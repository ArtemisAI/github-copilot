# VS Code Tunnels with Microsoft Account (Stack Overflow)

## Question Summary

**Title:** How do I use VSCode Tunnels using only a Microsoft account instead of a GitHub account?

**Asked:** November 30, 2023

**Tags:** visual-studio-code, vscode-remote, chromebook

## Problem Description

User wants to use VS Code tunnels on a school Chromebook where GitHub is blocked. They want to connect to their home PC using Microsoft account authentication instead of GitHub, since vscode.dev offers both Microsoft and GitHub login options.

The issue is that the PC only sets up remote tunnels with GitHub authentication.

## Solution 1: CLI Commands (mrBen)

### Using CLI with Microsoft Provider

On your home PC, you can start a tunnel with the `code` CLI using these commands:

```bash
code tunnel user login --provider microsoft
code tunnel
```

- First command logs you in with a Microsoft account
- Second command starts the tunnel
- If you want to log with GitHub, you don't need the first command

### Alternative: VS Code UI

An easier way is to use the VS Code UI. The user has the option to sign in with Microsoft, but as understood, the whole feature is still experimental and Microsoft might only be accessible with an insider build.

## Solution 2: Code Tunnel CLI Installation (Pablo Oliveira)

The user can try the Code Tunnel CLI installation from the official documentation. They had a similar problem when trying to connect to the Alpine Linux environment and discovered that it was a problem with the C execution and compilation libs. But when running the "Code Tunnel" CLI, the problem was resolved.

**Reference:** [VS Code Tunnels Documentation](https://code.visualstudio.com/docs/remote/tunnels)

## Context

This question addresses a common issue where educational institutions block GitHub but allow Microsoft accounts. VS Code tunnels are particularly useful for Chromebooks and other restricted environments where full development setup isn't possible.

## Related Questions

- How to make explorer in remote SSH extension in Visual Studio Code (VSCode) do show symlinks?
- How to use visual studio code with github and 2FA
- Use Settings Sync or Syncing vscode extensions with remote SSH environment
- How to fix VS Code error with Remote-SSH: "the terminal process failed to launch: A native exception occurred during launch (forkpty(3) failed)."
- Github Codespaces rejecting pushes to Origin from Collaborators
