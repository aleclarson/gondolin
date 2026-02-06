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

## 4. Additional Documentation Concepts

I have also evaluated several other potential documentation improvements. Here is my analysis on how to best integrate them:

### A. Demo Folder / Quickstart Script
**Proposal:** A `demo/` folder or script to get a working example immediately.
**Recommendation:** Integrate into `examples/` and referenced in `CONTRIBUTING.md`.
**Reasoning:** The current `examples/` folder exists but might not be "one-click" enough. A dedicated script that runs a cool demo (e.g., "fetch a secret and write to a file") without needing complex setup (other than `npm install`) would be a huge confidence booster. It validates the environment works before diving into code.

### B. Current Limitations
**Proposal:** A section explicitly listing what the system *cannot* do.
**Recommendation:** Add to `README.md` or `docs/ARCHITECTURE.md`.
**Reasoning:** Managing expectations is crucial. If a user expects full Docker compatibility or raw socket access, they might be disappointed. Explicitly listing limitations (e.g., "No raw TCP/UDP egress," "Filesystem performance overhead," "Single-threaded guest supervisor") saves time for everyone and helps define the project's scope.

### C. Pareto Features ("High Impact, Low Effort")
**Proposal:** A list of features that would skyrocket interest with minimal effort.
**Recommendation:** Add as a "Help Wanted / Roadmap" section in `CONTRIBUTING.md` or a pinned GitHub Issue.
**Reasoning:** New contributors often ask "What can I do?". A curated list of high-impact tasks (e.g., "Add support for X language in guest," "Implement Y VFS optimization") gamifies contribution and directs energy where it matters most.

### D. `DRAGONS.md` (Complexity Warnings)
**Proposal:** A warning label for the most complex parts of the codebase.
**Recommendation:** Merge into `docs/ARCHITECTURE.md` or keep as commented warnings in code.
**Reasoning:** While a standalone `DRAGONS.md` is fun, it might get out of date. It is better to highlight these areas in `ARCHITECTURE.md` (e.g., "The Network Stack is custom-written in JS; modify with caution") and ensure the code itself has extensive comments in those "dragon" areas (like `host/src/network-stack.ts`).

### E. Fun Experiments
**Proposal:** Fun things to try to get familiar with the project.
**Recommendation:** Add to `CONTRIBUTING.md` under "Getting Started" or a `docs/TUTORIAL.md`.
**Reasoning:** Learning by doing is best. Suggested experiments like "Try to break out of the sandbox," "Write a script that talks to a local server," or "Mount a huge directory and benchmark it" give users a structured way to explore the capabilities and boundaries of Gondolin.
