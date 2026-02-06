# Proposed Onboarding Documentation for Gondolin

To significantly improve the onboarding experience for new contributors to this Gondolin fork, I propose creating the following eight high-impact documents. These documents address the current gaps in documentation regarding contribution workflows, system architecture, and debugging.

Given that this is a fork not actively cooperating with the upstream creator, these documents are designed to be "snapshots" of knowledge to help new maintainers get up to speed quickly, without the expectation of constant upstream synchronization. The strategy favors **many focused documents** over fewer monolithic ones to improve discoverability and reduce update friction.

## 1. `CONTRIBUTING.md`

**Purpose:**
This document serves as the single source of truth for potential contributors to this fork. A unified `CONTRIBUTING.md` in the root directory will streamline the setup process.

**Proposed Contents:**
-   **Introduction:** Welcome to the fork. Explanation of the fork's goals (if any specific deviation exists).
-   **Prerequisites:** Consolidated list of tools (Zig 0.15.2, Node.js, QEMU, lz4, e2fsprogs).
-   **Development Environment Setup:** Step-by-step bootstrap (`pnpm install`, `make build`).
-   **Project Structure:** Map of `guest/`, `host/`, `scripts/`.
-   **Development Workflow:** How to run the CLI (`pnpm run bash`) and tests (`make test`).
-   **Code Standards:** TS and Zig styles.

**Reasoning:**
Centralizing setup instructions lowers the barrier to entry significantly.

**Value / Cost Rating:** **95/100** (Essential for any contributor; low cost to write once).

## 2. `docs/ARCHITECTURE.md`

**Purpose:**
Provides the "mental model" needed to make meaningful code changes by explaining the Host-Guest relationship.

**Proposed Contents:**
-   **High-Level Overview:** Host process (Node.js) vs Guest VM (QEMU/Linux).
-   **Components:** `sandboxd` (guest supervisor), Host Controller, Network Stack.
-   **Communication:** Virtio-serial control plane.
-   **Security Model:** Secret injection and network hooking strategy.

**Reasoning:**
Understanding the split architecture is the biggest hurdle for new devs.

**Value / Cost Rating:** **90/100** (Critical context; moderate effort to synthesize).

## 3. `docs/DEBUGGING.md`

**Purpose:**
Curates techniques for diagnosing issues in the complex Host-VM environment.

**Proposed Contents:**
-   **Logs:** Host logs vs Guest logs (captured stdout).
-   **Debugging Host:** Node.js inspector.
-   **Debugging Guest:** `gondolin exec` for inspection, QEMU monitor (if applicable).
-   **Common Issues:** "VM fails to boot", "Network request failed".

**Reasoning:**
VM debugging is opaque; a guide saves hours of frustration.

**Value / Cost Rating:** **85/100** (High time-saver; moderate effort to compile scenarios).

## 4. `docs/DEMO.md`

**Purpose:**
A quick-start guide to running a working example that demonstrates the core value prop (secure, networked micro-VMs).

**Proposed Contents:**
-   **One-Liner:** A simple command to spin up a VM and do something visible (e.g., `curl` a site).
-   **Walkthrough:** Explanation of what happened (VM boot -> Network Hook -> Response).
-   **Next Steps:** Pointer to `EXPERIMENTS.md`.

**Reasoning:**
Validates the environment works and builds confidence immediately.

**Value / Cost Rating:** **80/100** (High engagement; low cost).

## 5. `docs/LIMITATIONS.md`

**Purpose:**
Explicitly lists what the system *cannot* do to manage expectations.

**Proposed Contents:**
-   **Networking:** No raw TCP/UDP egress (only HTTP/interception).
-   **Performance:** Filesystem overhead due to FUSE/virtio.
-   **Concurrency:** Single-threaded guest supervisor limitations.
-   **Platform:** ARM64/Linux focus (limitations on x86 or Windows).

**Reasoning:**
Prevents users from wasting time trying to implement impossible features.

**Value / Cost Rating:** **75/100** (Prevents frustration; very low cost).

## 6. `docs/ROADMAP.md`

**Purpose:**
Highlights "Pareto Features" – high-impact changes that would significantly boost the project's value with minimal effort.

**Proposed Contents:**
-   **Language Support:** Adding Python/Go/Rust runtimes to the guest image.
-   **VFS Features:** Implementing a specific optimization or write-back cache.
-   **Tooling:** Better CLI output or logging.

**Reasoning:**
Gamifies contribution and directs energy to where it matters most for this fork.

**Value / Cost Rating:** **70/100** (Good for direction; requires triage effort).

## 7. `docs/DRAGONS.md`

**Purpose:**
A "Here Be Dragons" warning for the most complex, fragile, or nuanced parts of the codebase.

**Proposed Contents:**
-   **Custom Network Stack:** The pure-JS TCP/IP implementation in `host/src/network-stack.ts`.
-   **Virtio Protocol:** The binary framing and multiplexing logic.
-   **Guest Init:** The fragility of the Alpine initramfs build process.

**Reasoning:**
Prevents "chesterton's fence" removals and warns experienced devs where to tread carefully.

**Value / Cost Rating:** **65/100** (Crucial for deep dives; niche audience).

## 8. `docs/EXPERIMENTS.md`

**Purpose:**
A list of fun, educational tasks to help developers learn the system by doing.

**Proposed Contents:**
-   **Breakout:** Try to read a host file not mounted in the VFS.
-   **Benchmark:** Mount a large repo and time `git status`.
-   **Interception:** Write a hook to modify all HTTP responses to be "Hello World".

**Reasoning:**
Encourages playful exploration which leads to deeper understanding.

**Value / Cost Rating:** **60/100** (Fun; secondary to core docs).
