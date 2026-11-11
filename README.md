# sandbox-executors

The official executor repository for [`sbx`](https://github.com/toollyst/sandbox): an index at
<https://toollyst.github.io/sandbox-executors/> (`v2/index.json`) and the archives it points to, in this
repository's [releases](https://github.com/toollyst/sandbox-executors/releases).

`sbx` lists it as `official` by default. It is one repository like any other: `sbx executor repo rm official` drops it,
and `sbx executor repo add NAME URL --key KEY` adds yours. The index format, the package layout and the executor
protocol are in the sandbox repository
([how executors work](https://github.com/toollyst/sandbox/blob/main/docs/developers/how-it-works.md)); nothing here is
special to `sbx`.

## What it publishes

| Name | Kind | Contents |
|------|------|----------|
| `wick` | VMM package | `firecracker` from a [toollyst/wick](https://github.com/toollyst/wick) release, or upstream Firecracker of the same version on a platform that release has no build for, with its `LICENSE` and `NOTICE`. Run by the `firecracker` adapter. |
| `firecracker` | adapter | `sbx-executor` (the `sbx-executor-firecracker` crate) and the default guest kernel, built from a pinned [toollyst/sandbox](https://github.com/toollyst/sandbox) commit, with `KERNEL-SOURCE.txt` for the kernel's GPL-2.0 source. |

```sh
sbx executor install wick      # installs the firecracker adapter first if it isn't installed
```

`wick` versions are wick's release tags without the `v` (`v1.18.0+wick.1` is `wick@1.18.0+wick.1`). Once a version
is published for a platform it is never rebuilt or replaced.

## Publishing

Everything is published by hand from GitHub Actions:

```sh
gh workflow run executors.yml                                   # wick's latest release, adapter from sandbox main
gh workflow run executors.yml -f wick=v1.18.0+wick.1 -f sandbox=v0.2.0
gh workflow run executors-resign.yml                            # re-sign the index before it expires
```

[`executors.yml`](.github/workflows/executors.yml) pins the `sandbox` input to one commit and builds the adapter, the
kernel and the repository tools (`cargo xtask repo-add`, `repo-sign`) from it. On Linux x86_64 with KVM, that
commit's `sbx` installs the new packages from a local copy of the repository and boots a VM before anything is
uploaded. Archives go to releases first, then the index is pushed to `gh-pages`.

Signing: once the repository secret `EXECUTORS_SIGNING_KEY` (a minisign secret key) is set, every publish signs
`v2/index.json` with a fresh `serial` and a 90-day `expires`, and `sbx` checks it against its built-in `OFFICIAL_KEYS`.
Until then the index is published unsigned, and `executors.yml` refuses to publish unsigned once `sbx` has a key.

## Dependencies

This repository depends on toollyst/sandbox (for the adapter and the tools) and on toollyst/wick (for VMM builds).
Nothing depends on this repository: `sbx` builds, tests and releases without it, and an executor you already
installed keeps working if it disappears.

## License

Apache-2.0 or MIT, like sbx ([LICENSE-APACHE](LICENSE-APACHE), [LICENSE-MIT](LICENSE-MIT)). The published packages
carry their own licenses (Firecracker and wick: Apache-2.0; the Linux kernel: GPL-2.0). Security issues: see
[toollyst/sandbox SECURITY.md](https://github.com/toollyst/sandbox/blob/main/SECURITY.md).
