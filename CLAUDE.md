# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bedrockifier is a Swift-based multi-purpose tool for creating backups of Minecraft Java and Bedrock server worlds. It provides:
- A command-line tool (`bedrockifier-tool`) for manual backup operations (pack/unpack/trim)
- A backup service daemon (`bedrockifierd`) that runs as a containerized service
- Support for both Bedrock (.mcworld format) and Java (.zip format) backups
- Integration with Docker containers (particularly itzg's Minecraft containers)

## Build and Development Commands

### Building the Project
```bash
swift build
```

### Running Tests
```bash
swift test
```

### Building Docker Container
```bash
./build-docker.sh [tag] [commit] [push]
# Example: ./build-docker.sh dev main nopush
```

### Running Tools
```bash
# Run the CLI tool
swift run bedrockifier-tool --help

# Available commands:
swift run bedrockifier-tool pack <world-path> <output-path>
swift run bedrockifier-tool unpack <backup-path> <output-path>  
swift run bedrockifier-tool trim <backup-folder> <max-count>

# Run the service daemon
swift run bedrockifierd --help
```

## Architecture

### Core Components

**Main Executables:**
- `Sources/Tool/` - CLI tool for manual backup operations
- `Sources/Service/` - Daemon service for automated backups

**Core Library (`Sources/Bedrockifier/`):**
- `Client/` - SSH client implementation for connecting to containers
- `Model/` - Data models (BackupConfig, ContainerConfig, World, etc.)
- `Foundation/` - Logging, platform utilities, service timers
- `Extensions/` - Swift extensions for Date, String, etc.
- `Utilities/` - General utility classes

**Service Architecture:**
- `BackupActor` - Main actor handling backup operations
- `BackupService` - HTTP service with health endpoints and backup triggers
- `ContainerConnection` - Manages connections to Docker containers via SSH/RCON
- `ServiceTimer` - Handles scheduled backup intervals

### Key Dependencies
- **Hummingbird** - HTTP server framework for the service daemon
- **swift-nio-ssh** - SSH client for container communication
- **swift-argument-parser** - CLI argument parsing
- **Yams** - YAML configuration parsing
- **ZIPFoundation** - Archive creation/extraction
- **PTYKit** - Terminal/PTY handling

### Configuration
The service reads configuration from YAML files (typically `config.yml`) containing:
- Container definitions (SSH/RCON connection details)
- Backup schedules (interval/daily/event-based)
- World paths and backup destinations
- Logging levels and service settings

### Container Integration
- Connects to Minecraft containers via SSH or RCON
- Monitors container logs for player login/logout events
- Executes backup commands within containers
- Supports both Bedrock and Java server types

### HTTP Service Endpoints
- `/health`, `/live` - Health check endpoints
- `/status` - Service status and last backup info
- `/start-backup` - Trigger manual backup (token-protected)

The service runs on port 8080 and generates authentication tokens for secured endpoints.