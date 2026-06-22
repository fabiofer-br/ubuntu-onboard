# The "C# and .NET" on Linux is an incredibly performant stack

Especially when paired with a containerized SQL Server. 

## How to set up C# and .NET in Linux

Here is how to set up your C# environment on Ubuntu 26.04 and link it to your Docker database.

---

### Step 1: Install the .NET 8 / 9 SDK

Ubuntu and Microsoft collaborate closely to provide .NET packages directly through the native `apt` repositories. To install the SDK (which includes the runtime, compiler, and the `dotnet` CLI tool), run:

```bash
sudo apt update
sudo apt install dotnet-sdk-8.0 -y

```

*(Note: If your project requirements dictate the latest cutting-edge features, you can swap `8.0` for `9.0`).*

Verify the installation by checking the version:

```bash
dotnet --version

```

---

### Step 2: Supercharge VS Code for C#

Since your VS Code installation is a native `.deb` package, extension integration will be flawless. Open VS Code and install the official bundle:

* Search for **"C# Dev Kit"** by Microsoft in the Extensions marketplace (`Ctrl+Shift+X`).
* This kit automatically pulls in the C# language engine, debugging tools, and a structured "Solution Explorer" view that makes managing multi-project web APIs feel exactly like Visual Studio on Windows.

---

### Step 3: Create Your Backend Project

You can scaffold a modern, lightweight Web API directly from your new Zsh terminal. Let's create a clean project structure:

```bash
# Create a workspace directory
mkdir -p ~/projects/MyBackendApp
cd ~/projects/MyBackendApp

# Scaffold a new Web API project using Controller-less Minimal APIs
dotnet new webapi -n MyBackendApp.Api

# Open it inside your freshly working VS Code
code .

```

---

### Step 4: Connecting C# to your Docker SQL Server

To talk to your SQL Server container, you will need to add the Microsoft SQL client package to your C# project. Run this inside your project folder:

```bash
dotnet add package Microsoft.Data.SqlClient

```

#### The Connection String

Because your Docker container maps port `1433` directly to your localhost, your C# application connects to it just like a local database. Add this connection string to your `appsettings.json` file:

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost,1433;Database=MyDatabase;User Id=sa;Password=YourStrong@Password123;TrustServerCertificate=True;"
}

```

> **Why `TrustServerCertificate=True` matters:** Modern versions of `Microsoft.Data.SqlClient` strictly enforce encrypted connections by default. Since your Docker SQL Server relies on a self-signed development certificate, omitting this flag will cause your C# backend to reject the database connection.

---

### 💡 Workflow Pro-Tip: Hot Reloading

When developing backend endpoints, you don't need to manually stop and restart your application every time you edit a C# file. Instead, use the built-in file watcher inside your terminal:

```bash
dotnet watch run --project MyBackendApp.Api

```

This compiles your code in milliseconds in the background and applies updates immediately when you save a file, keeping your terminal and development loop completely seamless.

Would you like a minimal code snippet demonstrating how to execute a fast database connection check inside your new C# project?