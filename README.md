# GitHub MCP Server on Cloudflare Workers

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/leelakanakala/github-mcp)

A full-featured **Model Context Protocol (MCP)** server providing all 26 official GitHub tools, deployed as a serverless application on Cloudflare Workers with OAuth authentication and optional Cloudflare Access integration.

## Features

### All 26 Official GitHub Tools

#### File Operations (3 tools)
- `create_or_update_file` - Create or update single files
- `push_files` - Batch file operations in single commit
- `get_file_contents` - Read files and directories

#### Repository Management (4 tools)
- `search_repositories` - Search GitHub repositories
- `create_repository` - Create new repositories
- `fork_repository` - Fork repositories
- `create_branch` - Create branches

#### Issue Management (5 tools)
- `create_issue` - Create issues
- `list_issues` - List/filter issues
- `update_issue` - Update issues
- `add_issue_comment` - Comment on issues
- `get_issue` - Get issue details

#### Pull Request Management (10 tools)
- `create_pull_request` - Create PRs
- `list_pull_requests` - List/filter PRs
- `get_pull_request` - Get PR details
- `merge_pull_request` - Merge PRs
- `create_pull_request_review` - Review PRs
- `get_pull_request_files` - Get PR file changes
- `get_pull_request_status` - Get PR status checks
- `update_pull_request_branch` - Update PR branch
- `get_pull_request_comments` - Get PR comments
- `get_pull_request_reviews` - Get PR reviews

#### Search Operations (3 tools)
- `search_code` - Search code across GitHub
- `search_issues` - Search issues/PRs
- `search_users` - Search users

#### Commit Operations (1 tool)
- `list_commits` - List repository commits

### Security Features
- GitHub OAuth 2.0 authentication
- CSRF protection with one-time tokens
- State validation with session binding
- Secure cookie management (HttpOnly, Secure, SameSite)
- Optional Cloudflare Access integration
- Encrypted token storage

### Architecture
- **Serverless**: Cloudflare Workers for global edge deployment
- **Stateful**: Durable Objects for per-user state management
- **Scalable**: KV storage for OAuth state
- **Transport**: Server-Sent Events (SSE) for MCP protocol

## Prerequisites

