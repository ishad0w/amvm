# `amvm` _(Ansible with Mitogen Version Manager)_

`amvm` is a simple, powerful shell script for managing multiple versions of Ansible + Mitogen in isolated Python virtual environments and switching between them seamlessly.

It's designed for Ansible developers and operators who need to work with different versions for different projects without the hassle of manually managing virtual environments, control-node Python versions, or `ansible.cfg` files.

Version 2.0 uses [`uv`](https://docs.astral.sh/uv/) to create environments with uv-managed Python instead of relying on the system `python3` and `venv` module.

## Why `amvm`?

If you've ever found yourself juggling multiple Ansible projects, you've likely faced these challenges:

- Forgetting to activate the correct environment, leading to dependency errors.
- Managing different `ansible.cfg` files for standard vs. Mitogen-enabled runs.
- Manually creating and activating Python `venv`s for each version (`source .../activate`).
- Keeping Ansible, `ansible-core`, `ansible-lint`, Mitogen, and control-node Python mutually compatible.

`amvm` solves these problems by automating the entire workflow.
It's more than just a `venv` wrapper; it's a complete management tool that lets you **set your Ansible version once and forget about it**.

## Features
`amvm` provides flexibility and speed through a set of powerful, integrated features:

-   🚀 **Atomic Version & Config Switching**: This is the core feature. A single `amvm` command lets you switch both the active `ansible` version **and** its corresponding configuration (`ansible.cfg`) in one atomic operation. Instantly toggle between a standard setup, a Mitogen-optimized one, or your own project-specific config without any manual file editing. No more `source .../activate` or juggling paths by hand.

-   🐍 **uv-managed Python**: Built-in environments are created with `uv venv --managed-python` and an explicit Python pin from the compatibility matrix. amvm 2.0 no longer depends on your system Python having a working `venv` module.

-   🧭 **Built-in Compatibility Matrix**: Each built-in Ansible line pins `ansible`, `ansible-core`, `ansible-lint`, Mitogen, and Python. Print the current table at any time with `amvm --matrix`.

-   💻 **Fast Interactive UI**: Switch between any version and config combination in seconds using a modern, interactive menu powered by `fzf` (if installed), with a simple numbered menu as a fallback.

-   ⚡ **Integrated Mitogen & Custom Configs**:
    -   **Mitogen Support**: `amvm` automatically creates a Mitogen-ready configuration when installed. This appears as a separate option in the menu (e.g., `ansible-14.1.0 (mitogen)`), letting you enable or disable Mitogen with a single keystroke.
    -   **Per-Environment Configs**: Drop an `ansible-customized.cfg` file into any version's directory, and `amvm` automatically offers it as a switching option (e.g., `ansible-14.1.0 (custom)`) - perfect for project-specific settings.

-   🔧 **Extensive Customization**:
    -   **Custom Versions**: Easily define your own version sets (Ansible, ansible-core, ansible-lint, Mitogen, and optional Python) in a simple configuration file (`~/.amvm.cfg`).
    -   **Custom Root Directory**: Change the default installation path (`~/.amvm`) to anywhere on your system.

-   📦 **Batteries-Included Installation**: Each environment is created with not just Ansible, but also common dependencies like `ansible-lint`, `boto3`, `pywinrm`, and `jmespath`, saving you setup time. You can easily skip optional packages.

-   🛡️ **Safe and Clean Management**:
    -   **Isolated Environments**: Each version lives in its own uv-created virtual environment, preventing dependency conflicts.
    -   **Metadata & Stale Detection**: amvm 2.0 writes `.amvm.env` metadata into each environment and can identify old or mismatched environments.
    -   **Reinstall Support**: `amvm --reinstall <ver|all>` removes and rebuilds built-in environments after confirmation.
    -   **Interactive Uninstall**: Safely remove specific versions with a confirmation prompt.
    -   **Total Cleanup**: A single `amvm --cleanup` command interactively removes all `amvm`-managed files and directories.

-   🏠 **Self-Contained**: All environments, shims, metadata, and configurations are stored in a single directory (`~/.amvm` by default), making backups or removal trivial.

## Prerequisites
-   **Compatibility**: Works on **Linux** and **macOS**.
-   **Bash**: Version `3.2` or newer.
-   **`uv`**: Required in `PATH`. `amvm` does **not** auto-install it.
-   **`curl`**: Required to download the script.
-   **`fzf`** (Optional but Recommended): For the best interactive menu experience.

Install `uv` first if needed:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Version Matrix
`amvm` installs specific, tested combinations of packages. To use other versions, define them in your `~/.amvm.cfg`. To skip installing `ansible-lint` or `mitogen`, use `0` as the version number.

| Key |  `ansible` | `ansible-core` | `ansible-lint` | `mitogen` | `python` | Notes |
|:---:|:----------:|:--------------:|:--------------:|:---------:|:--------:|:------|
| `10` |  `10.7.0`  |    `2.17.14`   |    `25.8.2`    |  `0.3.50` |  `3.12`  | EOL; final 10.x package |
| `11` |  `11.13.0` |    `2.18.18`   |    `25.11.0`   |  `0.3.50` |  `3.12`  | EOL Dec 2025; strict-era lint |
| `12` |  `12.3.0`  |    `2.19.11`   |    `26.4.0`    |  `0.3.50` |  `3.12`  | EOL Dec 2025; strict-era lint |
| `13` |  `13.8.0`  |    `2.20.7`    |    `26.6.0`    |  `0.3.50` |  `3.12`  | Final Ansible 13/core 2.20 line |
| `14` |  `14.1.0`  |    `2.21.1`    |    `26.6.0`    |  `0.3.50` |  `3.12`  | Current tested Ansible 14/core 2.21 line |

Print the built-in matrix from the CLI:

```bash
amvm --matrix
```

Compatibility was checked against Ansible release maintenance data, PyPI package metadata, Ansible 14.1.0 release notes, and Mitogen 0.3.50 release notes.

---

## Installation

1.  **Install `uv`**
    `uv` must be available in your `PATH` before installing Ansible environments.

    ```bash
    curl -LsSf https://astral.sh/uv/install.sh | sh
    ```

2.  **Download the Script**
    Place the `amvm` script in a directory that is already in your `$PATH`, such as `~/.local/bin`.

    ```bash
    # Create the directory if it doesn't exist
    mkdir -p ~/.local/bin
    
    # Download and make executable
    curl -Lo ~/.local/bin/amvm https://raw.githubusercontent.com/ishad0w/amvm/main/amvm
    chmod +x ~/.local/bin/amvm
    ```

3.  **Configure Your PATH**
    `amvm` works by creating small wrapper scripts (shims) for the `ansible*` executables in `~/.amvm/bin`. For your system to find these shims, you must add this directory to your `$PATH`. **This is a one-time setup step.**

    The script will remind you if your `PATH` is not configured and will provide the exact command to run.

    For **zsh**:
    ```bash
    echo 'export PATH="$HOME/.amvm/bin:$PATH"' >> ~/.zshrc
    ```

    For **bash**:
    ```bash
    echo 'export PATH="$HOME/.amvm/bin:$PATH"' >> ~/.bashrc
    ```

    After running the command, **restart your terminal** or source your profile (`source ~/.zshrc`) to apply the changes.

---

## Usage

### 1. Install an Environment
Use the `amvm --install` command. You can install a specific predefined version or all of them.

```bash
# Install a specific built-in version (e.g., 14)
amvm --install 14

# Install all available built-in versions
amvm --install all
```

### 2. Reinstall an Existing Environment
Use `amvm --reinstall` when an old environment already exists but the matrix changed.

This is especially useful when upgrading from amvm 1.x, because old environments were created with system Python and do not contain amvm 2.0 metadata.

```bash
# Rebuild one built-in environment
amvm --reinstall 14

# Rebuild every current built-in environment
amvm --reinstall all
```

You will be asked for confirmation before anything is removed.

### 3. Switch the Active Version (it's not activated by default)
Run `amvm` with no arguments to open the interactive selection menu.

**With `fzf`:**
```
amvm>
┌───────────────────────────┐
│ > ansible-14.1.0 (mitogen)│
│   ansible-14.1.0          │
│   ansible-13.8.0 (mitogen)│
│   ansible-13.8.0 (custom) │
│   ansible-13.8.0          │
└───────────────────────────┘
```

**Without `fzf` (fallback):**
```
   1) ansible-14.1.0 (mitogen)
   2) ansible-14.1.0
   3) ansible-13.8.0 (mitogen)
   4) ansible-13.8.0 (custom)
   5) ansible-13.8.0
Enter a number: _
```

After switching, you can immediately verify the change:
```bash
$ ansible --version
ansible [core 2.21.1]
  config file = /home/user/.ansible.cfg
  ...
```

### 4. Print the Built-in Matrix
Use `amvm --matrix` to inspect the currently built-in compatibility table.

```bash
amvm --matrix
```

### 5. Uninstall an Environment
Use `amvm --uninstall` to open an interactive menu to select and remove a version.

```
uninstall>
┌──────────────────┐
│ > ansible-14.1.0 │
│   ansible-13.8.0 │
└──────────────────┘
```

You will be asked for confirmation before anything is deleted.

### Management Commands

```bash
amvm --matrix                         # Print the built-in compatibility matrix
amvm --install <10|11|12|13|14|all>   # Install a built-in version or all
amvm --reinstall <10|11|12|13|14|all> # Remove and reinstall a built-in version or all
amvm --install-custom <key>           # Install a user-defined version from your config
amvm --uninstall                      # Interactively remove an installed version
amvm --list                           # List all installed versions and stale status
amvm --cleanup                        # Interactively remove ALL amvm data
amvm --help, -h                       # Show the help message
amvm --version, -v                    # Show amvm version
```

---

## Customization

You can configure `amvm` by creating a file at `~/.amvm.cfg`.

### 1. Defining Custom Versions

To install versions not built into `amvm`, define them in `~/.amvm.cfg` using a Bash array named `AMVM_CUSTOM_VERSIONS`.
Each entry is a string with the format: `"KEY:ansible|ansible-core|ansible-lint|mitogen|python"`.

The final `python` field is optional for backward compatibility. If omitted, `amvm` defaults to Python `3.12`.

**Use `0` to skip installing `ansible-lint` or `mitogen`**.

**Example `~/.amvm.cfg`:**
```bash
# Define your custom versions here
AMVM_CUSTOM_VERSIONS=(
  # Key '9': Ansible 9 with Mitogen, no ansible-lint, and Python 3.11
  "9:9.13.0|2.16.14|0|0.3.50|3.11"
  # Key '14-no-lint': Ansible 14 without optional packages
  "14-no-lint:14.1.0|2.21.1|0|0"
)
```

With this config, you can now run:
```bash
amvm --install-custom 9
amvm --install-custom 14-no-lint
```

### 2. Using a Custom `ansible.cfg`

For ultimate control, you can use your own hand-tuned `ansible.cfg` for any version.
Simply create a file named `ansible-customized.cfg` inside the desired environment's directory:

`~/.amvm/ansible-X.Y.Z/ansible-customized.cfg`

The next time you run `amvm`, a new `(custom)` option will automatically appear in the menu for that version, allowing you to switch to it instantly.

### 3. Changing the Root Directory

You can change the main directory where `amvm` stores everything by setting `AMVM_ROOT` in your config.

```bash
# ~/.amvm.cfg
AMVM_ROOT="/opt/ansible_versions"
```

---

## How It Works

-   **`~/.amvm/`**: The root directory for everything `amvm` manages.
    -   `ansible-X.Y.Z/`: Each environment is a self-contained uv-created virtual environment. It contains `ansible.cfg`, `ansible-mitogen.cfg` (if applicable), your own `ansible-customized.cfg`, and `.amvm.env` metadata.
    -   `bin/`: This directory contains the active "shims" - small wrapper scripts that prepend the active environment's `bin` directory to `PATH` and then execute the matching `ansible*` command. This is the directory you add to your `PATH`.
-   **`~/.ansible.cfg`**: This is not a real file, but a symlink that always points to the active configuration file (`ansible.cfg`, `ansible-mitogen.cfg`, or `ansible-customized.cfg`) inside the active environment.

This shim-based approach is what makes switching seamless and eliminates the need to `source` or `deactivate` environments.

---

## Troubleshooting

-   **`uv` is missing**
    Install `uv` and confirm `uv --version` works in the same shell. amvm 2.0 intentionally does not auto-install `uv`.

-   **`amvm: command not found`**
    Ensure the `amvm` script is in a directory listed in your `PATH` (like `~/.local/bin`) and that it has execute permissions (`chmod +x amvm`).

-   **`ansible: command not found` after switching**
    Your `PATH` is likely misconfigured. Run `amvm` and it will show a warning with the exact command needed to fix your shell profile file. Remember to **restart your terminal** after making the change.

-   **Existing environment is skipped**
    If an environment already exists but does not match the current matrix, use `amvm --reinstall <key>`.
