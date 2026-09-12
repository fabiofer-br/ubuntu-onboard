# Bash CLI usage - Tips and Tricks



Because you are coming from a strong development background, the easiest way to think about this is mapping it to Windows concepts. In Linux, you have two primary ways to create these shortcuts, depending on how complex your sequence is:

1. **The Shell Script (The `.bat` or `.cmd` equivalent):** Best for multi-line sequences, logic, and variables.
2. **The Bash Alias (The macro equivalent):** Best for chaining two or three simple commands together on a single line.

---

## Creating Shell Scripts (command sequences)

In Linux, you don't need a specific file extension like `.bat`. Instead, the system looks at two things: a special first line of text inside the file (called the "shebang") and the file's execution permissions (using the `chmod` command you learned earlier).

To make your script accessible from *anywhere* in your folder structure, you must save it inside a directory that is registered in your system's `$PATH`. The standard folder for custom, user-created commands is `/usr/local/bin`.

1. **Create the file:** Use a text editor like nano.
Decide on the word you want to type in the terminal to run your sequence. Let's call it `start-dev`. Open a new file in the global binaries folder:

```bash
sudo nano /usr/local/bin/start-dev

```


2. **Add the Shebang and your commands:** Telling Linux what language to use.
The very first line of your file MUST be `#!/bin/bash`. This tells the system to interpret the following lines using the Bash shell.

Add your sequence below it. For example:

```bash
#!/bin/bash

echo "Starting development environment..."

# 1. Start the SQL container
docker start mssql2022

# 2. Navigate to your project folder
cd /home/username/projects/MyBackendApp

# 3. Open VS Code in that folder
code .

echo "Environment ready!"

```

*(To save and exit nano: Press `Ctrl+O`, `Enter`, then `Ctrl+X`)*


3. **Make the file executable:** Applying the execute permission.
By default, Linux creates text files as read/write only. You must explicitly tell the operating system that this text file is actually a program.

```bash
sudo chmod +x /usr/local/bin/start-dev

```


Now, no matter what folder you are currently in, you can simply type `start-dev` in your terminal, and it will execute that exact sequence of commands.

---

## Method 2: The Bash Alias (Best for simple one-liners)

If your sequence is just two commands that you want to fire off quickly (like updating the system), a script might be overkill. You can create an **alias** instead.

An alias acts as a text replacement. When you type the alias, Bash instantly replaces it with the full command string before hitting Enter.

1. Open your user's Bash configuration file:
```bash
nano ~/.bashrc

```


2. Scroll to the very bottom and add your alias. To chain commands together, use `&&` (this means "run the second command only if the first one succeeds").
```bash
alias sys-update='sudo apt update && sudo apt upgrade -y'

```


3. Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).
4. Reload your configuration so the terminal learns the new alias:
```bash
source ~/.bashrc

```



Now, typing `sys-update` will automatically run the full update sequence.

### Summary: Which should you use?

* Use an **Alias** in `~/.bashrc` if you are just passing long arguments to a single command, or stringing together a simple `command A && command B`.
* Use a **Shell Script** in `/usr/local/bin` if your sequence spans multiple lines, requires comments to explain what it does, or needs to accept dynamic arguments (like `start-dev python` vs `start-dev csharp`).