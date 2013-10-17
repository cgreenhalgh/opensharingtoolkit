# Notes on Using Mirage to Create Services

[Mirage](http://openmirage.org) is a Library operating system that initially target Xen VMs and Unix. There are some notes on [running Mirage on Android](mirageonandroid.md). This document concerns trying to build kiosk-related applications in Mirage. It is a work in progress and largely notes to self.

Mirage is continually evolving; this is written as of 2013-10-16.

## HTTP support

The mirage website is written using Mirage (see [source on github](https://github.com/mirage/mirage-www)). As such it represents the default strategy for implementing HTTP-based services. Essentially this means using [ocaml-cohttp](https://github.com/mirage/ocaml-cohttp) for the server (dispatch, etc.) and by default [ocaml-cow](https://github.com/mirage/ocaml-cow) syntax extensions for specifying dynamic/templated HTML, CSS, XML and JSON.

I dont think the HTTP client is currently implemented though... see [client.ml](https://github.com/mirage/ocaml-cohttp/blob/master/cohttp/client.ml).

## Low-level persistence

Low-level persistence is nominally provided by blkif (block interface) devices as in [devices.mli](https://github.com/mirage/mirage-platform/blob/master/unix/lib/devices.mli). 

In Xen (for a mirage guest), a blkif would typically be a xen frontend block device (see [ocaml-xen-block-driver](https://github.com/djs55/ocaml-xen-block-driver)), which just passes requests to/from a backend device (via a xen vchan) which in turn passes requests to/from the physical device driver or a further device abstraction.

There is currently an incomplete blkif device over a regular Unix file in the Mirage unix support [blkif.ml](https://github.com/mirage/mirage-platform/blob/master/unix/lib/blkif.ml). 

There is also ...

## File system support

...

## High-level persistence

Baardskeerder...

Irminsule...

 

