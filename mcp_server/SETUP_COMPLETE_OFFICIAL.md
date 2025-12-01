# Official Zep Graphiti Setup - Complete!

## What's Running

You're now using the **official Zep Graphiti MCP server** with Neo4j backend.

### Containers:
- **docker-neo4j-1**: Neo4j graph database (ports 7474, 7687)
- **docker-graphiti-mcp-1**: Official Graphiti MCP server (port 8000)

### Configuration:
- Location: `/Users/pierregallet/local-dev-environment/graphiti/mcp_server/`
- Docker Compose: `docker/docker-compose-neo4j.yml`
- Environment: `.env` file with your OpenAI API key

## Accessing the Services

### Neo4j Browser
- URL: http://localhost:7474
- Username: `neo4j`
- Password: `memorypassword`

### Graphiti MCP Server
- HTTP Endpoint: http://localhost:8000/mcp
- Used by Claude Desktop and other MCP clients

## Claude Desktop Integration

**Config File:** `~/Library/Application Support/Claude/claude_desktop_config.json`

**Configuration:**
```json
{
  "mcpServers": {
    "graphiti-memory": {
      "url": "http://localhost:8000/mcp"
    }
  }
}
```

## Next Steps

### 1. Restart Claude Desktop
The MCP server is configured. **Quit and reopen Claude Desktop** to load the Graphiti tools.

### 2. Test It
Try these commands in Claude Desktop:
- "Remember that I'm setting up Graphiti for shared AI memory"
- "Search your memory for information about my projects"
- "What do you know about Graphiti?"

### 3. Available Tools
The official MCP server provides these tools to LLMs:
- **add_episode**: Add new memories/episodes
- **search**: Search the knowledge graph
- **get_nodes**: Retrieve specific nodes
- **get_edges**: Get relationships between entities

## Managing the System

### Start/Stop Services
```bash
cd /Users/pierregallet/local-dev-environment/graphiti/mcp_server

# Start
docker compose -f docker/docker-compose-neo4j.yml up -d

# Stop
docker compose -f docker/docker-compose-neo4j.yml down

# View logs
docker compose -f docker/docker-compose-neo4j.yml logs -f

# Restart just Graphiti MCP
docker compose -f docker/docker-compose-neo4j.yml restart graphiti-mcp
```

### Configuration

Edit `.env` file in `/Users/pierregallet/local-dev-environment/graphiti/mcp_server/`:
- `NEO4J_URI`, `NEO4J_USER`, `NEO4J_PASSWORD` - Database connection
- `OPENAI_API_KEY` - Your OpenAI key
- `MODEL_NAME` - LLM model to use (default: gpt-4.1-mini)
- `SEMAPHORE_LIMIT` - Concurrency control (default: 10)

After changing `.env`, restart:
```bash
docker compose -f docker/docker-compose-neo4j.yml restart graphiti-mcp
```

## Architecture

```
┌──────────────────┐
│ Claude Desktop   │
│ Gemini, etc      │
└────────┬─────────┘
         │ HTTP
         ↓
┌──────────────────┐
│  Official Zep    │
│  Graphiti MCP    │ ← http://localhost:8000/mcp
│  Server          │
└────────┬─────────┘
         │ Bolt Protocol
         ↓
┌──────────────────┐
│   Neo4j DB       │ ← bolt://localhost:7687
│ (Graph Database) │
└──────────────────┘
```

## Deploying to VPS

1. **Copy to VPS:**
   ```bash
   scp -r /Users/pierregallet/local-dev-environment/graphiti/mcp_server your-vps:/path/to/graphiti
   ```

2. **Update .env on VPS:**
   - Set correct `NEO4J_URI`, `NEO4J_PASSWORD`
   - Add your `OPENAI_API_KEY`

3. **Start on VPS:**
   ```bash
   cd /path/to/graphiti/mcp_server
   docker compose -f docker/docker-compose-neo4j.yml up -d
   ```

4. **Update MCP Clients:**
   Change URL in Claude Desktop config to:
   ```json
   {
     "graphiti-memory": {
       "url": "http://your-vps-ip:8000/mcp"
     }
   }
   ```

## Troubleshooting

### Containers Won't Start
```bash
# Check logs
docker compose -f docker/docker-compose-neo4j.yml logs

# Ensure ports aren't in use
lsof -i :7474
lsof -i :7687
lsof -i :8000
```

### Claude Desktop Not Seeing Tools
1. Verify container is running: `docker ps | grep graphiti-mcp`
2. Test endpoint: `curl http://localhost:8000/health`
3. Restart Claude Desktop completely
4. Check Claude's developer console for errors

### Memory Not Persisting
- Neo4j data is stored in Docker volume `docker_neo4j_data`
- To backup: `docker run --rm -v docker_neo4j_data:/data -v $(pwd):/backup alpine tar czf /backup/neo4j-backup.tar.gz /data`

## Official Documentation

- **GitHub Repo**: https://github.com/getzep/graphiti
- **MCP Server Docs**: https://github.com/getzep/graphiti/blob/main/mcp_server/README.md
- **Graphiti Docs**: https://docs.getzep.com/graphiti

## Advantages of Official Setup

✅ Regular updates from Zep team
✅ Better error handling
✅ HTTP transport (works with more MCP clients)
✅ Production-ready configuration
✅ Official support and documentation

Enjoy your official Zep Graphiti memory system!
