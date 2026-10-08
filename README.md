# hotkey-add-journal

`hotkey-add-journal` is a small Bash utility for adding structured entries to an Obsidian daily note from a keyboard shortcut.

It opens a graphical menu and can record events, sleep times, activities, energy and mood, ideas, reminders, todo items, and optional Markdown tags. The script safely updates the note and refuses to overwrite it when its structure is not what the configuration expects.

## How it works

The script expects daily notes named like `Journal/2026-10-08.md`. It inserts normal entries below an exact heading, normally `## Log`. Sleep entries are inserted directly after YAML frontmatter.

The script edits Markdown files directly; it does not use the Obsidian API and Obsidian does not need to be open. Keep your normal backup or Git workflow enabled.

## 1. Install dependencies

The script needs Bash, standard command-line tools, `ripgrep`, one menu program, `yad`, and `zenity`.

### Ubuntu Linux

```bash
sudo apt update
sudo apt install bash coreutils gawk grep sed findutils ripgrep yad zenity wofi
```

If `wofi` is unavailable on your Ubuntu release, install `rofi` instead and set `MENU_TOOL="rofi"` in the script:

```bash
sudo apt install bash coreutils gawk grep sed findutils ripgrep yad zenity rofi
```

### Arch Linux

```bash
sudo pacman -Syu
sudo pacman -S bash coreutils gawk grep sed findutils ripgrep yad zenity wofi
```

For an X11 desktop, install `rofi` instead and set `MENU_TOOL="rofi"`:

```bash
sudo pacman -S rofi
```

## 2. Install the script

From this project directory:

```bash
mkdir -p "$HOME/.local/bin"
cp hotkey-add-journal "$HOME/.local/bin/hotkey-add-journal"
chmod +x "$HOME/.local/bin/hotkey-add-journal"
```

If `$HOME/.local/bin` is not in your `PATH`, add this to `~/.bashrc` and/or `~/.zshrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Open a new terminal afterwards, or run `source "$HOME/.bashrc"`.

## 3. Configure your vault

Open `hotkey-add-journal` and edit only the block marked `USER CONFIGURATION — EDIT THIS BLOCK`.

Example:

```bash
VAULT_DIR="$HOME/Documents/MyObsidianVault"
JOURNAL_SUBDIR="Journal"
TEMPLATE_RELATIVE_PATH="_Templates/Journal - Daily Note.md"
LOG_HEADING="## Log"
TAG_INDEX_RELATIVE_PATH="TAGS_COUNT.md"
MENU_TOOL="wofi"
```

| Setting | Meaning |
|---|---|
| `VAULT_DIR` | Absolute path to your Obsidian vault. `$HOME` is allowed. |
| `JOURNAL_SUBDIR` | Folder containing `YYYY-MM-DD.md` daily notes. |
| `TEMPLATE_RELATIVE_PATH` | Template path relative to the vault. |
| `LOG_HEADING` | Exact heading where normal entries are inserted. |
| `TAG_INDEX_RELATIVE_PATH` | Optional tag-count file. Set it to an empty value if you do not have one. |
| `MENU_TOOL` | `wofi` for Wayland or `rofi` for X11/Wayland. |

Linux paths are case-sensitive: `Journal` and `journal` are different folders.

## 4. Prepare the daily-note template

Your template must contain the exact heading configured in `LOG_HEADING`. The smallest usable template is:

```markdown
---
date: {{date:YYYY-MM-DD}}
---

# {{date:dddd D MMMM YYYY}}

## Log

## Notes
```

An example is included in `examples/daily-note-template.md`. If your template uses another heading, either change it to `## Log` or change `LOG_HEADING` in the script.

## 5. Check and run

Run this before using the hotkey:

```bash
hotkey-add-journal --check
```

The check verifies the vault, Journal folder, template, log heading and required programs. It does not create or modify a journal note.

Then try it manually:

```bash
hotkey-add-journal
```

Run it inside your graphical desktop session. A plain SSH session normally cannot display `wofi`, `rofi`, `yad` or `zenity`.

## 6. Keyboard shortcuts

### Niri

In `~/.config/niri/config.kdl`:

```kdl
binds {
    Mod+J { spawn "/home/YOUR_USER/.local/bin/hotkey-add-journal"; }
}
```

Replace `YOUR_USER` with your Linux username and reload Niri.

### Hyprland

In `~/.config/hypr/hyprland.conf`:

```ini
bind = SUPER, J, exec, /home/YOUR_USER/.local/bin/hotkey-add-journal
```

Reload Hyprland afterwards.

### GNOME

Open **Settings → Keyboard → View and Customize Shortcuts → Custom Shortcuts**, add a shortcut named `Add journal entry`, and use `/home/YOUR_USER/.local/bin/hotkey-add-journal` as the command.

### KDE Plasma

Open **System Settings → Keyboard Shortcuts → Add New → Command or Script**, choose the script and assign a shortcut.

## Tags

The tag dialog searches tags found in the vault. It reads the optional tag index and also scans Markdown files with `ripgrep`. The displayed name does not need a leading `#`; the script adds it when saving. **New tag** allows a tag that is not yet present. Tags may not contain spaces.

## Troubleshooting

### A command is missing

Install the package named by `--check`, using `apt` on Ubuntu or `pacman` on Arch, then run the check again.

### The menu does not appear

Check that the script is started from a graphical session. Use `MENU_TOOL="wofi"` for Wayland or `MENU_TOOL="rofi"` for X11. Test the script manually before assigning a hotkey.

### The Daily Note is not found

Check `VAULT_DIR`, `JOURNAL_SUBDIR`, capitalization and the date filename. The script expects exactly `YYYY-MM-DD.md` for today.

### The log heading is missing

Make the heading in the template exactly match `LOG_HEADING`, including the number of `#` characters and spaces.

## Safety notes

- The original personal script is not touched by this project.
- A lock directory prevents simultaneous updates.
- Temporary files are used while updating an existing note.
- A SHA-256 check prevents replacing the note if another program changed it during the update.
- The script does not delete journal content.

## Separate Git repository

Yes, a separate repository is worthwhile. This tool is independent from a personal Obsidian vault, keeps private paths and notes out of the public project, makes versioning easy, and lets other people install it without copying your vault.

This directory is already structured as a standalone project. Before publishing it, review the script once more for personal names, paths or private metadata.
