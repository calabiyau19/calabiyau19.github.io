---
layout: post
title: "Whisper AI self hosted complete installation guide"
draft: false
date: 2025-11-25
description: "Complete guide to download, setup and run Whisper AI on a Linux Mint (Ubuntu) laptop. I Will be adding actual performance results later since the installation allows you to increase the transcription accuracy for a trade off in slower speed supposedly. Have tried both the tiny and base models and they are both pretty fast. Definitely comparable to the Whisper AI online model. Note: this setup alsow gives you the abiltiy to explicitly set the number of processors to work om the translation part.  These instructions explicity set it to 8 threads, which you can change to whatever you want. "
---


The best benefit to using a locally hosted Whisper AI dictation model is that in addition to having all your data locally hosted, it can also be used in every application that you would normally type in - something Whisper AI online cannot do. Things that do not work in Whisper AI like Libra Office Writer or this program, VS Code, or even a simple text editor or a terminal, locally hosting Whisper AI will allow you to have dictation and transcription available for all those applications as well.

## Local Whisper Dictation Setup (Linux Mint 22.2)

**Project:** Install Whisper AI locally, so I can use it anytime I want in any application on any computer running on my home network.

**Project Goals:**

* Install needed tools
* Set up initial working dictation to text
* Real-time continuous dictation
* Type directly into the active window
* Global hotkey activation
* Test Better Whisper models - like base as opposed to tiny
* GUI app - dropped from project
* Integrate directly with ChatGPT / Claude Desktop - dropped from project, not needed
* Access Whisper from another machine on your LAN - dropped from project, use it exclusively on the laptop

---

### 0. (Optional but recommended) Update system

Not strictly required for Whisper, but recommended as a first step.

 ```sh
sudo apt update && sudo apt upgrade
 ```

---

### 1. Verify Python and pip

Check if `python3` and `pip` exist:

 ```sh
python3 --version && pip --version
 ```

**Expected results:**

* `python3` should be present (Python 3.12.3 or similar)
* `pip` may be missing

If `pip` is missing, install `python3-pip`:

 ```sh
sudo apt install -y python3-pip
 ```

---

### 2. Install `pipx` (because of PEP 668 / externally-managed-env)

Direct `pip install` into system Python will give this error:

 ```text
error: externally-managed-environment … See PEP 668 …
 ```

**Correct response:** Use `pipx`so we don't break Mint's system Python.

Install `pipx`:

 ```sh
sudo apt install -y pipx
 ```

---

### 3. Install Whisper (ctranslate2) via `pipx`

This gives you a fast, local Whisper CLI in an isolated environment:

 ```sh
pipx install whisper-ctranslate2
 ```

After this, the `whisper-ctranslate2` command is globally available.

---

### 4. Install audio + clipboard tools

We need:

* `arecord` (from `alsa-utils`) to record from your mic
* `xclip` to push the transcript into the X11 clipboard

Install both:

 ```sh
sudo apt install -y alsa-utils xclip
 ```

These may already be installed on your system, but this ensures they're present.

---

### 5. Ensure you have a `~/Scripts` directory

Create the directory if it doesn't exist:

 ```sh
mkdir -p ~/Scripts
 ```

All Whisper scripts will be stored here.

---

### 6. Create the dictation script

Open the script in `nano`:

 ```sh
nano ~/Scripts/whisper_dictate.sh
 ```

Paste this exact content:

 ```sh
#!/usr/bin/env bash
set -e

DURATION="${1:-30}"          # seconds to record; override with: ./whisper_dictate.sh 45
OUT_WAV="/tmp/whisper_dictate.wav"
OUT_TXT="/tmp/whisper_dictate.txt"

echo "Recording for $DURATION seconds... (speak now)"
arecord -q -f cd -t wav -d "$DURATION" -r 16000 -c 1 "$OUT_WAV"

echo "Transcribing with Whisper (tiny, English)..."
whisper-ctranslate2 \
  --model tiny \
  --language en \
  --output_format txt \
  --output_dir /tmp \
  "$OUT_WAV"

# If the expected output file doesn't exist, try to locate it
if [ ! -f "$OUT_TXT" ]; then
  CANDIDATE="$(ls /tmp | grep -E '^whisper_dictate.*\.txt$' | head -n 1 2>/dev/null || true)"
  if [ -n "$CANDIDATE" ]; then
    OUT_TXT="/tmp/$CANDIDATE"
  fi
fi

if [ -f "$OUT_TXT" ]; then
  xclip -selection clipboard < "$OUT_TXT"
  echo
  echo "Transcript (also copied to clipboard):"
  echo "--------------------------------------"
  cat "$OUT_TXT"
  echo
  echo "You can now paste into ChatGPT, Claude, Word, LibreOffice, Google Docs, etc."
else
  echo "Error: transcript file not found." >&2
fi
 ```

