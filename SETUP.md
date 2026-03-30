# Quick Setup Guide

Follow these steps to get your GitHub MCP Server running in under 10 minutes!

## ✅ Step 1: Login to Cloudflare (2 min)

```bash
npx wrangler login
```

This opens your browser for authentication. After logging in, return to terminal.

**Find your subdomain:**
```bash
npx wrangler whoami
```

Look for output like: `Account Name: <your-name>` 
Your subdomain is usually based on your account name or email.

---

## ✅ Step 2: Create GitHub OAuth Apps (5 min)

### Production App

1. Go to: https://github.com/settings/developers
2. Click **"New OAuth App"**
3. Fill in:
   - **Application name**: `GitHub MCP Server - Production`
   - **Homepage URL**: `https://github-mcp-server.<YOUR-SUBDOMAIN>.workers.dev`
   - **Callback URL**: `https://github-mcp-server.<YOUR-SUBDOMAIN>.workers.dev/callback`
4. Click **"Register application"**
5. Copy the **Client ID**
6. Click **"Generate a new client secret"**
7. Copy the **Client Secret** immediately (you won't see it again!)

### Development App

1. Create another OAuth App
2. Fill in:
   - **Application name**: `GitHub MCP Server - Development`
   - **Homepage URL**: `http://localhost:8788`
   - **Callback URL**: `http://localhost:8788/callback`
3. Copy **Client ID** and **Client Secret**

---

## ✅ Step 3: Configure Local Environment (1 min)

```bash
# Copy example environment file
cp .dev.vars.example .dev.vars

# Generate encryption key
openssl rand -hex 32
```

Edit `.dev.vars`:
```bash
nano .dev.vars
```

Paste your **development** credentials:
```
GITHUB_CLIENT_ID=<paste_dev_client_id>
GITHUB_CLIENT_SECRET=<paste_dev_client_secret>
COOKIE_ENCRYPTION_KEY=<paste_generated_key>
```

Save and exit (Ctrl+X, then Y, then Enter).

---

## ✅ Step 4: Create KV Namespace (1 min)

```bash
npx wrangler kv namespace create "OAUTH_KV"
```

You'll see output like:
```
{ binding = "OAUTH_KV", id = "abc123def456..." }
```

Copy the `id` and update `wrangler.jsonc`:
```bash
nano wrangler.jsonc
```

Find the `kv_namespaces` section and replace `<ADD_YOUR_KV_ID_HERE>` with your ID:
```jsonc
"kv_namespaces": [
  {
    "binding": "OAUTH_KV",
    "id": "abc123def456..."  // Your actual ID here
  }
]
```

Save and exit.

---

## ✅ Step 5: Test Locally (1 min)

```bash
npm run dev
```

You should see:
```
⎔ Starting local server...
⬣ Listening on http://localhost:8788
```

**Test in browser:**
Open http://localhost:8788 - you should see JSON response or redirect to OAuth.

**Test with MCP Inspector:**
```bash
# Open new terminal
npx @modelcontextprotocol/inspector@latest
```

Enter: `http://localhost:8788/sse`
Click **Connect**
Browser will open for authentication
After auth, tools will appear in Inspector!

---

## ✅ Step 6: Deploy to Production (2 min)

```bash
# Set production secrets
npx wrangler secret put GITHUB_CLIENT_ID
# Paste PRODUCTION client ID, press Enter

npx wrangler secret put GITHUB_CLIENT_SECRET
# Paste PRODUCTION client secret, press Enter

npx wrangler secret put COOKIE_ENCRYPTION_KEY
# Paste same encryption key from .dev.vars, press Enter

# Deploy!
npm run deploy
```

You'll see:
```
Published github-mcp-server
  https://github-mcp-server.<your-subdomain>.workers.dev
```

---

## ✅ Step 7: Test Production (1 min)

```bash
npx @modelcontextprotocol/inspector@latest
```

Enter: `https://github-mcp-server.<your-subdomain>.workers.dev/sse`

Authenticate and test tools!

---

## ✅ Step 8: Connect Claude Desktop

Edit Claude config:
```bash
# macOS
nano ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Linux
nano ~/.config/Claude/claude_desktop_config.json

# Windows
notepad %APPDATA%\Claude\claude_desktop_config.json
```

Add:
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://github-mcp-server.<YOUR-SUBDOMAIN>.workers.dev/sse"
      ]
    }
  }
}
```

Save, restart Claude Desktop, and you're done!

---

## 🎉 Success Checklist

- [x] Cloudflare account logged in
- [x] GitHub OAuth apps created (production + development)
- [x] Local environment configured (.dev.vars)
- [x] KV namespace created and configured
- [x] Local server tested
- [x] Production secrets set
- [x] Deployed to Cloudflare
- [x] Production server tested
- [x] Claude Desktop connected

---

## 🚀 What's Next?

### Optional: Set Up Cloudflare Access

For additional security, follow: [CLOUDFLARE_ACCESS.md](./CLOUDFLARE_ACCESS.md)

### Test the Tools

Ask Claude:
- "List all issues in the [repo] repository"
- "Create a new branch called 'feature/test' in [owner/repo]"
- "Search for TypeScript files containing 'async function' in [owner/repo]"
- "Create a pull request from branch [source] to [target]"

### Monitor Your Server

```bash
# View live logs
npx wrangler tail

# Check deployment status
npx wrangler deployments list
```

---

## ⚠️ Troubleshooting

### "Certificate Error" when logging in

Your network might be blocking the connection. Try:
```bash
export NODE_TLS_REJECT_UNAUTHORIZED=0  # Only for development!
npx wrangler login
```

### OAuth redirect fails

Double-check callback URLs match exactly:
- Development: `http://localhost:8788/callback`
- Production: `https://github-mcp-server.<subdomain>.workers.dev/callback`

### KV Namespace not found

Make sure you:
1. Created the namespace: `npx wrangler kv namespace create "OAUTH_KV"`
2. Copied the correct `id` to `wrangler.jsonc`
3. Saved the file

### Tools not appearing in Claude

1. Check server logs: `npx wrangler tail`
2. Test with MCP Inspector first
3. Restart Claude Desktop completely
4. Clear Claude's MCP cache

### "Access token is invalid"

GitHub token expired. Re-authenticate:
1. Disconnect from Claude
2. Clear browser cookies for your worker domain
3. Reconnect and complete OAuth flow again

---

## 📚 Documentation

- **README.md** - Full documentation and features
- **TOOLS.md** - Complete reference for all 26 tools
- **CLOUDFLARE_ACCESS.md** - Set up additional auth layer
- **SETUP.md** - This quick start guide

---

## 💬 Need Help?

- Check logs: `npx wrangler tail`
- View deployments: `npx wrangler deployments list`
- Test OAuth: Use MCP Inspector
- Cloudflare docs: https://developers.cloudflare.com/workers/
- MCP docs: https://modelcontextprotocol.io/

---

**Congratulations! Your GitHub MCP Server is running! 🎉**
