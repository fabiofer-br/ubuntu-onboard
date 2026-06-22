# Ubuntu Onboard — Developer Workstation Guides

A collection of concise, practical guides to set up an Ubuntu 26.04 LTS developer workstation for C#, .NET, Docker, and SQL Server development.

Contents

- [Basic setup](docs/Ubuntu2604-101-basic-setup.md): initial system updates, handy packages, GNOME tweaks, Chrome/VScode/Edge install, power and GRUB settings.
- [Install Docker](docs/Ubuntu2604-102-install-docker.md): official Docker Engine installation, docker-compose plugin, post-install user setup and verification.
- [Additional tools](docs/Ubuntu2604-103-additional-tools.md): recommended dev tools (DBeaver, uv, Zsh/Oh My Zsh, CopyQ) and workflow tips.
- [SQL Server on Linux (Docker)](docs/Ubuntu2604-201-sql-server-linux.md): quick-start `docker run` examples, data persistence, permissions, and connection tips.
- [Dotnet & C# on Linux](docs/Ubuntu2604-202-dotnet-and-csharp.md): install .NET SDK, VS Code integration, scaffold Web API, and connect to Docker-hosted SQL Server.

Quick start

1. Follow the basic system setup: [docs/Ubuntu2604-101-basic-setup.md](docs/Ubuntu2604-101-basic-setup.md)
2. Install Docker using the recommended repository: [docs/Ubuntu2604-102-install-docker.md](docs/Ubuntu2604-102-install-docker.md)
3. Run a local SQL Server container (example):

```bash
docker run -e "ACCEPT_EULA=Y" \
		   -e "MSSQL_SA_PASSWORD=YourStrong@Password123" \
		   -p 1433:1433 \
		   --name mssql2022 \
		   -d mcr.microsoft.com/mssql/server:2022-latest
```

4. Persist SQL data with a bind mount and correct ownership (see the SQL Server guide):

```bash
mkdir -p ~/projects/sql_data
sudo chown -R 10001:0 ~/projects/sql_data
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=YourStrong@Password123" -p 1433:1433 --name mssql2022 -v ~/projects/sql_data:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2022-latest
```

5. Install the .NET SDK and scaffold a project (see Dotnet guide):

```bash
sudo apt update
sudo apt install dotnet-sdk-8.0 -y
dotnet new webapi -n MyBackendApp.Api
```

Usage notes

- Verify Docker with `docker --version` and `docker run hello-world` after installation.
- Add your user to the `docker` group to run Docker without `sudo` (see Docker guide).
- Use `dotnet watch run` during development for hot reload of changes.
- When connecting from `.NET` to SQL Server in Docker, the sample connection string in the docs includes `TrustServerCertificate=True` for local development.

Contributing

Add or improve guides as separate Markdown files under the `docs/` folder and open a pull request. Keep each guide focused and include examples and verification steps.

Resources

- Docs folder: [docs](docs)
- Docker docs: https://docs.docker.com/get-started/
- SQL Server on Linux images: https://mcr.microsoft.com/mssql/server

License

No license is specified for this repository. Add a `LICENSE` file if you want to set terms for reuse.

---
Generated from the contents of the `docs/` folder.
