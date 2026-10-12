# cnips

Command-line client for the [CNIPS Integration Platform](https://cnips.io).

`cnips` is for local development, testing, and deployment of integration
pipelines, components, and functions: init projects, build and run pipelines
locally, sync with remote workspaces, and publish to the marketplace.

This repository distributes **release binaries**. The source is not published
here.

## Install

```bash
brew install cnips/tap/cnips
```

Homebrew works on macOS and Linux. Upgrade with `brew upgrade cnips`.

To download instead, pick the archive for your platform from the
[latest release](https://github.com/cnips/cnips-cli/releases/latest):

| Platform | Archive |
| --- | --- |
| macOS, Apple silicon | `cnips_<version>_darwin_arm64.tar.gz` |
| macOS, Intel | `cnips_<version>_darwin_amd64.tar.gz` |
| Linux, x86-64 | `cnips_<version>_linux_amd64.tar.gz` |
| Linux, arm64 | `cnips_<version>_linux_arm64.tar.gz` |
| Linux, 32-bit ARM (ARMv7) | `cnips_<version>_linux_armv7.tar.gz` |
| Windows, x86-64 | `cnips_<version>_windows_amd64.zip` |
| Windows, arm64 | `cnips_<version>_windows_arm64.zip` |

For example, on Linux x86-64:

```bash
VERSION=$(curl -fsSL https://api.github.com/repos/cnips/cnips-cli/releases/latest \
  | grep -o '"tag_name": *"[^"]*"' | cut -d'"' -f4) &&
curl -fsSLO "https://github.com/cnips/cnips-cli/releases/download/${VERSION}/cnips_${VERSION}_linux_amd64.tar.gz" &&
curl -fsSLO "https://github.com/cnips/cnips-cli/releases/download/${VERSION}/checksums.txt" &&
sha256sum -c checksums.txt --ignore-missing &&
tar -xzf "cnips_${VERSION}_linux_amd64.tar.gz" cnips &&
sudo mv cnips /usr/local/bin/
```

Each archive holds the `cnips` binary and the license. The binaries are
statically linked and have no runtime dependencies. The ARMv7 build does not run
on ARMv6 boards (Raspberry Pi 1, Pi Zero W).

On macOS, a binary downloaded through a browser is quarantined by Gatekeeper,
because it is not signed or notarized. Clear it with
`xattr -d com.apple.quarantine /usr/local/bin/cnips`. Downloads made with
`curl`, and installs through Homebrew, are not affected.

## Quick start

```bash
cnips version
cnips --help

cnips init my-integrations
cd my-integrations
cnips login --base-url https://your-cnips-instance.com
```

## Support

Report issues to your CNIPS contact. This repository is for distribution and
does not track source issues.

## License

[Apache License 2.0](LICENSE).
