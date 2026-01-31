# `amvm` _(Ansible with Mitogen Version Manager)_

`amvm` is a simple, powerful shell script for managing multiple versions of Ansible + Mitogen in isolated Python virtual environments (`venv`) and switching between them seamlessly.

It's designed for Ansible developers and operators who need to work with different versions for different projects without the hassle of manually managing virtual environments or `ansible.cfg` files.

## Why `amvm`?

If you've ever found yourself juggling multiple Ansible projects, you've likely faced these challenges:

- Forgetting to activate the correct environment, leading to dependency errors.
- Managing different `ansible.cfg` files for standard vs. Mitogen-enabled runs.
- Manually creating and activating Python `venv`s for each version (`source .../activate`).

`amvm` solves these problems by automating the entire workflow.
It's more than just a `venv` wrapper; it's a complete management tool that lets you **set your Ansible version once and forget about it**.

## Features
`amvm` provides flexibility and speed through a set of powerful, integrated features:

-   🚀 **Atomic Version & Config Switching**: This is the core feature. A single `amvm` command lets you switch both the active `ansible` version **and** its corresponding configuration (`ansible.cfg`) in one atomic operation. Instantly toggle between a standard setup, a Mitogen-optimized one, or your own project-specific config without any manual file editing. No more `source .../activate` or juggling symlinks.

-   💻 **Fast Interactive UI**: Switch between any version and config combination in seconds using a modern, interactive menu powered by `fzf` (if installed), with a simple numbered menu as a fallback.

-   ⚡ **Integrated Mitogen & Custom Configs**:
    -   **Mitogen Support**: `amvm` automatically creates a Mitogen-ready configuration when installed. This appears as a separate option in the menu (e.g., `ansible-11.7.0 (mitogen)`), letting you enable or disable Mitogen with a single keystroke.
    -   **Per-Environment Configs**: Drop an `ansible-customized.cfg` file into any version's directory, and `amvm` automatically offers it as a switching option (e.g., `ansible-11.7.0 (custom)`)—perfect for project-specific settings.

-   🔧 **Extensive Customization**:
    -   **Custom Versions**: Easily define your own version sets (Ansible, ansible-core, ansible-lint, Mitogen) in a simple configuration file (`~/.amvm.cfg`).
    -   **Custom Root Directory**: Change the default installation path (`~/.amvm`) to anywhere on your system.

-   📦 **Batteries-Included Installation**: Each environment is created with not just Ansible, but also common dependencies like `ansible-lint`, `boto3`, `pywinrm`, and `jmespath`, saving you setup time. You can easily skip optional packages.

-   🛡️ **Safe and Clean Management**:
    -   **Isolated Environments**: Each version lives in its own `venv`, preventing any dependency conflicts.
    -   **Interactive Uninstall**: Safely remove specific versions with a confirmation prompt.
    -   **Total Cleanup**: A single `amvm --cleanup` command interactively removes all `amvm`-managed files and directories.

-   🏠 **Self-Contained**: All environments, shims, and configurations are stored in a single directory (`~/.amvm` by default), making backups or removal trivial.

## Prerequisites
-   **Compatibility**: Works on **Linux** and **macOS**.
-   **Bash**: Version `3.2` or newer.
-   **Python 3**: The `python3` command must be available in your `PATH`.
-   **Python Venv Module**: The module for creating virtual environments is required.
-   **`curl`**: Required to download the script.
-   **`fzf`** (Optional but Recommended): For the best interactive menu experience.

## Version Matrix
`amvm` installs specific, tested combinations of packages. To use other versions, define them in your `~/.amvm.cfg`. To skip installing `ansible-lint` or `mitogen`, use `0` as the version number.

| Key |  `ansible` | `ansible-core` | `ansible-lint` | `mitogen` |
|:---:|:----------:|:--------------:|:--------------:|:---------:|
| `10` |  `10.7.0`  |    `2.17.14`   |    `26.1.1`    |  `0.3.39` |
| `11` |  `11.13.0` |    `2.18.13`   |    `26.1.1`    |  `0.3.39` |
| `12` |  `12.3.0`  |    `2.19.6`    |    `26.1.1`    |  `0.3.39` |
| `13` |  `13.3.0`  |    `2.20.2`    |    `26.1.1`    |  `0.3.39` |

---

## Installation

