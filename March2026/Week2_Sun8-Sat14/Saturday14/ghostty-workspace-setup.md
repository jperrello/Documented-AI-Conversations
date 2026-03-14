# Ghostty Workspace Setup for Multi-Agent Claude Code
**Date:** Saturday, March 14, 2026
**Time:** Afternoon session
---

## User

i finally want to spend some time customizing ghostty for myself and i need your help. here are docs: https://ghostty.org/docs/config let me explain my particular situation to you and we can figure out where to go from there.
When I use Ghosty right now, I typically have three different terminal windows open at a time. Those three terminal windows are always just instances of Claude code running. I will always at least have three open. Maybe in the future, I may want to change that to more, but I want to be able to clearly see all three of these terminals at once so I can see when Claude is talking to me or responding or working on something.

I would also like a spare ghosty terminal window open just for me to navigate through directories and make new directories. I basically want a hotkey to spin up those ghosty terminals every time. And I think we could utilize Mac pretty well to do this. Like I could send all of these ghosty terminals to A specific desktop. So when I'm using my Mac, I navigate to that desktop and I have all my ghosty terminals completely open. Then I can navigate back to my main desktop, which has something like Visual Studio or Firefox running. But with that being said, I kind of want it to be easy to get notified of when those terminals are done running and things like that.

Basically, my goal is to optimize my augmented workflow entirely. There are definitely people online who have been investigating this for months. So maybe look for some obscure blog posts or discussion board posts on how people kind of set up ghosty terminals in order to maximize how many agents they could run and their ability to focus on all those agents at once and kind of keep it all controlled.

## Claude

Let me research the Ghostty docs and what people have done for multi-agent terminal workflows.

```tool
[Tool: Agent] description: "Fetch Ghostty config docs", prompt: "Fetch the Ghostty configuration documentation from https://ghostty.org/docs/config..."
```

```tool
[Tool: Agent] description: "Research multi-agent terminal setups", prompt: "Search the web for blog posts, discussion threads, and guides about people setting up Ghostty terminals..."
```

Let me also check your current Ghostty config to see what you already have.

```tool
[Tool: Glob] pattern: "**/*ghostty*", path: "/Users/jperr/.config"
```

```tool
[Tool: Read] file_path: "/Users/jperr/Library/Application Support/com.mitchellh.ghostty/config"
```

Good — you're starting from a clean slate. I've got two research agents running in the background:

1. **Fetching the official Ghostty config docs** — to understand all available options for keybindings, window management, notifications, and macOS features
2. **Searching for multi-agent workflow setups** — looking for blog posts and discussions from people who've optimized Ghostty for running multiple AI agents

I'll synthesize everything once both come back and we can build out your config together. While we wait — a couple of quick questions:

- What's your monitor setup? (single display, ultrawide, multiple monitors, screen resolution)
- Do you already use any macOS automation tools like Raycast, Hammerspoon, or Keyboard Maestro?

## User

**What's your monitor setup?** → Single laptop screen
**Do you already use any macOS automation tools?** → None

## Claude

Good to know — single laptop screen and no existing automation. That shapes things significantly. Let me check on the research results.

[Fetched Ghostty config docs, keybinding reference, config reference, and searched for multi-agent workflow setups]

Key findings from research:

**Ghostty Config Options:**
- `keybind = trigger=action` syntax with `global:` prefix for system-wide hotkeys
- `notify-on-command-finish` with `unfocused`/`always`/`never` options
- `window-save-state = always` for macOS window restore
- Split navigation, creation, zoom, and equalize actions
- AppleScript API for programmatic window/split/tab creation

**Multi-Agent Ecosystem:**
- Ghostty + Git Worktrees + Claude Code is the dominant pattern
- cmux — native macOS app built on Ghostty specifically for AI agent workflows with notification rings
- ghostty-pane-splitter, ghostty-workspace tools
- AppleScript automation for window positioning

### My recommendation

Start with **cmux** for day-to-day use since it directly solves the notification/attention problem, and **also set up the Ghostty config + AppleScript** so you have the raw automation layer for custom workflows.

## User

**Which path do you want to start with?** → Both (Recommended)

## Claude

Let's do both. I'll set up your Ghostty config first, then the AppleScript launcher, then cmux.

### 1. Ghostty Config

```tool
[Tool: Write] file_path: "/Users/jperr/Library/Application Support/com.mitchellh.ghostty/config"
```

