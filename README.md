# freelens-nightly-builds

<!-- markdownlint-disable MD013 -->

[![Home](https://img.shields.io/badge/%F0%9F%8F%A0-freelens.app-02a7a0)](https://freelens.app)
[![GitHub](https://img.shields.io/github/v/release/freelensapp/freelens-nightly-builds?display_name=tag&sort=semver)](https://github.com/freelensapp/freelens-nightly-builds)
[![Release nightly](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/release-nightly.yaml/badge.svg)](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/release-nightly.yaml)

<!-- markdownlint-enable MD013 -->

This is a project with nightly builds.

You can find the packages built from the main branch here:

<https://github.com/freelensapp/freelens-nightly-builds/releases>

## APT repository

Debian 12 or Ubuntu 24.04 or newer:

```sh
curl -L https://github.com/freelensapp/freelens-nightly-builds/raw/refs/heads/main/apt/freelens-nightly-builds.sources | sudo tee /etc/apt/sources.list.d/freelens-nightly-builds.sources
```

Older:

```sh
mkdir -p /etc/apt/keyrings
curl -L https://github.com/freelensapp/freelens-nightly-builds/raw/refs/heads/main/apt/freelens.asc | sudo tee /etc/apt/keyrings/freelens.asc
curl -L https://github.com/freelensapp/freelens-nightly-builds/raw/refs/heads/main/apt/freelens-nightly-builds.list | sudo tee /etc/apt/sources.list.d/freelens-nightly-builds.list
```

Then:

```sh
sudo apt update
sudo apt install freelens
```
