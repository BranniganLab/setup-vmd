# setup-vmd

A GitHub composite action for installing and caching **VMD** (Visual Molecular Dynamics) in GitHub Actions workflows.

This action downloads a VMD installer from a private repository using a GitHub App, restores a cached installation when available, installs VMD on a cache miss, and makes the `vmd` executable available on the system `PATH` for the remainder of the workflow.

## Features

- Restores a cached VMD installation when available.
- Downloads the installer only on a cache miss.
- Authenticates to a private installer repository using a GitHub App.
- Installs VMD into a user-space directory.
- Adds VMD to the workflow `PATH`.
- Verifies that the installation completed successfully.
- Automatically invalidates the cache when the installer changes.

## Requirements

This action requires a GitHub App with read access to the repository containing the VMD installer archive. The calling workflow must provide the GitHub App client ID and private key as GitHub Actions secrets.

| Secret | Description |
|--------|-------------|
| `GH_APP_ID` | GitHub App client ID used to authenticate to the installer repository. |
| `GH_APP_PRIVATE_KEY` | Private key corresponding to the GitHub App. |

The GitHub App must have permission to read the repository containing the VMD installer archive.

## Usage

```yaml
steps:
  - uses: actions/checkout@v6

  - name: Install system dependencies
    run: |
      sudo apt-get update
      sudo apt-get install -y \
        build-essential \
        libx11-dev \
        libglu1-mesa-dev \
        libxi-dev \
        libxext-dev \
        libxmu-dev \
        libjpeg-dev \
        libpng-dev \
        tcl-dev \
        tk-dev \
        python3-dev \
        flex \
        bison \
        git

  - name: Set up VMD
    uses: BranniganLab/setup-vmd@ae464c279176585de8a94bd7c7bd11eee4da5783
    with:
      github-app-id: ${{ secrets.GH_APP_ID }}
      github-app-private-key: ${{ secrets.GH_APP_PRIVATE_KEY }}

  - name: Run VMD
    run: |
      vmd -dispdev text -e script.tcl
```

After the action completes successfully, `vmd` is available on the `PATH` and can be used by all subsequent workflow steps.

## Inputs

| Name | Required | Description |
|------|:--------:|-------------|
| `github-app-id` | Yes | GitHub App client ID used to authenticate to the installer repository. |
| `github-app-private-key` | Yes | GitHub App private key. |
| `source-owner` | No | Owner of the repository containing the VMD installer. Default: `BranniganLab`. |
| `source-repository` | No | Repository containing the installer archive. Default: `vmd_for_testing`. |
| `tarball-name` | No | Name of the VMD installer archive. |
| `source-directory` | No | Name of the top-level directory created when extracting the installer archive. |

## Outputs

| Name | Description |
|------|-------------|
| `cache-hit` | `true` if the VMD installation was restored from the cache; otherwise `false`. |

## Cache Behavior

The cache key is derived from:

- the runner operating system,
- the runner architecture, and
- the Git blob SHA of the VMD installer archive.

Updating the installer archive automatically creates a new cache entry, so manual cache invalidation is normally unnecessary.

## Security

This action requires a GitHub App credential to access the private repository containing the VMD installer archive.

The GitHub App private key is used only during installation and is **not** written to the cache or included in the installed VMD files.

For reproducibility and supply-chain security, workflows should reference a reviewed, immutable commit SHA rather than a mutable branch or tag. At the time of writing, the recommended revision is:

```yaml
uses: BranniganLab/setup-vmd@ae464c279176585de8a94bd7c7bd11eee4da5783
```

When a new version of this action is released, update the workflow to reference the corresponding reviewed commit SHA.

## License

This repository contains automation for installing VMD.

VMD is developed by the Theoretical and Computational Biophysics Group at the University of Illinois Urbana-Champaign and is distributed under its own license terms. This action automates the installation of VMD but does not include or redistribute the VMD software itself.
