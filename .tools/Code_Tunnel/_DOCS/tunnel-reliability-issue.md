# VS Code Tunnel Reliability Issue (#236724)

## Issue Summary

**Title:** Document how to use VS Code Tunnels reliably for more than 2 or 3 days without disconnection

**Status:** Open (Bug identified by VS Code Team member)

**Assignee:** connor4312

**Labels:** bug

## Problem Description

Many VS Code users have reported that it's difficult or impossible to keep VS Code tunnels running for multiple days. This is an ongoing problem with no clear solution.

Typically, a VS Code tunnel will stay online for roughly 2 to 5 days, and then will fail. The issue appears to have started around early August 2024, possibly due to a security update.

## Error Symptoms

When tunnels disconnect, users get an error message like:

```log
Dec 20 08:55:08 code-tunnel[3683048]: [2024-12-20 08:55:08] warn Error refreshing access token, will retry: failed to lookup tunnel for host token: response error: HTTP status 401 Unauthorized from https://usw3.rel.tunnels.api.visualstudio.com/tunnels/REDACTED-94lzwh5?tokenScopes=host&api-version=2023-09-27-preview (request ID fb9d5243-5aa8-48fd-aace-71f027a4fee8):
```

## Workarounds Tested

### Microsoft Account Login

- Switching from GitHub to Microsoft account authentication was suggested but did not resolve the issue
- Users still experience tunnel failures after 2-5 days

### Manual Restart Process

Current workaround requires GUI access:

1. Close VS Code if it's running
2. Re-open VS Code
3. Stop tunnel if it's running
4. Start tunnel
5. Login with Microsoft account

### Alternative Access Methods

- Maintain another way to access the dev machine (e.g., VNC)
- When tunnel stops working (usually 1-2 times per week), access via VNC and restart tunnel

## Community Impact

- Issue affects multiple users (@kingpalethe, @tachyon-beep)
- Requires manual intervention every few days
- Impacts development workflows and remote access reliability

## Historical Context

- Before early August 2024, tunnels would stay open for weeks or months without issues
- Issue likely introduced by a security or authentication update

## Requested Solution

The issue requests documentation for reliable long-term tunnel usage, or potentially a technical fix for the underlying authentication/refresh problem.

## Related Issues

- Referenced issue #230058 which was previously closed
- Multiple similar issues have been reported and closed without resolution
