# N8N Cybersecurity Intelligence Automation

An automated cybersecurity intelligence platform built with n8n that aggregates, analyzes, and reports on cybersecurity news and threat intelligence.

## Overview

This project provides a comprehensive n8n workflow that:

1. Collects cybersecurity news from RSS feeds
2. Filters out low-quality or irrelevant content
3. Extracts structured data (CVEs, entities, threats)
4. Enriches findings with threat intelligence using AI research
5. Generates detailed cybersecurity reports
6. Delivers reports via email

The workflow leverages multiple AI models through OpenRouter, integrates with external tools via MCP (Model Context Protocol), and uses Redis for memory management.

### Workflow Diagram

![N8N Cybersecurity Intelligence Workflow](/workflow-diagram.png)
*Visual representation of the n8n workflow showing data flow from RSS feeds through processing, AI research, and report generation*

## Prerequisites

- Docker and Docker Compose
- Exa AI API key (https://exa.ai)
- Mailgun account and API key (for email delivery)
- OpenRouter API key (https://openrouter.ai) - required for all LLM operations
- Git (to clone repositories)

## Setup Instructions

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/n8n-custom.git
cd n8n-custom
```

### 2. Create .env file

Create a `.env` file in the project root with the following variables:

```
# Required for n8n runners authentication
N8N_RUNNERS_AUTH_TOKEN=YOUR_RANDOM_SECRET_TOKEN

# Required for OpenRouter API access (for LLM operations)
OPENROUTER_API_KEY=your_openrouter_api_key
```

> ⚠️ **IMPORTANT**:
> - The `N8N_RUNNERS_AUTH_TOKEN` must be set to a secure random string. This token is used to authenticate the runners to the main n8n container.
> - The `OPENROUTER_API_KEY` is required for all language model operations in the workflow. You can get an API key by signing up at [OpenRouter](https://openrouter.ai).

### 3. Set up additional MCP containers

The workflow relies on several Model Context Protocol (MCP) servers. You need to set up these containers separately:

#### A. MITRE ATT&CK MCP Server

```bash
# Clone the MITRE MCP repository
git clone https://github.com/bradleyjlevine/mcp-mitre.git
cd mcp-mitre

# Build and run the MITRE MCP container
docker build -t mitre:latest .
docker run -p 8099:8099 --restart=unless-stopped -d mitre:latest

# Return to the main directory
cd ..
```

#### B. MarkItDown MCP Server

```bash
# Clone the MarkItDown MCP repository
git clone https://github.com/bradleyjlevine/mcp-markitdown.git
cd mcp-markitdown/docker/embedded

# Build and start the embedded MarkItDown container
docker compose up -d --build

# Return to the main directory
cd ../../..
```

> **Note**: The MarkItDown MCP server has additional dependencies including ffmpeg, poppler-utils, and libmagic1, but these are automatically installed in the Docker container.

#### C. Shodan CVE MCP Server

```bash
# Clone the Shodan CVE MCP repository
git clone https://github.com/bradleyjlevine/shodan_cve.git
cd shodan_cve

# Build and start the Shodan CVE container
docker-compose up -d

# Return to the main directory
cd ..
```

> **Note**: The Shodan CVE MCP server provides access to CVE and CPE databases for vulnerability information without requiring an API key.

### 4. Start the main containers

```bash
# Return to the n8n-custom directory
cd ..
docker-compose up -d
```

### 5. Access n8n

Open your browser and navigate to:

```
http://localhost:5678
```

### 6. Import the workflow

1. Go to the n8n dashboard
2. Click "Workflows" in the sidebar
3. Click "Import from File"
4. Select the `Cybersecurity News.json` file

### 7. Configure the Exa AI API Key

> ⚠️ **IMPORTANT**: You must update the Exa AI MCP node with your API key.

1. Open the imported workflow
2. Find the Exa MCP node (under the CyberNews Researcher section)
3. Edit the URL to include your API key as a parameter:
   ```
   https://mcp.exa.ai/mcp?exaApiKey=YOUR_EXA_API_KEY
   ```

### 8. Configure the Mailgun API Key

> ⚠️ **IMPORTANT**: You must configure the Mailgun node with your own Mailgun account credentials.

1. Sign up for a Mailgun account if you don't have one (https://www.mailgun.com/)
2. Obtain your API key from the Mailgun dashboard
3. In the workflow, locate the Mailgun node
4. Add your Mailgun credentials:
   - API Key
   - Domain name
5. Replace the email placeholders with your actual email addresses:
   - Replace `<YOUR_MAILGUN_FROM_EMAIL>` with your Mailgun sender email address (must be verified in your Mailgun account)
   - Replace `<YOUR_RECIPIENT_EMAIL>` with the email address where you want to receive reports

### 9. Configure the OpenRouter API Key

> ⚠️ **IMPORTANT**: You must configure the OpenAI nodes to use your OpenRouter API key.

1. Make sure you've added your OpenRouter API key to the `.env` file as described earlier
2. In the workflow, locate all OpenAI nodes (used for Claude, Gemini, and Grok models)
3. For each OpenAI node, simply provide your OpenRouter API key
4. If you prefer different language models, you can modify the model parameter in each node

> **Privacy Note**: OpenRouter routes requests to different AI providers. Review and update the 'Provider Restrictions' in your OpenRouter account settings to only allow providers you trust with your data. Some providers collect conversations for their own purposes. See the "OpenRouter Privacy Considerations" section in Maintenance Notes for more details.

## Key Components

- **Data Ingestion**: RSS feed reader to pull cybersecurity news
- **Data Processing**: Filtering, extraction, and quality control
- **Enrichment**: AI-powered research using Exa AI and other MCP tools
- **Analysis**: Multiple language models via OpenRouter:
  - Claude Haiku/Sonnet 4/4.5 for various analysis tasks
  - Gemini 2.5 Flash for additional research
  - Grok 4 Fast for CyberNews researcher agent
- **Reporting**: Comprehensive report generation with structured sections
- **Delivery**: Email distribution via Mailgun

## Docker Services

The project uses several Docker containers:

### Main containers (docker-compose.yml):
1. **n8n**: Main workflow orchestration service
2. **n8n Runners**: Separate container for executing Python and Node.js code
3. **Redis**: Memory database for conversation state

### Additional required containers:
1. **MITRE MCP**: Provides MITRE ATT&CK framework data for threat intelligence
   - Port: 8099
   - Repository: https://github.com/bradleyjlevine/mcp-mitre

2. **MarkItDown MCP**: Fetches and converts web content to markdown format
   - Port: 8085
   - Repository: https://github.com/bradleyjlevine/mcp-markitdown
   - Uses embedded Docker setup with bgutil provider

3. **Shodan CVE MCP**: Provides CVE and CPE database access for vulnerability information
   - Port: 8088
   - Repository: https://github.com/bradleyjlevine/shodan_cve
   - No API key required

## Customization

- Modify the RSS feed URLs to track different sources
- Adjust filtering parameters for different quality thresholds
- Update email addresses in the Mailgun node:
  - `<YOUR_MAILGUN_FROM_EMAIL>` for the sender address
  - `<YOUR_RECIPIENT_EMAIL>` for the recipient address
- Configure different language models in the OpenRouter nodes

## Maintenance Notes

### Container Updates

- The n8n and n8n runners containers are configured to always pull the latest image when using `docker-compose up`
- This ensures you get the latest features and security updates
- Be aware that n8n updates may occasionally require workflow adjustments or container rebuilds
- Use `docker-compose pull` followed by `docker-compose up -d` to update containers

### Language Models

The workflow uses specific models chosen for their capabilities:

- **Information Extraction**: Claude Haiku 4.5
  - Efficient for initial content parsing
  - Good balance of cost and performance

- **MCP AI Tool Nodes**:
  - Claude Haiku 4.5 for general queries
  - Gemini 2.5 Flash for Shodan CVE analysis (handles large token outputs)

- **Researcher Agent and Report Writer**:
  - Claude Sonnet 4.5 for the main researcher agent
  - Claude Sonnet 4.0 for report generation

### Cost Considerations

- Models with 1M+ context windows are required for processing large CVE datasets
- Newer models may have higher costs - users should monitor their OpenRouter usage
- Consider adjusting model selection based on your budget and performance needs
- Use OpenRouter's cost calculator to estimate expenses based on your usage patterns

### OpenRouter Privacy Considerations

> ⚠️ **IMPORTANT**: OpenRouter routes your LLM conversations to different AI providers.

- After creating your OpenRouter account, review and update the 'Provider Restrictions' settings
- Only allow providers you explicitly trust with your data
- Some providers are known to collect conversations for analysis or other uses
- Configure these settings at: https://openrouter.ai/settings/privacy (under "Provider Restrictions")
- Consider restricting to providers with clear data privacy policies for sensitive cybersecurity content

## License

MIT License

## Author

Bradley Levine