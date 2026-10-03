---
name: explain-for-understanding
description: Use when the user wants to understand, review, or oversee something complex (code, a system, a paper, a plan, a concept, an agent's work). Picks the most digestible output format instead of defaulting to a wall of prose, escalating from controlled-language text to diagram, to interactive HTML page, to explainer video.
---

# Explain for Understanding

Source: Andrej Karpathy, "We'll be spending a lot more time trying to understand the outputs of language models" (x.com/karpathy/status/2105819303471976479, Oct 2, 2026).

Premise: as LLMs do more work autonomously, the human job shifts to oversight and understanding. Code and intelligence are cheap, so large, custom, **discardable** artifacts (web apps, videos) are now worth making just to help someone understand. Don't default to long prose.

## Trigger

Use when the user says things like: "explain X", "help me understand", "walk me through", "what did you just change", "summarize this codebase/paper/PR", "make this easier to follow", or asks to review/oversee large output. Also use proactively when a plain-text explanation would be long, dense, or multi-part.

## The ladder (each rung is usually better than the one below)

| Rung | Format | Use when | Cost |
|---|---|---|---|
| 1 | Controlled-language text (ASD-STE100) | Short explanation, quick answer, docs, procedures | Lowest |
| 2 | Diagram / image | Structure, flow, relationships, architecture, state, sequences | Low |
| 3 | HTML web page | Anything with parts to explore, compare, step through, or animate | Medium |
| 4 | Explainer video (3Blue1Brown style) | Deep concept, math, algorithm, or system someone must really internalize | Highest |

### Choosing a rung
- Default to rung 2 or 3 for anything non-trivial. Stay on rung 1 only when the answer fits in a few sentences.
- Go to rung 4 only when the user asks for a video or the topic is dense and the user clearly has time to learn it. Say what it will cost (time, API key) first.
- If unsure, deliver rung 1 summary plus offer one higher rung. Do not ask a long list of questions.
- Always ask yourself: is a human going to *read* this or *check* this? If check, prefer diagram or page.

## Rung 1: Writing in ASD-STE100

ASD-STE100 (Simplified Technical English) is a controlled-language spec from aerospace maintenance docs. Its constraints make text clean and unambiguous. Apply these rules (working summary, not the full spec):

- Procedural sentences: max about 20 words. Descriptive sentences: max about 25 words.
- Paragraphs: max about 6 sentences. One topic per paragraph.
- One instruction per sentence. Use the imperative: "Remove the cover."
- Use active voice. Say who or what does the action.
- Use simple verb tenses (simple present, past, future). Avoid complex tenses.
- Keep articles ("the", "a"). Do not drop them to save space.
- One word, one meaning. Pick one term for a thing and keep it. Do not vary for style.
- Use plain, common words. Prefer "use" over "utilize". Avoid idioms and slang.
- Put warnings and cautions first, in their own short sentences, before the step they apply to.
- Use lists for sequences and parallel items.
- Do not stack nouns into long noun strings. Rewrite with prepositions.

The full spec is very strict. Default to **"80% of the way to ASD-STE100"**: follow the rules above, but allow a needed technical term or a slightly longer sentence rather than awkward phrasing. Use full-strict only if the user asks.

## Rung 2: Diagrams

- Prefer a diagram over paragraphs for: architecture, data flow, dependencies, sequences, state machines, org/ownership, timelines, before/after.
- Pick the lightest tool that works, in this order:
  1. **Mermaid** in a fenced code block (flowchart, sequenceDiagram, stateDiagram, classDiagram, erDiagram, gantt). Renders in many UIs and is easy to edit.
  2. **Inline SVG** in an HTML file when you need custom layout or labels.
  3. **Image output** (PNG/SVG file) when the user needs to paste it into slides or docs.
- Label every box and edge. Keep to about 12 nodes; split into several diagrams rather than one giant one.
- Follow with 1 to 3 sentences (ASD-STE100 style) saying what to look at. Do not repeat the diagram in prose.
- If the user prefers a formal notation (e.g., UML) use it; otherwise pick whatever is clearest.

## Rung 3: HTML pages

Ask yourself "can this be a page?" Say "in HTML" to yourself as the default for complex output.

- Produce **one self-contained .html file**: inline CSS and JS, no build step, no required network (CDN scripts only if they add real value).
- Design for understanding, not decoration:
  - Clear title and a 2-sentence summary at the top.
  - Collapsible sections, tabs, or a table of contents for long material.
  - Interactive elements that teach: sliders to change a parameter, step-through buttons, hover tooltips, clickable diagram nodes, filters, side-by-side compare.
  - Embedded diagrams (inline SVG or Mermaid) and small animations where motion shows cause and effect.
  - Real content only. No lorem ipsum.
- Make it readable: system font stack, good contrast, max line width about 70 characters, responsive layout, works in dark and light.
- Save to a sensible path (e.g. `./explainers/<topic>.html`), then **open it** (`start` on Windows, `open` on macOS, `xdg-open` on Linux) or tell the user the path.
- Check it renders. If a browser tool or screenshot tool is available, load the page and look at it. Fix layout bugs before handing it over.
- It is a discardable artifact. Do not over-engineer, do not add it to the repo unless asked.

## Rung 4: Explainer videos

Karpathy's most-bullish format. Starting to work, not guaranteed. Example prompt: "Create a 3b1b style video explainer on X. Use my ElevenLabs API key for audio narration."

Workflow:
1. **Script first.** Write a short scene-by-scene script (hook, intuition, build-up, example, recap). Write narration in ASD-STE100 style: short, plain sentences. Show the script to the user if the video will take long to render.
2. **Animation.** Use **Manim** (Community Edition, `pip install manim`) for 3Blue1Brown-style math and concept animation. Alternatives: Motion Canvas, Remotion, or an HTML/Canvas animation recorded to video. Check what is installed first (`python -c "import manim"`, `ffmpeg -version`). Install missing pieces or report the blocker.
3. **Narration.**
   - If the user has an **ElevenLabs API key** (env var `ELEVENLABS_API_KEY`), use it. Never ask for the key in chat to paste into files; read it from the environment, and never print or log it.
   - If no key, use a free local TTS: **Piper**, **Kokoro**, **Coqui TTS**, or OS speech (Windows SAPI / macOS `say` / `espeak-ng`). Pick whichever installs cleanly. Tell the user the quality trade-off.
4. **Assemble.** Sync narration to scenes by measuring each audio clip's duration and setting scene run times to match. Combine with `ffmpeg`. Add captions (SRT) if cheap.
5. **Render at low quality first** (e.g. Manim `-ql`) to check pacing and bugs. Render final quality only after the draft looks right.
6. **Deliver** the .mp4 path, its duration, and 2 sentences on what it covers. Keep length short (2 to 5 minutes) unless asked.

Be honest about failures. If rendering or TTS breaks and you cannot fix it, say so and fall back to rung 3. Do not claim a video exists that you did not render.

## Oversight use (reviewing agent/LLM work)

When the thing to understand is work the agent itself just did (a large diff, a refactor, a research result):
- Lead with a diagram or page of *what changed and why*, not a changelog dump.
- Show the 3 to 5 decisions that matter and the risks, in ASD-STE100-style bullets.
- Link each claim to a file/line or a test result so the user can verify it.
- Offer "want this as a diagram / page?" if you answered in prose and the topic was big.

## Quick prompts the user can give (and you can self-apply)

- "Explain X in ASD-STE100." / "80% of the way to ASD-STE100."
- "Draw a diagram of X."
- "Give me X in HTML."
- "Make a 3b1b-style video explainer on X."

## Pitfalls

- Do not pick a higher rung just because it is fancier. A 3-sentence answer stays 3 sentences.
- Do not use ASD-STE100 for creative or persuasive writing. It is for clarity of explanation and procedures.
- Do not produce a diagram or page with content you did not verify. A pretty wrong explanation is worse than plain text.
- Do not leave an HTML page unchecked. Open or render it once.
- Do not store secrets (API keys) in generated files.
