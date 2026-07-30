# setup-vmd

A GitHub composite action for installing and caching **VMD** (Visual
Molecular Dynamics) in GitHub Actions workflows.

This action downloads a VMD installer from a private repository using a
GitHub App, restores a cached installation when available, installs VMD
on a cache miss, and makes the `vmd` executable available on the system
`PATH` for the remainder of the workflow.

## Features

-   Restores a cached VMD installation when available
-   Downloads the installer only on a cache miss
-   Authenticates using a GitHub App
-   Installs VMD into a user-space directory
-   Adds VMD to the workflow `PATH`
-   Verifies the installation
-   Uses deterministic cache keys so caches are automatically
    invalidated when the installer changes

------------------------------------------------------------------------

## Requirements

This action requires a GitHub App with read access to the repository containing
the VMD installer archive. The calling workflow must provide the GitHub App
client ID and private key as secrets.

  -----------------------------------------------------------------------
  Secret                        Description
  ----------------------------- -----------------------------------------
  `GH_APP_ID`                   GitHub App client ID used to access the
                                private installer repository.

  `GH_APP_PRIVATE_KEY`          Private key corresponding to the GitHub
                                App.
  -----------------------------------------------------------------------

The GitHub App must have permission to read the repository containing
the VMD installer archive.

------------------------------------------------------------------------

## Usage

``` yaml
steps:
  - uses: actions/checkout@v5

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

After the action completes successfully, `vmd` is available to all
subsequent workflow steps.

## Inputs

  -----------------------------------------------------------------------------------
  Name                               Required         Description
  -------------------------- ------------------------ -------------------------------
  `github-app-id`                      Yes            GitHub App client ID used to
                                                      authenticate to the installer
                                                      repository.

  `github-app-private-key`             Yes            GitHub App private key.

  `source-owner`                        No            Owner of the repository
                                                      containing the VMD installer.
                                                      Default: `BranniganLab`.

  `source-repository`                   No            Repository containing the
                                                      installer archive. Default:
                                                      `vmd_for_testing`.

  `tarball-name`                        No            Name of the VMD installer
                                                      archive.

  `source-directory`                    No            Top-level directory created
                                                      when extracting the installer
                                                      archive.
  -----------------------------------------------------------------------------------

## Outputs

  -----------------------------------------------------------------------
  Name                   Description
  ---------------------- ------------------------------------------------
  `cache-hit`            `true` if the cached installation was restored;
                         otherwise `false`.

  -----------------------------------------------------------------------

## Cache Behavior

The cache key is derived from the operating system, runner architecture,
and the Git blob SHA of the installer archive. Updating the installer
automatically creates a new cache entry.

## Installation Location

VMD is installed under `.cache/vmd-install/`. The action automatically
adds `.cache/vmd-install/bin` to the workflow PATH.

## Security

This action requires a GitHub App credential to access the private
repository containing the VMD installer.

The private key is used only during installation and is never written to
the cache or installed files.

For reproducibility and supply-chain security, workflows should
reference a reviewed immutable commit SHA:

``` yaml
uses: BranniganLab/setup-vmd@ae464c279176585de8a94bd7c7bd11eee4da5783
```

Update the referenced SHA only after reviewing and testing a newer
revision of the action.

## Versioning

This repository follows Semantic Versioning. Releases receive semantic
version numbers, but consuming workflows should reference the reviewed
commit SHA associated with a release rather than a mutable tag.

## License

This repository contains automation for installing VMD.

VMD is developed by the Theoretical and Computational Biophysics Group
at the University of Illinois Urbana-Champaign and is distributed under
its own license terms. This action automates installation but does not
include or redistribute the VMD software itself.
