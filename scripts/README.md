# scripts/

Release and install helpers.

| Script | Purpose |
| --- | --- |
| [`install.sh`](install.sh) | Install AURA binaries from GitHub Releases |
| [`bump-homebrew-tap.sh`](bump-homebrew-tap.sh) | Bump `mezmo/homebrew-tap` formulae to a released version |
| [`set-version.sh`](set-version.sh) | Set the workspace and crate versions in `Cargo.toml` |

## `install.sh`

```bash
curl -fsSL https://raw.githubusercontent.com/mezmo/aura/main/scripts/install.sh | bash
```

Installs `aura` (CLI) and `aura-web-server` for `linux`/`darwin` on `amd64`/`arm64`.

The script takes no command-line arguments. Every switch is an environment
variable, so it works unchanged when piped into `bash`:

```bash
curl -fsSL .../install.sh | AURA_COMPONENT=cli AURA_VERSION=0.1.3 bash
```

### Switches

| Variable | Default | Effect |
| --- | --- | --- |
| `AURA_VERSION` | `latest` | Release tag to install. A leading `v` is optional (`0.1.3` and `v0.1.3` both work). `latest` is resolved by following the `releases/latest` redirect. |
| `AURA_INSTALL` | `~/.local/bin` | Install directory. Created if missing. |
| `AURA_COMPONENT` | `all` | Which binaries to install: `all`, `server` (`aura-web-server` only), or `cli` (`aura` only). Any other value is an error. |
| `AURA_REQUIRE_CHECKSUM` | `0` | `1` makes a missing `checksums.txt`, or a missing entry for an asset, a fatal error instead of a warning. A checksum *mismatch* is always fatal. |
| `AURA_CHECKSUMS` | unset | Path to a local `checksums.txt` to verify against, instead of downloading one from the release. |
| `AURA_NO_BREW` | `0` | `1` skips Homebrew on macOS and downloads the release assets directly. |

### Homebrew on macOS

On macOS the script installs from the `mezmo/tap/aura` tap instead of
downloading assets, but only when all of these hold:

- `brew` is on `PATH`
- `AURA_NO_BREW` is not `1`
- `AURA_VERSION` is `latest` (the tap cannot pin versions)
- `AURA_COMPONENT` is `all`

Otherwise it prints why and falls back to a direct download. `AURA_INSTALL` does
not apply to the Homebrew path.

The `mezmo/tap/aura` formula provides the `aura` CLI only, so this path does not
install `aura-web-server`. Set `AURA_NO_BREW=1` to get both binaries on macOS.

### Requirements

- `curl` or `wget` for downloads
- `sha256sum` or `shasum` for checksum verification. If neither is installed,
  verification is skipped with a warning, or fails when
  `AURA_REQUIRE_CHECKSUM=1`.

## `bump-homebrew-tap.sh`

```
bump-homebrew-tap.sh [--dry-run] <version>
```

Rewrites the `version` and `sha256` fields in each `Formula/*.rb` of
`mezmo/homebrew-tap` and pushes to `main`.

| Switch | Default | Effect |
| --- | --- | --- |
| `--dry-run` / `DRY_RUN=1` | off | Print the proposed commit and test the push with `git push --dry-run` without updating any refs. Tolerates a missing checksums file, validating the version bump only. |
| `CHECKSUMS_FILE` | `dist/checksums.txt` | Release checksums to source each `sha256` from. |
| `GH_TOKEN` / `GITHUB_TOKEN` | unset | Token used to clone and push the tap. Required unless `--dry-run`. |

Exits 0 without committing when the formulae already sit at the target version.

## `set-version.sh`

```
set-version.sh <version>
```

Sets `version` in the workspace `Cargo.toml` and in each `crates/*/Cargo.toml`
that carries its own version, then runs `make update-lockfile`. Stands in for
`cargo set-version`, which does not build against this workspace's edition 2024
requirements.