- [Node.js](https://nodejs.org/) v18+ 
- [Cloudflare account](https://dash.cloudflare.com/sign-up)
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/install-and-update/)
- [GitHub account](https://github.com/)

## Quick Start

### 1. Install Dependencies

```bash
cd github-mcp-server
npm install
```

### 2. Login to Cloudflare

```bash
npx wrangler login
```

This will open your browser to authenticate with Cloudflare.

### 3. Find Your Cloudflare Subdomain

```bash
npx wrangler whoami
```

Look for your account subdomain (e.g., `yourname.workers.dev`).

### 4. Create GitHub OAuth Apps

You need **TWO** OAuth Apps: one for production, one for development.

#### Production OAuth App

1. Go to: https://github.com/settings/developers
2. Click **"New OAuth App"**
3. Fill in:
   - **Application name**: `GitHub MCP Server - Production`
   - **Homepage URL**: `https://github-mcp-server.<your-subdomain>.workers.dev`
   - **Authorization callback URL**: `https://github-mcp-server.<your-subdomain>.workers.dev/callback`
4. Click **"Register application"**
5. Note your **Client ID**
6. Click **"Generate a new client secret"**
7. Copy the **Client Secret** (you won't see it again!)

#### Development OAuth App

1. Create another OAuth App:
   - **Application name**: `GitHub MCP Server - Development`
   - **Homepage URL**: `http://localhost:8788`
   - **Authorization callback URL**: `http://localhost:8788/callback`
2. Note the **Client ID** and **Client Secret**

### 5. Set Up Local Development

```bash
# Copy the example environment file
cp .dev.vars.example .dev.vars

# Generate cookie encryption key
openssl rand -hex 32

# Edit .dev.vars and add your development credentials
nano .dev.vars
```

Your `.dev.vars` should look like:
```
GITHUB_CLIENT_ID=your_dev_client_id_here
GITHUB_CLIENT_SECRET=your_dev_client_secret_here
COOKIE_ENCRYPTION_KEY=your_generated_key_here
```

### 6. Create KV Namespace

```bash
# Create KV namespace for OAuth state
npx wrangler kv namespace create "OAUTH_KV"
```

Copy the `id` from the output and update `wrangler.jsonc`:

```jsonc
"kv_namespaces": [
  {
    "binding": "OAUTH_KV",
    "id": "paste_your_kv_id_here"  // Replace this
  }
]
```

### 7. Test Locally

```bash
npm run dev
```

Visit http://localhost:8788 - you should see the server running.

Test with MCP Inspector:
```bash
npx @modelcontextprotocol/inspector@latest
```

Enter: `http://localhost:8788/sse` and click Connect.

### 8. Deploy to Production

```bash
# Set production secrets
npx wrangler secret put GITHUB_CLIENT_ID
# Paste your PRODUCTION client ID

npx wrangler secret put GITHUB_CLIENT_SECRET
# Paste your PRODUCTION client secret

npx wrangler secret put COOKIE_ENCRYPTION_KEY
# Paste the same encryption key from .dev.vars

# Deploy
npm run deploy
```

Your server will be live at: `https://github-mcp-server.<your-subdomain>.workers.dev`

## Usage

### With Claude Desktop

Add to your Claude Desktop config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://github-mcp-server.<your-subdomain>.workers.dev/sse"
      ]
    }
  }
}
```

Restart Claude Desktop. On first use, you'll authenticate with GitHub via your browser.

### With Cursor

In Cursor settings, add:
```json
{
  "Type": "Command",
  "Command": "npx mcp-remote https://github-mcp-server.<your-subdomain>.workers.dev/sse"
}
```

### With MCP Inspector

```bash
npx @modelcontextprotocol/inspector@latest
```

Enter your server URL: `https://github-mcp-server.<your-subdomain>.workers.dev/sse`

## Cloudflare Access Integration

To add an additional authentication layer with Cloudflare Access:

### 1. Set Up Cloudflare Access

