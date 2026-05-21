# Plan: Install Docker & docker-compose on Ubuntu 22.04

## TL;DR
Install Docker Engine and docker-compose using Ubuntu's official Docker repository via terminal commands. This approach is more maintainable than older `docker.io` package. Includes user group configuration for rootless operation.

## Steps

### Phase 1: Preparation
1. Update system package index
   ```bash
   sudo apt update
   ```
2. Install prerequisite packages
   ```bash
   sudo apt install -y util-linux-extra
   sudo apt install -y curl ca-certificates gnupg lsb-release
   ```

### Phase 2: Add Docker Repository
3. Create `/etc/apt/keyrings` directory
   ```bash
   sudo mkdir -p /etc/apt/keyrings
   ```
4. Download and add Docker's GPG key to system keyring
   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg

   ```
5. Add Docker's official repository to APT sources
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

### Phase 3: Install Docker
6. Update package index again
   ```bash
   sudo apt update
   ```
7. Install Docker packages
   ```bash
   sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

### Phase 4: Post-Installation Setup
8. Start and enable Docker service
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```
9. Create `docker` user group (safe to run even if it exists)
   ```bash
   sudo groupadd docker
   ```
10. Add current user to `docker` group
    ```bash
    sudo usermod -aG docker $USER
    ```
11. Apply new group membership (choose one method)
    ```bash
    # Option A: Use newgrp command (applies to current session only)
    newgrp docker
    
    # Option B: Log out and log back in (applies permanently)
    exit
    ```

### Phase 5: Verification
12. Verify Docker installation
    ```bash
    docker --version
    ```
13. Verify docker-compose installation
    ```bash
    docker compose version
    ```
14. Test Docker daemon
    ```bash
    docker run hello-world
    ```
    You should see:  
    ```bash
    Hello from Docker!
    This message shows that your installation appears to be working correctly.

    To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
        (amd64)
    3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.

    To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash

    Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/

    For more examples and ideas, visit:
    https://docs.docker.com/get-started/
    ```

## Key Details
- **Ubuntu version**: 26.04 LTS (from user profile)
- **Installation method**: Official Docker repository (recommended, more secure than PPA)
- **docker-compose**: Installing via `docker-compose-plugin` package (modern approach)
- **User permissions**: Adding user to `docker` group for rootless command execution
- **Service management**: systemd integration for auto-start capability

## Verification Checklist
- [ ] `docker --version` shows Docker version 24.x or later
- [ ] `docker compose version` confirms compose is available
- [ ] `docker run hello-world` executes without sudo errors
- [ ] User can run docker commands without sudo prefix

## Decisions & Scope
- **Included**: Full Docker Engine + docker-compose (via plugin)
- **Excluded**: Docker Desktop (vs Engine), rootless Docker mode (optional later), Docker in WSL
- **Assumption**: User has sudo privileges (typical for development workstation)
