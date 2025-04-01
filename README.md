# freelens-nightly-builds

<!-- markdownlint-disable MD013 -->

[![Home](https://img.shields.io/badge/%F0%9F%8F%A0-freelens.app-02a7a0)](https://freelens.app)
[![release](https://img.shields.io/github/v/release/freelensapp/freelens-nightly-builds?display_name=tag&sort=semver)](https://github.com/freelensapp/freelens-nightly-builds/releases/latest)
[![Release nightly](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/release-nightly.yaml/badge.svg)](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/release-nightly.yaml)

<!-- markdownlint-enable MD013 -->

This is a project with nightly builds for the
[Freelens](https://freelens.app) application.

You can find the packages built from the main branch here:

<https://github.com/freelensapp/freelens-nightly-builds/releases>

For official stable releases, check out:

<https://github.com/freelensapp/freelens/releases>

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
