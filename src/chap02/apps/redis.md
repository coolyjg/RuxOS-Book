
# Redis

RuxOS supports Redis Server on Qemu.

## Create disk.img

Run following command at root directory of RuxOS. 
```shell
make disk_img
```
This command will create `disk.img` to store data, and this image will be passed to qemu as a disk image.

## Run Redis-Server

Run following command to start a redis server at port 5555.
```shell
make A=apps/c/redis/ LOG=error NET=y BLK=y ARCH=aarch64 SMP=4 ARGS="./redis-server,--bind,0.0.0.0,--port,5555,--save,\"\",--appendonly,no,--protected-mode,no,--ignore-warnings,ARM64-COW-BUG" run
```
Parameter explanation:
* `A`: This paramter points to the directory where redis is placed. 
* `LOG`: `LOG` indicates the log level, which includes: `error`,  `warn`, `info`, `debug`, `trace`.
* `NET`: `NET` indicates that net device (virtio-net) is enabled.
* `BLK`: `BLK` indicates that storage device (virtio-blk) is enabled.
* `ARCH`: `ARCH` indicates what architecture RuxOS is running on, which covers: `x86_64`, `aarch64`, `riscv64`.
* `SMP`: `SMP` enables multi-core feature in RuxOS. The following number indicates the number of the cores.
* `ARGS`: `ARGS` provides the parameters needed for the redis-server to run. RuxOS binds redis server on 0.0.0.0:5555.

After running this command, a Redis Server is started on port 5555.

## Connect to Redis-Server

It is recommended to use [redis tools](https://redis.io/resources/tools/) to connect to Redis Server by:
```shell
sudo apt install redis-tools
```

### redis-cli

Then you can use:
```shell
redis-cli -p 5555
```

Then it is nothing different from a linux redis server.

### redis-benchmark

Use the Redis benchmark testing tool to test:
```shell
redis-benchmark -p 5555
```
More parameters can be found on [Redis Benchmark](https://redis.io/docs/management/optimization/benchmarks/).

## Use Musl libc

By default, RuxOS Redis Server is supported by ruxlibc, which is a customized standard C library. 

By adding `MUSL=y` to the make command, redis server will be compiled, linked by standard Musl libc.

## Use 9pfs

By default, RuxOS passes application arguments by `ARGS`, which can be inconvenient. RuxOS successfully integrate 9p protocol, and provide 9pfs to share directories and files between host and qemu. 

Run following command:
```shell
make A=apps/c/redis/ LOG=error NET=y V9P=y BLK=y V9P_PATH=apps/c/redis ARCH=aarch64 SMP=4 ARGS="./redis-server,/v9fs/redis.conf" run
```
Parameter explanation:
* `V9P`: Use `V9P=y` to enable virtio-9p on qemu.
* `V9P_PATH`: `V9P_PATH` points to the shared directory, which contains the redis configuration file.
