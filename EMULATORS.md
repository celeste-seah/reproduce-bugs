# Emulators & simulators

Reproducing on the reporter's real device usually isn't possible, so an emulator/simulator (or a resized desktop browser, for "desktop" as a target) is how you get a *real* screenshot instead of just trusting the reporter's own. Set one up before Phase 2 rather than skipping straight to "not reproducible."

## Running from a cloud/remote session (Cursor, etc.)

A cloud agent VM (e.g. Cursor's cloud VMs) may not be able to run an Android emulator or iOS Simulator directly, Android needs KVM (hardware virtualization), iOS Simulator needs macOS/Xcode, and neither is guaranteed on a given cloud VM. Check the provider's current docs before assuming either works, don't take it on faith from this file.

**Workaround: Claude Code Remote Control.** Keep the emulator/simulator on a real Mac or devbox, and control that machine's Claude Code session from wherever you're actually working:

1. **On the Mac/devbox that has (or will have) the emulator/Simulator**: open a terminal in the project directory and run `claude remote-control` (long-running server) or `claude --remote-control` (interactive session). Accept the one-time "Enable Remote Control?" prompt. This prints a session URL (and a QR code if you press spacebar).
   - If the emulator/Simulator isn't set up on this machine yet, do that first, see the platform sections below.
   - Requires being signed in via `claude /login` with a claude.ai account (Pro/Max/Team/Enterprise — API keys aren't supported for this), and the machine needs to keep running (use `tmux`/`screen` if it's headless/over SSH, closing the terminal drops the session).
2. **From your other session** (Cursor cloud agent, browser, phone): open the session URL, or find the session by name at [claude.ai/code](https://claude.ai/code) (green dot = online), and drive the skill from there as normal. Tool calls, including `adb`/`xcrun simctl`/the iOS Simulator MCP tool, execute on the Mac, not on the remote session.

This isn't confirmed by Anthropic's docs to specifically cover GUI-driven simulator/emulator control (only that local tool execution in general works over Remote Control). Sanity-check with a trivial tool call (e.g. a Bash `echo`) before relying on it for a real repro session.

## General principles (any platform)

- **Boot before you drive it.** Confirm the device is actually ready (booted, network up) before sending input — a tap or navigation against a half-booted device silently does nothing or hits the wrong screen.
- **Prefer exact bounds over pixel-guessing.** Before tapping anything, dump the accessibility tree (`uiautomator dump` on Android, an accessibility/`read_page`-style snapshot elsewhere) and compute the target's centre from its reported bounds. Screenshot pixels are a last resort for elements the tree doesn't expose — and even then, a screenshot tool's reported scale factor is per-call, not a fixed device constant, so re-derive it fresh each time rather than reusing one from an earlier screenshot. Repeated taps built on a stale or guessed scale factor compound into wrong fields getting text, wrong buttons getting pressed, and several wasted retries.
- **A newly created shortcut may not appear on the screen you're looking at.** Installing a home-screen icon (e.g. Android's "Add to Home screen") can place it on a different launcher page than the one currently in view. Search other pages/screens before concluding the install silently failed.
- **Not every on-screen control is a real form element.** Some custom dialers/keyboards/buttons ignore synthetic text input entirely even though they look like inputs. If typed text isn't landing, tap the specific coordinates of each character/digit instead of trusting a text-injection action.
- **Native OS-level dialogs (permission prompts, system alerts) may not be tappable at all** through the automation surface available to you. If a tap genuinely won't register after a couple of tries, don't burn time on it — capture the dialog's appearance as your repro evidence and either pre-set the permission via a device-settings command or move on.
- **Substitute devices explicitly, don't fake exactness.** Emulator/simulator catalogues typically only ship reference hardware profiles (e.g. Android's AVD manager ships Google's own device definitions, not third-party OEM models). When asked to reproduce on a device with no matching profile, pick the closest available one on the same OS version, and say so plainly in the write-up — name the substitution made and why — rather than presenting a stand-in as an exact match.
- **Note the gap between "device available" and "exact match."** An emulator/simulator can usually get you close (right OS, right browser) but some real-device conditions can't be faithfully reproduced (e.g. app-store-gated install flows, real cellular network drops, actual hardware sensors). When you hit one of these, say so explicitly in the writeup rather than presenting an approximation as exact.
- **Cross-check the real clock when correlating time-sensitive credentials.** Hunting for a fresh OTP/2FA code sent by email or SMS by matching timestamps only works if you trust the right clock — a sandboxed environment's stated "today" can lag or lead its actual host wall-clock. Check the real time directly (e.g. `date -u`, `adb shell date`) before deciding a message is or isn't the one you're waiting for.

## Android (emulator via AVD)

**Check first**: `which avdmanager emulator adb` and `avdmanager list avd`. If an AVD already exists, skip to boot. Otherwise walk through the one-time install below.

```bash
# one-time install
brew install --cask android-commandlinetools
brew install openjdk   # formula not cask, avoids a sudo prompt

# env (persist across shell calls — Bash tool calls don't share state)
export JAVA_HOME=/opt/homebrew/opt/openjdk/libexec/openjdk.jdk/Contents/Home
export ANDROID_SDK_ROOT=/opt/homebrew/share/android-commandlinetools
export PATH="$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/platform-tools:$ANDROID_SDK_ROOT/emulator:$PATH"

# packages + device
sdkmanager --install "platform-tools" "emulator" "platforms;android-34" "system-images;android-34;google_apis;arm64-v8a"
avdmanager create avd -n <avd-name> -k "system-images;android-34;google_apis;arm64-v8a"

# boot + confirm ready
emulator -avd <avd-name> -no-snapshot -no-boot-anim &
adb devices                              # wait for "device", not "offline"
adb shell getprop sys.boot_completed      # wait for 1
```

Drive it with `adb shell input tap x y`, `adb shell input text "..."`, `adb exec-out screencap -p > file.png`, `adb shell am start -a android.intent.action.VIEW -d "<url>"`.

**Known gap**: the `google_apis` system image has no Play Store, so Chrome can only offer a plain "Add to Home screen" bookmark shortcut, not a true installed PWA/WebAPK, even when the site's manifest is fully valid. Note this explicitly wherever an "ideal/installed" repro is attempted on this image.

## iOS (Simulator)

**Check first**: call the iOS Simulator MCP tool's `attach` action if available, or `xcrun simctl list devices`, to see if a usable device already exists and is booted. If nothing's booted, boot one (`xcrun simctl boot <udid>`) before proceeding.

Use `mcp__Claude_Code_iOS_Simulator__control` if available, else `xcrun simctl` directly.

```bash
xcrun simctl list devices                    # find/boot a device, get its UDID
xcrun simctl io booted screenshot <path>.png  # real screenshot file, only way to get one
```

- Coordinate space for tap/swipe is the simulator's own point space, **not** the screenshot's pixel dimensions — check the screenshot's reported scale and divide/multiply accordingly.
- Native (SpringBoard-owned) permission alerts often can't be dismissed through simulated tap/touch input. If several attempts fail, treat the alert's appearance as sufficient repro evidence and use `xcrun simctl privacy <udid> grant|revoke <service> <bundle-id>` to pre-set permission state for further navigation, instead of continuing to fight the dialog.
- Chrome-for-iOS may not be installable in a fresh simulator without an App Store sign-in, if the reporter used Chrome on iPhone, check this before promising an exact repro, it may be infeasible and worth flagging as such rather than substituting Safari silently.

## Desktop

No emulator needed, use a real browser window, but still match what the reporter actually had:
- resize the viewport to their reported resolution/breakpoint if known, rather than testing full-screen by default
- match the browser (Chrome vs Safari vs Firefox) rather than defaulting to whichever is open
- OS-level differences (Windows vs macOS chrome/scrollbars/font rendering) are usually not worth reproducing unless the bug is specifically about them, note that as a scoping decision rather than silently skipping it
