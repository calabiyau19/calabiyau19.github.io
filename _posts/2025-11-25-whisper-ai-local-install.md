---
layout: post
title: "Whisper AI self hosted complete installation guide"
draft: false
date: 2025-11-25
last_modified_at: 2026-06-13
description: "Complete guide to download, set up, and run Whisper AI on a Linux Mint (Ubuntu) laptop. Have tried the tiny, base and small models, and they are close in speed to Whisper AI. The small model is a little slower in my testing but very accurate. Note: this setup also gives you the ability to explicitly set the number of processors to work on the translation part.  These instructions explicitly set it to 8 threads, which you can change to whatever you want."
---

**NOTE: IF YOU JUST WANT TO INSTALL WHISPER LOCALLY AND HAVE IT WORK SIMILARLY TO WHISPER AI ONLINE, SKIP TO [Section 5](#section-5--whisper-ai-style-dictation-startstop-hotkeys).  SECTIONS 1-4 DOCUMENT THE PROCESS THAT I WENT THROUGH FROM KNOWING NOTHING ABOUT SELF HOSTING A WHISPER VERSION TO GETTING A FUNCTIONING VERSION RUNNING ON MY LAPTOP.**

The benefits to using a locally hosted Whisper AI dictation model are that your data is locally hosted, and that it can also be used in every application that you would normally type in - something Whisper AI online cannot do. Things that do not work in Whisper AI, like Libra Office Writer or this program, VS Code, a simple text editor or a terminal, Whisper Local will allow you to dictate and transcribe to all those applications as well.

## Local Whisper Dictation Setup (Linux Mint 22.2)

**Project:** Install Whisper AI locally, so it can be used in any application running on management PC.  Note: After installing and running Whisper Local, I realized that all the many servers available on the home network through my laptop (SSH terminals and web access pages) would also have Whisper Local available for dictation and transcription - through the laptop since that is how I access them all. This was a bonus.  

**Project Goals:**

* Install needed tools
* Set up initial working dictation to text
* Real-time continuous dictation
* Type directly into the active window
* Global hotkey activation
* Test Better Whisper models - like the base and small models in addition to the tiny model

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

  echo "Transcribing..."
  whisper-ctranslate2 \
    --model "${MODEL}" \
    --language en \
    --output_format txt \
    --output_dir "${OUT_DIR}" \
    "${WAV}"

  if [ -f "$TXT" ]; then
    cat "${TXT}" >> "${SESSION_TXT}"
    echo " " >> "${SESSION_TXT}"
    echo
    echo "--- Full transcript so far ---"
    cat "${SESSION_TXT}"
    echo "--- end ---"
    echo
    xclip -selection clipboard < "${SESSION_TXT}"
    echo "(Copied to clipboard.)"
  fi

  i=$((i + 1))
done
 ```

Then:

* **Ctrl+O**, **Enter** to save
* **Ctrl+X** to exit

---

### 2.2 Make it executable

 ```sh
chmod +x ~/Scripts/whisper_continuous.sh
 ```

---

### 2.3 Usage

Record 10-second chunks with the `base` model:

 ```sh
~/Scripts/whisper_continuous.sh 10 base
 ```

This will loop indefinitely, recording + transcribing 10-second chunks and appending to a session file. Press **Ctrl+C** to stop.

---

## Section 3 – One-Shot Typing with Hotkey

This is a short script that:

* Records for 10 seconds
* Transcribes (no display)
* Types the result directly into your active window

Bound to a global hotkey like **Ctrl+Alt+W**.

---

### 3.1 Create `whisper_type.sh`

Open a new script:

 ```sh
nano ~/Scripts/whisper_type.sh
 ```

Paste this:

 ```sh
#!/usr/bin/env bash
set -e

AUDIO_FILE="/tmp/whisper_type.wav"
TEXT_FILE="/tmp/whisper_type.txt"

# Record 10 seconds from microphone
arecord -q -f cd -t wav -d 10 -r 16000 -c 1 "$AUDIO_FILE"

# Transcribe with tiny model
whisper-ctranslate2 \
  --model tiny \
  --language en \
  --output_format txt \
  --output_dir /tmp \
  "$AUDIO_FILE" >/dev/null 2>&1

# If transcription succeeded, type it
if [ -f "$TEXT_FILE" ]; then
  TRANSCRIPT="$(cat "$TEXT_FILE")"
  xdotool type "$TRANSCRIPT"
  rm -f "$AUDIO_FILE" "$TEXT_FILE"
fi
 ```

Save and exit.

---

### 3.2 Make it executable

 ```sh
chmod +x ~/Scripts/whisper_type.sh
 ```

---

### 3.3 Bind to a hotkey

In **Cinnamon** (default on Mint):

1. Open **Settings** → **Keyboard** → **Shortcuts** → **Custom Shortcuts**
2. Click **+** to add a new shortcut
3. Name it: `Whisper Type`
4. Command: `bash -lc ~/Scripts/whisper_type.sh`
5. Bind it to: **Ctrl+Alt+W** (or any key combo you prefer)
6. Click **Add**

Now pressing **Ctrl+Alt+W** anywhere will:

* Record 10 seconds of speech
* Transcribe with the `tiny` model
* Type the result into your active window (web form, text editor, terminal, etc.)

---

## Section 4 – Advanced: Microphone Debugging

If Whisper doesn't record, the problem is usually the microphone or ALSA configuration.

### 4.1 List your audio devices

 ```sh
arecord -l
 ```

You'll see output like:

 ```text
**** PLAYBACK Devices ****
card 0: PCH [HDA Intel PCH], device 0: ALC295 Analog [ALC295 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0

**** CAPTURE Devices ****
card 0: PCH [HDA Intel PCH], device 0: ALC295 Analog [ALC295 Analog]
```

Your microphone is the **CAPTURE** device.

### 4.2 Test recording (5 seconds)

 ```sh
arecord -f cd -t wav -d 5 -r 16000 -c 1 /tmp/test.wav
 ```

You'll see:

 ```text
Recording WAVE '/tmp/test.wav' : Signed 16 bit Little Endian, Rate 16000 Hz, Mono
```

Speak into your mic for 5 seconds. When done:

 ```sh
aplay /tmp/test.wav
 ```

You should hear your recording played back.

---

## Section 5 – Whisper-AI-Style Dictation with Start/Stop Hotkeys

This is the "production" setup. Two hotkeys:

* **Ctrl+Alt+Q** = START recording
* **Ctrl+Alt+Z** = STOP, transcribe, and type into active window

No display, no dialogs. Works anywhere.

---

### 5.1 Prerequisites

Before starting, you need:

* `xdotool` (to type into active windows)
* `arecord` (already installed from Section 1)
* `paplay` (for audio feedback)

Install them:

 ```sh
sudo apt install -y xdotool pulseaudio-utils
 ```

---

### 5.2 Create the start script

Open a new script:

 ```sh
nano ~/Scripts/start_whisper_simple.sh
 ```

Paste this:

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
paplay --volume=34406 /usr/share/sounds/freedesktop/stereo/bell.oga 2>/dev/null || true

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

Save and exit.

---

### 5.3 Create the stop script

Open a new script:

 ```sh
nano ~/Scripts/stop_whisper_simple.sh
 ```

Paste this:

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
paplay --volume=34406 /usr/share/sounds/freedesktop/stereo/bell.oga 2>/dev/null || true

# Check if we have an audio file to transcribe
if [ ! -f "$AUDIO_FILE" ] || [ ! -s "$AUDIO_FILE" ]; then
    echo "No audio file found or file is empty."
    exit 1
fi

echo "Transcribing..."

# Transcribe with Whisper
whisper-ctranslate2 \
    --model small \
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
TRANSCRIPT="$(cat "$TEXT_FILE" | tr '\n' ' ')"
TRANSCRIPT="${TRANSCRIPT## }"  # Remove leading spaces
TRANSCRIPT="${TRANSCRIPT%% }"  # Remove trailing spaces

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

Save and exit.

---

### 5.4 Make both scripts executable

 ```sh
chmod +x ~/Scripts/start_whisper_simple.sh ~/Scripts/stop_whisper_simple.sh
 ```

---

### 5.5 Bind the hotkeys in Cinnamon

Open **Cinnamon Settings** → **Keyboard** → **Shortcuts** → **Custom Shortcuts**.

**Add hotkey 1: Start recording**

1. Click **+**
2. Name: `Whisper Start`
3. Command: `bash -lc "~/Scripts/start_whisper_simple.sh"`
4. Shortcut: **Ctrl+Alt+Q**
5. Click **Add**

**Add hotkey 2: Stop and transcribe**

1. Click **+**
2. Name: `Whisper Stop`
3. Command: `bash -lc "~/Scripts/stop_whisper_simple.sh"`
4. Shortcut: **Ctrl+Alt+Z**
5. Click **Add**

---

### 5.6 How to use it

Focus your cursor into any text field:

* ChatGPT / Claude web interface
* LibreOffice Writer
* Google Docs
* VS Code editor
* Any other text field in any application

1. Press **Ctrl+Alt+Q** to start recording
   * You'll hear a bell sound
   * Recording begins immediately
   * You can speak for up to 60 seconds

2. When done speaking, press **Ctrl+Alt+Z** to stop
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

### 5.7 Performance notes

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

### 5.8 Adjusting bell sound volume

The bell sounds use `--volume=34406` which is 52.5% system volume. If this is:

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

### 5.9 Cleanup: Remove old test scripts

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

### 5.10 Troubleshooting

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

### 5.11 Future enhancements

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

---

## Additional Notes: Switching Models and Performance Tips

### Switching to a Different Whisper Model

If you want to experiment with different Whisper models (`tiny`, `base`, `small`, `medium`, or `large-v3`), you can do so by editing your transcription script (`stop_whisper_simple.sh`).

Look for the `--model` line in the transcription command:

```sh
whisper-ctranslate2 \
  --model small \
  --language en \
  ...
```

To try a different model, simply change the `--model` argument. For example:

```sh
--model medium
```

### First-Time Use of a New Model

Each model must be downloaded the first time you use it. Because these models are large (e.g. `small` is ~244 MB, `medium` is ~769 MB), the first run will take longer while the model is downloaded from Hugging Face.

To allow this download, make sure offline mode is disabled during that first run:

```sh
# export HF_HUB_OFFLINE=1
```

Once the model is cached locally, you can re-enable offline mode:

```sh
export HF_HUB_OFFLINE=1
```

Transcription with the new model will then be just as fast as your previous one, with better accuracy depending on the model size.

### Comparing Model Performance (Optional)

If you're interested in comparing how different models perform on your system, you can add timing logic to your script.

Here's an example using `date`:

```sh
START=$(date +%s)

whisper-ctranslate2 \
  --model small \
  --language en \
  --threads 8 \
  --output_format txt \
  --output_dir /tmp \
  "$AUDIO_FILE"

END=$(date +%s)
echo "Transcription took $((END - START)) seconds"
```

This will show how long each transcription takes, making it easier to compare different models under the same conditions.

### Observing CPU Usage

You may notice that not all CPU cores are fully used, even if you set `--threads 8`. This is normal:

* Whisper processing is done in stages; not all stages can use all threads at once.
* Shorter audio clips and simpler speech may not utilize all available cores.
* Longer recordings (30+ seconds) or larger models (`medium`, `large-v3`) tend to scale CPU usage more effectively.

To monitor live CPU usage:

```sh
htop
```

Or:

```sh
top
```

Look for how many cores spike during transcription, and how long they stay active.

### Model Sizes and Tradeoffs

| Model | Size (approx) | Accuracy | Speed (CPU) |
|-------|---------------|----------|-------------|
| `tiny` | 39 MB | Lowest | Fastest |
| `base` | 74 MB | Better | Very fast |
| `small` | 244 MB | Good | Moderate |
| `medium` | 769 MB | Very good | Slower |
| `large-v3` | 1.5 GB | Best | Slowest |

Choose a model based on your balance of speed vs. transcription quality. For real-time use, `base` or `small` offer the best tradeoff.

---

## Model Selection: Real-World Testing vs. Assumptions

### The Flawed Assumption: "Bigger = Better and Faster"

A common assumption when choosing ML models is that larger models are always faster and more accurate. **This assumption is wrong**, especially on CPU-only systems. The only way to know is to test on your actual hardware.

### Your Test Setup

- **Hardware:** Linux Mint 22.2 on a 22-core laptop, CPU-only (no GPU)
- **Test audio:** Three sentences (~40 words), consistent phrasing
- **Measurement:** Wall-clock transcription time + core utilization (htop)
- **Models tested:** small, medium, large-v3

### Test Results

| Model | Time | Cores Used | Consistency | Accuracy |
|-------|------|-----------|--------------|----------|
| **small** | 13.9 seconds | Moderate (8 engaged) | **Reliable** ✓ | Excellent |
| **medium** | 11.53–18.38 seconds | Full (but inefficient) | **Inconsistent** ✗ | Excellent |
| **large-v3** | 28.6 seconds | Full (sustained) | Reliable | Perfect |

### Key Finding: Small Model Wins

Small was **both fastest and most consistent**:

- **First run:** 13.9 seconds (predictable)
- **Retest:** 13.9 seconds (identical)
- **CPU pattern:** Steady, moderate engagement across 8 threads
- **No variance:** No system noise, no surprises

Medium showed unpredictability:

- **First run:** 11.53 seconds (looked promising)
- **Retest:** 18.38 seconds (50% slower, different result)
- **CPU pattern:** Inefficient core utilization with overhead
- **Transcript degradation:** Second run had transcription errors ("is revolutionized" instead of "has revolutionized")

Large-v3 was predictably slow:

- **Consistent:** 28.6 seconds every run
- **Accuracy:** Perfect transcription
- **Cost:** 2x slower than small, requires 1.5 GB model download
- **Verdict:** Overkill for interactive dictation

### Why "Bigger" Doesn't Mean "Faster" on CPU

The official Whisper documentation indicates small to medium is "a 3x increase in model size for maybe 2% more accuracy in English."

On CPU-only systems:

1. **Larger models = more parameters to compute** — not automatically faster with naive threading
2. **Threading overhead increases** — managing more threads across 8 available slots causes contention, not parallelism
3. **Memory bandwidth becomes the bottleneck** — CPU cores idle waiting for parameter loads
4. **Optimization is architecture-specific** — `small` model is architected for this CPU/threading sweet spot

### Recommendation for Interactive Dictation

**Use `--model small` with `--threads 8`**

- 13.9-second transcription feel snappy for live typing
- Consistent, predictable performance (no surprises)
- Excellent accuracy for everyday English
- Frees up your 22 cores efficiently (no thrashing)

If you need absolute maximum accuracy and don't mind waiting: use `large-v3`. For everything else: small is the winner.

### The Lesson

Always test on your actual hardware before assuming model size = speed. Theory and benchmarks (which often use A100 GPUs) don't transfer to CPU-only laptops.

---

## Updating whisper-ctranslate2

### Check Your Current Version

To see which version of whisper-ctranslate2 you have installed:

```sh
pipx list | grep whisper
```

You can also check the version directly from the tool itself:

```sh
whisper-ctranslate2 --version
```

### Upgrade to the Latest Version

To upgrade to the latest version:

```sh
pipx upgrade whisper-ctranslate2
```

This will check for a newer version on PyPI and install it if available.

### Check Before Upgrading (Dry Run)

If you want to see what would be upgraded without actually doing it:

```sh
pipx upgrade whisper-ctranslate2 --dry-run
```

This shows you the version change without making any changes to your system.

### What's New in Recent Versions

The latest version is 0.5.7 (released Feb 8, 2026). For a detailed list of changes and bug fixes in each release, visit the official releases page at: https://github.com/Softcatala/whisper-ctranslate2/releases
