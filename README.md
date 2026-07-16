# unirtos-network-demos

[中文](README.zh.md) | English

This repository is recommended to be used through the unirtos-cli demo workflow to ensure consistency in project creation, environment setup, and build procedures.

## Feature Description

This demo is used to demonstrate the basic workflow of UniRTOS network capabilities and is suitable as a starting point for network-based applications.

- Demonstrates the initialization of network-related components and the basic invocation flow
- Provides a minimal runnable example for network feature verification
- Makes it easy to extend protocol, connection, and data send/receive logic on this basis

## Quick Start

### 1. Install the UniRTOS Toolchain

- [Development Preparation](https://www.quectel.com.cn/unirtos/docs?docs_page=快速上手/开发准备/开发准备.html)
- [Install the Cross-Compilation Toolchain](https://www.quectel.com.cn/unirtos/docs?docs_page=快速上手/环境搭建/环境搭建.html)
- [Install Python3](https://www.python.org/downloads/)
- [Install git](https://git-scm.com)
- Install `unirtos-cli`: `pip install unirtos-cli`

After the above tools are installed, confirm that the following commands are available:

```bash
python --version # Python3
git --version
unirtos --version # version 1.0.5 or later
unirtos-cli version # version 1.0.11 or later
```

### 2. Fetch the Demo Using unirtos-cli

First, check the available demos and versions:

```bash
unirtos-cli ls-demos
```

Create this demo project:

```bash
unirtos-cli new -r unirtos-network-demos
```

To specify a version:

```bash
unirtos-cli new -r unirtos-network-demos -v 1.0.0
```

### 3. Enter the Project and Build

```bash
cd unirtos-network-demos-1.0.0
unirtos-cli env-setup
unirtos-cli build
```

## Common Commands

```bash
# Open the SDK menu configuration
unirtos-cli menuconfig

# Clean build artifacts
unirtos-cli clean
```

## Technical Community

Technical Community: https://forumschinese.quectel.com/c/66-category/66

## Contribution Guide

Contributions are welcome. It is recommended to submit changes in the following way:
- Run a basic verification before submission: env-setup, build, clean.
- Use clear commit messages describing the purpose of the change, scope of impact, and verification results.
- When adding new features or changing behavior, update the README and related documentation accordingly.
- Submit bug fixes and feature improvements through Issues or Pull Requests.
