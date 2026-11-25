# OpenSEO Self-Hosted for Claude Code

The plugin includes the upstream OpenSEO skills and connects to your own OpenSEO instance. Claude Code stores the endpoint you enter in your user configuration.

Inside Claude Code, run:

```text
/plugin marketplace add elohimito/open-seo#claude-selfhost-plugin
/plugin install openseo-selfhost@openseo-selfhost
/reload-plugins
```

Choose **User** scope to make the plugin available across projects. When prompted for **OpenSEO MCP URL**, enter the full HTTPS address of your instance, including `/mcp`. You can update it in `/config`.

Open `/mcp`, select the OpenSEO server, and sign in. Cloudflare Access installations use the email in their Access policy and require Managed OAuth with loopback redirects enabled.

Use the `list_projects` tool for a connection check without spending SEO research credits. The bundled skills appear under the `/openseo-selfhost:` namespace.

- [Cloudflare Access and MCP setup](../../docs/SELF_HOSTING_CLOUDFLARE_OPERATIONS.md#connect-the-mcp-server)
- [Upstream OpenSEO project](https://github.com/every-app/open-seo)
