# Roadmap & Help Wanted

This document lists "Pareto Features" – high-impact improvements that would provide significant value with moderate effort.

## Language Runtimes
*   **Python/Go/Rust:** The current image is minimal. Creating a "fat" image with popular language runtimes pre-installed would make the tool useful for more agents.

## VFS Improvements
*   **Write Caching:** Implement a write-back cache in the Host VFS to improve performance for small, frequent writes.
*   **Git Support:** Optimize `git` operations, which are currently slow due to high metadata traffic.

## Developer Experience
*   **Better Logging:** Structured logging for the Host controller to make debugging network traces easier.
*   **Status Bar:** A visual indicator in the CLI showing VM status and network activity.
