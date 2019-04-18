# pkgbase-docs
Information / Documentation on TrueOS inspired base packages for FreeBSD

### What is package base?

FreeBSD has had the 'pkg' system for years now, allowing users easy installation and management of 3rd party applictions that originate in the ports tree. Package base refers to the process of getting the FreeBSD base system (userland and kernel) into package form, allowing the same management tools to be used.

### What base packages are available?

This package base system breaks the base OS down into the following sub-packages:
 * userland (Meta Package for other userland packages)
 * userland-base (Primary OS binaries / libraries, AKA 'runtime')
 * userland-docs (Manpages / info)
 * userland-debug (Debug files in /usr/lib/debug)
 * userland-lib32 (32bit compat libraries)
 * userland-tests (Testing frameworks)
 * kernel (Primary kernel / GENERIC)
 * kernel-debug (Kernel debug files)

### How do I manage these base packages?
