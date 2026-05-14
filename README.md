# freelens-nightly-builds

<!-- markdownlint-disable MD013 -->

[![Home](https://img.shields.io/badge/%F0%9F%8F%A0-freelens.app-02a7a0)](https://freelens.app)
[![GitHub](https://img.shields.io/github/stars/freelensapp/freelens-nightly-builds?style=flat&label=GitHub%20%E2%AD%90&logo=github)](https://github.com/freelensapp/freelens-nightly-builds)
[![nightly](https://img.shields.io/github/v/release/freelensapp/freelens-nightly-builds?display_name=tag&sort=semver&label=nightly)](https://github.com/freelensapp/freelens-nightly-builds/releases/latest)
[![Homebrew Cask Version](https://img.shields.io/homebrew/cask/v/freelens%40nightly?label=homebrew)](https://formulae.brew.sh/cask/freelens%40nightly#default)
[![Snapcraft](https://img.shields.io/snapcraft/v/freelens/latest/edge)](https://snapcraft.io/freelens)
[![Release nightly](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/release-nightly.yaml/badge.svg)](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/release-nightly.yaml)
[![Trunk Check](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/trunk-check.yaml/badge.svg?branch=main)](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/trunk-check.yaml)

<!-- markdownlint-enable MD013 -->

This is a project with nightly builds for the
[Freelens](https://freelens.app) application.

You can find the packages built from the main branch here:

<https://github.com/freelensapp/freelens-nightly-builds/releases>

For official stable releases, check out:

<https://github.com/freelensapp/freelens/releases>

## Homebrew

Run from the command line:

```sh
brew install freelens@nightly
```

You might need to uninstall `freelens` package first:

```sh
brew uninstall freelens
```

## Snap

Run from the command line:

```sh
snap install freelens --classic --channel=edge
```

Or if the stable package was already installed:

```sh
snap refresh freelens --channel=edge
```

## APT repository

Run from the command line:

<!-- markdownlint-disable MD013 -->

```sh
sudo mkdir -p /etc/apt/keyrings
curl -L https://raw.githubusercontent.com/freelensapp/freelens-nightly-builds/refs/heads/main/apt/freelens-nightly-builds.asc | sudo tee /etc/apt/keyrings/freelens-nightly-builds.asc
curl -L https://raw.githubusercontent.com/freelensapp/freelens-nightly-builds/refs/heads/main/apt/freelens-nightly-builds.sources | sudo tee /etc/apt/sources.list.d/freelens-nightly-builds.sources
sudo tee /etc/apt/preferences.d/freelens-nightly-builds <<END
Package: freelens
Pin: release l=freelens-nightly-builds
Pin-Priority: 1000
END
sudo apt update
sudo apt install freelens
```

Remove APT repository configuration if you no longer want to use nightly builds:

```sh
sudo rm -f /etc/apt/preferences.d/freelens-nightly-builds
```

then:

```sh
sudo apt purge freelens
# or
sudo rm -f /etc/apt/keyrings/freelens-nightly-builds.asc
sudo rm -f /etc/apt/sources.list.d/freelens-nightly-builds.list
```