Then in nano:

* **Ctrl+O**, **Enter** to save
* **Ctrl+X** to exit

---

### 7. Make the script executable

 ```sh
chmod +x ~/Scripts/whisper_dictate.sh
 ```

---

### 8. Usage: test 5-second dictation

From any terminal:

 ```sh
~/Scripts/whisper_dictate.sh 5
 ```

**What this does:**

1. Records 5 seconds from your default microphone:

```text
   Recording for 5 seconds... (speak now)
```

2. Runs local Whisper (`tiny` model, English) via `whisper-ctranslate2`.

3. Finds the output `.txt` file in `/tmp`.

4. Copies transcript into the clipboard with `xclip`.

5. Prints transcript to the terminal, e.g.:

 ```text
   Transcript (also copied to clipboard):
   --------------------------------------
   Testing 1, 2, 3, 4, 5, 6, 7, 8, 9, 10.
 ```

You can then paste into:

* ChatGPT / Claude / other web chat boxes
* LibreOffice Writer
* Word (via Wine or on another machine if clipboard shared)
* Google Docs in the browser

---

## Section 2 – Continuous, Chunked Dictation with Whisper

This adds a second script that:

* Records in repeating chunks (default: 10 seconds each)
* Transcribes each chunk with Whisper locally
* Appends each chunk to a session transcript
* Copies the full running transcript to your clipboard after every chunk
* Stops cleanly with Ctrl+C

All scripts live in `~/Scripts`.

---

### 2.1 Create `whisper_continuous.sh`

Open a new script in `nano`:

 ```sh
nano ~/Scripts/whisper_continuous.sh
 ```

Paste this content into the file:

 ```sh
#!/usr/bin/env bash
set -e

# Usage:
#   ./whisper_continuous.sh          # 10s chunks, tiny model
#   ./whisper_continuous.sh 15       # 15s chunks, tiny model
#   ./whisper_continuous.sh 10 base  # 10s chunks, base model
#
# Press Ctrl+C to stop recording/transcribing.

CHUNK_SECONDS="${1:-10}"           # length of each recording chunk
MODEL="${2:-tiny}"                 # whisper model: tiny, base, small, medium, large-v3, etc.

OUT_DIR="/tmp/whisper_continuous"
mkdir -p "$OUT_DIR"

SESSION_TS="$(date +%Y%m%d_%H%M%S)"
SESSION_TXT="${OUT_DIR}/session_${SESSION_TS}.txt"

echo "Whisper continuous dictation"
echo "  Chunk length : ${CHUNK_SECONDS} seconds"
echo "  Model        : ${MODEL}"
echo "  Session file : ${SESSION_TXT}"
echo
echo "Press Ctrl+C at any time to stop."
echo

trap 'echo; echo "Stopping. Final transcript saved at: ${SESSION_TXT}"; exit 0' INT

i=1
while true; do
  WAV="${OUT_DIR}/chunk_${SESSION_TS}_${i}.wav"
  TXT="${OUT_DIR}/chunk_${SESSION_TS}_${i}.txt"

  echo
  echo "Chunk #${i} - recording ${CHUNK_SECONDS} seconds... (speak now)"
  arecord -q -f cd -t wav -d "${CHUNK_SECONDS}" -r 16000 -c 1 "${WAV}"

  echo "Transcribing chunk #${i} with Whisper..."
  whisper-ctranslate2 \
    --model "${MODEL}" \
    --language en \
    --output_format txt \
    --output_dir "${OUT_DIR}" \
    "${WAV}"

  if [ -f "${TXT}" ]; then
    CHUNK_TEXT="$(cat "${TXT}")"
    echo "${CHUNK_TEXT}" >> "${SESSION_TXT}"

    # Copy entire running transcript to clipboard
    xclip -selection clipboard < "${SESSION_TXT}"

    echo
    echo "Last chunk:"
    echo "-----------"
    echo "${CHUNK_TEXT}"
    echo
    echo "Running transcript (also copied to clipboard):"
    echo "----------------------------------------------"
    cat "${SESSION_TXT}"
    echo
    echo "(You can paste this into ChatGPT, Claude, Word, LibreOffice, Google Docs, etc.)"
  else
    echo "Warning: transcript file not found for chunk #${i} (${TXT})." >&2
  fi

  i=$((i + 1))
done
 ```

