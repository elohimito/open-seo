# OpenSEO operations

This fork uses the [upstream Cloudflare self-host deployment](../SELF_HOSTING_CLOUDFLARE.md) and its built-in dashboard and MCP server. Keep the deployment on an identifiable upstream release. Keep the deployment inventory, project identifiers, test-account usage, and incident notes in a private record.

## Deploy and maintain

1. Use Node 22.6+ and the pnpm version in `package.json`. Install with `corepack pnpm install --frozen-lockfile`.
2. Select the intended Cloudflare account. Access requires an active Zero Trust organization. Run the upstream Alchemy login with its default scopes plus `access:write`, or use a short-lived token restricted to the target account and the resource operations required by the stack. Use Alchemy for the state-store bootstrap and deployment.
3. Start from `deploy/.env.selfhost.example`. Set `AUTH_MODE=cloudflare_access`, `ACCESS_ALLOWED_EMAILS`, and `OPENSEO_TELEMETRY_DISABLED=1`. Set the DataForSEO Base64 credential as `DATAFORSEO_API_KEY` in the ignored `.env.selfhost` file and restrict local file permissions. Alchemy writes the credential as a Worker secret. Keep its source in the operator's secret manager for redeploys.
4. Run `corepack pnpm deploy:selfhost --yes` on the intended release. Confirm dashboard sign-in, the authenticated health endpoint, access rejection on the dashboard and MCP, and existing project history after redeployment. Verify the Access app still matches the deployed hostname and the Worker has Preview URLs disabled.
5. Before merging upstream changes, review release notes, D1 migrations, and changes to Access and the provider gateway. Run `corepack pnpm test:ci`, the self-host build, and `corepack pnpm exec tsc --noEmit`. Deploy with the same Alchemy stack and stage to reuse resources. The pinned Alchemy release ties previews to the main Worker URL by default. This fork patches its Worker provider to honor `subdomain.previewsEnabled` separately. Check whether the upstream provider supports this setting before dropping the patch.

### Backup and recovery

Record a [D1 Time Travel bookmark](https://developers.cloudflare.com/d1/reference/time-travel/) and export D1 before a schema-changing deployment. Keep exports private. Copy valuable R2 artifacts separately. R2's provider cache can be rebuilt by research requests that may incur costs. Keep the Alchemy state store and deployment bootstrap credentials recoverable. Review restoration in a separate environment before touching live data. A Worker version rollback does not reverse D1 migrations. Upstream's destroy command deletes self-hosted resources and data.

## Operator and agent access

Use `https://WORKER_HOSTNAME/mcp` for Streamable HTTP MCP. Let Cloudflare Access Managed OAuth authenticate the operator. Keep the Access allowlist to named users and allow loopback callbacks for CLI clients. Add only exact HTTPS callback URLs shown by the relevant web client. Use `list_projects` for the first connectivity check.

| Client | Connection |
| --- | --- |
| [Claude Code](https://code.claude.com/docs/en/mcp) | `claude mcp add --transport http openseo --scope user https://WORKER_HOSTNAME/mcp`, then authenticate under `/mcp`. |
| [Codex CLI](https://developers.openai.com/codex/mcp) | `codex mcp add openseo --url https://WORKER_HOSTNAME/mcp`, then `codex mcp login openseo` if needed. |
| [Cursor](https://cursor.com/docs/mcp) | Add `{"mcpServers":{"openseo":{"url":"https://WORKER_HOSTNAME/mcp"}}}` to the user MCP config and follow OAuth. Dynamic registration can require `https://www.cursor.com/agents/mcp/oauth/callback` even from desktop. Allow that exact callback in the Access app. |
| [ChatGPT](https://openseo.so/docs/chatgpt) | Create a private MCP app using the endpoint and OAuth. Copy the exact redirect URI displayed in the [app's management page](https://developers.openai.com/plugins/build/auth) to the Access app. |
| Other clients | Select Streamable HTTP, then use an OAuth client that supports the Access resource parameter and the server's available registration method. |

Managed OAuth is configured by the upstream stack. See [OpenSEO Cloudflare operations](../SELF_HOSTING_CLOUDFLARE_OPERATIONS.md) and [Cloudflare Managed OAuth](https://developers.cloudflare.com/cloudflare-one/access-controls/applications/http-apps/managed-oauth/). Verify each client before using paid tools.

## Research spending

Use [DataForSEO API Settings](https://app.dataforseo.com/api-settings) for account-wide and endpoint-level daily enforcement. DataForSEO resets daily limits at midnight UTC. An account-wide cap applies to all consumers of the same account. Verify balance and cost using the free [user data endpoint](https://docs.dataforseo.com/v3/appendix/user_data/) with credentials supplied through a secure channel. Auto-recharge is a separate setting. Its monthly recharge limit bounds deposits and does not enforce usage.

Use existing results before issuing provider calls. The Alchemy stack provisions an R2 provider cache and a seven-day cache-object lifecycle. Individual operations apply their own cache semantics, input bounds, and concurrency controls. Repeated requests can still hit the provider. Self-hosted OpenSEO's estimates and usage views do not impose daily, monthly, or project-specific spend limits. Require an explicit budget and human approval for large or novel batches. Keep scheduled rank and AI-visibility tracking off unless recurring paid collection has been approved. Project creation through `create_project` creates the record without starting collection. Website setup and tracking tools can trigger paid work, including scheduled work.

## Integrating other systems

Use MCP for interactive competitor, keyword, backlink, and AI visibility research. Pass the OpenSEO project ID and market in requests where applicable. Keep exports or normalized results with the project ID, provider, fetched time, and actual cost so other workflows can reuse them. For an unattended job, design a dedicated integration that approves and caps each paid batch and writes an idempotent spend record before enabling it. Treat the dashboard's internal RPCs as implementation details.

The optional [Search Console](../SELF_HOSTING_GOOGLE_SEARCH_CONSOLE.md) and [GA4](../SELF_HOSTING_GOOGLE_ANALYTICS.md) links are useful for researching a project inside OpenSEO. They each require specific read-only Google consent and an OAuth client. Reuse an existing growth system's GA4 and Search Console data until there is an approved reason to connect the same properties in OpenSEO.
