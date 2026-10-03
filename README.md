# explain-for-understanding

An agent skill for **Claude Code** and **Codex** that makes the agent pick the most digestible output format when you need to understand, review, or oversee something complex, instead of defaulting to a wall of prose.

Based on Andrej Karpathy's post, ["We'll be spending a lot more time trying to understand the outputs of language models"](https://x.com/karpathy/status/2105819303471976479) (Oct 2, 2026).

## The idea

As LLMs do more work on their own, your job moves toward oversight and understanding. Code is cheap, so large, custom, throwaway artifacts are now worth making just to help you understand. The skill walks a ladder, where each rung is usually better than the one below:

| Rung | Format | Use when |
|---|---|---|
| 1 | Text in ASD-STE100 (controlled language) | Short explanations, procedures, docs |
| 2 | Diagram / image (Mermaid, SVG) | Structure, flow, architecture, sequences |
| 3 | Self-contained interactive HTML page | Anything to explore, compare, or step through |
| 4 | 3Blue1Brown-style explainer video | Dense concepts you must really internalize |

It also covers how to review the agent's own work (diff, refactor, research) with a diagram or page, not a changelog dump.

## Install

**Claude Code** (all projects):

```bash
git clone https://github.com/subhams07/explain-for-understanding-skill.git
mkdir -p ~/.claude/skills
cp -r explain-for-understanding-skill/skills/explain-for-understanding ~/.claude/skills/
```

**Codex**:

```bash
mkdir -p ~/.codex/skills
cp -r explain-for-understanding-skill/skills/explain-for-understanding ~/.codex/skills/
```

For a single project, copy the folder into `.claude/skills/` in that repo instead. Restart the tool or start a new session so it picks the skill up.

## Use

The skill triggers on requests like "explain X", "walk me through this", "help me understand this PR". You can also ask directly:

- "Explain X in ASD-STE100." or "80% of the way to ASD-STE100."
- "Draw a diagram of X."
- "Give me X in HTML."
- "Make a 3b1b-style video explainer on X."

## Notes

- The ASD-STE100 rules in the skill are a working summary, not the official spec. See [asd-ste100.org](https://www.asd-ste100.org/) for the real one.
- Video mode uses Manim and ffmpeg. For narration it reads `ELEVENLABS_API_KEY` from the environment if set, otherwise it falls back to a free local text-to-speech tool.
- Video generation is experimental and may need tool installs.

## License

MIT. See [LICENSE](LICENSE).
