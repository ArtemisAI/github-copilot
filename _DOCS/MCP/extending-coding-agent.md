# Extending GitHub Copilot coding agent with the Model Context Protocol (MCP)

## Prerequisite

Before setting up an MCP server for Copilot coding agent, read [Model Context Protocol (MCP) and GitHub Copilot coding agent](overview.md) to make sure you understand the concepts around MCP servers and Copilot coding agent.

## Introduction

As a repository administrator, you can configure MCP servers for use within your repository. You do this using a JSON-formatted configuration that specifies the details of the MCP servers you want to use. You enter the JSON configuration directly into the settings for the repository on GitHub.com.

Organization and enterprise administrators can also configure MCP servers as part of custom agents using the YAML frontmatter. For more information, see [Custom agents configuration](https://docs.github.com/en/enterprise-cloud@latest/copilot/reference/custom-agents-configuration#mcp-server-configuration-details).

**Warning:** Once you've configured an MCP server, Copilot will be able to use the tools provided by the server autonomously, and will not ask for your approval before using them.

**Note:** Copilot coding agent only supports tools provided by MCP servers. It does not support resources or prompts.

**Note:** Copilot coding agent does not currently support remote MCP servers that leverage OAuth for authentication and authorization.

## Adding an MCP configuration to your repository

Repository administrators can configure MCP servers by following these steps:

1. On GitHub, navigate to the main page of the repository.
2. Under your repository name, click Settings. If you cannot see the "Settings" tab, select the dropdown menu, then click Settings.
3. In the "Code & automation" section of the sidebar, click Copilot then Coding agent.
4. Add your configuration in the MCP configuration section.
5. Click Save.

Your configuration will be validated to ensure proper syntax.

If your MCP server requires a key or secret, add a secret to your Copilot environment. Only secrets with names prefixed with `COPILOT_MCP_` will be available to your MCP configuration. See Setting up a Copilot environment for Copilot coding agent.

## Writing a JSON configuration for MCP servers

You configure MCP servers using a special JSON format. The JSON must contain an `mcpServers` object, where the key is the name of the MCP server (for example, `sentry`), and the value is an object with the configuration for that MCP server.

```json
{
  "mcpServers": {
    "MCP SERVER 1": {
      "command": "VALUE",
      "args": [ VALUES ],
      ...
    },
    "MCP SERVER 2": {
      "command": "VALUE",
      "args": [ VALUES ],
      ...
    },
    ...
  }
}
```

The configuration object can contain the following keys:

### Required keys for local and remote MCP servers

- `tools` (`string[]`): The tools from the MCP server to enable. You may be able to find a list of tools in the server's documentation, or in its code. We strongly recommend that you allowlist specific read-only tools, since the agent will be able to use these tools autonomously and will not ask you for approval first. You can also enable all tools by including `*` in the array.
- `type` (`string`): Copilot coding agent accepts `"local"`, `"http"`, or `"sse"`.

### Local MCP specific keys

- `command` (`string`): Required. The command to run to start the MCP server.
- `args` (`string[]`): Required. The arguments to pass to the `command`.
- `env` (`object`): Optional. The environment variables to pass to the server. This object should map the name of the environment variable that should be exposed to your MCP server to either of the following:
  - The name of a GitHub Actions secret you have configured, beginning with `COPILOT_MCP_`.
  - A string value.

### Remote MCP specific keys

- `url` (`string`): Required. The MCP server's URL.
- `headers` (`object`): Optional. The headers to attach to requests to the server. This object should map the name of header keys to either of the following:
  - The name of a GitHub Actions secret you have configured, beginning with `COPILOT_MCP_` preceded by a `$`
  - A string value

## Example configurations

### Example: Sentry

The [Sentry MCP server](https://github.com/getsentry/sentry-mcp) gives Copilot authenticated access to exceptions recorded in [Sentry](https://sentry.io/).

```json
{
  "mcpServers": {
    "sentry": {
      "type": "local",
      "command": "npx",
      "args": ["@sentry/mcp-server@latest", "--host=$SENTRY_HOST"],
      "tools": ["get_issue_details", "get_issue_summary"],
      "env": {
        "SENTRY_HOST": "https://contoso.sentry.io",
        "SENTRY_ACCESS_TOKEN": "COPILOT_MCP_SENTRY_ACCESS_TOKEN"
      }
    }
  }
}
```

### Example: Notion

The [Notion MCP server](https://github.com/makenotion/notion-mcp-server) gives Copilot authenticated access to notes and other content from [Notion](https://notion.so/).

```json
{
  "mcpServers": {
    "notionApi": {
      "type": "local",
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-e",
        "OPENAPI_MCP_HEADERS={\"Authorization\": \"Bearer $NOTION_API_KEY\", \"Notion-Version\": \"2022-06-28\"}",
        "mcp/notion"
      ],
      "env": {
        "NOTION_API_KEY": "COPILOT_MCP_NOTION_API_KEY"
      },
      "tools": ["*"]
    }
  }
}
```

### Example: Azure

The [Azure MCP Server](https://github.com/Azure/azure-mcp) allows Copilot to understand your Azure-specific files and Azure resources within your subscription when making code changes.

To automatically configure your repository with a `copilot-setup-steps.yml` file to authenticate with Azure, plus secrets for authentication, clone the repository locally then run the [Azure Developer CLI](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/?ref_product=copilot&ref_type=engagement&ref_style=button)'s `azd coding-agent config` command in the root of the repository.

Once you've run the command and merged the created pull request, you can add the MCP configuration to your repository.

```json
{
  "mcpServers": {
    "Azure": {
      "type": "local",
      "command": "npx",
      "args": [
        "-y",
        "@azure/mcp@latest",
        "server",
        "start"
      ],
      "tools": ["*"]
    }
  }
}
```

### Example: Cloudflare

The [Cloudflare MCP server](https://github.com/cloudflare/mcp-server-cloudflare) creates connections between your Cloudflare services, including processing documentation and data analysis.

```json
{
  "mcpServers": {
    "cloudflare": {
      "type": "sse",
      "url": "https://docs.mcp.cloudflare.com/sse",
      "tools": ["*"]
    }
  }
}
```

### Example: Azure DevOps

The [Azure DevOps MCP server](https://github.com/microsoft/azure-devops-mcp) creates a seamless connection between Copilot and your Azure DevOps services, including work items, pipelines or documentation.

To use the Azure DevOps MCP server with Copilot coding agent, you must update the repository's copilot-setup-steps.yml file to include an Azure login workflow step.

1. Configure OIDC in a Microsoft Entra application, trusting GitHub. See [Use the Azure Login action with OpenID Connect](https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure-openid-connect).
2. Setup access to Azure DevOps organization and projects for the application identity. See [Add organization users and manage access](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/add-organization-users).
3. Add a `.github/workflows/copilot-setup-steps.yml` Actions workflow file in your repository if you do not already have one.
4. Add an Azure login step to the `copilot-setup-steps` workflow job.

```yaml
on:
  workflow_dispatch:
permissions:
  id-token: write
  contents: read
jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    environment: copilot
    steps:
      - name: Azure login
        uses: azure/login@a457da9ea143d694b1b9c7c869ebb04ebe844ef5
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          allow-no-subscriptions: true
```

This configuration ensures the `azure/login` action is executed when Copilot coding agent runs.

1. In your repository's Copilot environment, add secrets for your `AZURE_CLIENT_ID` and `AZURE_TENANT_ID`.
2. Configure the Azure DevOps MCP server by adding an `ado` object to your MCP configuration with defined tools you want Copilot coding agent to use.

```json
{
  "mcpServers": {
    "ado": {
      "type": "local",
      "command": "npx",
      "args": ["-y", "@azure-devops/mcp", "<your-azure-devops-organization>", "-a", "azcli"],
      "tools": ["wit_get_work_item", "wit_get_work_items_batch_by_ids", ...]
    }
  }
}
```

## Reusing your MCP configuration from Visual Studio Code

If you have already configured MCP servers in VS Code, you can leverage a similar configuration for Copilot coding agent.

Depending on how VS Code is configured, you may be able to find your MCP settings in your repository's `.vscode/mcp.json` file, or in your machine's private `settings.json` file.

To adapt the configuration for Copilot coding agent, you will need to:

1. Add a `tools` key for each MCP server, specifying which tools will be available to Copilot.
2. If you've configured `inputs`, switch to using `env` directly.
3. If you've configured an `envFile`, switch to using `env` directly.
4. Update any references to `inputs` in your `args` configuration to refer to environment variables from `env` instead.

For more information on MCP in VS Code, see the [VS Code docs](https://code.visualstudio.com/docs/copilot/chat/mcp-servers).

## Setting up a Copilot environment for Copilot coding agent

Some MCP servers will require keys or secrets. To leverage those servers in Copilot coding agent, you can add secrets to an environment for Copilot. This ensures the secrets are properly recognized and passed to the applicable MCP server that you have configured.

You must be a repository administrator to configure a Copilot environment for your repository.

1. On GitHub, navigate to the main page of the repository.
2. Under your repository name, click Settings.
3. In the left sidebar, click Environments.
4. Click New environment.
5. Call the new environment `copilot` and click Configure environment.
6. Under "Environment secrets", click Add environment secret.
7. Give the secret a name beginning `COPILOT_MCP_`, add the secret value, then click Add secret.

## Validating your MCP configuration

Once you've set up your MCP configuration, you should test it to make sure it is set up correctly.

1. Create an issue in the repository, then assign it to Copilot.
2. Wait a few seconds, and Copilot will leave an 👀 reaction on the issue.
3. Wait a few more seconds, and Copilot will create a pull request, which will appear in the issue's timeline.
4. Click the created pull request in the timeline, and wait until a "Copilot started work" timeline event appears.
5. Click View session to open the Copilot coding agent logs.
6. Click the ellipsis button (...) at the top right of the log viewer, then click Copilot in the sidebar.
7. Click the Start MCP Servers step to expand the logs.
8. If your MCP servers have been started successfully, you will see their tools listed at the bottom of the logs.

If your MCP servers require any dependencies that are not installed on the GitHub Actions runner by default, such as `uv` and `pipx`, or that need special setup steps, you may need to create a `copilot-setup-steps.yml` Actions workflow file to install them. For more information, see [Customizing the development environment for GitHub Copilot coding agent](https://docs.github.com/en/enterprise-cloud@latest/copilot/customizing-copilot/customizing-the-development-environment-for-copilot-coding-agent).

## Customizing the built-in GitHub MCP server

The GitHub MCP server is enabled by default and connects to GitHub with a specially scoped token that only has read-only access to the current repository.

If you want to allow Copilot to access data outside the current repository, you can give it a personal access token with wider access.

1. Create a personal access token with the appropriate permissions. We recommend using a fine-grained personal access token, where you can limit the token's access to read-only permissions on specific repositories. For more information on personal access tokens, see [Managing your personal access tokens](https://docs.github.com/en/enterprise-cloud@latest/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).
2. On GitHub, navigate to the main page of the repository.
3. Under your repository name, click Settings.
4. In the "Code & automation" section of the sidebar, click Copilot then Coding agent.
5. Add your configuration in the MCP configuration section. For example, you can add the following:

```json
{
  "mcpServers": {
    "github-mcp-server": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/readonly",
      "tools": ["*"]
    }
  }
}
```

1. Click Save.
2. In the left sidebar, click Environments.
3. Click the `copilot` environment.
4. Under "Environment secrets", click Add environment secret.
5. Call the secret `COPILOT_MCP_GITHUB_PERSONAL_ACCESS_TOKEN`, enter your personal access token in the "Value" field, then click Add secret.

For information on using the GitHub MCP server in other environments, see [Using the GitHub MCP Server](https://docs.github.com/en/enterprise-cloud@latest/copilot/customizing-copilot/using-model-context-protocol/using-the-github-mcp-server).

## Next steps

- [Creating custom agents](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)
- [Customizing the development environment for GitHub Copilot coding agent](https://docs.github.com/en/enterprise-cloud@latest/copilot/customizing-copilot/customizing-the-development-environment-for-copilot-coding-agent)
- [Extending GitHub Copilot Chat with Model Context Protocol (MCP) servers](https://docs.github.com/en/enterprise-cloud@latest/copilot/customizing-copilot/extending-copilot-chat-with-mcp)
