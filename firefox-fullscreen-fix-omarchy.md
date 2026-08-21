---
title: "Fixing Firefox Screensaver & Idle Issues on Fullscreen in Omarchy Quattro"
date: "2026-08-21"
excerpt: "I have recently updated to **Omarchy Quattro (v4.0)** and noticed that the screensaver kicks in or the screen locks while wathcing videos fullscreen on Firefox. If you have the same issue as me probably this post might help you too."
tags: ["omarchy", "firefox", "fullscreen", "idle", "issue", "config", "arch", "linux", "fix", "hyprland"]
---

I have recently updated to **Omarchy Quattro (v4.0)** and noticed that the screensaver kicks in or the screen locks while wathcing videos fullscreen on Firefox.
If you have the same issue as me probably this post might help you too.

---

### The Problem

In Omarchy Quattro, the desktop environment transitioned from standalone idle daemons like `hypridle` to a unified Quickshell-based idle service (`omarchy.idle`).

During this architecture shift and the migration of Hyprland configs to Lua, native Wayland idle inhibitor requests from browser video streams don't always propagate down to the shell's idle timer automatically. As a result, the system assumes you are idle and launches the screensaver mid-playback.

> **Disclaimer:** I am not familiar with the inner workings of the Omarchy in detail, but this is what my investigation and Gemini have explained to me.

---

### The Fix

You can explicitly enforce an idle inhibitor rule for Firefox inside your personal Hyprland Lua configuration.

1. Open your Hyprland configuration:
```bash
nvim ~/.config/hypr/hyprland.lua

```
Note here i am using nvim, you can use whatever editor you might want to.

2. Scroll down to the bottom of the file where personal overrides are defined and add:
```lua
-- Inhibit screensaver and sleep when Firefox is fullscreen
o.window("firefox", { idle_inhibit = "fullscreen" })

```

3. Save the file and reload Hyprland, it should auto reload, but to be certain lets run this command:
```bash
hyprctl reload

```
Once applied, fullscreen media playback will reliably hold off the screensaver without having to manually toggle sleep inhibitors from the top bar.
