# Here Be Dragons 🐉

This document warns of parts of the codebase that are particularly complex, fragile, or non-intuitive. Modify these areas with extreme caution.

## 1. The Network Stack (`host/src/network-stack.ts`)
*   **What it is:** A complete implementation of Ethernet, ARP, IP, TCP, and DHCP written in TypeScript.
*   **The Dragon:** It is *not* a standard OS stack. It creates "fake" TCP connections to satisfy the Guest's kernel, while proxying the actual data via Node.js `fetch` or `http` modules.
*   **Risk:** Touching state machine logic (SYN/ACK handling) can easily break connectivity or cause hang-ups.

## 2. Virtio Framing (`host/src/virtio-protocol.ts`)
*   **What it is:** Decodes the binary stream coming from the QEMU virtio-serial device.
*   **The Dragon:** It multiplexes multiple "channels" (stdin, stdout, fs-rpc) over a single pipe.
*   **Risk:** Off-by-one errors here will corrupt data streams effectively permanently.

## 3. Alpine Initramfs Build (`guest/image/`)
*   **What it is:** Scripts to download APKs and repack them into a cpio archive.
*   **The Dragon:** Alpine package versions drift. The build relies on specific mirrors and package behavior.
*   **Risk:** Upgrading Alpine versions often breaks the minimal boot process (missing shared libs, changed init paths).