Wrote config with:
- JetBrains Mono 12pt, catppuccin-mocha theme
- Vim-style split navigation (Ctrl+H/J/K/L)
- Split resizing (Ctrl+Shift+H/J/K/L)
- Quick split creation (Cmd+D / Cmd+Shift+D)
- Zoom toggle (Cmd+Shift+Z), equalize (Cmd+Shift+E)
- Global toggle visibility (Ctrl+`)
- Notifications when commands finish (3s threshold, unfocused only)
- Window state saved/restored

### 2. AppleScript Workspace Launcher

Created `~/Scripts/launch-workspace.applescript` — creates 4 Ghostty windows (3 Claude Code + 1 shell) in a 2x2 grid.

Created `~/Scripts/workspace` shell shortcut.

### 3. Install cmux

```tool
[Tool: Bash] brew install --cask cmux
```

cmux installed successfully.

**What you now have:**
- Ghostty config with keybindings, notifications, and window state
- AppleScript launcher for 2x2 workspace grid
- cmux for agent notification management

## User

can we have hotkey to run the workspace launcher

## Claude

The simplest way without installing anything new is a macOS Automator Quick Action with a keyboard shortcut.

**Which hotkey do you want for launching the workspace?**

## User

**Hotkey choice:** → Cmd+Shift+F1

## Claude

macOS Shortcuts can't be created programmatically. Here's what you need to do:

1. Open **Shortcuts.app** (Cmd+Space, type "Shortcuts")
2. Click **+** to create a new shortcut
3. Name it **"Launch Workspace"**
4. Add a **"Run Shell Script"** action
5. Paste: `osascript ~/Scripts/launch-workspace.applescript`
6. Click the shortcut name at the top → **Add Keyboard Shortcut** → press **Cmd+Shift+F1**

**Want me to test-run the workspace AppleScript now?**

## User

→ Yes, run it

## Claude

```tool
[Tool: Bash] osascript ~/Scripts/launch-workspace.applescript
```

First attempt failed: `execution error: Ghostty got an error: Can't set bounds of window id "tab-group-abac9b520" to {0, 25, 735.0, 455.5}. (-10006)`

Fixed the script to use System Events for positioning instead of Ghostty's bounds property.

```tool
[Tool: Bash] osascript ~/Scripts/launch-workspace.applescript
```

No errors. You should see 4 Ghostty windows tiled in a 2x2 grid right now.

Also added two useful config options from the research:
- `unfocused-split-opacity = 0.85` — dims inactive splits
- `focus-follows-mouse = true` — hovering over a split focuses it

## User

the command currently says "raw:claude" when it should say claude --dangerously-skip-permissions

## Claude

Updated all three Claude window configurations from `"raw:claude\n"` to `"raw:claude --dangerously-skip-permissions\n"`.

```tool
[Tool: Bash] osascript ~/Scripts/launch-workspace.applescript
```

No errors.

## User

not good, claude command still dont work, also how do i map this to the keybind again

[Screenshot showing terminals with error: `zsh: command not found: raw:claude`]

## Claude

The `raw:` prefix isn't being interpreted by AppleScript — it's being passed literally to the shell. Switched from `initial input` to using the `command` property on the surface configuration instead.

```tool
[Tool: Bash] osascript ~/Scripts/launch-workspace.applescript
```

For the **hotkey** (Cmd+Shift+F1):
1. Open **Shortcuts.app**
2. Click **+** to create a new shortcut
3. Name it **Launch Workspace**
4. Search for and add the **"Run Shell Script"** action
5. Replace the default script text with: `osascript ~/Scripts/launch-workspace.applescript`
6. Right-click the shortcut → **Add Keyboard Shortcut** → press **Cmd+Shift+F1**

## User

Ghostty failed to launch the requested command:

/usr/bin/login -flp jperr /bin/bash --noprofile --norc -c exec -l claude --dangerously-skip-permissions

Runtime: 33 ms

Press any key to close the window.

## Claude

The `claude` binary isn't in the default PATH. Found it at `/Users/jperr/.local/bin/claude`.

Updated all three command lines to use the full path: `/Users/jperr/.local/bin/claude --dangerously-skip-permissions`

```tool
[Tool: Bash] osascript ~/Scripts/launch-workspace.applescript
```

No errors.

## User

great. where does this script live on my computer? can we put it in documents under ghostty

## Claude

Moved both files to `~/Documents/ghostty/`:
- `launch-workspace.applescript` — the main script
- `workspace` — shell shortcut

Updated the workspace shell script to reference the new location.

When setting up the Shortcuts hotkey, use: `osascript ~/Documents/ghostty/launch-workspace.applescript`

## User

shortcuts is not really how you remember it i manually have to enter the script

[Screenshot showing Shortcuts "Run Shell Script" action with `echo "Hello World"` placeholder]

## Claude

Replace the `echo "Hello World"` with:

```
osascript ~/Documents/ghostty/launch-workspace.applescript
```

Then to add the keyboard shortcut: click the shortcut's name at the top of the editor, look for **"Add Keyboard Shortcut"** in the details panel on the right side, and press **Cmd+Shift+F1**.

## User

osascript: /var/root/Documents/ghostty/launch-workspace.applescript: No such file or directory

## Claude

The "Run as Administrator" checkbox is causing it to run as root, so `~` resolves to `/var/root/` instead of your home directory.

Two fixes — either:
1. **Uncheck "Run as Administrator"** (recommended), or
2. Use the full path: `/Users/jperr/Documents/ghostty/launch-workspace.applescript`

## User

/Users/jperr/Documents/ghostty/launch-workspace.applescript:1390:1402: execution error: System Events got an error: osascript is not allowed assistive access. (-25211)

## Claude

That error means macOS needs to grant accessibility permissions to the process running the script.

Go to **System Settings → Privacy & Security → Accessibility** and add:
- **Shortcuts** (if not already listed)
- **osascript** — it might show up as `/usr/bin/osascript`. If it doesn't appear automatically, click **+** and press Cmd+Shift+G to navigate to `/usr/bin/osascript`

Then try the shortcut again.

## User

/document this whole conversation. I am done documenting.

## Claude

Documenting and committing the full conversation.
