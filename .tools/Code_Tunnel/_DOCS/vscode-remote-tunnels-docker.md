# VS Code Remote Tunnels Docker (ilteoood/vscode-remote-tunnels)

## Overview

A Docker image for VS Code remote tunnels that can be easily deployed anywhere. This is a multi-arch image, updated automatically thanks to GitHub Actions.

Its purpose is to provide a Visual Studio Code Server instance accessible through vscode.dev.

## Features

- Multi-architecture support
- Automated builds via GitHub Actions
- Docker Compose support
- Easy deployment anywhere

## Configuration

The container is configurable using 1 environment variable:

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| MACHINE_NAME | No | vscode-remote | The name of the machine that will be used to access the tunnel. It must be less than 20 characters. |

Additionally, it is also possible to override the script at path `/usr/local/bin/init` to install additional software when the container boots.

## Execution

### Using Docker Compose

You can run this image using Docker Compose and the sample file provided.

### Using Docker Run

```bash
sudo docker run --name CONTAINER_NAME -e MACHINE_NAME=YOUR_MACHINE_NAME ilteoood/vscode-remote-tunnels
```

## Repository Structure

- `.github/`: GitHub Actions workflows
- `src/`: Source code
- `Dockerfile`: Docker image definition
- `docker-compose.yml`: Docker Compose configuration
- `README.md`: Documentation

## Languages

- Dockerfile (51.1%)
- Shell (48.9%)

## Statistics

- 16 stars
- 1 watching
- 6 forks

## Contributing

This project welcomes contributions. See the repository for contribution guidelines.

## License

MIT License

## Support

If you like this work, consider supporting the author:

- [Patreon](https://www.patreon.com/ilteoood)
- [Buy Me a Coffee](https://www.buymeacoffee.com/ilteoood)
