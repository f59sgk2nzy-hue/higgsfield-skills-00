# Narrated picture story

Use this path when the user explicitly requests still scenes or a picture story. Preserve the requested topic, character identity, style, aspect ratio, duration, narration language and voice. Use a persistent working directory and save every job ID and returned URL. Before paid work verify the prerequisites in `references/captions.md`, including an available local transcription model; a timed narration transcript is required even when captions are off.

Write one continuous narration and a sequence of numbered beats in `story.json`: `{"beats":[{"n":1,"phrase":"..."},{"n":2,"phrase":"..."}]}`. Research factual topics and retain sources. Generate one continuous `seed_audio` take with the selected voice pair, with no speed adjustment. Measure it. If it differs from the requested duration by more than the larger of three seconds and ten percent, rewrite the text and retry at most twice; never change the requested duration. For long narration, join a few completed same-voice paragraph takes in order without changing their speed.

Run the local transcript and build a frame timeline:

```bash
python3 "$SKILL_DIR/scripts/audio_to_captions.py" narration.wav --script story.json \
  --language en --model "$LOCAL_MODEL_DIR" --json words.json
python3 "$SKILL_DIR/scripts/build_scene_timeline.py" --script story.json \
  --timestamps words.json --audio-duration "$MEASURED_SECONDS" \
  --requested-duration "$REQUESTED_SECONDS" --out timeline.json
```

Follow the timeline order, not completion order. Create style and character references; generate a distinct numbered image for every frame in `timeline.json`, using the preceding frame as a reference for each declared variation. Describe only the intended change. Store each completed result as `frameNNN.png` and keep the mapping from `output_slot` to actual job ID and result URL. Do not claim a variation used a reference that was not supplied to its generation. Inspect the pictures against the narration and repair only failed or incorrect frames. Never repeat one picture to satisfy the count.

Make `results.json` with `{"results":[{"output_slot":"frame-001","job_id":"...","url":"..."}, ...]}` from the actual completed jobs. Bind it and assemble:

```bash
python3 "$SKILL_DIR/scripts/bind_scene_frame_results.py" --manifest timeline.json \
  --results results.json --out bound.json
bash "$SKILL_DIR/scripts/assemble_slides.sh" --audio narration.wav --timeline bound.json \
  --frames-dir frames --blocks "$FRAME_COUNT" --requested-seconds "$REQUESTED_SECONDS" --out final.mp4
```

The assembler verifies ordering, references, aspect, frame density, duration and decode. Stop on a failed check. If captions were requested, time them against the continuous narration with `audio_to_captions.py --script story.json`, then use the caption burn and final review in `references/captions.md`. Verify three points in the result against the script, then deliver and optionally upload the actual final file.
