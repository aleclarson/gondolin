# Agent Integration Guide

This document is a comprehensive, compact usage guide for AI agents and developers integrating with Gondolin. It focuses on how to use the library to create, control, and secure micro-VMs.

## Core Concepts

Gondolin runs a lightweight Linux VM controlled by a Node.js process.
*   **Host:** Your Node.js application.
*   **Guest:** The VM environment where untrusted code runs.
*   **Isolation:** The Guest has NO network access by default. All HTTP/HTTPS traffic is intercepted and proxied by the Host.

## Guest Environment

You have access to a Linux Micro VM (Alpine). Beyond standard BusyBox utilities, the following toolset is pre-installed:

*   **Languages:** Python 3.12, Node.js (with npm/npx).
*   **Package Management:** `uv` (use this for fast Python pkg installs), `npm`.
*   **Network:** `curl`, `wget`, `wcurl`, `nc` (netcat), `nslookup`, `whois`.
*   **Utilities:** `tree` (use this to explore file structure), `lsof`, `pstree`, `bc`, `xxd`.

> **Note:** `git` is **not** present in `/usr/bin`, so plan accordingly.

## API Reference

### 1. Creating a VM

```ts
import { VM, createHttpHooks, MemoryProvider, RealFSProvider } from "@earendil-works/gondolin";

// Define Network Policy
const { httpHooks, env } = createHttpHooks({
  // Only allow specific domains
  allowedHosts: ["api.openai.com", "github.com"],
  // Inject secrets ONLY for specific hosts (Guest never sees the raw value)
  secrets: {
    OPENAI_API_KEY: {
      hosts: ["api.openai.com"],
      value: process.env.OPENAI_API_KEY,
    },
  },
});

// Initialize VM
const vm = await VM.create({
  // Network hooks for interception
  httpHooks,
  // Environment variables for the guest (e.g. placeholder secrets)
  env,
  // Virtual Filesystem Configuration
  vfs: {
    mounts: {
      // Ephemeral memory filesystem at /workspace
      "/workspace": new MemoryProvider(),
      // Read-only mount of a host directory
      "/data": new ReadonlyProvider(new RealFSProvider("./host-data")),
    },
  },
});
```

### 2. Executing Commands

Commands run inside the Guest. Standard streams are returned.

```ts
const result = await vm.exec("ls -la /workspace");
if (result.exitCode !== 0) {
  console.error("Command failed:", result.stderr);
} else {
  console.log("Output:", result.stdout);
}

// Streaming execution (for long-running processes)
await vm.exec("npm install", [], {
  onStdout: (chunk) => process.stdout.write(chunk),
  onStderr: (chunk) => process.stderr.write(chunk),
});
```

### 3. Cleanup

Always close the VM to kill the QEMU process.

```ts
await vm.close();
```

## Best Practices

*   **Secret Safety:** NEVER pass raw secrets via `env` or `exec`. Use the `secrets` option in `createHttpHooks`. The Guest receives a placeholder (e.g., `$OPENAI_API_KEY`), and the Host injects the real value at the network layer.
*   **Filesystem:** Prefer `MemoryProvider` for temporary work. Use `RealFSProvider` sparingly and consider `ReadonlyProvider` wrapper for safety.
*   **Networking:** The Guest cannot use raw TCP/UDP. Do not try to `ping` or open database connections unless they are HTTP-based.
*   **Concurrency:** The VM is single-threaded. Avoid running massive parallel builds inside one VM.

## Internal Architecture (For Debugging)

*   **Virtio-Serial:** Used for all Host<->Guest communication (exec, fs-rpc).
*   **Network:** The Host implements a JS-based TCP/IP stack. It terminates Guest TCP connections and proxies them via Node's `fetch`.