Save and exit:

* **Ctrl+O**, **Enter**
* **Ctrl+X**

---

### 2.2 Make the script executable

 ```sh
chmod +x ~/Scripts/whisper_continuous.sh
 ```

---

### 2.3 Usage examples

**Default continuous dictation (10s chunks, `tiny` model):**

 ```sh
~/Scripts/whisper_continuous.sh
 ```

* Records chunk #1 for 10 seconds, transcribes, prints:
  * "Last chunk" (the latest 10 seconds)
  * "Running transcript" (all chunks so far)
* Copies the full running transcript to the clipboard.
* Immediately starts chunk #2, then #3, etc.
* Press **Ctrl+C** to stop.
* Final transcript is saved as:

 ```sh
/tmp/whisper_continuous/session_YYYYMMDD_HHMMSS.txt
 ```

**Change chunk length (e.g., 15 seconds):**

 ```sh
~/Scripts/whisper_continuous.sh 15
 ```

**Use a different Whisper model (e.g., `base` instead of `tiny`):**

 ```sh
~/Scripts/whisper_continuous.sh 10 base
 ```

---

## Section 3 — Whisper Dictation That Types Directly Into the Active Window

This script allows Whisper to "type" the transcribed text into whatever window you have focused. Works with: ChatGPT, Claude, LibreOffice, Word, Google Docs, any browser field, any terminal, anything with a text cursor.

---

### 3.1 Install typing tool: xdotool

 ```sh
sudo apt install -y xdotool
 ```

(This will show "already the newest version" if previously installed.)

---

### 3.2 Create the typing dictation script

Open the script:

 ```sh
nano ~/Scripts/whisper_type.sh
 ```

Paste the following content:

 ```sh
#!/usr/bin/env bash
set -e

# Usage:
#   ./whisper_type.sh              # 10s, tiny model
#   ./whisper_type.sh 5            # 5s, tiny model
#   ./whisper_type.sh 8 base       # 8s, base model
#
# IMPORTANT:
#   - Click in the target window (ChatGPT, Claude, Word, LibreOffice, Google Docs, etc.)
#   - Make sure the text cursor is where you want the text
#   - Then run this script from a terminal
#   - When recording finishes, the script will "type" the text into the active window

DURATION="${1:-10}"          # seconds to record
MODEL="${2:-tiny}"           # whisper model (tiny, base, small, etc.)

OUT_WAV="/tmp/whisper_type.wav"
OUT_TXT="/tmp/whisper_type.txt"

echo "Recording for ${DURATION} seconds... (speak now)"
arecord -q -f cd -t wav -d "${DURATION}" -r 16000 -c 1 "${OUT_WAV}"

echo "Transcribing with Whisper (${MODEL}, English)..."
whisper-ctranslate2 \
  --model "${MODEL}" \
  --language en \
  --output_format txt \
  --output_dir /tmp \
  "${OUT_WAV}"

if [ ! -f "${OUT_TXT}" ]; then
  # Try to locate a matching txt file if the expected one isn't there
  CANDIDATE="$(ls /tmp | grep -E '^whisper_type.*\.txt$' | head -n 1 2>/dev/null || true)"
  if [ -n "${CANDIDATE}" ]; then
    OUT_TXT="/tmp/${CANDIDATE}"
  fi
fi

if [ -f "${OUT_TXT}" ]; then
  TEXT="$(cat "${OUT_TXT}")"

  echo
  echo "Transcript:"
  echo "-----------"
  echo "${TEXT}"
  echo
  echo "Typing transcript into the active window using xdotool..."
  echo "(Make sure your cursor is in the target text box.)"
  echo

  # Type the text into the currently active window
  xdotool type --delay 0 "${TEXT}"

  echo
  echo "Done."
else
  echo "Error: transcript file not found (${OUT_TXT})." >&2
fi
 ```

