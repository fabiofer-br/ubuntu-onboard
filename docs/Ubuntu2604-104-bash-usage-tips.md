# Bash CLI usage - Tips and Tricks

## Creating a Bash Alias 

If there is a sequence of just two commands that you want to fire off quickly (like updating the system), a script might be overkill. 
You can create an _**alias**_ for it.

An _**alias**_ acts as a text replacement: when you type the alias, Bash instantly replaces it with the full command string before hitting Enter.

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
<br/><br/>

---

## Creating Shell Scripts

In Linux, you don't need a specific file extension like `.bat` or `.bat` to create a _**Shell Script**_. 
Instead, the system looks at two things: a special first line of text inside the file (called _"the Shebang"_), and the file's execution permissions (using the `chmod` command).

To make your script accessible from *anywhere* in your folder structure, you must save it inside a directory that is registered in your system's `$PATH`. The standard folder for custom, user-created commands is `/usr/local/bin`.

1. **Create the file:** Use a text editor like nano.
Decide on the word you want to type in the terminal to run your sequence. Let's call it `start-dev`. 
Open a new file in the global binaries folder:
```bash
sudo nano /usr/local/bin/start-dev
```

2. **Add the Shebang and your commands:** Tell Linux what language to use.
The very first line of your file MUST be `#!/bin/bash`. 
This tells the system to interpret the following lines using the Bash shell.

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

3. **Make the file executable:** Apply the execute permission.
By default, Linux creates text files as read/write only. You must explicitly tell the operating system that this text file is actually a program.
```bash
sudo chmod +x /usr/local/bin/start-dev
```

Now, no matter what folder you are currently in, you can simply type `start-dev` in your terminal, and it will execute that exact sequence of commands.

---

### Checking sudo `root` user context inside Shell Scripts

If you want to ensure that a script is running as a `root` (and exit if it is not), you can simply put the following clause at the very beginning of the script:
```bash
#!/bin/bash

if [ "$EUID" -ne 0 ]; then
  echo "Please run as root"
  exit
fi

# The rest of script code follow here...
```

---

### Changing sudo user context inside Shell Scripts

A script started with:

```bash
sudo ./script.sh
```
runs entirely as `root`. 
To run a command as the original user inside it:
```bash
sudo -u "$SUDO_USER" -- command
```

For example:
```bash
sudo -u "$SUDO_USER" -- touch "$SUDO_USER_HOME/file.txt"
```

You can determine the original user and home directory with:
```bash
original_user="${SUDO_USER:-$USER}"
original_home=$(getent passwd "$original_user" | cut -d: -f6)
```

If you entered an interactive root shell with `sudo -i` or `sudo -s`, use:
```bash
exit
```

A script cannot “return” the parent shell to the original user. `exit` only terminates the script or current shell; after the script finishes, your calling shell remains the original user automatically.

---

### Scripts that run automatically (_"Shell Startup Files"_)

* `~/.bashrc`  
   Runs for interactive, non-login Bash sessions, such as a typical new terminal tab. _**Affects only the current user**_.  
   It is commonly used for aliases, shell functions, and interactive settings.
* `~/.bash_profile`, `~/.bash_login`, and `~/.profile`  
  These are user login-shell files. _**Affect only the current user**_.  
  Important: Bash reads only the _first one it finds_, in _that order_. Ubuntu commonly uses the last one (`~/.profile`): it is a good place for login-session environment settings. 
* `/etc/bash.bashrc`  
  Provides system-wide settings for interactive Bash sessions. _**Affects all users**_. 
* `/etc/profile` 
  This is the system-wide login-shell file and typically loads scripts in `/etc/profile.d/`. Those scripts configure login sessions _**for all users in the system**_.

In short, use `~/.bashrc` for interactive shell behavior and `~/.profile` for settings needed at login.  
A terminal can start a shell as either a _login_ or _non-login_ shell, so which files run depends on its configuration.


<br/>  <br/>

---
