# pkgbase-docs
Information / Documentation on TrueOS inspired base packages for FreeBSD

### What is package base?

FreeBSD has had the **pkg** system for years. It allows users easy installation and management of 3rd party applictions that originate in the ports tree. *Package base* refers to the process of transforming FreeBSD userland and kernel into packages. This allows the package management tools to be used for the FreeBSD base system.

### What base packages are available?

This *package base* system organizes the FreeBSD core into these sub-packages:
 * userland (Meta Package for other userland packages)
 * userland-base (Primary OS binaries / libraries, AKA 'runtime')
 * userland-docs (Manpages / info)
 * userland-debug (Debug files in /usr/lib/debug)
 * userland-lib32 (32bit compat libraries)
 * userland-tests (Testing frameworks)
 * kernel (Primary kernel / GENERIC)
 * kernel-debug (Kernel debug files)
 * src (FreeBSD base system sources used in pkg builds)

### How are these base packages managed?

Manage these base packages with the usual [pkg(8)](https://www.freebsd.org/cgi/man.cgi?query=pkg>) commands. During upgrades, various conf files in */etc* will be merged with local changes and updated. If an unmergeable conflict exists, **pkg** creates a *<file>.pkgnew* file with the new file contents.

### How are these packages built?

Base packages and regular ports are both built with a version of [Poudriere(8)](https://www.freebsd.org/cgi/man.cgi?query=poudriere) that is [patched to add base port support](https://github.com/freebsd/poudriere/pull/664). This patch allows boot-strapping a *poudriere* jail from the [base-ports added to TrueOS](https://github.com/trueos/trueos-ports/tree/trueos-master/os).

These ports are also buildable as normal ports post-install. New OS packages can be built anytime, with or without *poudriere*.

Using a *poudriere* jail to run `poudriere bulk` includes the base system ports in the resulting package repo.