Save and exit:

* **Ctrl+O**, **Enter**
* **Ctrl+X**

---

### 3.3 Make it executable

 ```sh
chmod +x ~/Scripts/whisper_type.sh
 ```

---

### 3.4 Usage

1. Open any text box anywhere (ChatGPT, Claude, LibreOffice, Google Docs, etc.).
2. Run the script from your terminal:

 ```sh
~/Scripts/whisper_type.sh 5
 ```

3. Immediately click into your target window and leave the text cursor there.
4. Speak while it records.
5. When recording completes, Whisper transcribes and `xdotool` types your words directly into the active window.

**Test results:**

* One 5-second recording
* Whisper correctly transcribed
* The script printed the text in your terminal
* Then it successfully typed the same text into your target application

This confirms Section 3 is complete.

---

## Section 4 – Global Hotkey for Whisper Typing (Ctrl+Alt+W)

This section wires your `whisper_type.sh` script to a global keyboard shortcut, so you can start a short dictation and have it typed into the active window with a single key combo.

---

### 4.1 Ensure the script exists and is executable

(Already done in Section 3, but for completeness.)

 ```sh
nano ~/Scripts/whisper_type.sh
chmod +x ~/Scripts/whisper_type.sh
 ```

The `whisper_type.sh` script content is documented in Section 3.2 above.

---

### 4.2 Open Keyboard Shortcuts

From a terminal:

 ```sh
cinnamon-settings keyboard
 ```

Then:

1. Go to the **Shortcuts** tab.
2. In the left "Categories" pane, click **Custom Shortcuts**.

---

### 4.3 Add the custom shortcut

1. Click **Add custom shortcut**.
2. In the dialog:
   * **Name:**

 ```text
   Whisper Type
 ```

* **Command:**

 ```sh
   bash -lc "~/Scripts/whisper_type.sh 10"
 ```

3. Click **Add**.

You will now see **Whisper Type** in the list, with **unassigned** as its keybinding.

---

### 4.4 Assign Ctrl+Alt+W

1. Click on the word **unassigned** next to **Whisper Type**.
2. When it says **Pick an accelerator…**, press:

 ```text
   Ctrl + Alt + W
 ```

3. It should now display: **Ctrl+Alt+W**.

This finishes the hotkey setup for the one-shot typing script.

---

### 4.5 Using the hotkey

1. Click into any text box (ChatGPT, Claude, LibreOffice, Google Docs, etc.) so the text cursor is where you want the dictation to appear.
2. Press **Ctrl+Alt+W**.
3. Speak for ~10 seconds (or whatever duration you configured).
4. When the script finishes:
   * Whisper transcribes your audio locally.
   * `xdotool` types the text directly into the active window.
5. The script exits automatically — no separate "stop" key is required for this short, one-shot mode.

---

## Section 5 – Whisper-AI-Style Dictation (Start/Stop Hotkeys)

This section creates a true Whisper-AI-style dictation system with two separate hotkeys:

* **Ctrl+Alt+Q** = Start recording
* **Ctrl+Alt+Z** = Stop recording and immediately transcribe

Unlike the previous sections which used fixed-duration recording, this system lets you:

* Record for as long as you need (up to 60 seconds max)
* Stop whenever you're done speaking
* Get instant transcription typed into your active window
* Hear audio feedback (bell sounds) when starting and stopping

---

### 5.1 Create the START script

Open a new script in `nano`:

 ```sh
nano ~/Scripts/start_whisper_simple.sh
 ```

