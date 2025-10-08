# benb-mcp-azure
Bits &amp; Bytes MCP Azure tutorial

# Build and Deploy a Remote MCP Server with Azure Functions

## Workshop Overview

In this hands-on tutorial, you'll learn how to create, test, and deploy MCP (Model Context Protocol) servers using Microsoft Azure Functions with C#. By the end of this session, you'll have a working remote MCP server deployed to Azure.

---

## Prerequisites

Before starting this tutorial, ensure you have:

- Azure CLI (`az`) installed
- .NET SDK installed
- Azure Functions Core Tools
- Visual Studio Code with GitHub Copilot
- An active Azure subscription

---

## Part 1: Setting Up Your Azure Functions Project

### Step 1: Create a New Azure Functions Project

Open your terminal and run the following command to create a new Azure Functions project:

```bash
func init --worker-runtime dotnet-isolated
```

### Step 2: Verify the Basic Setup

Test that your Azure Function works as expected by running:

```bash
func start
```

You should see output displaying a local URL. Open your browser and navigate to the provided URL to see a welcome message.

Press `CTRL + C` to stop the application.

---

## Part 2: Adding MCP Capabilities

### Step 3: Install the MCP Package

Add the MCP extension package to your project:

```bash
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.Mcp --prerelease
```

> **Note:** This package is currently experimental and in pre-release mode.

---

## Part 3: Creating MCP Tools

You'll create three MCP server functions with the following capabilities:

| Tool Name | Purpose | Parameters |
|-----------|---------|------------|
| HelloWorldTool | Displays 'Hello World' | None |
| ReverseMessageTool | Reverses text | string |
| MultiplyNumbersTool | Multiplies two numbers | number, number |

### Step 4: Create the Tool Definitions

Create a new file named `ToolDefinitions.cs` and add the following code:

```csharp
public static class ToolDefinitions {
  public static class HelloWorldTool {
    public const string Name = "HelloWorldTool";
    public const string Description = "A simple tool that says: Hello World!";
  }

  public static class ReverseMessageTool {
    public const string Name = "ReverseMessageTool";
    public const string Description = "Echoes back message in reverse.";

    public static class Param {
      public const string Name = "Message";
      public const string Description = "The Message to reverse";
    }
  }

  public static class MultiplyNumbersTool {
    public const string Name = "MultiplyNumbersTool";
    public const string Description = "A tool that shows parameter usage by asking for two numbers and multiplying them together";

    public static class Param1 {
      public const string Name = "FirstNumber";
      public const string Description = "The first number to multiply";
    }

    public static class Param2 {
      public const string Name = "SecondNumber";
      public const string Description = "The second number to multiply";
    }
  }

  public static class DataTypes {
    public const string Number = "number";
    public const string String = "string";
  }
}
```

### Step 5: Create the HelloWorldTool

Create a new file named `HelloWorldMcpTool.cs`:

```csharp
public class HelloWorldMcpTool {
  [Function("HelloWorldMcpTool")]
  public IActionResult Run(
    [McpToolTrigger(ToolDefinitions.HelloWorldTool.Name, ToolDefinitions.HelloWorldTool.Description)]
    ToolInvocationContext context
  ) {
      return new OkObjectResult($"Hi. I am {ToolDefinitions.HelloWorldTool.Name} and my message is 'HELLO WORLD!'");
  }
}
```

### Step 6: Create the ReverseMessageTool

Create a new file named `ReverseMessageMcpTool.cs`:

```csharp
public class ReverseMessageMcpTool {
  [Function("ReverseMessageMcpTool")]
  public IActionResult Run(
    [McpToolTrigger(ToolDefinitions.ReverseMessageTool.Name, ToolDefinitions.ReverseMessageTool.Description)]
    ToolInvocationContext context,
    [McpToolProperty(ToolDefinitions.ReverseMessageTool.Param.Name, ToolDefinitions.DataTypes.String, ToolDefinitions.ReverseMessageTool.Param.Description)]
    string message
  ) {
    string reversedMessage = new string(message.ToCharArray().Reverse().ToArray());
    return new OkObjectResult($"Hi. I'm {ToolDefinitions.ReverseMessageTool.Name}!. The reversed message is: {reversedMessage}");
  }
}
```

### Step 7: Create the MultiplyNumbersTool

Create a new file named `MultiplyNumbersMcpTool.cs`:

```csharp
public class MultiplyNumbersMcpTool {
  [Function("MultiplyNumbersMcpTool")]
  public IActionResult Run(
    [McpToolTrigger(ToolDefinitions.MultiplyNumbersTool.Name, ToolDefinitions.MultiplyNumbersTool.Description)]
    ToolInvocationContext context,
    [McpToolProperty(ToolDefinitions.MultiplyNumbersTool.Param1.Name, ToolDefinitions.DataTypes.Number, ToolDefinitions.MultiplyNumbersTool.Param1.Description)]
    int firstNumber,
    [McpToolProperty(ToolDefinitions.MultiplyNumbersTool.Param2.Name, ToolDefinitions.DataTypes.Number, ToolDefinitions.MultiplyNumbersTool.Param2.Description)]
    int secondNumber) {
    return new OkObjectResult($"Hi. I am {ToolDefinitions.MultiplyNumbersTool.Name}!. The result of {firstNumber} multiplied by {secondNumber} is: {firstNumber * secondNumber}");
  }
}
```

---

## Part 4: Configuring the MCP Tools

### Step 8: Update Program.cs

Open `Program.cs` and add the following code **before** the `builder.Build().Run();` line:

```csharp
builder.EnableMcpToolMetadata();

builder.ConfigureMcpTool(ToolDefinitions.HelloWorldTool.Name);

builder.ConfigureMcpTool(ToolDefinitions.ReverseMessageTool.Name)
  .WithProperty(ToolDefinitions.ReverseMessageTool.Param.Name, ToolDefinitions.DataTypes.String, ToolDefinitions.ReverseMessageTool.Param.Description);

builder.ConfigureMcpTool(ToolDefinitions.MultiplyNumbersTool.Name)
  .WithProperty(ToolDefinitions.MultiplyNumbersTool.Param1.Name, ToolDefinitions.DataTypes.Number, ToolDefinitions.MultiplyNumbersTool.Param1.Description)
  .WithProperty(ToolDefinitions.MultiplyNumbersTool.Param2.Name, ToolDefinitions.DataTypes.Number, ToolDefinitions.MultiplyNumbersTool.Param2.Description);
```

### Step 9: Configure Local Settings

Open `local.settings.json` and make the following changes:

1. Set `AzureWebJobsStorage` to `"UseDevelopmentStorage=true"` for local development
2. Add `"AzureWebJobsSecretStorageType": "Files"` inside the `"Values"` section

Your `local.settings.json` should look like this:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
    "AzureWebJobsSecretStorageType": "Files"
  }
}
```

---

## Part 5: Testing Locally

### Step 10: Start the Azure Functions App

Run the following command:

```bash
func start
```

Note the **MCP server SSE endpoint** displayed in the output (e.g., `http://localhost:7071/runtime/webhooks/mcp/sse`). You'll need this in the next step.

### Step 11: Configure the MCP Server in VS Code

1. Open the VS Code Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`)
2. Select **"MCP: Add Server..."**
3. Choose **"HTTP (HTTP or Server-Sent Events)"**
4. Paste your MCP server SSE endpoint (e.g., `http://localhost:7071/runtime/webhooks/mcp/sse`)
5. Give your MCP server a name (or accept the default)
6. Select **"Workspace Settings"** to save the configuration in `.vscode/mcp.json`

### Step 12: Start the MCP Server

In the `.vscode/mcp.json` file that opens, click **Start** to start the MCP server.

### Step 13: Test the Tools

1. Open the **GitHub Copilot Chat** panel in VS Code
2. Click on the **Agent** dropdown and select any Claude model
3. Click on the **tools icon** to see your three tools listed

#### Test HelloWorldTool

Enter the following prompt:

```
Call the HelloWorldTool.
```

Click **Continue** when prompted.

#### Test ReverseMessageTool

Enter the following prompt:

```
Reverse the message: MCP is very cool
```

#### Test MultiplyNumbersTool

Enter the following prompt:

```
Multiply 100 and 89.
```

---

## Part 6: Deploying to Azure

### Step 14: Login to Azure

```bash
az login
```

### Step 15: Create a Resource Group

```bash
az group create --name rg-mcp-func-server --location eastus
```

### Step 16: Create a Storage Account

```bash
az storage account create --name mcpstoreacc --resource-group rg-mcp-func-server --location eastus --sku Standard_LRS
```

### Step 17: Create a Function App

```bash
az functionapp create --resource-group rg-mcp-func-server --consumption-plan-location eastus --runtime dotnet-isolated --functions-version 4 --name mcp-func-app --storage-account mcpstoreacc
```

### Step 18: Deploy the Function App

```bash
func azure functionapp publish mcp-func-app
```

Upon successful deployment, note the endpoint URL displayed in your terminal.

---

## Part 7: Configuring the Remote MCP Server

### Step 19: Get the Azure Function URL

1. Login to the [Azure Portal](https://portal.azure.com)
2. Search for your resource group: `rg-mcp-func-server`
3. Click on the function app: `mcp-func-app`
4. Copy the **Default Domain** (e.g., `mcp-func-app.azurewebsites.net`)

### Step 20: Update mcp.json with the Remote URL

Open `.vscode/mcp.json` and replace the local URL with your Azure URL:

```json
{
    "servers": {
        "my-mcp-server-c5a639d4": {
            "url": "https://mcp-func-app.azurewebsites.net/runtime/webhooks/mcp/sse"
        }
    }
}
```

### Step 21: Get the MCP Extension Key from Azure

1. In the Azure Portal, navigate to your function app
2. Click on **"App keys"** in the left navigation
3. Copy the **mcp_extension** key

### Step 22: Configure Authentication

Update `.vscode/mcp.json` to prompt for the extension key:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "functions-mcp-extension-system-key",
      "description": "Azure Functions MCP Extension System Key",
      "password": true
    }
  ],
  "servers": {
    "my-mcp-server-c5a639d4": {
      "url": "https://mcp-func-app.azurewebsites.net/runtime/webhooks/mcp/sse",
      "headers": {
        "x-functions-key": "${input:functions-mcp-extension-system-key}"
      }
    }
  }
}
```

### Step 23: Test the Remote Server

1. Start the server in VS Code
2. When prompted, enter the `mcp_extension` key from Azure
3. Test with a prompt:

```
Multiply 11 and 22 and provide the answer.
```

You should receive a response from your remotely deployed MCP server!

---

## Summary

Congratulations! You have successfully:

- Created an Azure Functions project
- Added MCP capabilities with three custom tools
- Tested the MCP server locally
- Deployed the MCP server to Azure
- Configured and tested the remote MCP server

This foundation enables you to build more sophisticated MCP servers for large-scale implementations.

---

## Next Steps

- Explore additional MCP tool capabilities
- Implement more complex business logic
- Add error handling and logging
- Secure your endpoints with proper authentication
- Scale your Azure Functions deployment

---

## Resources

- [Azure Functions Documentation](https://docs.microsoft.com/azure/azure-functions/)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [GitHub Copilot Documentation](https://docs.github.com/copilot)