1. Go to [Cloudflare Zero Trust](https://one.dash.cloudflare.com/)
2. Navigate to **Access** > **Applications**
3. Click **Add an application**
4. Choose **Self-hosted**
5. Configure:
   - **Application name**: `GitHub MCP Server`
   - **Subdomain**: `github-mcp-server`
   - **Domain**: `<your-subdomain>.workers.dev`
   - **Path**: Include all paths `/`

### 2. Configure Access Policies

Create policies for who can access:

**Example Policy: Email-based Access**
```
Policy name: Allow Your Team
Action: Allow
Include:
  - Emails: you@example.com, teammate@example.com
```

**Example Policy: Group-based Access**
```
Policy name: Engineering Team
Action: Allow
Include:
  - Email domain: yourcompany.com
  - Groups: engineering
```

### 3. Update Your Application

No code changes needed! Cloudflare Access will automatically:
1. Intercept requests to your Worker
2. Present login page if not authenticated
3. Validate JWT tokens
4. Pass through to your Worker if authorized

### Access Flow

1. User clicks MCP server link
2. Cloudflare Access checks authentication
3. If not authenticated → Login page (email, SSO, etc.)
4. If authenticated but not authorized → Access denied page
5. If authorized → Request passes to Worker
6. Worker validates GitHub OAuth
7. User gets access to GitHub tools

## Tools Reference

### Example: Create a File

```javascript
// Tool: create_or_update_file
{
  "owner": "your-username",
  "repo": "your-repo",
  "path": "src/example.ts",
  "content": "console.log('Hello, World!');",
  "message": "Add example file",
  "branch": "main"
}
```

### Example: Create an Issue

```javascript
// Tool: create_issue
{
  "owner": "your-username",
  "repo": "your-repo",
  "title": "Bug: Something is broken",
  "body": "Description of the issue...",
  "labels": ["bug", "priority-high"]
}
```

### Example: Search Code

```javascript
// Tool: search_code
{
  "q": "language:typescript path:src/ fetch",
  "per_page": 20
}
```

For detailed tool documentation, see [TOOLS.md](./TOOLS.md).

## GitHub API Scopes

The server requests these GitHub OAuth scopes:

- `repo` - Full control of private repositories (includes public repos)
- `read:user` - Read user profile data
- `read:org` - Read organization data

Users grant these permissions during OAuth flow.

## Development

### Project Structure

```
github-mcp-server/
├── src/
│   ├── index.ts                    # Main MCP server with 26 tools
│   ├── github-handler.ts           # OAuth flow handlers
│   ├── utils.ts                    # GitHub OAuth utilities
│   └── workers-oauth-utils.ts      # Security utilities
├── package.json                    # Dependencies
├── tsconfig.json                   # TypeScript config
├── wrangler.jsonc                  # Cloudflare Workers config
├── .dev.vars.example               # Example environment variables
└── README.md                       # This file
```

### Adding Custom Tools

Edit `src/index.ts` and add your tool after the existing 26:

```typescript
this.server.tool(
  "my_custom_tool",
  "Description of what it does",
  {
    param1: z.string().describe("Parameter description"),
  },
  async ({ param1 }) => {
    const octokit = new Octokit({ auth: this.props!.accessToken });
    // Your implementation using octokit
    return {
      content: [{ type: "text", text: "Result" }]
    };
  }
);
```

### Running Tests

```bash
# Type checking
npm run type-check

# Local development server
npm run dev

# Test with Inspector
npx @modelcontextprotocol/inspector@latest
```

## Troubleshooting

### "Not logged in" Error

```bash
npx wrangler login
```

### "Failed to fetch" or Certificate Errors

Check your network connection and proxy settings. If behind a corporate proxy, configure:

```bash
export NODE_TLS_REJECT_UNAUTHORIZED=0  # Only for development!
```

### OAuth Flow Issues

1. Verify your OAuth App callback URLs match exactly
2. Check that secrets are set correctly:
   ```bash
   npx wrangler secret list
   ```
3. Ensure KV namespace ID is correct in `wrangler.jsonc`

### Tools Not Showing Up

1. Check server logs:
   ```bash
   npx wrangler tail
   ```
2. Verify GitHub token has correct scopes
3. Test authentication with MCP Inspector

### Cloudflare Access Not Working

1. Verify Access application is configured correctly
2. Check Access policies allow your identity
3. Test Access separately: visit your worker URL in browser
4. Review Access logs in Cloudflare dashboard

## Cost Estimate

### Cloudflare Workers Free Tier
- **100,000 requests/day**
- **Durable Objects**: First 1M operations/month free
- **KV**: First 100K reads/day free

### Paid Plan (if needed)
- **Workers**: $5/month for 10M requests
- **Durable Objects**: $0.15 per million requests
- **KV**: $0.50 per million reads

For typical personal/team use, the **free tier is sufficient**.

### Cloudflare Access Pricing
- **Free**: Up to 50 users
- **Paid**: $3/user/month for additional users

## Security Considerations

✅ **Implemented:**
- OAuth 2.0 with PKCE flow
- CSRF protection
- State validation with session binding
- Secure cookies (HttpOnly, Secure, SameSite)
- Input validation with Zod
- Error handling without leaking sensitive data

⚠️ **Additional Recommendations:**
- Enable Cloudflare Access for production
- Rotate OAuth secrets regularly
- Monitor usage and set up alerts
- Review Cloudflare logs periodically
- Implement rate limiting if needed

## Resources

- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Cloudflare Workers Docs](https://developers.cloudflare.com/workers/)
- [GitHub OAuth Apps](https://docs.github.com/en/apps/oauth-apps)
- [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/applications/)
- [Octokit REST API](https://octokit.github.io/rest.js/)

## License

MIT

## Contributing

Contributions welcome! Please open an issue or PR.

## Support

- Issues: [GitHub Issues](https://github.com/your-username/github-mcp-server/issues)
- Discussions: [GitHub Discussions](https://github.com/your-username/github-mcp-server/discussions)
- MCP Community: [Discord](https://discord.gg/modelcontextprotocol)
