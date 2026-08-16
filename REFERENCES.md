# References — verified, current as of 2026-08-15

Pre-filled so no session tonight re-spends time or usage budget re-verifying
what's already confirmed. Update the "last verified" date if anything here
is re-checked later.

## Primary model — use this first

- **Repo:** https://huggingface.co/Erebus007/NCERT-qwen-1.5B-Q4_K_M (public,
  no token required)
- **File:** `qwen2-1.5b-instruct.Q4_K_M.gguf`
- **Size:** 986 MB
- **Base model:** Qwen2-1.5B-Instruct, fine-tuned via Unsloth on NCERT
  curriculum data
- **Verified:** 2026-08-15, via direct fetch of the repo's file listing —
  the file's existence and exact name are confirmed, not guessed
- **Download:**
  ```
  huggingface-cli download Erebus007/NCERT-qwen-1.5B-Q4_K_M qwen2-1.5b-instruct.Q4_K_M.gguf --local-dir ./models/
  ```
- **NOT yet verified:** whether the chat template survived GGUF conversion.
  This is the first real task of M1 — load it and send a real message.

## Fallback models — only if the primary needs replacing

- Gemma-2-2b-it (GGUF) — re-verify current repo path before use, do not
  assume from training data
- Qwen2.5-1.5B-Instruct (GGUF) — same caveat

## Sarvam-1 — v2 research thread only, not for tonight

Base release is completion-only, not chat-ready as shipped. Not relevant
to tonight's session at all — the primary model above already solves this
problem by starting from an instruct-tuned base.

## GitHub

Project repo: **[fill in once created — see TONIGHT.md]**
