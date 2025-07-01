# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Himmelblau is a Rust-based authentication and device management suite that enables Linux systems to authenticate against Microsoft Azure Entra ID via PAM and NSS modules. It also provides Intune device enrollment and policy enforcement capabilities.

## Development Commands

### Prerequisites
**Install Rust** (if not already installed):
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```

**Install system dependencies** (Ubuntu/Debian):
```bash
sudo apt-get install make gcc libpam0g-dev libudev-dev libssl-dev pkg-config libtss2-dev libcap-dev libtalloc-dev libtevent-dev libldb-dev libdhash-dev libkrb5-dev libpcre2-dev libclang-18-dev autoconf gettext libsqlite3-dev build-essential libdbus-1-dev libunistring-dev
```

### Building
```bash
# Build release version (primary build command)
make

# Build debug version  
cargo build

# Clean build artifacts
make clean
cargo clean
```

### Testing
```bash
# Run container-based integration tests
make test

# Build test environment only
make build-tests

# Interactive test container for debugging
make exec
```

### Linting and Code Quality
```bash
# Run Clippy linting with project-specific configuration
cargo clippy

# Format code
cargo fmt

# Check formatting without modifying files
cargo fmt --check
```

## Architecture Overview

### Core Components

**Authentication Stack:**
- `himmelblaud` - Main authentication daemon that interfaces with Azure Entra ID
- `himmelblaud_tasks` - Background tasks daemon for async operations
- `pam_himmelblau.so` - PAM module for Linux authentication integration  
- `libnss_himmelblau.so` - NSS module for user/group name resolution
- `aad-tool` - Command-line administration tool

**Supporting Services:**
- `broker/` - Authentication broker service handling OAuth flows
- `broker-client/` - Client library for broker communication
- `sso/` - Single Sign-On browser integration for desktop environments
- `qr-greeter/` - QR code authentication for desktop login

### Key Libraries and Modules

**Core Authentication (`src/common/`):**
- Central shared library (`himmelblau_unix_common`) containing core authentication logic
- Handles Azure Entra ID protocol implementation and token management

**Cryptography (`src/crypto/`):**
- Certificate handling and cryptographic operations
- TPM integration for hardware-backed security

**Policy Management (`src/policies/`):**
- Intune MDM policy parsing and enforcement
- Device compliance evaluation

**Identity Mapping (`src/idmap/`):**
- UID/GID mapping between Azure Entra ID and Linux
- Contains both Rust and C code for system integration

**Protocol Definitions (`src/proto/`):**
- Azure Entra ID and internal protocol structures
- Shared data types across components

### Build System Architecture

This is a **Cargo workspace** with 20 member crates. Key workspace dependencies include:
- `libhimmelblau` - Core authentication library with broker, TPM, and MFA features
- `tokio` - Async runtime for networking operations
- Kanidm libraries - Identity management components (crypto, protocols, users)
- OpenSSL/TLS libraries for secure communications

### Installation and Platform Support

The project uses platform-specific installation via Makefile:
- `make install` - Auto-detects platform and installs appropriate binaries
- Supports openSUSE/SUSE, Ubuntu/Debian, Fedora/RHEL distributions
- SystemD service integration for daemon management
- NSS and PAM system integration

### Container-Based Testing

Testing uses containerized environments to ensure platform compatibility:
- Tests run in isolated Podman/Docker containers
- Python-based NSS and PAM integration tests (`nsstest.py`, `pamtest.py`)
- Container builds for multiple Linux distributions

### Configuration

**Main Configuration:**
- `/etc/himmelblau/himmelblau.conf` - Primary daemon configuration
- Requires `domains` and `pam_allow_groups` settings for authentication
- SystemD service files in `platform/` directories

**Development Configuration:**
- `clippy.toml` - Rust linting configuration with relaxed complexity thresholds
- Minimum Rust version: 1.70
- Uses Rust 2021 edition

### Package Distribution

Multi-platform packaging system:
- DEB packages for Ubuntu/Debian (`make deb`)
- RPM packages for RHEL/Fedora/SUSE (`make rpm`) 
- Nix packages and flakes for NixOS (`make nix`)
- Container-based builds ensure reproducibility across distributions