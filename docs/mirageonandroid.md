# Mirage on Android

Trying to get [Mirage](http://openmirage.org) running on Android...

Status: work in progress (2013-09-13) - Ocaml 4.00.1 cross-compiler working; Mirage (mostly) working on Linux.

## Introduction

[Mirage](http://openmirage.org/) is an experimental "unikernel" project using Ocaml, i.e. compiles applications to bootable VM images suitable for Xen (or to a Unix process for testing/development).

It might be a useful way to create services in Ocaml that are safe, optimised and reasonably portable, at least to Unix and Xen.

## Mirage on Unix

More Mirage on the Xen Wiki: [Mirage architecture](http://wiki.xenproject.org/wiki/Mirage_architecture), including links to other docs including [Compile Mirage](http://wiki.xenproject.org/wiki/Compile_Mirage), which in turn points to [OPAM install](http://opam.ocamlpro.com/doc/Quick_Install.html) which has Ubuntu-specific instructions. This gives ocaml-4.00.1 and opam 1.0.0 as of 2013-09-12. 

A few notes on getting started with Mirage on Unix as of 2013-09-12:

* Right now, this seems to be the most complete/up-to-data [install guide](http://openmirage.org/wiki/install)
* it wasn't obvious from the introductory documentation on the Xen wiki that you can't opam install mirage-xen and mirage-unix at the same time, so if you have installed for one target (or mirari has done for you) then you need to "opam switch ..." (and/or "opem remove ..." it) before you can install/build for the other target.
* mirari 0.9.7 seems to require --socket as an explicit argument whereas some documentation suggests it was the default before (now defaults to direct?!).
* mirage 0.9.6 doesn't seem to print console output; had to use Mort's version, https://github.com/mor1/mirage
* mirage is a kind of virtual package in opam, and the actual building is done by mirage-unix (or mirage-xen); consequently you have to opam pin mirage-unix as well or instead of mirage to get that version
* most of the [mirage-skeleton examples](https://github.com/mirage/mirage-skeleton) compile and run, including basic and the simple web server, but some don't (I've only tried the UNIX socket targets).

## Ocaml on Android 

[Keigo](https://sites.google.com/site/keigoattic/ocaml-on-android) seems to be the root of much OCaml on Android work. His patch is Ocaml 3.12.1 (and NDK n7). He also wrote an initial [top-level](https://bitbucket.org/keigoi/ocaml-toplevel-android/src). This includes [vouillon](https://github.com/vouillon/ocaml-android) which was active approx. Feb 2013, and Vernoux's [Ocaml top-level for Android](https://play.google.com/store/apps/details?id=fr.vernoux.ocaml&hl=en). 

Vouillon's version appears to be Ocaml 4.00.1 and include LWT according to [this post](https://sympa.inria.fr/sympa/arc/caml-list/2013-01/msg00173.html). He also has an [Opam respository](https://github.com/vouillon/opam-android-repository/) for this.

Trying Vouillon's cross compiler:

* do "opam switch", e.g. "opam switch 4.00.1" or "opam switch install 4.00.1android --alias-of 4.00.1" (so that you are using a compiler within opam, not just the system default).
* add repo as per github readme: "opam repo add android https://github.com/vouillon/opam-android-repository.git"
* "opam install ocaml-android"
* One package in Vouillon's opam repository have hard-coded paths, e.g. https://github.com/vouillon/opam-android-repository/blob/master/packages/android-lwt.2.4.3/opam has "/home/jerome/.opam/4.00.1/bin/arm-linux-androideabi-ocamlfind". Edit in ~/.opam/repo/android/packages/android-lwt.2.4.3/opam to "%{prefix}%/bin/arm-linux-androideabi-ocamlfind" and then run "opam update" (before trying to install android-lwt).
* "opam install android-lwt"

To test it out try the instructions under Test on [Keigo's page](https://sites.google.com/site/keigoattic/ocaml-on-android)... but (having used opam) 

* with ocamlrun ~/.opam/4.00.1.android/arm-linux-androideabi/bin/, 
* ocaml in ~/.opam/4.00.1.android/arm-linux-androideabi/ and 
* stdlib files in ~/.opam/4.00.1.android/build/ocaml/stdlib (adjust for your chosen compiler version).
* The bin dir with ocamlopt is ~/.opam/4.00.1.android/bin/arm-linux-androideabi.

Interpreter and native both work for me ;-)

I'm not sure that camlp4 is visible initially (i.e. after ocaml-android) but it does seem to be after installing android-lwt. Note that it adds a symlink to camlp4 libraries on the system in $(ANDROID_PREFIX)/lib/ocaml/camlp4 (since these will be running on the host, not the target). 

## Notes on Building Mirage

The Mirage release (currently 0.9.6) is via the standard opam repository. There is also a [mirage project fork of the opam repository](https://github.com/mirage/opam-repository).

Opam packages and dependencies as of mirage 0.9.6:

* mirage-net-socket 0.9.4 requires mirage >= 0.9.5 & ocamlfind
* mirage 0.9.6 requires mirage-unix 0.9.6 OR mirage-xen 0.9.6
* mirage-unix 0.9.6 requires cstruct >= 0.7.1 & ocamlfind & lwt >= 2.4.0 & shared-memory-ring >= 0.4.1 & tuntap >= 0.6 & ipaddr >= 0.2.2 & fd-send-recv
* (cstruct)[https://github.com/mirage/ocaml-cstruct] >= 0.7.1 requires ocamlfind & ocplib-endian, optionally async | lwt
* ocamlfind has no dependencies
* (ocplib-endian)[https://github.com/OCamlPro/ocplib-endian] requires ocamlfind, optcomp
* optcomp requires ocamlfind
* lwt >= 2.4.0 depends on ocamlfind, optionally base-threads (installed by default) | base-unix (installed by default) | conf-libev (not installed) | ssl (not installed) | react (not installed) | lablgtk (not installed) | ocaml-text (not installed)
* shared-memory-ring >= 0.4.1 depends on cstruct >= 0.6.0 & lwt & ocamlfind & ounit
* ounit depends on ocamlfind
* tuntap >= 0.6 depends on ocamlfind & ipaddr >= 0.2.2
* (ipaddr)[https://github.com/mirage/ocaml-ipaddr] >= 0.2.2 depends on ocamlfind
* fd-send-recv depends on ocamlfind

optional:

* (async)[https://github.com/janestreet/async] (optional) - dependencies not shown here

Mirage applications are build using [mirari](https://github.com/mirage/mirari), currently release 0.9.7. This requires cmdliner & tuntap >= 0.5 & fd-send-recv.

Mirari uses a *.conf file to work out what to do. This includes:

* depends - a list of ocamlfind libraries
* packages - a list a opam packages that provide the depends libraries
* compiler - an opam compiler (i.e. argument for opam switch)

"mirari configure --socket" sets up the build environment for the application, including using opam to install the listed packages. The github readme is not completely consistent with mirari 0.9.7. In particular, mirari configure createss:

* backend.ml - for Unix, anyway, which provides a runtime interface for mirari to talk to the application
* main.ml - a standard main, which depends on the type of application, i.e. main-http = Cohttp callback, main-ip = with Netmanager (called with args net mgr, interface, ip), main-noip = without networking (called with no args) 
* Makefile - a pretty standard makefile which has targets to clean and to build using ocamlbuild configured to use ocamlfind  
* myocamlbuild.ml - a standard ocamlbuild, which I'm not clear exactly why it is needed at the moment as options look fairly standard

For Unix, mirari build just calls "make build" and links the native executable to mir-NAME. 

### Package specific details

#### fd-send-recv

[archive]("https://github.com/xen-org/ocaml-fd-send-recv/archive/ocaml-fd-send-recv-1.0.1.tar.gz")

Implements send_fd, recv_fd, fd_of_int and int_of_fd. Includes native C library which uses sendmsg and recvmsg. I'm not sure that this is strictly required for the UNIX socket version. 

Opam build:
* make
* make install

Includes Makefile, plus myocamlbuild.ml and setup.ml generated by oasis. Includes _oasis and _tags.

Initially: lib/fd_send_recv_stubs.c:24:25: fatal error: sys/statvfs.h: No such file or directory
It doesn't actually look like it needs this include anyway! 

See [git fork](https://github.com/cgreenhalgh/ocaml-fd-send-recv/tree/ocaml-fd-send-recv-1.0.1-nostatvfs) where I have removed it. Check this out and...
	oasis setup
	ocaml setup.ml -configure --override ocamlfind `opam config var prefix`/bin/arm-linux-androideabi/ocamlfind
	ocaml setup.ml -build
	ocaml setup.ml -install

#### optcomp

[archive](https://forge.ocamlcore.org/frs/download.php/1011/optcomp-1.4.tar.gz)

Version 1.4. Required by ocplib-endian. 

Has _oasis. No native files. 
Opam build: "./configure" "--prefix" "%{prefix}%"; make; make "install"

Requires camlp4, camlp4.lib & camlp4.quotations.o.

This is actually a compile-time tool so in is the host version that is required, not the target version. So I think we need to fool the android toolchain into reporting that it has it. 

	opam install optcomp

Now try and make the android toolchain think it has it too - let's try the following and see...
	ln -s `opam config var prefix`/lib/optcomp `opam config var prefix`/arm-linux-androideabi/lib/

#### ocplib-endian

[ocplib-endian](https://github.com/OCamlPro/ocplib-endian 
[archive](https://github.com/OCamlPro/ocplib-endian/archive/0.4.tar.gz)

Required by cstruct. No native code. Has _oasis.

Requires package optcomp.

Trying...
	oasis setup
	ocaml setup.ml -configure --override ocamlfind `opam config var prefix`/bin/arm-linux-androideabi/ocamlfind
	ocaml setup.ml -build
	ocaml setup.ml -install

Seemed to work.

#### cstruct

[cstruct](https://github.com/mirage/ocaml-cstruct)
[archive](https://github.com/mirage/ocaml-cstruct/archive/ocaml-cstruct-0.7.1.tar.gz)

Configured using Oasis. Includes native cstruct_stubs.c - portable memory copying only.
Has extra Makefile configuration to check if lwt and async installed, and if target is Xen (disable unix).

Hmm. But probably has both compiler syntax extensions and target support code. Former is Library cstruct-syntax; latter are cstruct, lwt_cstruct and unix_cstruct.

Try host build first: opam install cstruct  (requires host opam install lwt; opam install ocplib-endian)

Now target...
	oasis setup
	ocaml setup.ml -configure --enable-lwt --enable-unix --override ocamlfind `opam config var prefix`/bin/arm-linux-androideabi/ocamlfind
	ocaml setup.ml -build
	ocaml setup.ml -install

#### lwt

This seems to also use oasis.

An android-lwt opam module has been created by Vouillon; this has patches and opam commands which include some overrides for the oasis-defined build to use the arm-android compilers, which may be a useful model for the above (note, I have removed the fixed path below):
 
	build: [
	  ["oasis" "setup"]
	  ["ocaml" "setup.ml" "-configure"
	      "--override" "ocamlfind"
	         "%{prefix}%/bin/arm-linux-androideabi-ocamlfind"
	      "--override" "android_target" "true"
	      "--enable-ssl"]
	  ["ocaml" "setup.ml" "-build"]
	  ["ocaml" "setup.ml" "-install"]
	]

#### ocamlfind

build-time only. There is a version (1.3.3.1, with a patch) in the ocaml-android repository, so that's what I'm using by default.
It was built for android-lwt by default in this build sequence.

#### base-threads

Part of core system built by ocaml-android (?!)

#### base-unix

Part of core system built by ocaml-android (?!)

#### shared-memory-ring

[archive](https://github.com/mirage/shared-memory-ring/archive/shared-memory-ring-0.4.1.tar.gz)

Build: make all; make install.

Has _oasis. Has native code, including architecture-dependent __asm__ - checks for defined __i386__ __x86_64__ __arm__ ... - will probably need patching, or something custom to set target architecture.

Hopefully just getting the cross-compiler used will set this correctly?!

Oasis build for target...
	#	oasis setup
	ocaml setup.ml -configure --override ocamlfind `opam config var prefix`/bin/arm-linux-androideabi/ocamlfind
	ocaml setup.ml -build
	ocaml setup.ml -install

Was OK once (why??); no more:

+ /home/pszcmg/.opam/4.00.1.android/bin/arm-linux-androideabi/ocamlfind ocamlc -c lib/barrier_stubs.c
/tmp/ccwwXn9Q.s: Assembler messages:
/tmp/ccwwXn9Q.s:23: Error: selected processor does not support ARM mode `dmb'

What processor?? Time to look in ocaml-android, I suppose...

#### ounit

[archive](http://forge.ocamlcore.org/frs/download.php/886/ounit-1.1.2.tar.gz)

Build: make build; make install

Has _oasis. Makefile accepts BUILDFLAGS and INSTALLFLAGS. No native code.

Oasis build for target...
	# oasis setup
	ocaml setup.ml -configure --override ocamlfind `opam config var prefix`/bin/arm-linux-androideabi/ocamlfind
	ocaml setup.ml -build
	ocaml setup.ml -install


#### ipaddr

(ipaddr)[https://github.com/mirage/ocaml-ipaddr] 

[archive](https://github.com/mirage/ocaml-ipaddr/archive/0.2.2.tar.gz)

Build: "ocaml" "setup.ml" "-configure" "--prefix" "%{prefix}%"; make "build"; make "install"

Has _oasis. No native code. 

Oasis build for target...
	oasis setup
	ocaml setup.ml -configure --override ocamlfind `opam config var prefix`/bin/arm-linux-androideabi/ocamlfind
	ocaml setup.ml -build
	ocaml setup.ml -install

#### tuntap

[archive](https://github.com/mirage/ocaml-tuntap/archive/0.6.tar.gz)

Build: make "PREFIX=%{prefix}%"; make "PREFIX=%{prefix}%" "install"

Has _oasis.

Oasis build for target...
	# needs setup
	oasis setup
	ocaml setup.ml -configure --override ocamlfind `opam config var prefix`/bin/arm-linux-androideabi/ocamlfind
	ocaml setup.ml -build
lib/tuntap_stubs.c:30:21: fatal error: ifaddrs.h: No such file or directory
This defines getifaddrs and freeifaddrs
Seems to be known - this might fix it [getifaddrs for android](https://github.com/kmackay/android-ifaddrs)
Not sure how to set up conditional compilation at the moment though, or to get compiler to add lib/ to C-compiler include path...

	ocaml setup.ml -install


#### mirage-unix

[archive](https://github.com/mirage/mirage-platform/archive/v0.9.6.tar.gz).

Opam build:
	make "unix-build"
	make "unix-install" "PREFIX=%{prefix}%"

Makefile calls relevant sub-dir make, i.e. unix/ ...

Has configure.os script which does (host)OS-specific C-flag setting and selects tap_stubs_linux.c or tab_stubs_macosx.c.
Makefile uses cmd. Configure -> "configure unix".  

Requires (ocamlfind) cstruct cstruct.syntax lwt lwt.syntax lwt.unix tuntap ipaddr.

Has native files; one checksum speedup; other tap_stubs_linux which just emits an error if pcap_opendev called! Apparently this functionality was replaced by ocaml tuntap. 

#### mirage

No-op

#### mirage-net-socket

0.9.4. [archive](https://github.com/mirage/mirage-net/archive/v0.9.4.tar.gz)

Has custom myocamlbuild.ml, which includes at least some Xen-specific stuff. 

Has fairly standard Makefile choosing directory to build: Expects env MIRAGE_NET to be set to "socket" or "direct" for default build.

Has cmd script but unclear if used/needed.

Socket sub-directory has Makefile which uses cmd (duplicate of top-level) to configure/build/install. Optionally set PREFIX. Has META.in (conjecture: used by cmd), _tags and _vars. Has duplicate myocamlbuild.ml. Socket has no native code (relies on standard Unix module). 

Direct sub-directory ditto.


Has native code, tuntap_stubs.c. Has branches for __linux__ and __APPLE__/__MACH__. 

Hopefully just getting the cross-compiler used will set this correctly?!


