# gprbuild-pkg

## Overview

While `gprbuild` is available in the FreeBSD package repositories, it is currently dependent on `gnat12`. This creates compatibility issues and inconvenience for users who wish to utilize the latest GNAT 14 toolchain.

**gprbuild-pkg** provides a script to generate a `gprbuild` package that is compatible with `gcc14-ada`.

## Prerequisites

To use this package, you need `gcc14-ada`. You can build the `gcc14-ada` package using the following repository:

* [https://github.com/hodong-kim/gcc14-ada](https://github.com/hodong-kim/gcc14-ada)

## Build and Installation

Follow these steps to build and install the package.

### 1. Install Build Dependencies

Ensure that Ruby, Rake, and GNU Make are installed on your system.

```sh
sudo pkg install ruby rubygem-rake gmake

```

### 2. Build the Package

Run the `rake` command to start the build process. This script will automatically download the source code, bootstrap `gprbuild`, and create a FreeBSD package file.

> **Note:** The script requires `sudo` privileges to create necessary symbolic links for the compiler during the build process.

```sh
rake
```

### 3. Install the Generated Package

Once the build is complete, a `.pkg` file (e.g., `gprbuild-gcc14-25.0.0.pkg`) will be generated in the current directory. Install it using the `pkg` command.

```sh
sudo pkg add ./gprbuild-gcc14-25.0.0.pkg
```

### 4. Cleaning Up (Optional)

To remove temporary build files and directories, use the clean task:

```sh
rake clean
```