pkgbase-docs
=========

Information / Documentation on TrueOS inspired base packages for FreeBSD

Table of contents
=================
   * [What is package base?](#what-is-package-base)
   * [Download Images](#download-images)
   * [Users FAQ](#users-faq)
      * [What base packages are available](#what-base-packages-are-available)
      * [How do I manage these base packages](#how-do-i-manage-these-base-packages)
   * [Developers FAQ](#developers-faq)
      * [How do I build packages](#how-do-i-build-packages)
      * [How can I customize world or kernel](#how-can-i-customize-world-or-kernel)
      * [Why buildworld and buildkernel](#why-buildworld-and-buildkernel)
      

What is package base?
=========

FreeBSD has had the 'pkg' system for years now, allowing users easy installation and management of 3rd party applictions that originate in the ports tree. Package base refers to the process of getting the FreeBSD base system (userland and kernel) into package form, allowing the same management tools to be used. This is ends the requirement of needing to use multiple tools to update a system, such as "freebsd-update" and then "pkg". Additionally, this also allows tighter integration between the standard FreeBSD base system and the Ports / Package repos. Now updates will be done consistently across the two, ensuring packages and userland don't run into problems that result from a mis-matched userland/kernel and 3rd party applications.


Download Images
=========

The TrueOS Project (& iXsystems) are helping to maintain both FreeBSD 13 (HEAD) and FreeBSD 12 (Stable) images of the base system and complete ports tree. The master images will be updated weekly, while the 12 images will be updated more aggressively, usually daily.

[FreeBSD 13 - master](https://pkg.trueos.org/iso/freebsd-pkgbase/)

[FreeBSD 12 - stable](https://pkg.trueos.org/iso/freebsd12-pkgbase/)


Users FAQ
=========

What base packages are available
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

How do I manage these base packages
-----

Management is done via the usual 'pkg' commands. During upgrades, various conf files in /etc will be merged with local changes and updated. Should an un-mergable conflict exist, pkg will create a "<file>.pkgnew" file which contains the new file contents. These files can be manually inspected and merged into your existing /etc conf files.
  
To find a list of any files that need merging you can use the following command:

` # find /etc | grep '.pkgnew$'`


Developers FAQ
=========

How do I build packages
-----

**Note: This assumes you are already comfortable with FreeBSD's ports system and poudriere build tool.**

Base packages are built with poudriere. To get started, you'll to install `ports-mgmt/poudriere-devel` plus a [patch to add base port support](https://github.com/freebsd/poudriere/pull/664), or if you are using our package-base images, you can install `ports-mgmt/poudriere-trueos` to grab a pre-patched version.

This adds the ability to boot-strap a poudriere jail from the [base-ports added to TrueOS](https://github.com/trueos/trueos-ports/tree/trueos-master/os). These ports can be built as normal ports post-install as well, allowing new OS packages to be built anytime, with or without poudriere.

After a normal run of 'poudriere bulk' using a jail created in the above manner, the base-system ports are automatically included in the resulting package repo and ready for usage.

**Example (Creating a HEAD package set)**
```
 # poudriere ports -c -p myports -m git -U "https://github.com/trueos/trueos-ports" -B trueos-master
 # poudriere jail -c -j currentpkgbase -m ports=myports -v 13-CURRENT
 # poudriere bulk -a -j currentpkgbase -p myports
```

**Example (Creating a 12-Stable package set)**
```
 # poudriere ports -c -p myports -m git -U "https://github.com/trueos/trueos-ports" -B trueos-master
 # cd /usr/local/poudriere/ports/myports && ./update-branch-os.sh os freebsd/stable/12
 # poudriere jail -c -j currentpkgbase -m ports=myports -v 12-STABLE
 # poudriere bulk -a -j currentpkgbase -p myports
```

**Note: The 'update-branch-os.sh' script is a helper script to make it easy to switch OS sources in ports. This works with GitHub only. If you wish to pull sources from another location you will need to update the os/src port to reflect this.**


How can I customize world or kernel
-----

The [os/buildworld](https://github.com/trueos/trueos-ports/tree/trueos-master/os/buildworld) and [os/buildkernel](https://github.com/trueos/trueos-ports/tree/trueos-master/os/buildkernel) ports each support the typical `make config` usage you would expect via ports. It is possible to set various WITH/WITHOUT options via this method.


Why buildworld and buildkernel
-----

These ports are special to the base package process. They are used to run FreeBSD's **buildworld**/ **buildkernel** targerts once, and assemble the final output into a single tarball stored in /usr/dist/world.txz and /usr/dist/kernel.txz respectively. These are used by the other os/userland-* and os/kernel-* packages to re-pack into the user-facing package without needing to do multiple runs of 'buildworld / buildkernel'. Additionally it will provide single tarball files of each, which can be fed into other tools that cannot use pkgs natively. 


