# Notes on Using Mirage to Create Services

[Mirage](http://openmirage.org) is a Library operating system that initially target Xen VMs and Unix. There are some notes on [running Mirage on Android](mirageonandroid.md). This document concerns trying to build kiosk-related applications in Mirage. It is a work in progress and largely notes to self.

Mirage is continually evolving; this is written as of 2013-10-16.

## HTTP support

The mirage website is written using Mirage (see [source on github](https://github.com/mirage/mirage-www)). As such it represents the default strategy for implementing HTTP-based services. Essentially this means using [ocaml-cohttp](https://github.com/mirage/ocaml-cohttp) for the server (dispatch, etc.) and by default [ocaml-cow](https://github.com/mirage/ocaml-cow) syntax extensions for specifying dynamic/templated HTML, CSS, XML and JSON.

I dont think the HTTP client is currently implemented though... see [client.ml](https://github.com/mirage/ocaml-cohttp/blob/master/cohttp/client.ml).

## Low-level persistence

Low-level persistence is nominally provided by blkif (block interface) devices as in [devices.mli](https://github.com/mirage/mirage-platform/blob/master/unix/lib/devices.mli). The main methods are:
'''
read_512: int64 -> int64 -> Cstruct.t Lwt_stream.t;
write_page: int64 -> Io_page.t -> unit Lwt.t;
sector_size : int;
size: int64;
'''

Note that a [Cstruct](https://github.com/mirage/ocaml-cstruct/blob/master/lib/cstruct.mli) is an offset and length over  Bigarray.Array1 of int8_unsigned elements. There are standard methods to create over a bigarray, create (by copy) from a string, to convert (copy) to a string and to blit. The main function of `Lwt_stream.t` is (blocking) get (next element).

An [Io_page.t](https://github.com/mirage/mirage-platform/blob/master/unix/lib/io_page.mli) is (also) a Bigarray.Array1, and supports `to_cstruct` without copy and `to_string` with copy. On Xen these appear to be allocated using contigmalloc, while they are regular Ocaml arrays on Unix. Page size is hard-coded to 4096 bytes on both.

In Xen (for a mirage guest), a blkif would typically be a xen frontend block device (see [ocaml-xen-block-driver](https://github.com/djs55/ocaml-xen-block-driver)), which just passes requests to/from a backend device (via a xen vchan) which in turn passes requests to/from the physical device driver or a further device abstraction.

There is currently an incomplete blkif device over a regular Unix file in the Mirage unix support [blkif.ml](https://github.com/mirage/mirage-platform/blob/master/unix/lib/blkif.ml). 

There is also Dave Scott's replacement for crunchfs over a tar-formatted blkif [here](https://github.com/mirage/mirage-www/pull/38).

Dave Scott says:
'''
I'd like to see unix blkif implemented (without using the xen implementation). I've got some patches lying around which open files with O_DIRECT and perform unbuffered sector-aligned I/O. I was thinking of making a 'unix-block-driver' with this code in it, mirroring the xen implementation. It's mostly a thin veneer over Lwt_bytes, replacing Bigarray.t with Cstruct.t. In future I was thinking the mirage-platform repo could become the minimal 'boot' code plus module types for Blkif, Netif etc; and all the concrete implementations could be spun out into other repos.
'''

## Configuration

In principle, an application's configuration, including network config and device config, is specified in a `.conf` file that is used by [Mirari](https://github.com/mirage/mirari) to create skeleton code for running the application during the config process.

Currently a Mirari config option like `fs-NAME` uses [mir-crunch](https://github.com/mirage/mirage-fs) to convert the specific directory into a crunch filesystem (readonly key-value device built into the application). [crunch.ml](https://github.com/mirage/mirage-fs/blob/master/crunch/crunch.ml) generates code which creates a registers (with `OS.Devices.new_provider`) a provider for the generated `OS.Devices.kv_ro` device and registers a sunction with `OS.Main.at_enter` to `plug` that device.
 
Dave Scott's [mirage-www variant](https://github.com/mirage/mirage-www/pull/38) explicitly performs `Blkfront.register` which in turns registers a `Xen.Blkif` provider and then enumerates virtual block devices listed in Xenstore and generates a plug event for each. Using `OS.Devices.listen` it detects the first `blkif` that appears and creates an object implementing the read method of a `kv_ro` over it, assuming that it is tar-encoded. This is not a full `Devices.kv_ro` and is not registered with OS.Devices. The application explicitly attempts to use this and a registered `kv_ro` called `static`. 

If we assume that mirari should be responsible for VM-related configuration then there should be an option which specifies the name of a blkif expected by the application and (a) on Xen the corresponding virtual device name (b) on Unix (or both) the local file that should contain the block device image. The file itself could either be generated by mirari, assuming it knows how to make (e.g.) tar image of a directory, or whatever, or could be generated by explicit actions in the makefile. For Xen, mirari would include a variant of the Blkfront.register code in the generated main.ml which would create blkif providers with mapped names for virtual block devices. For Unix, mirari would include code in main.ml to register and try to plug a provider for each configured file/name. 

This assumes that the expected blkif device names (but not the Xen vbd or Unix file names) are effectively hard-coded into the application (or other application-specific configuration files). 

## File system support

There is Dave Scott's [FAT filesystem support](https://github.com/djs55/ocaml-fat), although this doesn't use the latest interface(s)/types (it has an abstrct BLOCK interface based on sector read/write and Bitstring.t).

## High-level persistence

### Mirage readonly key-value store

Mirage [OS.Devices](https://github.com/mirage/mirage-platform/blob/master/unix/lib/devices.mli) so far defines an abstract key-value store:
'''
type kv_ro = <
  iter_s : (string -> unit Lwt.t) -> unit Lwt.t;
  read : string -> Cstruct.t Lwt_stream.t option Lwt.t;
  size : string -> int64 option Lwt.t 
>
'''

The key is a string, the value a `Cstruct.t Lwt_stream.t` of knowable size (via `size`). The set of all keys can also be iterated.

This interface is supported by the crunchfs (see above), and partially by [Dave's mirage-www variant](https://github.com/mirage/mirage-www/pull/38) (which is over a blkif exposed by a Xen blockfront device).

### Baardskeerder

[Baardskeerder](https://github.com/Incubaid/baardskeerder) is a copy-on-write B-Tree. Dave Scott did a partial port [here](https://github.com/djs55/mirage/tree/baardskeerder/lib/btree), but apparently with no persistent store option (in memory only).

The interface provided is of a named set of b-trees (each of type `t`), where key is a string and value is a string. There is one transactionless (blocking) action, to get the latest value of key: `get_latest : t -> k -> (v,k) result`. The full interface is used within a transaction:
'''
val with_tx : t -> (tx -> ('a,'b) result) -> ('a,'b) result
val get   : tx -> k -> (v,k) result
val set   : tx -> k -> v -> unit
val delete: tx -> k -> (unit,k) result
'''

There are also key range and key prefix variants of get (with and without transaction) and delete (within a transaction).

Mapping from value (string) to Cstruct.t Lwt_stream.t would be simple but with a copy. Similarly a write would require a copy-based mapping; presumably the value written should be non-blocking, e.g. list of Cstruct.t. The `range_latest` function should allow creating an iteration interface.

Support for transactions would presumably be useful - some sort of atomic test-and-set as minimum! By default the `with_tx` approach could be used.

This would suggest a minimal Mirage interface of:
'''
type kv_rw = <
  iter_s : (string -> unit Lwt.t) -> unit Lwt.t;
  read : string -> Cstruct.t Lwt_stream.t option Lwt.t;
  size : string -> int64 option Lwt.t;
  write : string -> Cstruct.t list -> unit Lwt.t;
  delete : string -> unit Lwt.t;
  with_tx : (tx -> ('a Lwt.t)) -> 'a Lwt.t; 
  iter_tx : tx -> (tx -> string -> unit Lwt.t) -> unit Lwt.t;
  read_tx : tx -> string -> Cstruct.t Lwt_stream.t option Lwt.t;
  size_tx : tx -> string -> int64 option Lwt.t;
  write_tx : tx -> string -> Cstruct.t list -> unit Lwt.t;
  delete_tx : tx -> string -> unit Lwt.t;
>
'''

The Baardskeerder backing interface is an open set of named byte arrays, with unaligned random readwrite access via string buffers. There are three backing store implementations provided in [store.ml](https://github.com/Incubaid/baardskeerder/blob/master/src/store.ml): in memory, synchronous unix, LWT unix. 

This would require a copy-based type mapping and (if non-block reads/write ever actually occur) a cacheing/block layer on top of `blkif`. I'm also not completely certain if a b-tree can be implemented over exactly one file (in the general case it can be several files aka [spindles](https://github.com/Incubaid/baardskeerder/blob/master/doc/disk_format.rst)). 

### Irminsule

[Irminsule](https://github.com/samoht/irminsule) is "a distributed filesystem and block store that follows the same design principles as Git". Apparently not ready for production use, and right now I am not sure how in principle it should be used in this scenario!

