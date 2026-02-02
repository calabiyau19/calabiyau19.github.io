---
layout: post
draft: false
title: "Troubleshooting local Whisper AI install after it stops working."
date: 2026-02-01
description: "Occasionally, Whisper AI stops transcribing, and this post just serves to remind me how to quickly get it back up and running again, as it is only one small piece of code to change once, then change back immediately after it starts working again. Less than a five-minute fix now that I have it documented."
---


## Troubleshooting: Whisper Local Stops Transcribing

### TL;DR - Quick Fix

If you hear the beeps but no text appears:

 ```sh
nano ~/Scripts/stop_whisper_simple.sh
 ```

Comment out line 3: `# export HF_HUB_OFFLINE=1`

Save, run one transcription (takes 1-2 minutes), then uncomment line 3 and save again.

---

### Common Cause

**BleachBit or similar cleanup tools delete Whisper model files** from `~/.cache/huggingface/`. With offline mode enabled, Whisper can't re-download the model and hangs silently during transcription.

---

### Step-by-Step Fix

#### Step 1: Verify the problem

 ```sh
~/Scripts/start_whisper_simple.sh
 ```

Speak for 5 seconds, then:

 ```sh
~/Scripts/stop_whisper_simple.sh
 ```

If it shows "Transcribing..." and hangs (nothing happens), continue to Step 2.

#### Step 2: Temporarily allow online access

 ```sh
nano ~/Scripts/stop_whisper_simple.sh
 ```

Find line 3:
 ```sh
export HF_HUB_OFFLINE=1
 ```

Add `#` at the beginning:
 ```sh
# export HF_HUB_OFFLINE=1
 ```

Save (Ctrl+X, Y, Enter).

#### Step 3: Re-download the model

 ```sh
~/Scripts/start_whisper_simple.sh
 ```

Speak for 5 seconds, then:

 ```sh
~/Scripts/stop_whisper_simple.sh
 ```

This will download the base model (~150 MB). Takes 1-2 minutes. Wait for transcription to complete.

#### Step 4: Re-enable offline mode

 ```sh
nano ~/Scripts/stop_whisper_simple.sh
 ```

Remove the `#` from line 3:
 ```sh
export HF_HUB_OFFLINE=1
 ```

Save (Ctrl+X, Y, Enter).

#### Step 5: Test

Press Ctrl+Alt+Q, speak, press Ctrl+Alt+Z. Transcription should be fast (2-3 seconds).

---

### Prevention

**Configure BleachBit to exclude:** `~/.cache/huggingface/` from cleaning

OR

**Accept:** Re-downloading model (~150 MB) after each BleachBit run using steps above.
