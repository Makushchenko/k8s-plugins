# kubectl-kubeplugin

A `kubectl` plugin that prints live CPU/Memory metrics as **CSV**:

```
Resource,Namespace,Name,CPU,Memory
```

It uses `kubectl top` under the hood and falls back to `N/A` if metrics are unavailable.

---

## Requirements

* `kubectl` configured to access your cluster
* **Metrics Server** installed and healthy

  * Docs: [https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/)

---

## Quick Start

1. Save the plugin script as `kubectl-kubeplugin` (exact filename) somewhere in your `$PATH`.
2. Make it executable:

   ```bash
   chmod +x kubectl-kubeplugin
   ```
3. Verify `kubectl` can see it:

   ```bash
   kubectl plugin list
   ```
4. Run:

   ```bash
   kubectl kubeplugin                 # kube-system pods (default)
   kubectl kubeplugin default pods    # pods in 'default'
   kubectl kubeplugin nodes           # cluster nodes (Namespace column '-')
   kubectl kubeplugin -A              # pods across all namespaces
   kubectl kubeplugin -h              # help
   ```

### Flags

* `-A, --all-namespaces` — show metrics for all accessible namespaces (pods)
* `-h, --help` — show help

---

## How to Install Self‑Written kubectl Plugins

`kubectl` automatically discovers **executable** files on your `$PATH` with names that start with `kubectl-`. Each file defines a plugin named after the suffix.

* **File name → command name**: `kubectl-kubeplugin` ⇒ `kubectl kubeplugin`
* **Placement**: put the file in a directory on your `$PATH` (e.g., `~/.local/bin`, `/usr/local/bin`)
* **Permissions**: `chmod +x ~/.local/bin/kubectl-kubeplugin`
* **Verify**:

  ```bash
  kubectl plugin list
  kubectl kubeplugin -h
  ```

> Tip (Linux/macOS): add `~/.local/bin` to PATH in your shell config, e.g. `~/.bashrc` or `~/.zshrc`:
>
> ```bash
> export PATH="$HOME/.local/bin:$PATH"
> ```

---

## Using Plugins with **Krew** (Plugin Manager)

[Krew](https://krew.sigs.k8s.io/) is the package manager for `kubectl` plugins. It lets you **discover**, **install**, **update**, and **remove** community plugins.

### Install Krew

* **Linux/macOS (bash/zsh):**

  ```bash
  set -x; cd "$(mktemp -d)" && \\
  OS="$(uname | tr '[:upper:]' '[:lower:]')" && ARCH="$(uname -m)" && \\
  if [ "$ARCH" = "x86_64" ]; then ARCH=amd64; fi; \\
  if [ "$ARCH" = "aarch64" ]; then ARCH=arm64; fi; \\
  KREW="krew-${OS}_${ARCH}" && \\
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" && \\
  tar zxvf "${KREW}.tar.gz" && ./${KREW} install krew
  ```

  Then add Krew’s bin dir to your PATH and restart your shell:

  ```bash
  export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
  ```
* **Windows (PowerShell):** see [https://krew.sigs.k8s.io/docs/user-guide/setup/install/](https://krew.sigs.k8s.io/docs/user-guide/setup/install/)

### Use Krew

* **Search plugins:**

  ```bash
  kubectl krew search
  kubectl krew search metrics
  ```
* **Install a plugin:**

  ```bash
  kubectl krew install ctx
  kubectl krew install ns
  ```
* **List installed plugins:**

  ```bash
  kubectl krew list
  ```
* **Upgrade everything:**

  ```bash
  kubectl krew upgrade
  ```
* **Remove a plugin:**

  ```bash
  kubectl krew uninstall ctx
  ```

> Note: This repository’s `kubectl-kubeplugin` is a **self‑written** plugin you install manually. Packaging it for Krew requires creating a Krew plugin manifest and hosting release assets. See: [https://krew.sigs.k8s.io/docs/developer-guide/](https://krew.sigs.k8s.io/docs/developer-guide/) for publishing.

---

## Examples & Output

```bash
kubectl kubeplugin
kubectl kubeplugin -A
kubectl kubeplugin nodes
```

Sample output:

```text
Resource,Namespace,Name,CPU,Memory
pods,default,web-7b6c8d7f9d-t8s4z,3m,18Mi
nodes,-,ip-10-0-1-23,120m,1024Mi
```

---

## Troubleshooting

* **Only header printed**: Metrics exist but `kubectl top` returns no rows yet. Wait a few seconds after pod start; ensure Metrics Server is healthy.
* **Plugin not found**: confirm filename is `kubectl-kubeplugin`, it’s executable, and the directory is on your `$PATH`.
* **Permission denied**: `chmod +x kubectl-kubeplugin` and re-run.