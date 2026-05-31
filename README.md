# Setup Pebble SDK

A GitHub Action that sets up the [Pebble (Core Devices) SDK](https://developer.repebble.com/sdk/) build environment and, optionally, builds your Pebble app or watchface into a `.pbw`.

It behaves like `setup-node` / `setup-go`: it installs the toolchain so subsequent steps can run `pebble` commands, and with `build: true` it runs `pebble build` for you and reports the resulting `.pbw`.

> **Linux only.** Runs on GitHub-hosted `ubuntu-latest` (and compatible self-hosted Linux runners). It sets up the **build** environment — it does not install the QEMU emulator, so `pebble install --emulator` / screenshots are out of scope.

## Usage

### Set up the SDK (no build)

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: levonn-dev/setup-pebble@v1
  - run: pebble build   # or any other pebble command
```

### Set up and build a `.pbw`

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: levonn-dev/setup-pebble@v1
    id: pebble
    with:
      build: 'true'
  - uses: actions/upload-artifact@v4
    with:
      name: app-pbw
      path: ${{ steps.pebble.outputs.pbw-path }}
```

### Build a project in a subdirectory, with a pinned SDK

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: levonn-dev/setup-pebble@v1
    with:
      build: 'true'
      working-directory: watchfaces/my-face
      sdk-version: '4.4'   # pin instead of "latest"
```

## Inputs

| Name | Default | Description |
|------|---------|-------------|
| `build` | `false` | Run `pebble build` after setup to produce a `.pbw`. |
| `working-directory` | `.` | Project directory (contains `package.json` + `wscript`). Used only when `build: true`. |
| `sdk-version` | `latest` | SDK version passed to `pebble sdk install`. |
| `cache` | `true` | Cache `~/.pebble-sdk` and the `pebble-tool` install across runs. |

## Outputs

| Name | Description |
|------|-------------|
| `pbw-path` | Absolute path to the built `.pbw`. Set only when `build: true`. |

## What it does

1. Installs the system libraries the SDK needs (`libsdl2`, `libglib2.0`, `libpixman-1`, `zlib1g`, `libsndio`). Node.js is assumed to be present (it is preinstalled on GitHub-hosted `ubuntu-latest`).
2. Installs [`uv`](https://docs.astral.sh/uv/) and the Pebble CLI (`uv tool install --force pebble-tool --python 3.13`; `--force` keeps the `pebble` shim correct across cache restores).
3. Runs `pebble sdk install <sdk-version>`, which downloads the ARM toolchain and SDK into `~/.pebble-sdk`.
4. If `build: true`, runs `pebble build` in `working-directory` and exposes the resulting `.pbw` as the `pbw-path` output.

## Caching

With `cache: true` (the default), the action caches `~/.pebble-sdk` (SDK + toolchain) and `~/.local/share/uv` (the `pebble-tool` install and its Python), keyed by runner OS and `sdk-version`.

> With `sdk-version: latest`, the cached SDK is reused until the cache key changes. Pin `sdk-version` for fully reproducible builds, or set `cache: false` to always fetch fresh.

## Used by

- [pebble-adventure](https://github.com/levonn-dev/pebble-adventure)
- [pebble-adventure-wf](https://github.com/levonn-dev/pebble-adventure-wf)

A typical workflow for those projects:

```yaml
name: Build
on:
  push:
    branches: [main]
  pull_request:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: levonn-dev/setup-pebble@v1
        id: pebble
        with:
          build: 'true'
      - uses: actions/upload-artifact@v4
        with:
          name: pbw
          path: ${{ steps.pebble.outputs.pbw-path }}
```

## Example project

A minimal watchface used by this repo's test workflow lives in
[`examples/sample-watchface`](examples/sample-watchface). The
[`Test setup-pebble`](.github/workflows/test.yml) workflow exercises both
`build: false` (setup only) and `build: true` (setup + build) against it.

## License

[MIT](LICENSE) © 2026 levonn-dev
