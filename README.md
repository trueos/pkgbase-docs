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
      * [Why not X release version?](#why-not-x-release-version)
      * [Where can I discuss further?](#where-can-i-discuss-further)
   * [NEW! Flavors support](#flavors-support)
     * [How do I use flavors?](how-do-i-use-flavors)
   * [Developer FAQ](#developers-faq)
      * [How do I build packages?](#how-do-i-build-packages)
      * [Changing OS Target](#changing-os-target)
      * [How can I customize world or kernel?](#how-can-i-customize-world-or-kernel)
      * [Why buildworld and buildkernel?](#why-buildworld-and-buildkernel)
      

What is pkgbase?
=========

FreeBSD has had the `pkg` system for years now, giving users easy installation and management of third party applications from the ports tree. "pkgbase" refers to the process of getting the FreeBSD base system (userland and kernel) into package form, allowing the same management tools to be used. This ends the requirement of multiple tools to update a system, like `freebsd-update` and then `pkg`. Additionally, it more tightly integrates the standard FreeBSD base system and the ports/package repos. Now updates are done consistently across the two, ensuring packages and userland don't run into problems that result from a mis-matched userland/kernel and third party applications.


What is a CFT?
=========

A CFT is a *Call For Testing*. It's a notice that a project is ready for wide spread review and testing. This CFT is to raise awareness and gather input about the work being done. Comments, constructive criticsim,and even patches are all a helpful part of the process.


Download Images
=========

The [TrueOS Project](https://www.trueos.org) / [iXsystems](https://www.ixsystems.com) is maintaining both FreeBSD 13 (HEAD) and FreeBSD 12 (Stable) images of the base system and complete ports tree.

[FreeBSD 13-CURRENT](https://pkg.trueos.org/iso/freebsd-pkgbase/)

[FreeBSD 12-STABLE](https://pkg.trueos.org/iso/freebsd12-pkgbase/)




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
 * kernel (Primary kernel / GENERIC or -NODEBUG)
 * kernel-debug (Kernel with Witness and other debugging)
 * kernel-symbols (Kernel debug symbols in /use/lib/debug)
 * kernel-debug-symbols (Kernel debug symbols in /use/lib/debug for Witness kernel)
 
Additional packages related to building are also available:

 * src (FreeBSD base system sources used in pkg builds, pkg installs into `/usr/src`)
 * buildworld (Installs tarball: /usr/dist/world.txz which contains entire buildworld output)
 * buildkernel (Installs tarball: /usr/dist/kernel.txz which contains entire buildkernel output)
 * buildkernel-debug (Installs tarball: /usr/dist/kernel-debug.txz which contains entire buildkernel output for Witness enabled kernel)



How do I manage these base packages?
-----

Management is done with the usual `pkg` commands. During upgrades, various conf files in `/etc` will be merged with local changes and updated. If an unmergeable conflict exists, pkg creates a _`file`_`.pkgnew` file containing the new file contents. These files can be manually inspected and merged into existing `/etc` conf files.
  
To find a list of any files that need merging:

` # find /etc | grep '.pkgnew$'`


Update Frequency
-----
The 13-CURRENT images will be updated weekly, while the 12-STABLE images will be updated more aggressively, usually every 48 hours or so. 


How long will these repos get updated?
-----

Right now we are looking to keep building and publishing these base package repos for as long as users find them useful. When the FreeBSD project begins publishing base packages for their official releases we will probably wind down the updates or reduce the frequency.


Why not X release version?
-----
This CFT is intended to help test pkg base, of which updating is a huge part. Release versions of FreeBSD tend to not change that often. We may consider a release build at a later point after the initial images have shaken out issues.


Where can I discuss further?
-----

Users are encouraged to join the discussion on the [pkgbase mailing list](https://lists.freebsd.org/mailman/listinfo/freebsd-pkgbase).



Flavors support
=========

As of May 11th, 2019 our package base repositories have gained the support of FLAVORS. What does this mean? It means with pkg it’s now possible to switch between different types of base system builds. We’ve created three flavors for testing:

‘generic’ - Default FreeBSD world/kernel flags
‘minimal’ - Build with a bunch of WITHOUT flags set
https://github.com/trueos/trueos-ports/blob/334b9246300fc5b9f7dc2057b48464ff193f4f9d/Mk/Uses/os.mk#L72-L86
‘zol’ - Build of ‘generic’ FreeBSD flavor, but setting WITHOUT_ZFS and a dependency on ‘sysutils/zol’

How do I use flavors?
-----

If you are running the pre-flavor package set we originally published in the Call for Testing, you’ll need to follow these steps to switch to the package set with flavors:

```
# pkg set --change-name userland:os-generic-userland
# pkg set --change-name userland-base:os-generic-userland-base
# pkg set --change-name userland-docs:os-generic-userland-docs
# pkg set --change-name kernel:os-generic-kernel
```

Repeat that process for lib32, debug, tests, or any other optional base packages you have installed. After you’re finished switching to the flavors package set,  run ‘pkg upgrade’ and it’ll upgrade you to the new packages.


You can also switch between flavor packages using a similar method. For example, to switch from the generic -> zol flavors:

```
# pkg set --change-name os-generic-userland:os-zol-userland
# pkg set --change-name os-generic-userland-base:os-zol-userland-base
# pkg set --change-name os-generic-userland-docs:os-zol-userland-docs
# pkg set --change-name os-generic-kernel:os-zol-kernel
```

Repeat this process for any other base packages you have installed. “#pkg info | grep ‘^os-’” can help you find any other installed base packages.

**Word of caution!!**

No matter how tempted you are, avoid using ‘pkg install’ try and change flavors. Pkg will detect a conflict with the currently installed flavor and proceed to de-install it, destroying all your config files in /etc. That will ruin your day. Using name-changes and ‘pkg upgrade’ seems to be the safest route at this time, but please let us know if you find any other working alternatives.




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

**Note: The `update-branch-os.sh` script is a local TrueOS helper script to make it easy to switch OS sources in ports. This works with GitHub only. If you wish to pull sources from another location, update the os/src port to reflect this.**


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

These ports are special to the base package process. They run FreeBSD's **buildworld**/ **buildkernel** targets once and assemble the final outputs into single tarballs stored in `/usr/dist/world.txz` and `/usr/dist/kernel.txz` respectively. These are used by the other `os/userland-*` and `os/kernel-*` packages to re-pack into the user-facing package without needing multiple runs of `buildworld/buildkernel`. Additionally, it provides single tarball files of each, which can be fed into other tools that cannot use packages natively. 

