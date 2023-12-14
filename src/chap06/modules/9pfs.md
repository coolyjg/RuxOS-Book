
# 9pfs

This chapter mainly introduces an overview of 9p protocol and how to use the 9pfs module of RuxOS.

9PFS, 9p-filesystem, is a file system based on the 9P protocol that reads and writes the server-side file system.

In RuxOS, 9pfs corresponds to two different features, namely `virtio-9p` and `net-9p`. Among them, `virtio-9p` is based on the `virtio-9p` virtual device in the hypervisor to communicate and control the reading and writing of the host file system, while `net-9p` realizes connection communication and controls the server-side file system through the network TCP protocol. These two different features can be enabled at the same time and work properly.

9P protocol is basically a communication protocol used for file sharing. It is a the session layer protocol. This means that 9P has nothing to do with the physical layer, data link layer, etc. There are three versions of 9P communication protocol: `9P2000`, `9P2000.L` and `9P2000.u`. For the protocol content, please refer to the following link:

* [Basic protocol content of 9P2000](https://ericvh.github.io/9p-rfc/rfc9p2000.html)
* [9P2000.L extended protocol version](https://github.com/chaos/diod/blob/master/protocol.md)
* [9P2000.u extended protocol version](http://ericvh.github.io/9p-rfc/rfc9p2000.u.html)

## virtio-9p

In order to use `virtio-9p`, users should add this feature to the APP.

For C applications, add a line under features.txt in the APP directory:

```txt
virtio-9p
```

For Rust applications,  add the `virtio-9p` feature to the axstd feature of Cargo.toml in the APP directory, as follows (this example is Cargo.toml in `application/fs/shell`):

```txt
axstd = { path = "../../../ulib/axstd", features = ["alloc", "fs", "virtio-9p"], optional = true }
```

Then, in order to enable the host (currently qemu-system-aarch64) to enable the virtio-9p backend, you need to add the parameter `V9P=y` to the `make` command line.

For example:

```shell
make A=xxxx LOG=error V9P=y V9P_PATH=xxxx ARCH=xxxx
```

`V9P_PATH` is an **environment variable**, corresponding to the file directory mapping path of the host. If this parameter is not set, it will be the current directory `./`.

After completing the above steps, you can create a `v9fs` directory in the root directory (or load it as a root file system if `blkfs` is not specified in features.txt). This directory should contain the file directory mapped by the host backend. For example, when `V9P_PATH=./temp/`, the v9fs (or root file) directory will be consistent with the ./temp directory.

In addition, RuxOS provide more flexible parameters:

* `V9P_PATH`: Default value is current directory. This parameter specified the shared directory on the host.
* `PROTOCOL_9P`: The 9P protocol selected, which can be either `9P2000.L` or `9P2000.u`. Default value is `9P2000.L`.


## net-9p

In order to use `net-9p`, users should add this feature to the APP.

For C applications, add a line under features.txt in the APP directory:

```txt
net-9p
```

For Rust applications, add the `net-9p` feature to the axstd feature of Cargo.toml in the APP directory, as follows:

```txt
axstd = { path = "../../../ulib/axstd", features = ["alloc", "fs", "net-9p"], optional = true }
```

Then, `net-9p` protocol needs TCP protocol to communicate, so the parameter `NET=y` needs to be added to the `make` command line.

For example:

```shell
make A=xxxx LOG=error NET=y ARCH=xxxx
```

After completing the above steps, a directory named `v9fs` is created at the root directory (or loaded as root directory if `blkfs` is specified). This directory contains the file directory mapped by the host backend. The specific path of the `net-9p` backend mapping may need to be controlled through `ANAME_9P`, but this depends on the design of the server side (it may also be passed in parameters when starting the server or changing some server-side codes).

In addition, RuxOS provide more flexible parameters for users:

* `NET_9P_ADDR`: The IP address and PORT port number of the 9P server. The default value is `127.0.0.1:564`.
* `ANAME_9P`: The aname path parameter corresponding to `tattach`, which can be set to the specifically selected corresponding host file directory path. The default value points to current directory.
* `PROTOCOL_9P`: The 9P protocol selected, which can be either `9P2000.L` or `9P2000.u`. Default value is `9P2000.L`.

