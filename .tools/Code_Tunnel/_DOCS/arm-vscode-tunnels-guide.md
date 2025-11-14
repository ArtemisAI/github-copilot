# ARM Learn: VS Code Tunnels Installation Guide

## About this Install Guide

**Reading time:** 10 min

**Last updated:** 30 Apr 2025

**Author:** Jason Andrews, Arm

**Official docs:** [View](https://code.visualstudio.com/docs/remote/vscode-server)

This guide is intended to get you up and running with this tool quickly with the most common settings. For a thorough review of all options, refer to the official documentation.

## Overview

The Arm architecture is well supported by the popular Visual Studio Code development tool. The Download page has binaries for all popular Arm architectures and operating systems. Desktop installation is straightforward and there are plenty of tutorials.

VS Code Remote Tunnels is a useful feature for non-desktop Arm development.

## Tunnel Use Cases

- Remote Linux servers, including Arm cloud instances, with no Linux desktop installed
- Developer virtual machines such as Multipass
- Arm single board computers running Linux

To create a tunnel a command-line instance of VS Code is run on the remote machine (server) which registers a tunnel. A browser or a VS Code desktop instance (client) is used to connect to the tunnel and access the remote machine. The tunnel removes the need for open ports or SSH port forwarding. VS Code tunnels also work well for connecting to Arm boards on your local network when you are away because no changes to the network are needed.

**Note:** Creating VS Code tunnels requires a GitHub account.

## Prerequisites

Follow the instructions below to install either the CLI version or the desktop version of VS Code on an Arm Linux machine. This is the remote machine you want to use for development.

The use cases for VS Code tunnels assume SSH or command line access to the remote Linux machine is available, but no Linux desktop or browser access is required. This means VS Code must be installed from the command line.

Confirm you are using an Arm machine by running:

```bash
uname -m
```

Depending on the version of the Arm architecture, the results are shown below. Both Armv8-A and Armv7-A are supported.

For the Armv8-A architecture the output should be:

```bash
aarch64
```

For the Armv7-A architecture the output should be:

```bash
armv7l
```

## Which Version of VS Code Should I Download?

Download either the CLI version or the desktop version of VS Code. The desktop version is the best choice if the machine has a Linux desktop and you want to use both a remote tunnel and do desktop development. The CLI version is best for tunneling only.

### CLI Version for Tunneling

Download a command line (CLI) release of VS Code from the Downloads page.

#### Armv8-A Download

```bash
wget -O vscode.tgz 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-arm64'
```

#### Armv7-A Download

```bash
wget -O vscode.tgz 'https://code.visualstudio.com/sha/download?build=stable&os=cli-linux-armhf'
```

### Desktop Version for Tunneling and Desktop Development

VS Code desktop also works for the remote machine.

Download the Debian package for VS Code desktop using the command line. This is for Ubuntu or Debian Linux machines. An RPM version is also available for Red Hat and Fedora.

```bash
wget -O vscode.deb 'https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-arm64'
```

**Warning:** The `snap` install is for a non-Arm architecture and does not work.

## Installation

### CLI Installation

For CLI, extract the file. The result is a single executable named `code`. It is placed in the current directory.

```bash
tar xvf vscode.tgz
```

### Desktop Installation

For the desktop, install the Debian package.

```bash
sudo apt install ./vscode.deb
```

## Starting a VS Code Tunnel

On the remote machine start a VS Code tunnel. The `./` is used to indicate the CLI version which was extracted in the current directory.

```bash
./code tunnel --name my-tunnel-1 --accept-server-license-terms
```

For the desktop, ensure the `code` executable is on `PATH`. The `./` should be omitted:

```bash
code tunnel --name my-tunnel-1 --accept-server-license-terms
```

If `--name` is not used, an arbitrary name will be assigned to the tunnel.

The VS Code tunnel server will print a URL to authenticate and link it to your GitHub account.

```bash
* Visual Studio Code Server
*
* By using the software, you agree to
* the Visual Studio Code Server License Terms (https://aka.ms/vscode-server-license) and
* the Microsoft Privacy Statement (https://privacy.microsoft.com/en-US/privacystatement).
*
To grant access to the server, please log into https://github.com/login/device and use code XXXX-YYYY
```

Visit [device link](https://github.com/login/device) and then enter the code shown.

After the code is entered, the remote device is connected to your GitHub account.

## Connecting to the Remote Machine

The remote machine is now ready to accept client connections. These can be from a browser or VS Code desktop.

### Browser Connection

The quickest way to connect to the remote machine is using a browser. Even a tablet or phone can be used.

Connect using a browser by visiting the link which is printed by the server. The link will contain the tunnel name specified with `--name` or a created name if no name was provided.

In the example above, the tunnel link is:

```bash
https://vscode.dev/tunnel/my-tunnel-1
```

If you do not remember or cannot see the tunnel name, open:

```bash
https://vscode.dev
```

When VS Code opens, click on the lower left corner and select `Connect to Tunnel`.

A list of tunnels attached to your account will be opened. If you are not signed in to GitHub it will require authentication before showing the list of tunnels. The tunnel list also shows if the tunnel is online and ready for a connection.

Select the tunnel to open and VS Code will connect to the remote machine and you are ready to use VS Code on the remote machine.

### VS Code Desktop Connection

Connecting to a tunnel from VS Code Desktop requires the [Remote - Tunnels Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.remote-server).

Search for `remote tunnels` in the Extensions Marketplace and install the extension.

Just as with the browser connection, click on the lower left corner and select `Connect to Tunnel`.

A list of tunnels attached to your account will be opened. If you are not signed in to GitHub it will require authentication before showing the list of tunnels. The tunnel list also shows if the tunnel is online and ready for a connection.

Select the tunnel to open and VS Code will connect to the remote machine and you are ready to use VS Code on the remote machine.

## Closing a Tunnel Connection

On both browser and desktop, click the lower left (which now shows the name of the connected tunnel) and select `Close Remote Connection` to disconnect from the tunnel server.

## Key Takeaways

VS Code tunnels and your GitHub account make it easy to connect to Arm machines running Linux for terminal access or to do development on a remote machine with VS Code. This is done without any open ports or SSH port forwarding.
