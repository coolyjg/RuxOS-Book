
# Iperf3

[iPerf3](https://github.com/esnet/iperf) is a tool for active measurements of the maximum achievable bandwidth on IP networks.

## Build & run

Build and start the [`iperf3`](https://github.com/esnet/iperf) server on Ruxos:

```bash
# in ruxos root directory
make A=apps/c/iperf BLK=y NET=y ARCH=<arch> run
```

## Benchmark

In another shell, run the `iperf3` client:

* iperf on Ruxos as the receiver:

    ```bash
    # TCP
    iperf3 -c 127.0.0.1 -p 5555
    # UDP
    iperf3 -uc 127.0.0.1 -p 5555 -b <sender_bitrate> -l <buffer_len>
    ```

    You need to set the `<sender_bitrate>` (in bits/sec) to avoid sending packets too fast from the client when use UDP.

* iperf on Ruxos as the sender:

    ```bash
    # TCP
    iperf3 -c 127.0.0.1 -p 5555 -R
    # UDP
    iperf3 -uc 127.0.0.1 -p 5555 -b 0 -l <buffer_len> -R
    ```

By default, the `<buffer_len>` is 128 KB for TCP and 8 KB for UDP. Larger buffer length may improve the performance. You can change it by the `-l` option of `iperf3`.

Note that if the `<buffer_len>` is greater than `1472` (total packet length is exceeded the MTU of the NIC) when use UDP, packets fragmentation will occur. You should enable fragmentation features in [smoltcp](https://github.com/smoltcp-rs/smoltcp):

```toml
# in ruxos/modules/axnet/Cargo.toml
[dependencies.smoltcp]
git = "https://github.com/rcore-os/smoltcp.git"
rev = "2ade274"
default-features = false
features = [
  "alloc", "log",   # no std
  "medium-ethernet",
  "proto-ipv4",
  "socket-raw", "socket-icmp", "socket-udp", "socket-tcp", "socket-dns",
  # "fragmentation-buffer-size-65536", "proto-ipv4-fragmentation",
  # "reassembly-buffer-size-65536", "reassembly-buffer-count-32",
  # "assembler-max-segment-count-32",
]
```

