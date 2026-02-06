# Debugging Guide

Debugging the boundary between a Node.js process and a QEMU VM can be tricky.

## Viewing Logs

### Host Logs
Host logs are printed to the terminal where you run `gondolin`. You can often increase verbosity by setting `DEBUG=*` (if supported by the specific modules) or checking `console.log` outputs in the `host/` code.

### Guest Logs
The Guest's `stdout` and `stderr` are typically captured by the `sandboxd` protocol and streamed back to the Host.
*   If the VM crashes *before* `sandboxd` starts, you might miss logs.
*   **Tip:** You can edit the QEMU invocation in `host/src/vm.ts` to redirect serial output to a file or stdio for debugging early boot issues.

## Debugging Techniques

### Inspecting the Guest
If the VM boots but behaves oddly, you can explore it interactively:
```bash
# Launch a shell inside the VM
gondolin bash
```
Once inside, you can use standard Linux tools (`ls`, `ps`, `cat`) to inspect the state.

### Debugging the Host
Since the Host is just a Node.js process, you can attach a debugger:
```bash
node --inspect-brk node_modules/.bin/tsx host/src/index.ts ...
```

## Common Issues

*   **"VM fails to boot":** Often due to missing QEMU installation or architecture mismatch (trying to run ARM64 image on x86). Check your `qemu-system-*` installation.
*   **"Network request failed":** The internal network stack is strict. Ensure the domain is allowed in your `httpHooks` configuration. Check for TLS certificate errors.
*   **"Build failed":** Verify you are using **Zig 0.15.2**. Newer or older versions often break the build.