Paste this content:

 ```sh
#!/usr/bin/env bash
set -e

# Start recording (Option A - Whisper-AI style)
# Bound to: Ctrl+Alt+Q

PIDFILE="/tmp/whisper_recording.pid"
AUDIO_FILE="/tmp/whisper_recording.wav"

# Don't start if already recording
if [ -f "$PIDFILE" ] && ps -p "$(cat "$PIDFILE")" >/dev/null 2>&1; then
    echo "Already recording."
    exit 0
fi

# Play start sound (optional - comment out if you don't want it)
paplay --volume=32768 /usr/share/sounds/freedesktop/stereo/bell.oga 2>/dev/null || true

# Start recording in background (max 60 seconds)
# -f cd = CD quality
# -t wav = WAV format
# -d 60 = max 60 seconds (will be killed early by stop script)
# -r 16000 = 16kHz sample rate (what Whisper expects)
# -c 1 = mono
arecord -q -f cd -t wav -d 60 -r 16000 -c 1 "$AUDIO_FILE" &

PID=$!
echo "$PID" > "$PIDFILE"

echo "Recording started (PID: $PID)"
 ```

Save and exit:

* **Ctrl+O**, **Enter**
* **Ctrl+X**

Make it executable:

 ```sh
chmod +x ~/Scripts/start_whisper_simple.sh
 ```

**What this script does:**

1. Checks if a recording is already in progress (prevents double-recording)
2. Plays a bell sound at 50% volume as audio feedback
3. Starts `arecord` in the background to record audio (max 60 seconds)
4. Saves the process ID (PID) so the stop script can kill it
5. Exits immediately, leaving recording running in background

---

### 5.2 Create the STOP script

Open a new script in `nano`:

 ```sh
nano ~/Scripts/stop_whisper_simple.sh
 ```

Paste this content:

 ```sh
#!/usr/bin/env bash
set -e
export HF_HUB_OFFLINE=1

# Stop recording and transcribe (Option A - Whisper-AI style)
# Bound to: Ctrl+Alt+Z

PIDFILE="/tmp/whisper_recording.pid"
AUDIO_FILE="/tmp/whisper_recording.wav"
TEXT_FILE="/tmp/whisper_recording.txt"

# Check if recording is running
if [ ! -f "$PIDFILE" ]; then
    echo "No recording in progress."
    exit 0
fi

PID="$(cat "$PIDFILE")"

# Kill the recording process if it's still running
if ps -p "$PID" >/dev/null 2>&1; then
    kill "$PID" 2>/dev/null || true
    sleep 0.2  # Give arecord time to finish writing the file
fi

rm -f "$PIDFILE"

# Play stop sound (optional - comment out if you don't want it)
paplay --volume=32768 /usr/share/sounds/freedesktop/stereo/bell.oga 2>/dev/null || true

# Check if we have an audio file to transcribe
if [ ! -f "$AUDIO_FILE" ] || [ ! -s "$AUDIO_FILE" ]; then
    echo "No audio file found or file is empty."
    exit 1
fi

echo "Transcribing..."

# Transcribe with Whisper
whisper-ctranslate2 \
    --model base \
    --language en \
    --device cpu \
    --threads 8 \
    --output_format txt \
    --output_dir /tmp \
    "$AUDIO_FILE" >/dev/null 2>&1

# Check if transcription succeeded
if [ ! -f "$TEXT_FILE" ]; then
    echo "Error: Transcription failed (no text file generated)."
    exit 1
fi

# Get the transcript
TRANSCRIPT="$(cat "$TEXT_FILE")"

if [ -z "$TRANSCRIPT" ]; then
    echo "Warning: Transcript is empty."
    exit 0
fi

echo "Transcript: $TRANSCRIPT"
echo "Typing into active window..."

# Type into the active window
xdotool type --delay 0 "$TRANSCRIPT"

# Clean up temp files
rm -f "$AUDIO_FILE" "$TEXT_FILE"

echo "Done."
 ```

Save and exit:

* **Ctrl+O**, **Enter**
* **Ctrl+X**

Make it executable:

 ```sh
chmod +x ~/Scripts/stop_whisper_simple.sh
 ```

**What this script does:**

1. Checks if a recording is in progress
2. Kills the `arecord` process to stop recording
3. Plays a bell sound as audio feedback
4. Verifies the audio file exists and has content
5. Transcribes using Whisper with the cached model (offline mode)
6. Types the transcript into whatever window has focus
7. Cleans up all temporary files

