# FastMCP Azure App Service Template

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the information here.

This is a minimal FastMCP (Fast Model Context Protocol) server application designed for deployment on Azure App Service. It provides a simple MCP server with tools and resources accessible via Server-Sent Events (SSE).

## Working Effectively

### Initial Setup
- Install Python dependencies: `pip install -r requirements.txt` -- takes under 1 minute. NEVER CANCEL.
- The application requires Python 3.11+ (tested with Python 3.12.3).
- No additional build steps or compilation required.
- **No existing test infrastructure** - create tests when adding new functionality.

### Running the Application
- **Local Development**: `cd src && python -m uvicorn app:app --host 0.0.0.0 --port 8000` -- starts in under 5 seconds. NEVER CANCEL.
- **MCP Development**: `cd src && mcp dev app.py` -- may take 2+ minutes to install npm inspector. NEVER CANCEL. Set timeout to 5+ minutes.
  - Provides local inspector at http://127.0.0.1:6274 with authentication token
  - Use inspector to interactively test tools and resources
- **MCP Server**: `cd src && mcp run app.py` -- starts immediately.

### Azure Deployment
- **Prerequisites**: Install Azure Developer CLI (azd).
- **Initialize**: `azd init` -- prompts for environment name.
- **Login**: `azd auth login` -- opens browser for authentication.
- **Deploy**: `azd up` -- takes 5-15 minutes depending on Azure provisioning. NEVER CANCEL. Set timeout to 30+ minutes.
- **Update**: `azd deploy` -- takes 2-5 minutes for code updates only.

## Validation

### Functional Testing
- ALWAYS test that the Python app imports successfully: `cd src && python -c "import app; print('FastMCP app imported successfully')"`
- ALWAYS verify the server starts: Start uvicorn and confirm "Application startup complete" message appears.
- Test MCP client connection using any MCP-compatible client at the `/sse` endpoint.
- The application provides:
  - `add` tool: Adds two integers
  - `greeting://{name}` resource: Returns personalized greeting

### End-to-End Scenario Testing
- Start the server locally with uvicorn.
- Verify the server responds on http://localhost:8000 (root returns 404, this is expected).
- For MCP testing, use `mcp dev app.py` to start the inspector.
- Open the provided inspector URL (http://127.0.0.1:6274) with the authentication token.
- Test the `add` tool by calling it with two numbers in the inspector.
- Test the greeting resource by accessing `greeting://YourName` in the inspector.
- Connect with an MCP client to http://localhost:8000/sse for production testing.

### Azure Validation
- After deployment, verify the application is accessible at the Azure App Service URL.
- Test MCP client connectivity to `https://yourapp.azurewebsites.net/sse`.
- Confirm logs are available in Azure portal.

## Common Tasks

### Development Workflow
- Make changes to `src/app.py` to add tools or resources.
- Test locally with `python -m uvicorn app:app --host 0.0.0.0 --port 8000`.
- Validate with MCP client or `mcp dev app.py` for interactive debugging.
- **No linting tools configured** - add flake8/black/pylint if code quality checks needed.
- Deploy changes with `azd deploy`.

### Adding MCP Tools
- Use `@mcp.tool()` decorator to add new tools.
- Tools must have type hints and docstrings.
- Test locally before deploying.

### Adding MCP Resources
- Use `@mcp.resource()` decorator with URI patterns.
- Resources provide read-only data to MCP clients.
- Test resource URIs match expected patterns.

## Repository Structure

### Key Files
```
├── src/app.py              # Main FastMCP application
├── requirements.txt        # Python dependencies
├── requirements.in         # Dependency source file
├── azure.yaml             # Azure Developer CLI configuration
├── infra/                 # Azure Bicep infrastructure templates
│   ├── main.bicep         # Main infrastructure template
│   └── resources.bicep    # App Service resources
├── README.md              # Project documentation
└── CONTRIBUTING.md        # Contribution guidelines
```

### Dependencies
- **Core**: mcp[cli], uvicorn, gunicorn
- **Web**: starlette, httpx, sse-starlette
- **Utilities**: pydantic, typer, rich
- **Azure**: Configured for Python 3.11 runtime

## Infrastructure

### Azure Resources
- **App Service Plan**: S1 tier (Standard)
- **Web App**: Linux-based Python 3.11
- **Configuration**: 
  - Command: `python -m uvicorn app:app --host 0.0.0.0 --port 8000`
  - HTTPS only enabled
  - Build during deployment enabled

### Local vs Azure Differences
- Local: Direct uvicorn execution
- Azure: Uses the configured app command in Bicep template
- Azure: F1 (free) tier has CPU/RAM limitations (noted in README)

## Troubleshooting

### Common Issues
- **Import errors**: Ensure `pip install -r requirements.txt` completed successfully.
- **Port conflicts**: Default port 8000, change if needed.
- **Azure deployment failures**: Check `azd auth login` status and subscription permissions.
- **MCP client connection**: Verify the `/sse` endpoint is accessible and responds to MCP protocol.

### Debugging Steps
1. Test basic Python import: `python -c "import app"`
2. Start server and check logs for startup messages
3. Test with curl: `curl http://localhost:8000/` (should return "Not Found" for root - this is expected behavior)
4. For MCP testing: `curl http://127.0.0.1:6274` (should return HTML when inspector is running)
5. Use `mcp dev app.py` for interactive MCP debugging with web inspector

## Performance Expectations

### Timing Guidelines
- **Dependency Installation**: Under 1 minute
- **Application Startup**: Under 5 seconds  
- **MCP Dev Setup**: 2-5 minutes (installs npm dependencies)
- **Azure Deployment**: 5-30 minutes (infrastructure + code)
- **Code-only Updates**: 2-5 minutes

### Resource Usage
- **Local**: Minimal resource usage, single process
- **Azure F1**: Limited CPU/RAM, suitable for development/testing
- **Azure S1**: Production-ready with better performance

NEVER CANCEL long-running commands. Azure deployments and MCP dev setup require patience.