# Performance & Caching

Guidance for faster feedback cycles.

## Cold vs Warm Scan
- Cold scan: first run with no cache; gathers full dependency & metadata context.
- Warm scan: subsequent run leveraging cached intermediate results (if configured).

## Optimization Tips
- Narrow policy scope early (start with critical vulnerabilities only).
- Disable optional expensive features (e.g., dependency tree expansion) if latency sensitive.
- Parallelize scans at CI layer if repository is polyglot (future enhancement potential).

## Caching Strategy
- Use CI cache keys on lockfile hashes (`yarn.lock`, `package-lock.json`, etc.).
- Bust cache when configuration (`.codeward/`) changes.

## Measuring
Track job duration over several PRs; expect improvement after first warm cache use.

See also: [Troubleshooting & FAQ](troubleshooting-faq.md), [Architecture](../concepts/architecture.md).
