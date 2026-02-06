# Proposed Onboarding Documentation for Gondolin

To significantly improve the onboarding experience for new contributors to the Gondolin project, I propose creating the following three high-impact documents. These documents address the current gaps in documentation regarding contribution workflows, system architecture, and debugging.

## 1. `CONTRIBUTING.md`

**Purpose:**
This document serves as the single source of truth for potential contributors. Currently, development instructions are scattered across `README.md`, `host/README.md`, `guest/README.md`, and `AGENTS.md`. A unified `CONTRIBUTING.md` in the root directory will streamline the setup process and reduce friction for new developers.

**Proposed Contents:**
-   **Introduction:** A warm welcome and a brief explanation of what contributions are helpful (code, docs, issues).
-   **Prerequisites:** A consolidated list of tools required to build both Host and Guest (Zig 0.15.2, Node.js/pnpm, QEMU, lz4, e2fsprogs, etc.). It should explicitly mention version requirements (e.g., Zig 0.15.2).
-   **Development Environment Setup:** Step-by-step instructions to bootstrap the repo:
    -   `pnpm install`
    -   `make build` (explaining it builds guest kernel/initramfs and host code).
-   **Project Structure:** A quick map of the repo (`guest/`, `host/`, `scripts/`) so developers know where to look.
-   **Development Workflow:**
    -   How to run the development CLI (`pnpm run bash` in `host/` or `npx ...` equivalent).
    -   How to run tests (`make test`, `cd host && pnpm test`). Mention the VM vs Unit test distinction found in `AGENTS.md`.
-   **Code Standards:** Mentioning TypeScript and Zig styles, and the `make check` command for linting.

**Reasoning:**
New contributors often struggle with setting up the environment, especially in a polyglot repo (TypeScript + Zig + System tools). `AGENTS.md` contains valuable info (like `make build` and test commands) that isn't prominently visible to human contributors in the root `README`. Centralizing this lowers the barrier to entry.

## 2. `docs/ARCHITECTURE.md`

**Purpose:**
Gondolin has a complex architecture involving a host process (Node.js) managing a guest VM (QEMU/Linux) via custom protocols (Virtio-serial) and network stacks. Understanding how these pieces fit together is non-trivial. This document will provide the "mental model" needed to make meaningful code changes.

**Proposed Contents:**
-   **High-Level Overview:** A diagram or description of the Host-Guest relationship.
-   **Components:**
    -   **Guest (`guest/`):** The role of `sandboxd` (supervisor) and the minimal Alpine initramfs.
    -   **Host (`host/`):** The role of the Controller, Network Stack, and VFS.
-   **Communication Channels:**
    -   **Control Plane:** Explanation of the virtio-serial connection, how the host sends commands (`exec`), and how streams (stdin/out/err) are multiplexed.
    -   **Data Plane (Networking):** How the custom JavaScript network stack (Ethernet/IP/TCP) interacts with the VM. Explanation of how packets are intercepted and how `fetch` is used for egress.
    -   **Filesystem (VFS):** How the guest's FUSE filesystem (`sandboxfs`) communicates with the host to lazy-load files or write data.
-   **Security Model:** Deep dive into how secrets are injected (network hooking) and why the guest never sees the raw secrets.

**Reasoning:**
The codebase is split between `guest` (Zig) and `host` (TS). A contributor wanting to add a feature (e.g., a new VFS capability) needs to understand both sides of the bridge. Currently, this knowledge requires reading the source code of `sandboxd` and `sandbox-controller.ts`. An architecture guide accelerates understanding of these interactions.

## 3. `docs/DEBUGGING.md`

**Purpose:**
Debugging a system that spans a host process and a virtualized guest is difficult. Standard debugging tools don't always apply seamlessly. This document will curate techniques for diagnosing issues in this specific environment.

**Proposed Contents:**
-   **Logs:**
    -   Where to find Host logs.
    -   How to view Guest logs (since stdout is captured by the protocol, is there a debug channel? or `dmesg` access?).
-   **Debugging the Host:** Standard Node.js debugging (inspector) attaching to the host process.
-   **Debugging the Guest:**
    -   How to inspect the guest state (e.g., running `ps` or `ls` inside the guest via `gondolin exec`).
    -   Advanced: Attaching GDB to QEMU (if supported/documented).
-   **Common Issues & Solutions:**
    -   "VM fails to boot" (Architecture mismatch? Missing QEMU?).
    -   "Network request failed" (TLS cert issues? Allowlist blocks?).
    -   "Build failed" (Zig version mismatch?).

**Reasoning:**
When things go wrong in a VM, they fail silently or cryptically. Providing a guide on how to peek inside the black box (the VM) and the complex network stack is essential for keeping contributors from giving up when they hit a bug.