**Critical performance fix:** The line `export HF_HUB_OFFLINE=1` forces Whisper to use the cached model without checking Hugging Face servers online. Without this line, transcription takes 25+ seconds. With it, transcription takes 2-3 seconds.

---

### 5.3 Test the scripts manually (before hotkeys)

Before binding to hotkeys, test manually to verify everything works:

1. Open any text editor (gedit, xed, LibreOffice Writer, etc.)
2. Click in the text field so your cursor is positioned
3. In a terminal, run:

 ```sh
~/Scripts/start_whisper_simple.sh
 ```

You should hear a bell sound and see: `Recording started (PID: xxxx)`

4. Speak for 5-10 seconds
5. Run:

 ```sh
~/Scripts/stop_whisper_simple.sh
 ```

You should:

* Hear another bell sound
* See: `Transcribing...`
* See: `Transcript: [your words]`
* See: `Typing into active window...`
* See the text appear in your text editor

If this works, proceed to hotkey setup.

---

### 5.4 Set up keyboard shortcuts

Open Keyboard Settings from the terminal:

 ```sh
cinnamon-settings keyboard
 ```

Then:

1. Click the **Shortcuts** tab
2. In the left "Categories" pane, click **Custom Shortcuts**
3. Click **Add custom shortcut**

#### Create the START shortcut

In the dialog:

* **Name:** `Whisper Start Recording`
* **Command:** `bash -lc "~/Scripts/start_whisper_simple.sh"`
* Click **Add**

Now bind it to a key:

1. Click on the word **unassigned** next to "Whisper Start Recording"
2. When prompted, press: **Ctrl + Alt + Q**
3. You should see it change to: `Ctrl+Alt+Q`

#### Create the STOP shortcut

Click **Add custom shortcut** again.

In the dialog:

* **Name:** `Whisper Stop and Transcribe`
* **Command:** `bash -lc "~/Scripts/stop_whisper_simple.sh"`
* Click **Add**

Now bind it to a key:

1. Click on the word **unassigned** next to "Whisper Stop and Transcribe"
2. When prompted, press: **Ctrl + Alt + Z**
3. You should see it change to: `Ctrl+Alt+Z`

Close the Keyboard Settings window. Your hotkeys are now active.

---

### 5.5 Using the hotkey dictation system

**Basic workflow:**

1. Click in any text field where you want the dictation to appear:
   * ChatGPT or Claude web interface
   * LibreOffice Writer
   * Google Docs
   * Any text editor
   * Any browser text field
   * Terminal applications

2. Press **Ctrl+Alt+Q** to start recording
   * You'll hear a bell sound
   * Recording begins immediately
   * You can speak for up to 60 seconds

3. When done speaking, press **Ctrl+Alt+Z** to stop
   * You'll hear another bell sound
   * Whisper transcribes your audio (1-2 seconds)
   * Text is automatically typed into your active window
   * All temp files are cleaned up

**Important notes:**

* The focus must remain in your target text field for the typing to work correctly
* If you switch windows after pressing Ctrl+Alt+Z, the text will type into whatever window has focus when transcription completes
* Maximum recording time is 60 seconds (script will auto-stop if you reach this limit)
* You can record for as little as 1 second - there's no minimum

---

### 5.6 Performance notes

**Transcription speed:**

* 10 seconds of speech = ~2.0 seconds to transcribe
* 30 seconds of speech = ~3.0-4.0 seconds to transcribe
* 60 seconds of speech = ~6.0-8.0 seconds to transcribe

These times are for the `base` model on CPU with 8 threads.

**Why offline mode is critical:**

The line `export HF_HUB_OFFLINE=1` in the stop script prevents Whisper from checking Hugging Face servers online before transcribing. Without this line:

* Transcription takes 25-35 seconds (unacceptable)
* Only ~6 seconds of actual CPU work, rest is network overhead
* Multiple CPU cores sit idle

With offline mode enabled:

* Transcription takes 2-3 seconds (excellent)
* CPU cores properly utilized during transcription
* No network calls = faster and more reliable

The Whisper model is already cached at:

 ```text
~/.cache/huggingface/hub/models--Systran--faster-whisper-base/
 ```

