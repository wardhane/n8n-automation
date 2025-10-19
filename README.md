# n8n-automation

## Setup

We use lima and nerdctl as an alternative to Docker. Lima provides lightweight Linux VMs and a convenient environment to run containerd-based workloads, and `nerdctl` is a Docker-compatible CLI that talks to containerd (so you can mostly use the same commands you're used to with Docker).

Below are recommended additions to your Oh My Zsh configuration to make working with `nerdctl` and the Google Cloud SDK easier. These examples are safe wrappers that forward all arguments, mount your local config directories (so credentials persist), and run interactively when appropriate.

Add the following to `~/.oh-my-zsh/custom/aliases.zsh` (or to your `~/.zshrc`):

```bash
# Make common Docker commands use nerdctl
alias docker='lima nerdctl'
```

Notes and tips:

- The `alias docker='lima nerdctl'` line lets you run most Docker commands (`docker run`, `docker build`, `docker ps`, etc.) using `nerdctl` without changing your muscle memory. If you prefer, use `alias nerdctl=nerdctl` (not required).
- The `gcloud` function uses `--entrypoint gcloud` so the container runs the `gcloud` binary directly. All command-line arguments are passed through via `"$@"`.
- We mount `~/.config/gcloud` into the container so your credentials and settings are available. If you store credentials elsewhere (or use service account JSONs), add `-v` mounts or `-e GOOGLE_APPLICATION_CREDENTIALS=/path/in/container` as needed.
- `-v "$(pwd):/workspace" -w /workspace` mounts the current working directory into the container and sets it as the working directory. This makes commands like `gcloud beta functions deploy ...` work against files in your repo.
- On systems that enforce SELinux labels the `:Z` mount option is helpful; it's harmless on systems that don't need it.

Examples:

```bash
# Use the containerized gcloud to list projects
gcloud projects list

# Use nerdctl via the docker alias
docker ps

# Run an interactive shell in the cloud-sdk container (useful for one-off tasks)
lima nerdctl run --rm -it \
	-v "${HOME}/.config/gcloud:/root/.config/gcloud:Z" \
	-v "${HOME}/.ssh:/root/.ssh:Z" \
	-v "$(pwd):/workspace:Z" \
	-w /workspace \
	gcr.io/google.com/cloudsdktool/google-cloud-cli:stable /bin/bash
```

Reload your zsh configuration after adding the snippets above:

```bash
source ~/.zshrc
```

Quick verification commands:

```bash
gcloud --version    # should run the cloud-sdk image and print the version
```

If you'd like, you can make the wrappers more advanced (for example, automatically adding `-e GOOGLE_APPLICATION_CREDENTIALS` when a local service account file is present, or adding a small helper to safely forward Docker socket access). Open an issue or PR with details if you want me to add those helpers.
