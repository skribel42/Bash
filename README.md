# install-pentest-tools

A single Bash script that turns a stock **Debian 13 (trixie)** box into a usable
security/pentest workstation, without turning it into Kali.

## What it does

Kali Linux is great, but running it as a daily driver is a pain, and bolting
Kali's rolling repos onto a Debian install is a well-known way to eventually
break your system. This script takes the saner path:

- **By default**, it installs the large chunk of the Kali toolset that already
  lives in Debian's own repositories (nmap, hydra, john, hashcat, sqlmap, ffuf,
  aircrack-ng, wireshark, radare2, impacket, netexec, seclists, and ~70 more).
  No foreign packages, nothing that can wreck your base system.
- **Optionally**, it can pull the real `kali-linux-default` metapackage from
  Kali's official repo, pinned low so it never overrides your Debian base.
- **Optionally**, it installs Metasploit (which isn't in Debian) via Rapid7's
  official installer.

It's idempotent (safe to re-run), installs packages one at a time so a single
missing name never kills the whole run, verifies Kali's signing key by
fingerprint before trusting it, and has a `--dry-run` mode that changes nothing.

## Requirements

- Debian 13 (trixie), amd64. It'll warn but still run on other Debian-likes.
- `sudo` / root access.
- An internet connection.

## How to run it (beginner-friendly)

**1. Get the script.** Either clone the repo:

```bash
git clone https://github.com/skribel41/Bash.git
cd Bash
```

...or just grab the one file:

```bash
wget https://raw.githubusercontent.com/skribel41/Bash/main/install-pentest-tools.sh
```

**2. Make it executable.** This tells Linux the file is allowed to run:

```bash
chmod +x install-pentest-tools.sh
```

**3. See what it *would* do first (recommended).** Nothing gets changed:

```bash
./install-pentest-tools.sh --dry-run
```

**4. Run it for real.** It needs root because it installs system packages:

```bash
sudo ./install-pentest-tools.sh
```

That's the default install. When it finishes it prints a summary of what got
installed and what was skipped.

## Options

| Flag                | What it does                                                        |
|---------------------|---------------------------------------------------------------------|
| *(none)*            | Install the Debian-native tool set only. Safe. Start here.           |
| `--with-kali-repo`  | Also add the pinned Kali repo and install the full `kali-linux-default` set. |
| `--with-metasploit` | Install Metasploit Framework (Rapid7 official installer).            |
| `--dry-run`         | Print everything, change nothing.                                    |
| `-h`, `--help`      | Show usage.                                                          |

Example — everything:

```bash
sudo ./install-pentest-tools.sh --with-kali-repo --with-metasploit
```

## ⚠️ Read before using `--with-kali-repo`

Kali is a **rolling** distro. Even with the low APT pin this script sets up,
mixing Kali packages into Debian stable carries real breakage risk over time.
**Take a snapshot first** (Timeshift, or a Btrfs/LVM snapshot) so you can roll
back if something goes sideways. If you only want the common tools, skip this
flag entirely — the default install covers most of what newcomers actually use.

If you ever hit an `EXPKEYSIG` error from the Kali repo later on, that just means
the signing key needs refreshing. Fix:

```bash
sudo wget https://archive.kali.org/archive-keyring.gpg -O /usr/share/keyrings/kali-archive-keyring.gpg
sudo apt update
```

## Keeping the tools updated

How you update depends on where a tool came from.

**The default (Debian-native) tools** are now just regular system packages, so
they update with everything else:

```bash
sudo apt update && sudo apt upgrade
```

Run that whenever you'd normally update Debian. Nothing special to remember.

**The exploit database** (`searchsploit`) ships exploit content that updates
separately from its package. Refresh it with:

```bash
searchsploit -u
```

**Metasploit** (if you installed it) updates either through apt — Rapid7's
installer adds its own repo, so the normal `apt upgrade` above catches it — or
directly with:

```bash
sudo msfupdate
```

**Kali-repo packages** (only if you used `--with-kali-repo`) are the one catch.
Because the script pins Kali at low priority to protect your Debian base, a
plain `apt upgrade` will **not** pull newer Kali versions — those tools stay at
the version you installed. That's the safety tradeoff working as intended. When
you do want to update a specific Kali tool, target it explicitly:

```bash
sudo apt install <toolname>/kali-rolling
```

Don't remove the pin and run a blanket upgrade across the Kali repo — that's
exactly the rolling-into-stable breakage the pin exists to prevent.



These are security testing tools. Only use them against systems you own or have
explicit written permission to test. You're responsible for what you do with them.

## License

MIT
