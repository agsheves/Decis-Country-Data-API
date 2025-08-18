# Decis Intelligence Country Info API
The code repository and API docs for Decis users

*NOTE* - these documents refer to the test version of the **agent** endpoint. The **country information** endpoint is better suited for routine and bulk country data access. [That endpoint is explained here](https://github.com/agsheves/Decis-Country-Data-API)

# Overview of the API

The Decis agent API allows authorized users to access the Decis agent workspace programmatically to make use of the Decis agents in stand-alone applications or via other AI tools. Access to the API endpoint is via **HTTPS GET request** initiated by the user and requires [valid workspace credentials](https://workspace.decis.ai/) for access. 

# Decis  Agent API - Complete Usage Guide

## Overview

The Decis Agent API provides access to multiple specialized AI agents for intelligence analysis, research, and risk assessment. This is a session-based API that requires authentication and provides conversation continuity across requests.

## Quick Start

### 1. Initialize a Session

First, create an authenticated session to get your session credentials.

# ⚠️ This uses your [WORKSPACE credentials](https://workspace.decis.ai/). Do not use your [country endpoint credentials]()

```python
import requests

# Step 1: Initialize session (requires your Decis workspace credentials)
session_url = "https://workspace.decis.ai/_/api/sessionInit"
auth_response = requests.get(session_url, auth=('your-email@domain.com', 'your-password'))


if auth_response.status_code == 200:
    session_data = auth_response.json()
    session_id = session_data['session_id']
    access_token = session_data['access_token']
    available_tools = session_data['tools_available']
    print(f"Session initialized: {session_id}")
else:
    print(f"Authentication failed: {auth_response.status_code}")
```

### 2. Make Agent Requests

```python
def query_agent(session_id, access_token, tool, query, assets=None, query_specifics=None):
    url = "https://workspace.decis.ai/_/api/sessions"
    
    data = {
        "session_id": session_id,
        "access_token": access_token,
        "tool": tool,
        "query": query
    }
    
    if assets:
        data["assets"] = assets
    if query_specifics:
        data["query_specifics"] = query_specifics
    
    response = requests.post(url, data=data)
    return response.json() if response.status_code == 200 else response.text

# Example usage
result = query_agent(
    session_id=session_id,
    access_token=access_token,
    tool="general_chat",
    query="What are the main sections in a risk assessment?"
)
```

## Available Tools/Agents

The ```session initiation ``` call returns a list of the available agents and their descriptions in the ```available_tools``` field. The available agents as at August 2025 are:

| Tool Name | Function | Description | Best For |
|-----------|----------|-------------|----------|
| `general_chat` | General conversation | Standard AI assistant for general queries | Basic questions, explanations |
| `risk_copilot` | Risk analysis assistant | Specialized in risk assessment and management | Risk analysis, threat evaluation |
| `research_mode` | Web search & research | Performs live web searches using Perplexity. Ignores conversation history to send a clean request. | Current events, fact-checking |
| `deep_reasoning` | Advanced reasoning | Enhanced logical analysis and complex problem solving | Complex analytical tasks |
| `country_research` | Country intelligence | Accesses Decis country database and analysis | Country risk, geopolitical analysis |
| `none` | Default (general_chat) | Falls back to general conversation | When unsure which tool to use |

## Special Parameters

### Query Specifics Parameter

Use `query_specifics` to provide additional context:
- **Country codes** for country research
- **Asset lists** for risk analysis (*Not yet available*)
- **Geo coordinates or imagess** for targeted analysis (*Not yet available*)

### Country Research Tool

The `country_research` tool has special behavior based on query content:

- **Clean data requests**: Include "clean" in your query to get raw country data from the Decis database. (Note, the [country api endpoint](https://github.com/agsheves/Decis-Country-Data-API/tree/main) is still preferable for bulk or scheduled data downloads.)
- **Analysis requests**: Any other query performs country analysis using the Decis country info database and returns that payload plus the user query to a deep reasoning model.

```python
# Get clean country data
clean_data = query_agent(
    session_id=session_id,
    access_token=access_token,
    tool="country_research",
    query="clean data request", # ⬅️ Include 'clean' here
    query_specifics="USA,GBR,FRA"  # ISO3 country codes
)

# Get country analysis
analysis = query_agent(
    session_id=session_id,
    access_token=access_token,
    tool="country_research",
    query="What are the main risks in Eastern Europe?", # ⬅️ The  query
    query_specifics="POL,CZE,HUN"  # ⬅️ The specific countries to consider. **These must be included**
)
```

**Note** ⚠️ Country analysis requests must have countries specified. If you want to pose a general question without country data, use a different agent. 

## Complete Python Example

```python
import requests
import json

class DecisAgentAPI:
    def __init__(self, email, password):
        self.base_url = "https://workspace.decis.ai/_/api"
        self.session_id = None
        self.access_token = None
        self.initialize_session(email, password)
    
    def initialize_session(self, email, password):
        """Initialize a new session with Decis API"""
        response = requests.get(
            f"{self.base_url}/sessionInit", 
            auth=(email, password)
        )
        
        if response.status_code == 200:
            data = response.json()
            self.session_id = data['session_id']
            self.access_token = data['access_token']
            print(f"✅ Session initialized: {self.session_id}")
            return data['tools_available']
        else:
            raise Exception(f"Session initialization failed: {response.status_code}")
    
    def query(self, tool, query, assets=None, query_specifics=None):
        """Send a query to the specified agent tool"""
        data = {
            "session_id": self.session_id,
            "access_token": self.access_token,
            "tool": tool,
            "query": query
        }
        
        if assets:
            data["assets"] = assets
        if query_specifics:
            data["query_specifics"] = query_specifics
        
        response = requests.post(f"{self.base_url}/sessions", data=data)
        
        if response.status_code == 200:
            return response.json()
        else:
            return {"error": f"Request failed: {response.status_code} - {response.text}"}

# Usage example
if __name__ == "__main__":
    # Initialize API client
    api = DecisAgentAPI("your-email@domain.com", "your-password")
    
    # Example 1: General conversation
    result1 = api.query("general_chat", "Explain the components of country risk assessment")
    print("General Chat:", result1['response'])
    
    # Example 2: Country research
    result2 = api.query(
        "country_research", 
        "Analyze current stability in the Balkans",
        query_specifics="SRB,BIH,MKD,MNE"
    )
    print("Country Analysis:", result2['response'])
    
    # Example 3: Research mode
    result3 = api.query("research_mode", "Latest developments in EU sanctions policy")
    print("Research:", result3['response'])
```

## Error Handling & Response Codes

### Session Initialization Errors
- `401`: Invalid credentials
- `403`: Access denied

### Agent Query Errors
- `400`: Incomplete request (missing required parameters)
- `403`: Session ID not found, expired session, or safety violation
- `429`: API call limit reached
- `200` with error message: Invalid token or unknown tool

### Safety Filtering

The API includes built-in safety filtering that blocks:
- SQL injection attempts
- Command injection patterns
- Prompt injection attacks
- Requests for system information or credentials

**Warning** ⚠️ Multiple malicious injections will end the session and suspend that account's access privileges. 

## Best Practices

### 1. Session Management
- Initialize once per conversation/workflow
- Reuse session credentials across related queries
- Handle session expiration gracefully

### 2. Tool Selection
- Use `general_chat` for explanatory content
- Use `risk_copilot` for risk-specific analysis
- Use `research_mode` for current/factual information
- Use `country_research` for geopolitical intelligence

### 3. Error Handling
```python
def safe_query(api, tool, query):
    try:
        result = api.query(tool, query)
        if "error" in result:
            print(f"API Error: {result['error']}")
            return None
        return result['response']
    except Exception as e:
        print(f"Request failed: {e}")
        return None
```

### 4. Conversation Context
The API maintains conversation history automatically within a session, so follow-up questions will have context from previous exchanges.

**Note** Future versions will allow a previous session to be restarted, maintaning cnversation history and context

## Security Notes

- **Never share session credentials** - each user should have their own
- **Sessions are logged** with username and IP address
- **Credential sharing may result in access loss**
- **Malicious queries are blocked** and may impact account standing

## Support

For API access requests or technical support:
- Email: [support@decis.ai](mailto:support@decis.ai)
- Subject line: "Agent API Support" 
- Include your organization details and use case

## Rate Limits

The API includes usage limits per billing period. Monitor your usage and contact support if you need increased limits for production applications.
