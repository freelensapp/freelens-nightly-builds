# freelens-nightly-builds

<!-- markdownlint-disable MD013 -->

[![Home](https://img.shields.io/badge/%F0%9F%8F%A0-freelens.app-02a7a0)](https://freelens.app)
[![release](https://img.shields.io/github/v/release/freelensapp/freelens-nightly-builds?display_name=tag&sort=semver)](https://github.com/freelensapp/freelens-nightly-builds/releases)
[![Release nightly](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/release-nightly.yaml/badge.svg)](https://github.com/freelensapp/freelens-nightly-builds/actions/workflows/release-nightly.yaml)

<!-- markdownlint-enable MD013 -->

This is a project with nightly builds.

You can find the packages built from the main branch here:

<https://github.com/freelensapp/freelens-nightly-builds/releases>

## APT repository

Run from the command line:

<!-- markdownlint-disable MD013 -->

```sh
sudo mkdir -p /etc/apt/keyrings
curl -L https://raw.githubusercontent.com/freelensapp/freelens-nightly-builds/refs/heads/main/apt/freelens-nightly-builds.asc | sudo tee /etc/apt/keyrings/freelens-nightly-builds.asc
curl -L https://raw.githubusercontent.com/freelensapp/freelens-nightly-builds/refs/heads/main/apt/freelens-nightly-builds.sources | sudo tee /etc/apt/sources.list.d/freelens-nightly-builds.sources
sudo apt update
sudo apt install freelens
```

Remove them if you no longer want to use the APT repository for nightly builds:

```sh
sudo apt purge freelens
# or
sudo rm -f /etc/apt/keyrings/freelens-nightly-builds.asc
sudo rm -f /etc/apt/sources.list.d/freelens-nightly-builds.list
```
