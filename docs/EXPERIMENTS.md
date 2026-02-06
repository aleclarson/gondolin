# Fun Experiments

Learning by doing is the best way to understand Gondolin. Try these experiments!

## 1. The Jailbreak Attempt
**Goal:** Try to access a file on your Host machine that you *didn't* mount.
*   **Steps:**
    1.  Start the shell: `gondolin bash`
    2.  Try to read `/etc/passwd` (this is the Guest's passwd).
    3.  Try to find a way to escape to the Host.
*   **Lesson:** Understand the isolation boundaries.

## 2. The Man-in-the-Middle
**Goal:** Intercept and rewrite a web request.
*   **Steps:**
    1.  Modify `host/src/http-hooks.ts` (or wherever you initialize the VM).
    2.  Add an `onResponse` hook that replaces the body of `example.com` with "HACKED".
    3.  Run `curl example.com` inside the Guest.
*   **Lesson:** See how the Network Stack's interception capability works.

## 3. The Big Mount
**Goal:** Test VFS performance.
*   **Steps:**
    1.  Mount a large local git repository (like the Linux kernel) into the VM.
    2.  Run `git status` inside the Guest.
    3.  Watch it crawl.
*   **Lesson:** Feel the performance cost of FUSE/Virtio and understand why `LIMITATIONS.md` exists.
