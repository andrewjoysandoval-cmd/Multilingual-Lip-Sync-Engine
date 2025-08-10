✨ Multilingual‑Lip‑Sync‑Engine (MLE)
Real‑time, multilingual mouth‑sync for humanoid robots.
Text → TTS visemes → smoothed servo motions — on an embedded SBC — with caching and fallbacks.

Built to slot into Realbotix F‑Series style heads. Works with Azure TTS viseme events; degrades gracefully when visemes aren’t available.

🔍 Why this exists 
Realbotix’s current public flow historically centered on English lip‑sync; expanding to Spanish / Cantonese / Mandarin / French and more means the robot must map any language’s phonemes → a fixed set of visemes, time them to the audio, and drive actuators smoothly in real‑time. MLE does exactly that using Azure Neural TTS viseme events (and a forced‑alignment fallback when visemes aren’t provided).

🧩 What you get
Unified 22‑viseme schema (Microsoft‑style) → 10 normalized servo channels

Coarticulation blending (attack/decay cross‑fades) for natural motion

Azure streaming support (viseme IDs + audio offsets in ticks)

File cache (audio + viseme timeline) to cut latency & cloud calls

Forced‑alignment fallback (simple, swappable for MFA or your aligner)

CLI + modular code (engine.py, tts.py, mapping.py, coarticulation.py, cache.py, fallback.py)

🗂️ Repo layout
bash
Copy
mle_demo/            # Minimal demo (fast evaluation)
mle_full_release/    # Production-ready engine (use this for integration)
⚙️ Requirements
Python 3.10+

For Azure mode: an Azure Speech resource (Key + Region)

bash
Copy
python -m venv .venv && source .venv/bin/activate
pip install azure-cognitiveservices-speech pydub pandas
🚀 Quick start
Demo (offline or Azure)
bash
Copy
# Offline (simulated visemes)
python mle_demo/mle_demo.py --text "Hola, ¿cómo estás?" --language es --output demo_es.csv

# Azure (live visemes)
python mle_demo/mle_demo.py \
  --text "Hello, how are you?" \
  --language en-US \
  --azure-key $AZURE_SPEECH_KEY \
  --azure-region $AZURE_SPEECH_REGION \
  --use-azure \
  --output demo_en.csv
Full Release (recommended)
bash
Copy
# Azure
python -m mle_full_release.main \
  --text "你好，很高兴见到你" \
  --language zh-CN \
  --azure-key $AZURE_SPEECH_KEY \
  --azure-region $AZURE_SPEECH_REGION \
  --output out_zh.csv

# Offline / Simulated
python -m mle_full_release.main \
  --text "Hola, ¿cómo estás?" \
  --language es-ES \
  --output out_es.csv
Output format: CSV with time_ms, servo_0..servo_9.
Sample period defaults to 50 ms (configurable).

🔧 Mapping to your head (F‑Series style)
Open mle_full_release/mapping.py.

Calibrate the 10 normalized channels [0.0..1.0] to your servo IDs & travel ranges.

Keep jaw safety limits conservative; increase gradually after visual checks.

Stream the CSV to your motion controller at the same sample period you used to generate it.

Tip: start with vowels (/A/ /E/ /I/ /O/ /U/) and bilabials (/P/ /B/ /M/). Tune lip‑corner, pucker, jaw.

🧪 How coarticulation works 
MLE blends neighboring visemes with attack/decay windows so the mouth begins forming the next shape before the current one fully ends. This avoids “flapping” and matches how human articulators overlap in time. Tweak the window (e.g., 50–120 ms) in coarticulation.py for your servos’ speed.

⚡ Caching
Keyed by (text, language, voice).

Stores: audio.wav + visemes.json.

Turn it on with --cache-dir .mle_cache (CLI) or via LipSyncEngine(..., cache_dir=...).

🆘 Fallback alignment
If your TTS doesn’t emit viseme events, MLE can approximate timings over the audio duration. The included fallback is intentionally simple so you can swap in Montreal Forced Aligner (or any phoneme aligner) for production.

🖧 Real‑time notes
Azure viseme events include ID + audio_offset (100‑ns ticks). We convert to ms and schedule servo targets accordingly.

Keep your audio output buffer predictable; introduce a small visual lead (e.g., ~40–60 ms) if your audio path is slower than the motion loop.

🔒 Privacy & safety
Cache lives locally; clear it for sensitive phrases.

If you use face tracking / personalization elsewhere, ensure consent & retention policies suit your market.

🗺️ Roadmap
Plug‑in MFA for accurate phoneme timings (drop‑in replacement for simplified fallback)

Export binary streaming format for controllers (Protobuf)

More viseme presets (12–14 channel rigs)

Sample ROS node & WebSocket streamer

Automated MOS and latency harness

📎 References 
Realbotix language expansion coverage (Business Wire)

Microsoft Learn — Visemes & viseme/phoneme groupings

Azure Speech SDK — Viseme events + audio_offset ticks

Coarticulation — overlapping gestures (intro)


📄 License
MIT (or your preferred license). Add a LICENSE file.

🤝 Contributing
Issues and PRs welcome. Please include a short clip or CSV snippet with any bug report so we can reproduce timing problems.

🧰 Maintainer shortcuts
Demo CSV quick‑gen: python mle_demo/mle_demo.py …

Full engine CSV: python -m mle_full_release.main …

Clear cache: delete --cache-dir folder.

