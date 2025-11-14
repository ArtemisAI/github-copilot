# VS Code Remote - Tunnels Marketplace Extension

## Overview

**Extension:** Remote - Tunnels (ms-vscode.remote-server)

**Publisher:** Microsoft

**Installs:** 7,310,962

**Rating:** 4.4 out of 5

The Remote - Tunnels extension lets you connect to a remote machine, like a desktop PC or virtual machine (VM), via a secure tunnel. You can then securely connect to that machine from anywhere, without the requirement of SSH.

## Getting Started

### Make a Tunnel Available

Begin by making your remote machine accessible through a tunnel:

#### Option 1: VS Code UI

Run VS Code on the remote machine and use the `Turn on Remote Tunnel Access` command.

#### Option 2: CLI

Run the `tunnel` command in the `code` CLI.

The code CLI is part of the regular Windows, Mac and Linux installs, but can also be downloaded separately from the VS Code download page.

### Connect to a Tunnel

To connect to a tunnel, have the `Remote Tunnels` extension installed and run the command: `Remote - Tunnels: Connect to Tunnel...`.

You can find the command by:

- Pressing `F1` to open the Command Palette
- Clicking on the remote indicator in the lower left corner

You'll be prompted to log into GitHub and will get a list of available tunnels to connect to.

After selecting a tunnel, the window will reload to connect to the remote machine. Note that the remote indicator in the lower left corner now shows the name of the tunnel.

You can also open a tunnel in the Remote Explorer.

**Documentation Links:**

- [Remote - Tunnels Documentation](https://code.visualstudio.com/docs/remote/tunnels)
- [VS Code Server Documentation](https://aka.ms/vscode-server-doc)

## Release Notes

This extension releases with VS Code. VS Code release notes include a summary of changes to all Remote Development extensions with a link to detailed release notes.

As with VS Code itself, the extensions update during a development iteration. You can use the pre-release version of this extension to regularly get the latest extension updates before the official extension release.

## Questions, Feedback, Contributing

Have a question or feedback?

- See the [documentation](https://aka.ms/vscode-remote) or the [troubleshooting guide](https://aka.ms/vscode-remote/troubleshooting)
- [Up-vote a feature or request a new one](https://aka.ms/vscode-remote/feature-requests), search [existing issues](https://aka.ms/vscode-remote/issues), or [report a problem](https://aka.ms/vscode-remote/issues/new)
- Contribute to [our documentation](https://github.com/Microsoft/vscode-docs)
- See our [CONTRIBUTING](https://aka.ms/vscode-remote/contributing) guide for details

## Telemetry

Visual Studio Code Remote - Tunnels and related extensions collect telemetry data to help us build a better experience working remotely from VS Code. We only collect data on which commands are executed. We do not collect any information about image names, paths, etc. The extension respects the `telemetry.enableTelemetry` setting which you can learn more about in the Visual Studio Code FAQ.

## License

By downloading and using the Visual Studio Remote - Tunnels extension and its related components, you agree to the product license terms and privacy statement.

## Installation

Launch VS Code Quick Open (`Ctrl+P`), paste the following command, and press enter:

```bash
ext install ms-vscode.remote-server
```
