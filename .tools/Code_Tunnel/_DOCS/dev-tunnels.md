# Dev Tunnels SDK (microsoft/dev-tunnels)

## Overview

Dev Tunnels allows developers to securely expose local web services to the Internet, control who has access, and easily debug web applications from anywhere.

## SDK Feature Matrix

| Feature | .NET | TypeScript | Java | Go | Rust |
|---------|------|------------|------|----|------|
| Management API | ✅ | ✅ | ✅ | ✅ | ✅ |
| Tunnel Client Connections | ✅ | ✅ | ✅ | ✅ | ✅ |
| Tunnel Host Connections | ✅ | ✅ | ❌ | ❌ | ✅ |
| Reconnection | ✅ | ✅ | ❌ | ❌ | ❌ |
| SSH-level Reconnection | ✅ | ✅ | ❌ | ❌ | ❌ |
| Automatic tunnel access token refresh | ✅ | ✅ | ❌ | ❌ | ❌ |
| SSH Keep-alive | ✅ | ✅ | ❌ | ❌ | ❌ |

✅ - Supported, 🚧 - In Progress, ❌ - Not Supported, 🗓️ - Planned

## Resources

### Documentation

- [Dev tunnels documentation](https://aka.ms/devtunnels/docs)
- [Announcing the public preview of the devtunnel CLI](http://aka.ms/devtunnels/blog/cli)
- [Dev tunnels in Visual Studio 2022](http://aka.ms/devtunnels/vs)
- [Where else is dev tunnels used?](https://learn.microsoft.com/en-us/azure/developer/dev-tunnels/faq#where-else-is-dev-tunnels-used)

### Videos

#### Official

- [Securely test and debug your web apps and webhooks with dev tunnels](https://www.youtube.com/watch?v=yBiOGgUFD68)
- [Advanced developer tips and tricks in Visual Studio](https://youtu.be/Czr2M9qcdW4?t=491)

#### Community-created

- [ASP.NET Community Standup - Dev tunnels in Visual Studio for ASP.NET Core projects](https://youtu.be/B9K9eseNcKE?t=185)
- [New Visual Studio Feature is a Game Changer for API Developers - Put localhost Online](https://www.youtube.com/watch?v=NPJhrftkqeg)
- [Connect Any Client, Anywhere to localhost with Visual Studio Dev Tunnels!](https://www.youtube.com/watch?v=azuC8SFHWp8)
- [Dev Tunnels Visual Studio in 10 Minutes or Less](https://www.youtube.com/watch?v=kdaHwOkQf7c)
- [Share Local Web Services Across the Internet with Dev Tunnels CLI](https://www.youtube.com/watch?v=doUDcQNoy38)
- [ChatGPT Plugin development with Visual Studio](https://www.youtube.com/watch?v=iB9oxyJZhSA)

## Feedback

Have a question or feedback? There are many ways to submit feedback:

- [Up-vote a feature or request a new one](https://github.com/microsoft/dev-tunnels/issues?q=is%3Aissue+is%3Aopen+label%3Afeature-request)
- Search [existing bugs](https://github.com/microsoft/dev-tunnels/issues?q=is%3Aissue+is%3Aopen+label%3Abug) or [file a new issue](https://github.com/microsoft/dev-tunnels/issues/new)

## Repository Structure

- `.github/`: GitHub workflows and configurations
- `.pipelines/`: CI/CD pipeline configurations
- `.vscode/`: VS Code workspace settings
- `cs/`: C# SDK implementation
- `go/tunnels/`: Go SDK implementation
- `java/`: Java SDK implementation
- `rs/`: Rust SDK implementation
- `ts/`: TypeScript SDK implementation
- `samples/ts/`: TypeScript sample code

## Languages

- C# (43.2%)
- TypeScript (25.4%)
- Java (10.9%)
- Go (10.6%)
- Rust (9.2%)
- JavaScript (0.7%)

## Contributing

This project welcomes contributions and suggestions. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution.

## Security

Microsoft takes the security of our software products and services seriously. If you believe you have found a security vulnerability, please report it as described in the SECURITY.md file.

## Statistics

- 416 stars
- 7 watching
- 33 forks
- Used by 405 repositories
