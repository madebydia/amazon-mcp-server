# Amazon Cart MCP Server

[![SafeSkill 95/100](https://img.shields.io/badge/SafeSkill-95%2F100_Verified%20Safe-brightgreen)](https://safeskill.dev/scan/meimakes-amazon-mcp-server)
Local MCP (Model Context Protocol) server that enables AI assistants to interact with your personal Amazon cart through browser automation. Works with [Claude Desktop](https://claude.ai/download), [Poke](https://poke.com), and any MCP-compatible client.

## ⚠️ Important Disclaimer

**This tool uses browser automation to interact with Amazon.com.**

- **Users are solely responsible** for ensuring their use complies with [Amazon's Terms of Service](https://www.amazon.com/gp/help/customer/display.html?nodeId=508088)
- This project is **for personal, educational use only** - not for commercial automation or reselling
- **Use at your own risk** - the authors assume no liability for any violations of Amazon's policies or consequences thereof
- **Not affiliated with Amazon** - this is an independent, unofficial tool
- Amazon may change their website or policies at any time, potentially breaking functionality
- Excessive automation may result in account restrictions or bans

By using this software, you acknowledge and accept these risks.

## Features

- 🔍 **Search Amazon** - Find products by search query
- 🛒 **Add to Cart** - Add items to your Amazon cart automatically
- 👀 **View Cart** - Check current cart contents and subtotal
- 🔐 **Login Persistence** - Session saved locally for seamless use
- 🌐 **Secure Access** - Bearer token authentication via ngrok tunnel

## Quick Start

### Prerequisites

- Node.js v20 or higher
- npm or yarn
- ngrok account (free tier works)

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/meimakes/amazon-mcp-server.git
   cd amazon-mcp-server
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Configure environment:**
   ```bash
   cp .env .env.local  # Optional: keep your settings separate
   ```

   Edit `.env` and set:
   - `AUTH_TOKEN` - Generate a secure random token (required)
   - `HEADLESS=false` - For first-time login
   - `AMAZON_DOMAIN=amazon.com` - Or your local Amazon domain

4. **Build the project:**
   ```bash
   npm run build
   ```

5. **Start the server:**
   ```bash
   npm start
   ```

6. **First-time login:**
   - A Chrome browser window will open
   - Log into your Amazon account manually
   - Session will be saved in `./user-data/`
   - **After logging in once**, you can:
     - Stop the server (Ctrl+C)
     - Set `HEADLESS=true` in `.env`
     - Restart with headless mode

7. **Expose via ngrok (in a separate terminal):**
   ```bash
   npm run tunnel
   # Note the HTTPS URL (e.g., https://abc123.ngrok.io)
   ```

## Connecting to Poke

1. Copy your ngrok URL from the terminal
2. In Poke, add a custom MCP integration:
   - **URL:** `https://your-ngrok-url.ngrok.io/sse`
   - **API Key:** Your `AUTH_TOKEN` from `.env`
   - **Type:** MCP Server

3. **Important:** Always use the `/sse` endpoint!

4. Test the connection by asking Poke:
   - "What tools do you have?"
   - "Search Amazon for wireless mouse"

## Connecting to Claude Desktop

1. Build the project: `npm run build`
2. Open Claude Desktop → Settings → Developer → Edit Config
3. Add to `mcpServers`:

```json
{
  "mcpServers": {
    "amazon-cart": {
      "command": "node",
      "args": ["/absolute/path/to/amazon-mcp-server/dist/server.js"],
      "env": {
        "AUTH_TOKEN": "your-token-here",
        "HEADLESS": "true",
        "AMAZON_DOMAIN": "amazon.com"
      }
    }
  }
}
```

4. Restart Claude Desktop
5. You should see the Amazon tools available in the tools menu (🔧)

> **First-time setup:** Run the server once with `HEADLESS=false` to log into Amazon manually. After that, set `HEADLESS=true` for Claude Desktop.

## Available Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `search_amazon` | Search for products on Amazon | `query` (required) |
| `add_to_cart` | Add a product to cart | `query` or `asin`, `quantity` (optional) |
| `view_cart` | View current cart contents | None |
| `check_login` | Verify Amazon login status | None |

## Architecture

```
┌─────────────────┐
│   Poke.com      │ (Remote AI Assistant)
│   (Cloud)       │
└────────┬────────┘
         │ HTTPS
         ↓
┌─────────────────┐
│     ngrok       │ (Secure Tunnel)
│  Public HTTPS   │
└────────┬────────┘
         │ Local
         ↓
┌─────────────────┐
│   MCP Server    │ (Port 3000)
│   SSE + HTTP    │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│   Puppeteer     │ (Browser Automation)
│  + Chrome       │
│  (Persistent    │
│   Session)      │
└─────────────────┘
```

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3000` | Server port |
| `AUTH_TOKEN` | *required* | Bearer token for authentication |
| `AMAZON_DOMAIN` | `amazon.com` | Amazon domain (e.g., amazon.co.uk) |
| `HEADLESS` | `false` | Run browser in headless mode |
| `USER_DATA_DIR` | `./user-data` | Chrome user data directory |

### Example .env

```bash
PORT=3000
AUTH_TOKEN=a1b2c3d4-e5f6-4789-a012-3b4c5d6e7f8a
AMAZON_DOMAIN=amazon.com
HEADLESS=false
USER_DATA_DIR=./user-data
```

## Security

### ⚠️ Important Security Considerations

1. **AUTH_TOKEN Protection**
   - Never commit `.env` to Git (already in `.gitignore`)
   - Use a cryptographically secure random token
   - Generate with: `openssl rand -hex 32`

2. **ngrok Security**
   - Free tier URLs are public but unguessable
   - Consider ngrok's authentication features for extra security
   - Upgrade to ngrok paid plan for reserved domains and IP restrictions

3. **Session Data**
   - Login sessions stored in `./user-data/`
   - Contains cookies and authentication tokens
   - Never share or commit this directory
   - Already excluded via `.gitignore`

4. **Network Security**
   - Server only accepts authenticated requests
   - All traffic through ngrok is HTTPS encrypted
   - Local server binds to localhost only

5. **Browser Automation**
   - Puppeteer runs with sandbox disabled (required for some systems)
   - Session isolation via Chrome user data directory
   - No data sent to third parties

### Best Practices

- ✅ Use strong, unique AUTH_TOKEN
- ✅ Never share your ngrok URL publicly
- ✅ Regularly rotate AUTH_TOKEN
- ✅ Monitor server logs for suspicious activity
- ✅ Keep dependencies updated (`npm audit`)
- ✅ Use HEADLESS=true in production
- ⚠️ This is for personal use only - not production-ready for multi-user scenarios

## Troubleshooting

### Tools Not Showing in Poke

1. Restart the server
2. Delete and re-add the MCP connection in Poke
3. Check server logs for `tools/list` request
4. Verify ngrok tunnel is active

### Items Not Added to Cart

1. Verify you're logged into Amazon:
   - Check the browser window (if visible)
   - Or ask Poke to run `check_login`
2. If not logged in:
   - Set `HEADLESS=false`
   - Restart server
   - Log in manually in the browser window

### Connection Keeps Dropping

- Normal behavior - Poke reconnects as needed
- If persistent, check ngrok connection: `curl https://your-url.ngrok.io/health`

### Computer Sleep Mode

- Server and ngrok pause when computer sleeps
- Poke will reconnect automatically on wake
- To prevent sleep: Run `caffeinate` in a separate terminal (macOS)

## Development

### Project Structure

```
amazon-mcp/
├── src/
│   ├── server.ts       # MCP server + SSE implementation
│   ├── amazon.ts       # Amazon automation logic
│   ├── browser.ts      # Puppeteer browser management
│   └── types.ts        # TypeScript interfaces
├── dist/               # Compiled JavaScript (gitignored)
├── user-data/          # Chrome session data (gitignored)
├── .env                # Environment config (gitignored)
└── package.json
```

### Running in Development

```bash
npm run dev    # Uses ts-node, no build required
```

### Building

```bash
npm run build  # Compiles TypeScript to dist/
```

## Testing

### Health Check

```bash
curl http://localhost:3000/health
```

Expected response:
```json
{"status":"ok","server":"amazon-mcp-server"}
```

### Test SSE Connection

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  http://localhost:3000/sse
```

Should maintain an open connection with heartbeats.

## Compliance Notes

This project is designed for **personal, single-user use only**. It is not intended for:

- ❌ Multi-tenant deployments
- ❌ Production SaaS applications
- ❌ SOC 2 Type II compliance scenarios
- ❌ HIPAA or other regulated data handling
- ❌ Commercial automation at scale

If you need enterprise-grade compliance, consider:
- Implementing proper authentication (OAuth 2.0)
- Adding audit logging
- Using encrypted storage for sessions
- Deploying to compliant infrastructure (AWS, GCP with compliance certifications)
- Implementing rate limiting and abuse prevention

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Support

- 🐛 **Issues:** [GitHub Issues](https://github.com/meimakes/amazon-mcp-server/issues)
- 📧 **Contact:** via GitHub

## Author

Created by [@meimakes](https://github.com/meimakes)

---

**Note:** Keep your computer awake while running the server. The ngrok tunnel and SSE connections are sensitive to network interruptions.
