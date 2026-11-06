# Nx "auto" affected: a package removed from the lockfile marks every project

Minimal reproduction for `projectsAffectedByDependencyUpdates: "auto"` (Nx 24.0.4, bun 1.4.4).

Two projects: `a` depends on `ms`, `b` has no dependencies. Three commits:

1. `a depends on ms`
2. `a: change ms 2.1.3 -> 2.1.2`, a version change
3. `a: remove ms`, a removal

```bash
bun install

# Version change: only `a` (correct)
git checkout HEAD~1 && bun install
bunx nx show projects --affected --base=HEAD~1 --head=HEAD   # ["a"]

# Removal: `a` and `b` (expected only `a`)
git checkout main && bun install
bunx nx show projects --affected --base=HEAD~1 --head=HEAD   # ["a","b"]
```

`b` is marked because `ms` is gone from the head lockfile, so it has no external node in the head graph, and `changedExternalNodes` falls back to every project.
