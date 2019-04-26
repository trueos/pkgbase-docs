pkgbase-docs
=========

Information and Documentation on TrueOS-Inspired Base Packages for FreeBSD

Table of Contents
=================
   * [What is pkgbase?](#what-is-pkgbase)
   * [Download Images](#download-images)
   * [User FAQ](#user-faq)
      * [Which base packages are available?](#which-base-packages-are-available)
      * [How do I manage these base packages?](#how-do-i-manage-these-base-packages)
      * [Update Frequency](#update-frequency)
      * [How long will these repos get updated?](#how-long-will-these-repos-get-updated)
      * [Where can I discuss further?](#where-can-i-discuss-further)
   * [Developer FAQ](#developers-faq)
      * [How do I build packages?](#how-do-i-build-packages)
      * [Changing OS Target](#changing-os-target)
      * [How can I customize world or kernel?](#how-can-i-customize-world-or-kernel)
      * [Why buildworld and buildkernel?](#why-buildworld-and-buildkernel)
      

What is pkgbase?
=========

FreeBSD has had the 'pkg' system for years now, giving users easy installation and management of third party applications from the ports tree. "pkgbase" refers to the process of getting the FreeBSD base system (userland and kernel) into package form, allowing the same management tools to be used. This ends the requirement of multiple tools to update a system, like "freebsd-update" and then "pkg". Additionally, it more tightly integrates the standard FreeBSD base system and the ports/package repos. Now updates are done consistently across the two, ensuring packages and userland don't run into problems that result from a mis-matched userland/kernel and third party applications.




Download Images
=========

The [TrueOS Project](https://www.trueos.org) / [iXsystems](https://www.ixsystems.com) is maintaining both FreeBSD 13 (HEAD) and FreeBSD 12 (Stable) images of the base system and complete ports tree.

[FreeBSD 13 - master](https://pkg.trueos.org/iso/freebsd-pkgbase/)

[FreeBSD 12 - stable](https://pkg.trueos.org/iso/freebsd12-pkgbase/)




User FAQ
=========

Which base packages are available?
-----

This pkgbase system breaks the base OS into these sub-packages:
 * userland (Meta Package for other userland packages)
 * userland-base (Primary OS binaries / libraries, AKA 'runtime')
 * userland-docs (Manpages / info)
 * userland-debug (Debug files in /usr/lib/debug)
 * userland-lib32 (32bit compat libraries)
 * userland-tests (Testing frameworks)
 * kernel (Primary kernel / GENERIC)
 * kernel-debug (Kernel debug files)
 
Additional packages related to building are also available:

 * src (FreeBSD base system sources used in pkg builds, pkg installs into /usr/src)
 * buildworld (Installs tarball: /usr/dist/world.txz which contains entire buildworld output)
 * buildkernel (Installs tarball: /usr/dist/kernel.txz which contains entire buildkernel output)



How do I manage these base packages?
-----

Management is done with the usual 'pkg' commands. During upgrades, various conf files in `/etc` will be merged with local changes and updated. If an unmergeable conflict exists, pkg creates a _`file`_`.pkgnew` file containing the new file contents. These files can be manually inspected and merged into existing `/etc` conf files.
  
To find a list of any files that need merging:

` # find /etc | grep '.pkgnew$'`


Update Frequency
-----
The master images will be updated weekly, while the 12 images will be updated more aggressively, usually every 48 hours or so. 


How long will these repos get updated?
-----

Right now we are looking to keep building and publishing these base package repos for as long as users find them useful. When the FreeBSD project begins publishing base packages for their official releases we will probably wind down the updates or reduce the frequency.

Where can I discuss further?
-----

Users are encouraged to join the discussion on the [pkgbase mailing list](https://lists.freebsd.org/mailman/listinfo/freebsd-pkgbase).


Developers FAQ
=========



How do I build packages?
-----

**Note: This assumes you are already comfortable with FreeBSD's ports system and poudriere build tool.**

Base packages are built with poudriere. To get started, install `ports-mgmt/poudriere-devel` plus a [patch to add base port support](https://github.com/freebsd/poudriere/pull/664). Or if you are using our package-base images, you can install `ports-mgmt/poudriere-trueos` with a pre-patched version.

This adds the ability to bootstrap a poudriere jail from the [base-ports added to TrueOS](https://github.com/trueos/trueos-ports/tree/trueos-master/os). These ports can be built as normal ports post-install as well, allowing new OS packages to be built any time, with or without poudriere.

After a normal run of `poudriere bulk` using a jail created in the above manner, the base-system ports are automatically included in the resulting package repo and ready for use.

**Example (Creating a HEAD package set)**
```
 # poudriere ports -c -p myports -m git -U "https://github.com/trueos/trueos-ports" -B trueos-master
 # poudriere jail -c -j myjail -m ports=myports -v 13-CURRENT
 # poudriere bulk -a -j myjail -p myports
```

**Example (Creating a 12-Stable package set)**
```
 # poudriere ports -c -p myports -m git -U "https://github.com/trueos/trueos-ports" -B trueos-master
 # cd /usr/local/poudriere/ports/myports && ./update-branch-os.sh os freebsd/stable/12
 # poudriere jail -c -j myjail -m ports=myports -v 12-STABLE
 # poudriere bulk -a -j myjail -p myports
```

**Note: The 'update-branch-os.sh' script is a local TrueOS helper script to make it easy to switch OS sources in ports. This works with GitHub only. If you wish to pull sources from another location, update the os/src port to reflect this.**


Changing OS Target
-----

Within ports, the [os/src](https://github.com/trueos/trueos-ports/tree/trueos-master/os/src) port determines what version / branch of FreeBSD is used to build a jail and packages. This port can be edited to pull sources from any location which ports can support. 

Within the TrueOS ports tree, we ship the [update-branch-os.sh](https://github.com/trueos/trueos-ports/tree/trueos-master/update-branch-os.sh) utility which can be used to quickly switch sources between branches and tags on the [trueos/trueos](https://github.com/trueos/trueos) GitHub repo.

**Example Switching to freebsd/stable/12 branch **
```
# cd /usr/local/poudriere/ports/myports
# ./update-branch-os.sh os freebsd/stable/12
```


How can I customize world or kernel?
-----

The [os/buildworld](https://github.com/trueos/trueos-ports/tree/trueos-master/os/buildworld) and [os/buildkernel](https://github.com/trueos/trueos-ports/tree/trueos-master/os/buildkernel) ports each support the typical `make config` usage expected in ports. It is possible to set various WITH/WITHOUT options through this method. When using poudriere, refer to the poudriere man pages which explain how to set various port options for jails and ports trees.




Why buildworld and buildkernel?
-----

These ports are special to the base package process. They run FreeBSD's **buildworld**/ **buildkernel** targets once and assemble the final outputs into single tarballs stored in /usr/dist/world.txz and /usr/dist/kernel.txz respectively. These are used by the other os/userland-* and os/kernel-* packages to re-pack into the user-facing package without needing multiple runs of 'buildworld/buildkernel'. Additionally, it provides single tarball files of each, which can be fed into other tools that cannot use packages natively. 

