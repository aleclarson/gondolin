# Current Limitations

Gondolin is a specialized tool. It is NOT a general-purpose Docker replacement.

## Networking

*   **No Raw Sockets:** The Guest cannot open raw TCP or UDP sockets to the outside world. All traffic must be HTTP/HTTPS.
*   **No ICMP:** `ping` inside the guest is faked. It does not actually send ICMP packets.
*   **Protocol Support:** Only IPv4 is fully supported in the JS stack.

## Filesystem

*   **Performance:** The VFS (Virtual Filesystem) runs over virtio-serial and FUSE. It is significantly slower than a native disk. Heavy I/O operations (like compiling large projects inside the Guest) will be slow.
*   **Persistence:** By default, changes to the rootfs are lost on reboot (it's an overlay). You must mount a specific volume for persistence.

## Platform

*   **Architecture:** Primary support is for **ARM64 (Apple Silicon)** and **Linux aarch64**. x86_64 support exists but is less battle-tested.
*   **Windows:** Windows Host support is experimental/non-existent.
