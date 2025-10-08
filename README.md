## Part 1: Setting Up Your Azure Functions Project

### Step 1: Create a New Azure Functions Project

In Visual Studio Code, open the Command Palette and select "Azure Functions: Create New Project".

On the "Create new project" panel, choose the already highlighted current directory.

### Step 2: Select the Language

Choose **C#** as the programming language.

### Step 3: Select the .NET Version

Choose the latest version of .NET. At the time of writing, this is **".NET 9.0 Isolated"**.

### Step 4: Choose the Template

Select the **"HTTP trigger"** template to keep things simple.

### Step 5: Name Your Function

In the "Create new HTTP trigger" panel, provide a name for the Azure Function class file. For example: `HttpMCP`.

### Step 6: Provide a Namespace

Enter a namespace for your project, such as `Mcp.Function`.

### Step 7: Set Authorization Level

Choose **Anonymous** to keep the setup simple.

### Step 8: Verify the Basic Setup

To make sure that your basic Azure Function works as expected, type this command in the root of your project:

```bash
func start
```

This should show you output with a local URL (e.g., `http://localhost:7071/api/HttpMCP`).

If you point your browser to the given URL, you will see a welcome message confirming the function is working.

Press `CTRL + C` to stop the application.