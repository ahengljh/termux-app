# Termux App - Modified Fork

## Project Goal

We are modifying the termux-app to create a **one-click launcher** for a shell script. The target users are non-technical people who currently struggle with running commands in a pure shell. Instead of teaching them complex terminal commands, we provide a modified Termux that automatically runs a predefined script on launch, with a simplified UI.

## Architecture Overview

This is a multi-module Android project:

- **app/** - Main Termux application (Activity, Service, Installer)
- **terminal-view/** - Terminal UI rendering library (`TerminalView`, `TerminalRenderer`)
- **terminal-emulator/** - Terminal emulation engine with JNI (`TerminalSession`, `TerminalEmulator`)
- **termux-shared/** - Shared constants, utilities, settings for all Termux apps

## Key Entry Points & Flow

1. `TermuxApplication.onCreate()` - App init, bootstrap variant, properties, shell environment
2. `TermuxActivity.onCreate()` - Main UI, binds `TermuxService`
3. `TermuxActivity.onServiceConnected()` - Calls `TermuxInstaller.setupBootstrapIfNeeded()`, then creates terminal session
4. `TermuxTerminalSessionActivityClient.addNewSession()` - Creates shell session via `TermuxShellManager`
5. `TerminalSession` - Spawns shell process with PTY via JNI

## Key Files for UI Modification

- `app/src/main/res/layout/activity_termux.xml` - Main layout (DrawerLayout with TerminalView + side drawer + toolbar)
- `app/src/main/java/com/termux/app/TermuxActivity.java` - Main activity, UI initialization
- `app/src/main/java/com/termux/app/TermuxService.java` - Foreground service, session management
- `app/src/main/java/com/termux/app/TermuxInstaller.java` - Bootstrap installation
- `app/src/main/java/com/termux/app/terminal/TermuxTerminalSessionActivityClient.java` - Session callbacks
- `app/src/main/java/com/termux/app/terminal/TermuxTerminalViewClient.java` - Input/view event handling
- `app/src/main/java/com/termux/app/terminal/io/TermuxTerminalExtraKeys.java` - Extra keys toolbar
- `app/src/main/java/com/termux/app/RunCommandService.java` - Intent-based command execution

## Existing Customization Mechanisms

- **`~/.termux/termux.properties`** - Runtime config (extra-keys, theme, fullscreen, etc.)
- **`RunCommandService`** - Accepts `com.termux.RUN_COMMAND` intents with command path, args, stdin
- **App Shortcuts** (`res/xml/shortcuts.xml`) - Android launcher shortcuts
- **SharedPreferences** - Runtime settings via `TermuxAppSharedPreferences`

## Build Info

- Compile SDK: 36, Target SDK: 28, Min SDK: 21
- NDK: 29.0.14206865
- AGP: 8.13.2
- App Version: 0.118.0
- Debug signing: `app/testkey_untrusted.jks`
- Bootstrap variant: controlled by `TERMUX_PACKAGE_VARIANT` env var (default: `apt-android-7`)

## Commit Message Convention

Follow [Conventional Commits](https://www.conventionalcommits.org):
- Format: `<Type>[optional scope]: <Description>` (first letter capitalized, present tense)
- Types: **Added**, **Changed**, **Deprecated**, **Removed**, **Fixed**, **Security**
- Examples: `Added: Add one-click script launcher UI`, `Changed: Change default session to run startup script`

## Approach for Modifications

Our modifications focus on:
1. **UI changes** - Adding a button/launcher to the main activity for one-click script execution
2. **Auto-run on startup** - Optionally running the script automatically when the app starts
3. **Simplified interface** - Hiding complexity that end users don't need
4. Keeping the terminal accessible for advanced users who want it
