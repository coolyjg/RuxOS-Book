
# Arguments

RuxOS provide many, but flexible and pricise arguments to build and run various applications. 

**General options** about building process:

| General Options | |
| --- | --- |
| ARCH | Target architecture: x86_64, riscv64, aarch64. Default value is x86_64. |
| PLATFORM | Target platform in the `platforms` directory. This can be user-specified or generated automatically by `ARCH`. |
| SMP | Number of CPUs. If this is greater than one, multiple cores will be started. Default is one. |
| MODE | Build mode: release, debug, corresponed to `cargo build` mode. Default is release. |
| LOG | Logging level: warn, error, info, debug, trace. Default is warn. |
| V | Verbose level: (empty), 1, 2 |
| ARGS | Command-line arguments separated by comma, which can be none. It is used to pass specific variables, like `argc`, `argv`. |
| ENVS | Environment variables, separated by comma between key value pairs, which can be none. |

**Application options** about the application directory and features:

| App Options | |
| --- | --- |
| A/APP | Path to the application directory.Both relative and absolute path are right. Default path points to `apps/c/helloworld.` |
| FEATURES | Features of Ruxos modules to be enabled. This can be used to specific features without modifying corresponding `features.txt` |
| APP_FEATURES | Features of (rust) apps to be enabled. |

**Qemu options**, mainly used to enable qemu startup parameters:

| Qemu Options | |
|---|---|
| BLK | Enable storage devices (virtio-blk). This should be enabled for data persistence and configuration file passing via root filesystem. |
| NET | Enable network devices (virtio-net). This shoule be enabled if applications need network service. |
| GRAPHIC | Enable display devices and graphic output (virtio-gpu). |
| V9P | Enable virtio-9p devices. |
| BUS | Device bus type: mmio, pci. |
| DISK_IMG | Path to the virtual disk image. Default value is `/path/to/RuxOS/disk.img`. |
| ACCEL | Enable hardware acceleration (KVM on linux). |
| QEMU_LOG | Enable QEMU logging (log file is "qemu.log") to dump out assembly code. |
| NET_DUMP | Enable network packet dump (log file is "netdump.pcap"). |
| NET_DEV | QEMU netdev backend types: user, tap. |

**9P options** used to configure 9pfs:

| 9P Options | |
|---|---|
| V9P_PATH | Host path for backend of virtio-9p, points to the shared directory on host. Default value is RuxOS root directory. |
| NET_9P_ADDR| Server address and port for 9P netdev. Default value is `127.0.0.1:564`. |
| ANAME_9P | Path for root of 9pfs(parameter of TATTACH for root). |
| PROTOCOL_9P | Default protocol version selected for 9P. |

**Network options**, default port is 5555:

| Network Options | |
|---|---|
| IP | Ruxos IPv4 address (default is 10.0.2.15 for QEMU user netdev). |
| GW | Gateway IPv4 address (default is 10.0.2.2 for QEMU user netdev). |

**Libc options** for libc selection:

| Libc Options | |
|---|---|
| MUSL | Compile, link C app with musl libc. By default, RuxOS use `ruxlibc`. Users can specify `MUSL=y` to enable musl-libc. |



