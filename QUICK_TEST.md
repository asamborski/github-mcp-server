# Quick Test Guide

Your GitHub MCP Server is deployed and working! ✅

**URL:** https://github-mcp-server.zero-security.workers.dev

## ⚠️ Important: MCP Inspector Limitation

The MCP Inspector **does not support OAuth authentication**. Your server requires OAuth, so you need to use `mcp-remote` which handles the OAuth flow automatically.

---

## ✅ Option 1: Test with Claude Desktop (Recommended)

### Step 1: Install mcp-remote globally
```bash
npm install -g mcp-remote
```

### Step 2: Configure Claude Desktop

Edit your Claude Desktop config file:

**macOS:**
```bash
nano ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

**Linux:**
```bash
nano ~/.config/Claude/claude_desktop_config.json
```

**Windows:**
```bash
notepad %APPDATA%\Claude\claude_desktop_config.json
```

### Step 3: Add this configuration:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://github-mcp-server.zero-security.workers.dev/mcp"
      ]
    }
  }
}
```

### Step 4: Restart Claude Desktop

1. Quit Claude Desktop completely
2. Reopen Claude Desktop
3. A browser window will open for GitHub OAuth
4. Approve the authorization
5. Tools will appear in Claude!

---

## ✅ Option 2: Test OAuth Flow Manually

### Test OAuth Registration:
```bash
curl -X POST https://github-mcp-server.zero-security.workers.dev/register \
  -H "Content-Type: application/json" \
  -d '{"client_name":"Test Client","redirect_uris":["http://localhost:3000/callback"]}' | jq .
```

**Expected:** You get a `client_id` and `client_secret`

### Test Authorization Endpoint:
Open in browser:
```
https://github-mcp-server.zero-security.workers.dev/authorize?client_id=YOUR_CLIENT_ID&redirect_uri=http://localhost:3000/callback&response_type=code&scope=test
```

**Expected:** You see the approval dialog

---

## ✅ Option 3: Test with Cursor IDE

In Cursor settings, add:

**Type:** Command  
**Command:** `npx -y mcp-remote https://github-mcp-server.zero-security.workers.dev/mcp`

---

## 🔍 Verify Deployment

### Check Worker is Running:
```bash
curl https://github-mcp-server.zero-security.workers.dev/mcp
```

**Expected Response:**
```json
{
  "error": "invalid_token",
  "error_description": "Missing or invalid access token"
}
```

This is **correct** - it means the server is running and expecting OAuth authentication.

### Check Secrets are Set:
```bash
npx wrangler secret list
```

**Expected:**
```json
[
  {"name": "OAUTH_GITHUB_CLIENT_ID", "type": "secret_text"},
  {"name": "OAUTH_GITHUB_CLIENT_SECRET", "type": "secret_text"},
  {"name": "COOKIE_ENCRYPTION_KEY", "type": "secret_text"}
]
```

### View Live Logs:
```bash
NODE_TLS_REJECT_UNAUTHORIZED=0 npx wrangler tail
```

Then try connecting from Claude Desktop and watch the logs in real-time!

---

## 📊 What's Deployed

### All 26 GitHub Tools:
- `create_or_update_file` - Create/update files
- `push_files` - Batch file operations
- `get_file_contents` - Read files
- `search_repositories` - Search repos
- `create_repository` - Create repos
- `fork_repository` - Fork repos
- `create_branch` - Create branches
- `create_issue` - Create issues
- `list_issues` - List issues
- `update_issue` - Update issues
- `add_issue_comment` - Comment on issues
- `get_issue` - Get issue details
- `create_pull_request` - Create PRs
- `list_pull_requests` - List PRs
- `get_pull_request` - Get PR details
- `merge_pull_request` - Merge PRs
- `create_pull_request_review` - Review PRs
- `get_pull_request_files` - Get PR files
- `get_pull_request_status` - Get PR status
- `update_pull_request_branch` - Update PR branch
- `get_pull_request_comments` - Get PR comments
- `get_pull_request_reviews` - Get PR reviews
- `search_code` - Search code
- `search_issues` - Search issues/PRs
- `search_users` - Search users
- `list_commits` - List commits

### Security Features:
- ✅ GitHub OAuth 2.0
- ✅ CSRF protection
- ✅ State validation
- ✅ Secure cookies
- ✅ Encrypted tokens

---

## 🎯 Try These Commands in Claude

Once connected, try:

1. **"List all open issues in cloudflare/workers-sdk"**
2. **"Search for TypeScript files containing 'fetch' in cloudflare/workers-sdk"**
3. **"Show me the last 10 commits in cloudflare/ai"**
4. **"Create a new issue in my test repo about adding a README"**

---

## 🐛 Troubleshooting

### Browser doesn't open for OAuth
- Make sure `mcp-remote` is installed: `npm install -g mcp-remote`
- Try using `npx -y mcp-remote` in the config
- Check Claude Desktop logs

### "Invalid token" error
- This is normal before OAuth - means server is working!
- Complete OAuth flow via Claude Desktop or Cursor

### Tools not appearing
1. Restart Claude Desktop completely
2. Check config file is valid JSON
3. View Claude logs: `~/Library/Logs/Claude/`
4. Watch server logs: `npx wrangler tail`

### OAuth redirect fails
- Verify callback URL in GitHub OAuth app matches
- Should be: `https://github-mcp-server.zero-security.workers.dev/callback`

---

## 📚 Next Steps

1. ✅ Connect Claude Desktop (above)
2. ⬜ Test a few tools
3. ⬜ Set up Cloudflare Access (optional) - See [CLOUDFLARE_ACCESS.md](./CLOUDFLARE_ACCESS.md)
4. ⬜ Add team members
5. ⬜ Monitor usage with `wrangler tail`

---

## 🎉 Success!

Your GitHub MCP Server with OAuth is fully deployed and operational!

**Deployment URL:** https://github-mcp-server.zero-security.workers.dev
**Account:** Zero Security Prod
**Version:** 0f545de8-701c-4744-a030-ce3a76da2c1e
