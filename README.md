# Higress OPS MCP Server


[![smithery badge](https://smithery.ai/badge/@higress-group/higress-ops-mcp-server)](https://smithery.ai/server/@higress-group/higress-ops-mcp-server)

A Model Context Protocol (MCP) server implementation that enables comprehensive configuration and management of [Higress](https://higress.cn/). This repository also provides an MCP client built on top of [LangGraph](https://www.langchain.com/langgraph) and [LangChain MCP Adapters](https://github.com/langchain-ai/langchain-mcp-adapters), facilitating interaction with the Higress MCP Server through a well-designed agent flow architecture.

## Demo

https://github.com/user-attachments/assets/bae66b77-a158-452e-9196-98060bac0df7

## Config Environment Variables

Copy the `.env.example` file to `.env` and fill in the corresponding values.

## Start MCP Client and MCP Server

In stdio mode, the MCP server process is started by the MCP client program. Run the following command to start the MCP client and MCP server:

```python
uv run client.py
```

## Add a new tool

**Step 1: Create a new tool class or extend an existing one**

- Create a new file in the tools directory if adding a completely new tool category
- Or add your tool to an existing class if it fits an existing category

```python
from typing import Dict, List, Any
from fastmcp import FastMCP

class YourTools:
    def register_tools(self, mcp: FastMCP):
        @mcp.tool()
        async def your_tool_function(arg1: str, arg2: int) -> List[Dict]:
            """
            Your tool description.
            
            Args:
                arg1: Description of arg1
                arg2: Description of arg2

            Returns:
                Description of the return value
            
            Raises:
                ValueError: If the request fails
            """
            # Implementation using self.higress_client to make API calls
            return self.higress_client.your_api_method(arg1, arg2)
```


**Step 2: Add a new method to HigressClient if your tool needs to interact with the Higress Console API**

- Add methods to utils/higress_client.py that encapsulate API calls
- Use the existing HTTP methods (get, put, post) for actual API communication


```python
def your_api_method(self, arg1: str, arg2: int) -> List[Dict]:
    """
    Description of what this API method does.
    
    Args:
        arg1: Description of arg1
        arg2: Description of arg2
        
    Returns:
        Response data
        
    Raises:
        ValueError: If the request fails
    """
    path = "/v1/your/api/endpoint"
    data = {"arg1": arg1, "arg2": arg2}
    return self.put(path, data)  # or self.get(path) or self.post(path, data)
```

**Step 3: Register your tool class in the server**

- Add your tool class to the tool_classes list in server.py
- This list is used by ToolsRegister to instantiate and register all tools
- The ToolsRegister will automatically set logger and higress_client attributes

```python
tool_classes = [
    CommonTools,
    RequestBlockTools,
    RouteTools,
    ServiceSourceTools,
    YourTools  # Add your tool class here
]
```

**Step 4: Add your tool to `SENSITIVE_TOOLS` if it requires human confirmation**

- Tools in this list will require human confirmation before execution

```python
# Define write operations that require human confirmation
SENSITIVE_TOOLS = [
    "add_route", 
    "add_service_source",
    "update_route",
    "update_request_block_plugin", 
    "update_service_source",
    "your_tool_function"  # Add your tool name here if it requires confirmation
]
```