Offline mode simply tells Whisper to use this cache without verifying online.

---

### 5.7 Adjusting bell sound volume

The bell sounds use `--volume=32768` which is 50% system volume. If this is:

**Too loud:** Change both scripts to use a lower volume:

 ```sh
paplay --volume=16384 /usr/share/sounds/freedesktop/stereo/bell.oga 2>/dev/null || true
 ```

**Too quiet:** Change both scripts to use a higher volume:

 ```sh
paplay --volume=49152 /usr/share/sounds/freedesktop/stereo/bell.oga 2>/dev/null || true
 ```

**Remove sounds entirely:** Comment out or delete the `paplay` lines in both scripts.

The volume scale is:

* 0 = silent
* 32768 = 50% volume
* 65536 = 100% volume

---

### 5.8 Cleanup: Remove old test scripts

If you followed earlier sections, you may have old test scripts that are no longer needed. Keep only these three Whisper scripts:

**Scripts to KEEP:**

* `start_whisper_simple.sh` - START recording (Ctrl+Alt+Q)
* `stop_whisper_simple.sh` - STOP and transcribe (Ctrl+Alt+Z)
* `whisper_type.sh` - Original one-shot script (useful for manual testing)

**Scripts to DELETE** (if they exist):

 ```sh
cd ~/Scripts
rm -f start_whisper.sh stop_whisper.sh whisper_continuous.sh whisper_toggle.sh whisper_forever_test.sh whisper_60s.sh whisper_dictate.sh
 ```

**Clean up old temp directories:**

 ```sh
rm -rf /tmp/whisper_continuous
 ```

The new scripts automatically clean up their own temp files in `/tmp` after each use.

---

### 5.9 Troubleshooting

**Problem: Bell sound doesn't play**

Check if the sound file exists:

 ```sh
ls -l /usr/share/sounds/freedesktop/stereo/bell.oga
 ```

If missing, you can either:

* Install the freedesktop sound theme: `sudo apt install sound-theme-freedesktop`
* Or comment out the `paplay` lines in both scripts

**Problem: "Already recording" message when starting**

A stale PID file exists. Clean it up:

 ```sh
rm -f /tmp/whisper_recording.pid
 ```

**Problem: Text types into wrong window**

Make sure your target text field has focus when you press Ctrl+Alt+Z. The text types into whatever window is active when transcription completes.

**Problem: Transcription takes 20+ seconds**

The `HF_HUB_OFFLINE=1` line is missing from the stop script. Edit the script:

 ```sh
nano ~/Scripts/stop_whisper_simple.sh
 ```

Make sure this line appears near the top (around line 3):

 ```sh
export HF_HUB_OFFLINE=1
 ```

**Problem: No sound recorded**

Check your default audio device:

 ```sh
arecord -l
 ```

If your microphone is not the default device, you may need to specify the device in the start script. See `arecord --help` for device selection options.

---

### 5.10 Future enhancements

Possible improvements to consider:

**Use an even better model:**
Change `--model base` to `--model small` in the stop script for even better transcription accuracy (adds ~2-3 seconds to transcription time). Note: You're currently using the `base` model which offers a good balance of speed and accuracy.

**Add visual notifications:**
Replace or supplement the bell sounds with desktop notifications showing "Recording started" and "Transcribing..." messages.

**Increase max recording time:**
Change `-d 60` to `-d 120` in the start script to allow 2-minute recordings instead of 60 seconds.

**Add a cancel key:**
Create a third script that kills recording without transcribing, bound to something like Ctrl+Alt+X.

---

## Summary

You now have a complete local Whisper dictation system with multiple usage modes:

1. **Clipboard mode** - `whisper_dictate.sh` records and copies to clipboard
2. **Chunked mode** - `whisper_continuous.sh` records continuous chunks
3. **One-shot typing** - `whisper_type.sh` or Ctrl+Alt+W for 10-second quick dictation
4. **Whisper-AI-style** - Ctrl+Alt+Q (start) and Ctrl+Alt+Z (stop) for flexible dictation

All scripts use local processing - no internet required for transcription. The system works in any application on your Linux Mint system.
