pkgbase-docs
=========

Information / Documentation on TrueOS inspired base packages for FreeBSD

Table of contents
=================
   * [What is package base?](#what-is-package-base)
   * [Download Images](#download-images)
   * [FAQ](#faq)
      * [What base packages are available?](#what-base-packages-are-available)
      * [How do I manage these base packages?](#how-do-i-manage-these-base-packages)
      * [How are these packages built?](#how-are-these-packages-built)
      * [How can I customize world or kernel?](#how-can-i-customize-world-or-kernel)

What is package base?
=========

FreeBSD has had the 'pkg' system for years now, allowing users easy installation and management of 3rd party applictions that originate in the ports tree. Package base refers to the process of getting the FreeBSD base system (userland and kernel) into package form, allowing the same management tools to be used.


Download Images
=========

The TrueOS Project (& iXsystems) are helping to maintain both FreeBSD 13 (HEAD) and FreeBSD 12 (Stable) images of the base system and complete ports tree. The master images will be updated weekly, while the 12 images will be updated more aggressively, usually daily.

[FreeBSD 13 - master](https://pkg.trueos.org/iso/freebsd-pkgbase/)

[FreeBSD 12 - stable](https://pkg.trueos.org/iso/freebsd12-pkgbase/)


FAQ
=========

What base packages are available?
-----

This package base system breaks the base OS down into the following sub-packages:
 * userland (Meta Package for other userland packages)
 * userland-base (Primary OS binaries / libraries, AKA 'runtime')
 * userland-docs (Manpages / info)
 * userland-debug (Debug files in /usr/lib/debug)
 * userland-lib32 (32bit compat libraries)
 * userland-tests (Testing frameworks)
 * kernel (Primary kernel / GENERIC)
 * kernel-debug (Kernel debug files)
 
Additionally the following packages related to building are also available:

 * src (FreeBSD base system sources used in pkg builds, pkg installs into /usr/src)
 * buildworld (Installs tarball: /usr/dist/world.txz which contains entire buildworld output)
 * buildkernel (Installs tarball: /usr/dist/kernel.txz which contains entire buildkernel output)

How do I manage these base packages?
-----

Management is done via the usual 'pkg' commands. During upgrades, various conf files in /etc will be merged with local changes and updated. Should an un-mergable conflict exist, pkg will create a "<file>.pkgnew" file which contains the new file contents.

How are these packages built?
-----

Base packages are built with poudriere along with regular ports. This is done by using poudriere plus a [patch to add base port support](https://github.com/freebsd/poudriere/pull/664). This adds the ability to boot-strap a poudriere jail from the [base-ports added to TrueOS](https://github.com/trueos/trueos-ports/tree/trueos-master/os). These ports can be built as normal ports post-install as well, allowing new OS packages to be built anytime, with or without poudriere. 

After a normal run of 'poudriere bulk' using a jail created in the above manner, the base-system ports are automatically included in the resulting package repo and ready for usage.

How can I customize world or kernel?
-----

The [os/buildworld](https://github.com/trueos/trueos-ports/tree/trueos-master/os/buildworld) and [os/buildkernel](https://github.com/trueos/trueos-ports/tree/trueos-master/os/buildkernel) ports each support the typical "make config" usage you would expect via ports. It is possible to set various WITH/WITHOUT options via this method. These ports are used to run buildworld / buildkernel once, and assemble the final output into a single tarball stored in /usr/dist/world.txz and /usr/dist/kernel.txz respectively. These are used by the other userland-* and kernel-* packages to re-pack into final form without needing to do multiple runs of 'buildworld/buildkernel'. Additionally it will provide single tarball files of each, which can be fed into other tools that cannot use pkgs natively. 


