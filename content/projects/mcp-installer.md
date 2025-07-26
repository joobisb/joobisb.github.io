---
title: "mcp-installer: one-click MCP server installation"
date: 2025-07-26
---

*Repo*: [joobisb/mcp-installer](https://github.com/joobisb/mcp-installer)  
*Website*: [mcp-installer.com](https://mcp-installer.com)  
*NPM*: [@mcp-installer/cli](https://www.npmjs.com/package/@mcp-installer/cli)

A command-line tool that simplifies the installation and management of MCP (Model Context Protocol) servers across different AI clients like Claude Code, Claude Desktop, Cursor, Gemini CLI and VS Code.

## Key Features

- **One-click installation** of MCP servers across multiple AI clients
- **Cross-platform support** for Claude Code, Cursor, Gemini, VS Code and Claude Desktop
- **Backup & restore** functionality for configuration safety
- **System diagnostics** to troubleshoot installation issues
- **Registry management** with automatic updates from remote sources
- **Dry-run mode** to preview changes before execution

## Tech Stack

- **Runtime**: Node.js
- **Language**: TypeScript
- **Package Manager**: pnpm with workspace configuration
installation and a website

## Quick Start

```
# Install globally
npm install -g @mcp-installer/cli

# Install a server to all detected clients
mcp-installer install playwright

# Install to specific clients
mcp-installer install filesystem --clients=cursor,gemini
```

This project emerged from the need to simplify MCP server adoption across the growing ecosystem of AI-powered development tools.