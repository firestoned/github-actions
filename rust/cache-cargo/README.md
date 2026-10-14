# Cache Cargo Dependencies

**Caches the cargo registry, git checkouts and compiled dependencies for Rust
builds, keyed per job, toolchain and architecture, and pruned before saving.**

This action is a thin, opinionated wrapper around
[`Swatinem/rust-cache`](https://github.com/Swatinem/rust-cache), pinned to the
same release that [`rust/setup-rust-build`](../setup-rust-build/README.md)
uses. It caches:

1. **Cargo registry** (`~/.cargo/registry`): downloaded `.crate` archives and
   the sparse index.
2. **Cargo git checkouts** (`~/.cargo/git`): dependencies sourced from git.
3. **Compiled dependencies** in `target/`: the `.rlib`, `.rmeta` and build
   script outputs of your **dependencies**, not of your own workspace crates.
4. **Installed cargo binaries** (`~/.cargo/bin`): tools installed with
   `cargo install` during the job, such as `cargo-cyclonedx`.

## Features

- ✅ **No required inputs**: existing `uses:` lines keep working unchanged
- ✅ **Per-job keys**: a clippy job, a test job and a release build no longer
  overwrite each other's `target/`
- ✅ **Toolchain and architecture aware**: the key includes `rustc -vV`, so a
  new stable release or an ARM64 runner gets its own entry
- ✅ **Pruned before saving**: workspace crates and stale artifacts are
  removed, so the cache holds dependencies only and stays small
- ✅ **Bounded fallback**: a partial restore only ever comes from the same job
  on the same toolchain, never from a different job's leftovers
- ✅ **Quota control**: `save-if` lets pull requests restore without saving
- ✅ **Fails loudly** when no toolchain is installed, instead of caching under
  a meaningless key

## Usage

### Basic Usage

The toolchain must be installed **before** this action, because the cache key
is derived from `rustc -vV`.

```yaml
- uses: actions/checkout@v4

- uses: dtolnay/rust-toolchain@stable

- uses: firestoned/github-actions/rust/cache-cargo@v1

- name: Build
  run: cargo build --release
```

### Complete CI Workflow

Each job gets its own cache automatically. Nothing needs to be configured to
keep `clippy`, `test` and `build` apart.

```yaml
name: Rust CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  clippy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: clippy
      - uses: firestoned/github-actions/rust/cache-cargo@v1
        with:
          save-if: ${{ github.ref == 'refs/heads/main' }}
      - run: cargo clippy --all-targets -- -D warnings

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - uses: firestoned/github-actions/rust/cache-cargo@v1
        with:
          save-if: ${{ github.ref == 'refs/heads/main' }}
      - run: cargo test

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - uses: firestoned/github-actions/rust/cache-cargo@v1
        with:
          save-if: ${{ github.ref == 'refs/heads/main' }}
      - run: cargo build --release
```

### Multi-Platform Builds

The key includes the host triple reported by `rustc -vV`, so x86_64 and ARM64
runners never share an entry even when they run the same job id.

```yaml
jobs:
  build:
    strategy:
      matrix:
        runner: [ubuntu-latest, ubuntu-24.04-arm, macos-latest]
    runs-on: ${{ matrix.runner }}
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - uses: firestoned/github-actions/rust/cache-cargo@v1
      - run: cargo build --release
```

### Matrix Over Profiles or Targets on One Runner

When one job id builds different things on the same runner, add `key` so the
matrix legs do not overwrite each other:

```yaml
jobs:
  build:
    strategy:
      matrix:
        target: [x86_64-unknown-linux-gnu, x86_64-unknown-linux-musl]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.target }}
      - uses: firestoned/github-actions/rust/cache-cargo@v1
        with:
          key: ${{ matrix.target }}
      - run: cargo build --release --target ${{ matrix.target }}
```

### Sharing One Cache Across Jobs

`shared-key` replaces the per-job key. Use it only for jobs that compile
**exactly** the same profile, features and targets; otherwise they evict each
other, which is the problem this action's per-job default exists to prevent.

```yaml
- uses: firestoned/github-actions/rust/cache-cargo@v1
  with:
    shared-key: debug-tests
```

### Workspace Not at the Repository Root

```yaml
- uses: firestoned/github-actions/rust/cache-cargo@v1
  with:
    workspaces: |
      crates/server -> target
      tools/xtask -> target
```

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `key` | No | `''` | Extra key component added alongside the automatic job-based key. For matrix legs that share a job id and runner but build different things. |
| `shared-key` | No | `''` | Replaces the job-based key so several jobs share one cache. Only for jobs that build identical profiles, features and targets. |
| `save-if` | No | `'true'` | Whether to save at the end of the job. Pass `${{ github.ref == 'refs/heads/main' }}` to save only on the default branch. |
| `cache-on-failure` | No | `'false'` | Save the cache even if the job fails. |
| `workspaces` | No | `'. -> target'` | Newline-separated `workspace -> target` pairs. |
| `prefix-key` | No | `'v1-cargo'` | Key prefix. Change it to discard every existing entry and start over. |

## Outputs

| Output | Description |
| --- | --- |
| `cache-hit` | `true` when an exact key match was restored, `false` otherwise (including partial restores from a fallback key). |

## How It Works

### Cache key

The key is assembled by `rust-cache` from:

```
<prefix-key>-<job id>[-<key>]-<hash of: rustc -vV, CARGO_* and RUST* env, Cargo.toml / Cargo.lock / rust-toolchain files>
```

So a new entry is created when:

- the job id changes, or `key` differs;
- the toolchain changes: a new stable release, or a different host triple
  (x86_64 against ARM64);
- `Cargo.lock`, any `Cargo.toml`, `rust-toolchain(.toml)` or `.cargo/config`
  changes;
- a `CARGO_*`, `RUST*` or `CC`-style environment variable that affects the build
  changes.

### Restore fallback

On an exact miss, the most recent entry for the **same job and toolchain** is
restored and cargo rebuilds only what changed. A fallback never crosses jobs or
toolchains, so a test job cannot inherit a release build's `target/`.

### Pruning before save

Before saving, `rust-cache` removes from `target/`:

- the artifacts of your own workspace crates, which change on every commit and
  would make every entry unique anyway;
- dependency artifacts not used by the current build;
- incremental compilation directories.

and from `~/.cargo/registry` the extracted sources that can be recreated from
the archives. The result is an entry that holds what is expensive to rebuild
and nothing else.

### Saving

The entry is saved in the job's post step, only when:

- `save-if` is `true`;
- the job succeeded, or `cache-on-failure` is `true`;
- the key was not an exact hit (an existing entry is never re-uploaded).

## Why This Changed (v1.4.0)

Up to v1.3.x, this action cached the whole `target/` directory under a single
key per OS and `Cargo.lock` hash:

```
Linux-cargo-build-target-<Cargo.lock hash>
```

Every job in a workflow used that same key. In practice that meant:

- **Jobs evicted each other.** A clippy job (check-only metadata), a test job
  (`dev` profile with dev-dependencies), a release build, and an ARM64 build on
  a different architecture all restored whichever `target/` was saved first.
  `runner.os` is `Linux` on ARM64 runners too, so even the architecture did not
  separate them.
- **The entry only grew.** The prefix `restore-keys` carried the previous
  entry forward on every lockfile change and nothing was ever pruned.
- **Restores cost more than they saved.** In one consumer the entry reached
  4.4 GB. Restoring it took 2 minutes 42 seconds, after which `cargo test`
  still recompiled 38 crates because the cached artifacts were built for a
  different profile and feature set. Together with the other caches it held
  that repository at 9.85 GB of GitHub's 10 GB limit, so GitHub was evicting
  entries least-recently-used.

The interface is unchanged: no input is required, and existing `uses:` lines
work as before. The first run after upgrading starts from an empty cache,
because the keys are new.

## Troubleshooting

### `rust/cache-cargo needs a Rust toolchain installed first`

The cache key is computed from `rustc -vV`. Move the toolchain step above this
action:

```yaml
- uses: dtolnay/rust-toolchain@stable
- uses: firestoned/github-actions/rust/cache-cargo@v1
```

### Cache Miss on Every Run

1. **Is `Cargo.lock` committed?** Without it the key changes whenever
   resolution does.
2. **Is the toolchain floating?** `@stable` moves roughly every six weeks and
   each release starts a new entry. Pin a version, or accept one cold run per
   release.
3. **Is `save-if` false for the branch that should populate it?** With
   `save-if: ${{ github.ref == 'refs/heads/main' }}`, a pull request only reads
   entries that a run on `main` has saved.
4. **Has the entry been evicted?** Check with `gh cache list`. Repositories
   share a 10 GB quota and GitHub evicts the least recently used entries above
   it.

### Builds Still Recompile Dependencies After a Hit

Cargo reuses an artifact only when the profile, the feature set and the
compiler flags all match. Common causes:

- a different `RUSTFLAGS` or `CARGO_PROFILE_*` value than the run that saved
  the entry (both are part of the key, so this usually shows as a miss rather
  than a hit);
- `--all-features` in one job and default features in another sharing a
  `shared-key`. Give them separate keys.

### Cache Size Limits

```bash
# Show entries, largest first
gh cache list --sort size_in_bytes --order desc

# Delete an entry
gh cache delete <key-or-id>
```

Saving only on the default branch is usually the biggest single reduction.

## Migrating From v1.3.x

No workflow change is required. Recommended follow-ups:

1. **Make sure the toolchain step comes first.** v1.3.x did not need it;
   v1.4.0 fails fast without it.
2. **Add `save-if` for the default branch** to stop pull requests spending the
   quota.
3. **Delete the old entries once**, so they stop counting against the quota:

   ```bash
   gh cache list --key Linux-cargo-build-target- --json id --jq '.[].id' \
     | xargs -n1 gh cache delete
   gh cache list --key Linux-cargo-registry- --json id --jq '.[].id' \
     | xargs -n1 gh cache delete
   gh cache list --key Linux-cargo-git- --json id --jq '.[].id' \
     | xargs -n1 gh cache delete
   ```

4. **Remove `needs:` edges that only existed to share a cache.** Jobs no
   longer share `target/`, so a test job does not gain anything by waiting for
   a build job.

## Compatibility

- **Rust versions:** stable, beta, nightly
- **Operating systems:** Linux, macOS, Windows
- **Runners:** GitHub-hosted (x86_64 and ARM64) and self-hosted
- **Workspaces:** yes, including several workspaces via `workspaces`
- **Cross-compilation:** yes; the target directory layout under `target/<triple>/`
  is cached and pruned like the host one

## Comparison With Using `Swatinem/rust-cache` Directly

| | `rust/cache-cargo` | `Swatinem/rust-cache` |
| --- | --- | --- |
| Pinned to a reviewed SHA | ✅ maintained here | you pin it |
| Fails without a toolchain | ✅ | ❌ (keys on an empty `rustc -vV`) |
| Step summary | ✅ | ❌ |
| Per-job keys, pruning, arch awareness | ✅ (from rust-cache) | ✅ |
| Every rust-cache input | a curated subset | ✅ |

Use this action to keep the pin and the defaults in one place across
repositories. If you need an input it does not expose, open an issue rather than
inlining `Swatinem/rust-cache` in a consumer.

## Related Actions

- [`rust/setup-rust-build`](../setup-rust-build/README.md): toolchain, cache
  and `cross` setup for multi-architecture builds
- [`rust/build-binary`](../build-binary/README.md): build Rust binaries
- [`rust/build-library`](../build-library/README.md): build Rust libraries
- [`rust/security-scan`](../security-scan/README.md): scan dependencies for
  vulnerabilities

## License

MIT License - see [LICENSE](../../LICENSE) for details.

## Contributing

Contributions welcome! Please see [CONTRIBUTING.md](../../CONTRIBUTING.md) for
guidelines.
