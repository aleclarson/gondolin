# Contributing to Gondolin

Welcome! This is a fork of the Gondolin project. We appreciate your interest in contributing.

## Prerequisites

To build and run Gondolin, you will need the following tools:

*   **Node.js & pnpm:** For the Host controller and build scripts.
*   **Zig 0.15.2:** Strictly required for building the Guest binaries (`sandboxd`).
*   **QEMU:** To run the micro-VMs (`qemu-system-aarch64` or `qemu-system-x86_64`).
*   **System Tools:** `lz4` (initramfs compression), `e2fsprogs` (rootfs creation), `curl`.

## Development Setup

1.  **Clone the repository:**
    ```bash
    git clone <fork-url>
    cd gondolin
    ```

2.  **Install Node dependencies:**
    ```bash
    pnpm install
    ```

3.  **Build everything:**
    ```bash
    make build
    ```
    This command builds the Guest kernel, initramfs, and `sandboxd` binary, and prepares the Host TypeScript code.

## Project Structure

*   `guest/`: Zig source code for the in-VM daemon and build logic.
*   `host/`: TypeScript source code for the VM controller, network stack, and CLI.
*   `scripts/`: Build and utility scripts.
*   `docs/`: Documentation.

## Workflow

*   **Run the CLI:**
    ```bash
    cd host
    pnpm run bash
    ```
    This launches a shell inside the Gondolin VM.

*   **Run Tests:**
    ```bash
    make test
    ```
    **Note:** Some tests require a KVM/HVF capable environment.

## Code Standards

*   **TypeScript:** Follow the existing style (Prettier/ESLint are configured).
*   **Zig:** Use `zig fmt` to format code.
