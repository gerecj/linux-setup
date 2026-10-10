# linux-setup

Ansible config for Ubuntu Server. It runs on the machine itself, so
there's no control node, inventory or SSH. Clone the repo and run the playbook.

## Prerequisites

Ubuntu Server 26.04, a user with sudo rights, `ansible-core` and `git`:

Over SSH, start `tmux` first, so `apt` and the playbook keep running if the connection drops.

```sh
sudo apt update && sudo apt upgrade
sudo apt install ansible-core git
```

## Usage

```sh
git clone https://github.com/gerecj/linux-setup.git ~/linux-setup
cd ~/linux-setup
ansible-playbook local.yml -K
```

`-K` asks for your sudo password. Leave it out if you have passwordless sudo. Don't
run it with `sudo`: fnm, pnpm, uv, Codex and Claude Code are installed for the user
running the playbook, so the playbook refuses to run as root.

```sh
ansible-playbook local.yml -K --check --diff    # dry run, show what would change
ansible-playbook local.yml -K --tags packages   # run only part of it
```

Every run, dry runs included, is appended to `ansible.log` in the repo, so the output of
a failed run is still there after the terminal is gone.

## After the first run

1. Reboot (`sudo reboot`). That finishes handing the network from systemd-networkd
   to NetworkManager, puts `~/.local/bin`, fnm and pnpm on your PATH, and makes the
   `docker` group apply, so `docker` works without sudo.
1. Run `gh auth login` for GitHub, then `codex` and `claude` once each to sign in.

The network switch, as on Kubuntu, can give the machine a new IP. If SSH drops at the
end of the run, look up the new one on the console with `ip -4 a`.

## Layout

```
local.yml        the playbook, imports everything from tasks/
tasks/*.yml      one file per thing being set up
```

To add something, create `tasks/<name>.yml` and import it in `local.yml` with a tag.
Extra apt packages go in the list in `tasks/packages.yml`.

## SSH

The SSH server starts at boot with Ubuntu's defaults: log in with your user's
password, `ssh <user>@<server>`. If the password is refused, the Ubuntu installer
probably turned password logins off (that happens when you import SSH keys during
the install), see `/etc/ssh/sshd_config.d/50-cloud-init.conf`.

## Desktop

The server runs headless, but KDE Plasma is installed for when you need it. It boots
to the text console, so to get a desktop, plug in a monitor and start the login
screen:

```sh
sudo systemctl start display-manager   # SDDM login screen, pick the Plasma session
sudo systemctl stop display-manager    # back to headless, ends the desktop session
```

## What's installed

| What                                                  | For      | From                          | Updating                                                |
| ----------------------------------------------------- | -------- | ----------------------------- | ------------------------------------------------------- |
| Packages in `tasks/packages.yml`, KDE Plasma          | everyone | Ubuntu                        | `sudo apt upgrade`                                      |
| VS Code, Brave, Docker, GitHub CLI                    | everyone | the vendors' apt repositories | `sudo apt upgrade`                                      |
| Node.js, through [fnm](https://github.com/Schniz/fnm) | you      | fnm's installer, current LTS  | `fnm install <version>`                                 |
| pnpm                                                  | you      | standalone installer          | `pnpm self-update`                                      |
| uv                                                    | you      | official installer            | `uv self update`                                        |
| Codex                                                 | you      | official installer            | `curl -fsSL https://chatgpt.com/codex/install.sh \| sh` |
| Claude Code                                           | you      | official installer            | updates itself                                          |

"You" means the user who ran the playbook; those tools live under your home
directory. pnpm doesn't depend on the Node version and switches to the version a
project pins. Being in the `docker` group is effectively root access.

## Linting

```sh
pipx install ansible-lint
ansible-lint
```

Run it from the repo root. It runs yamllint too, with settings that fit Ansible, so
use it instead of plain `yamllint`, whose defaults clash with Ansible style.