1.  **Download the Script**
    Place the `amvm` script in a directory that is already in your `$PATH`, such as `~/.local/bin`.

    ```bash
    # Create the directory if it doesn't exist
    mkdir -p ~/.local/bin
    
    # Download and make executable
    curl -Lo ~/.local/bin/amvm https://raw.githubusercontent.com/ishad0w/amvm/main/amvm
    chmod +x ~/.local/bin/amvm
    ```

2.  **Configure Your PATH**
    `amvm` works by creating symlinks (shims) to the `ansible*` executables in `~/.amvm/bin`. For your system to find these shims, you must add this directory to your `$PATH`. **This is a one-time setup step.**

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
# Install a specific built-in version (e.g., 11)
amvm --install 11

# Install all available built-in versions
amvm --install all
```

### 2. Switch the Active Version (it's not activated by default)
Run `amvm` with no arguments to open the interactive selection menu.

**With `fzf`:**
```
amvm>
┌───────────────────────────┐
│ > ansible-11.7.0 (mitogen)│
│   ansible-11.7.0          │
│   ansible-10.7.0 (mitogen)│
│   ansible-10.7.0 (custom) │
│   ansible-10.7.0          │
└───────────────────────────┘
```

**Without `fzf` (fallback):**
```
   1) ansible-11.7.0 (mitogen)
   2) ansible-11.7.0
   3) ansible-10.7.0 (mitogen)
   4. ansible-10.7.0 (custom)
   5. ansible-10.7.0
Enter a number: _
```

After switching, you can immediately verify the change:
```bash
$ ansible --version
ansible [core 2.18.6]
  config file = /home/user/.ansible.cfg
  ...
```

### 3. Uninstall an Environment
Use `amvm --uninstall` to open an interactive menu to select and remove a version.

```
uninstall>
┌──────────────────┐
│ > ansible-11.7.0 │
│   ansible-10.7.0 │
└──────────────────┘
```

You will be asked for confirmation before anything is deleted.

### Management Commands

```bash
amvm --install <10, 11 | all>     # Install a built-in version (e.g., 10, 11) or all
amvm --install-custom <key>  # Install a user-defined version from your config
amvm --uninstall             # Interactively remove an installed version
amvm --list                  # List all installed versions
amvm --cleanup               # Interactively remove ALL amvm data
amvm --help, -h              # Show the help message
amvm --version, -v           # Show amvm version
```

---

## Customization

You can configure `amvm` by creating a file at `~/.amvm.cfg`.

### 1. Defining Custom Versions

To install versions not built into `amvm`, define them in `~/.amvm.cfg` using a Bash array named `AMVM_CUSTOM_VERSIONS`.
Each entry is a string with the format: `"KEY:ansible|ansible-core|ansible-lint|mitogen"`.

**Use `0` to skip installing `ansible-lint` or `mitogen`**.

**Example `~/.amvm.cfg`:**
```bash
# Define your custom versions here
AMVM_CUSTOM_VERSIONS=(
  # Key '9': Ansible 9 with Mitogen, but no ansible-lint
  "9:9.5.1|2.16.7|0|0.3.18"
  # Key '8': Ansible 8 without any optional packages
  "8:8.7.0|2.15.10|0|0"
)
```

With this config, you can now run:
```bash
amvm --install-custom 9
amvm --install-custom 8
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
    -   `ansible-X.Y.Z/`: Each environment is a self-contained Python `venv`. It contains `ansible.cfg`, `ansible-mitogen.cfg` (if applicable), and your own `ansible-customized.cfg`.
    -   `bin/`: This directory contains the "shims" (symlinks) pointing to executables in the **currently active** environment. This is the directory you add to your `PATH`.
-   **`~/.ansible.cfg`**: This is not a real file, but a symlink that always points to the active configuration file (`ansible.cfg`, `ansible-mitogen.cfg`, or `ansible-customized.cfg`) inside the active environment.

This shim-based approach is what makes switching seamless and eliminates the need to `source` or `deactivate` environments.

---

## Troubleshooting

-   **`amvm: command not found`**
    Ensure the `amvm` script is in a directory listed in your `PATH` (like `~/.local/bin`) and that it has execute permissions (`chmod +x amvm`).

-   **`ansible: command not found` after switching**
    Your `PATH` is likely misconfigured. Run `amvm` and it will show a warning with the exact command needed to fix your shell profile file. Remember to **restart your terminal** after making the change.
