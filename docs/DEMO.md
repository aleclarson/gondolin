# Quick Demo

Want to see Gondolin in action immediately?

## The "One-Liner"

Assuming you have `npm` and `qemu` installed, run this command to verify everything is working:

```bash
npx @earendil-works/gondolin bash
```

This will download the Guest image (if not present) and drop you into a shell inside the micro-VM.

## Verify Networking

Inside the guest shell, try:
```bash
curl https://www.google.com
```
You should see the HTML response. This confirms:
1.  The VM booted.
2.  The Guest sent an HTTP request.
3.  The Host intercepted it, performed the fetch, and returned the result.

## Verify Filesystem

```bash
ls -la /
```
You are seeing the minimal Alpine rootfs.

## Next Steps

Check out `docs/EXPERIMENTS.md` for more fun things to try!
