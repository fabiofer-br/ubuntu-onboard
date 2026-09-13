
Use a credential helper backed by Ubuntu’s keyring, and keep the GitHub username in each remote URL.

git config list --show-scope
git config list --show-scope --show-origin

---

Configre it once:

```bash
sudo apt install git-credential-libsecret

git config --global credential.helper /usr/lib/git-core/git-credential-libsecret
git config --global credential.useHttpPath true
```

The `useHttpPath` setting makes Git include the repository path when looking up credentials. Your remotes can remain in this form:

```text
https://fabiofer-br@github.com/fabiofer-br/repository.git
https://potencia40@github.com/potencia40/repository.git
https://w510664@github.com/w510664/repository.git
```

The first push for each account will ask for its PAT. Select the matching username and enter that account’s PAT. Ubuntu’s keyring will store the credentials and Git will reuse them afterward.

Check the configuration:

```bash
git config --global --get-all credential.helper
git config --global --get credential.useHttpPath
```

Avoid putting PATs directly into remote URLs, because they can appear in shell history and configuration files. If Git keeps using an old credential, remove the saved GitHub entries from **Passwords and Keys** or clear one with:

```bash
printf "protocol=https\nhost=github.com\nusername=fabiofer-br\n\n" | git credential reject
```

Repeat with `potencia40` or `w510664` if necessary.

---

I’m checking which package provides `git-credential-libsecret` on this Ubuntu installation, then I’ll give you the exact install/configuration commands for the available package.

Ran terminal command:  apt-cache search git-credential && apt-cache policy git libsecret-1-0 libsecret-1-dev libsecret-tools

The package name is not present in your enabled Ubuntu 26.04 repositories, so the earlier command is not applicable to this system. I found Ubuntu’s packaged `git-credential-oauth`, which is a better fit for GitHub and avoids storing PATs in plaintext; I’m verifying its behavior before recommending the replacement.

Ran terminal command:  apt-cache show git-credential-oauth | sed -n '1,80p'

Ran terminal command:  command -v git-credential-libsecret || true; command -v git-credential-oauth || true; git config --global --get-all credential.helper || true

The package lookup confirms the exact cause: Ubuntu 26.04 has no `git-credential-libsecret` package in the configured repositories, while `libsecret-tools` is a different utility and does not provide that Git helper. For GitHub, the practical packaged replacement is OAuth-based authentication; I’m checking whether a packaged credential manager is also available for your PAT-specific workflow.

Ran terminal command:  apt-cache search '^git-credential|credential-manager'

Ran terminal command:  apt-cache show git-credential-oauth | grep -E '^(Package|Description-en|Recommends|Depends):'

That error is expected on your Ubuntu 26.04 installation: `git-credential-libsecret` is not available in the Ubuntu repositories you are using. `libsecret-tools` is a different package and does not provide that Git helper.

For GitHub, use Ubuntu’s packaged OAuth helper instead:

```bash
sudo apt update
sudo apt install git-credential-oauth
```

Configure Git to use it:

```bash
git config --global --unset-all credential.helper 2>/dev/null || true
git config --global credential.helper oauth
git config --global credential.useHttpPath true
```

Then push:

```bash
git push
```

The first time, it should open GitHub authentication in your browser. After authorization, Git reuses the credential without asking for your PAT each time.

Keep your remotes in this form so Git can distinguish the accounts:

```text
https://fabiofer-br@github.com/fabiofer-br/repository.git
https://potencia40@github.com/potencia40/repository.git
https://w510664@github.com/w510664/repository.git
```

The earlier `git-credential-libsecret` recommendation was incorrect for this Ubuntu release. Do not install `libsecret-tools` as a substitute; it does not include `git-credential-libsecret`.

---

cat ~/.git-credentials

```bash
git config --global --unset-all credential.helper 2>/dev/null || true

git config --global credential.helper store
git config --global credential.useHttpPath true
```

git config --global --get credential.helper

cat ~/.git-credentials


Your configuration is correct as written. You only need `useHttpPath` once in the general `[credential]` section:

```ini
[credential]
    helper = storecat
    useHttpPath = true
```

It applies globally to all credential sections. These sections only need to define the username for each URL prefix:

```ini
[credential "https://github.com/fabiofer-br"]
    username = fabiofer-br
```

Git matches the most specific applicable credential section. With `useHttpPath = true`, it includes the repository path when looking up credentials, so credentials for different repositories or GitHub accounts can remain separate.

You can verify the effective configuration for a repository with:

```bash
git config --show-origin --get-regexp '^credential'
```

One important detail: because you use `helper = store`, PATs are saved unencrypted in `~/.git-credentials`. Ensure that file has restrictive permissions:

```bash
chmod 600 ~/.git-credentials
```

