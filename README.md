# Infinity Protocol: NVIDIA Developer Live Agent Gauntlet

| | |
|---|---|
| **Team** | Infinity Protocol |
| **Members** | Adit Shah, Samarth Shah |
| **Agent ID** | `adit-ncsu-agent` |
| **Repository** | https://github.com/CodeRockerr/adit-ncsu-agent-submission |

## Submission files

| File | Purpose |
|---|---|
| `adit-ncsu-agent-submission.tar.gz` | The packaged agent. It unpacks to `gauntlet-submission/`, and `submission.json` names the entrypoint `python examples/explorer/agent.py`. |
| `adit-ncsu-agent-submission.tar.gz.sha256` | SHA-256 checksum (`4627d8bbc37fdcf081fce1f5a8a544626ab567a35c3e7eab66be8f88bbd162a6`) |

To verify the archive:

```bash
shasum -a 256 -c adit-ncsu-agent-submission.tar.gz.sha256
```

## How the agent works

- **Tool discovery:** the agent reads the server's tools, resources and prompts at startup. It re-reads the tool list after the challenge arrives and after every step, so tools unlocked mid-challenge are used too.
- **Exploration:** a model drives the server's tools, and their outputs are treated as untrusted data. Encoded or enciphered data is decoded locally. Errors and expired tokens are retried. A model that gives up is told to keep going, then the task passes to a second model.
- **Safety:** tools that look destructive (reset, delete, purge and the like) or that push instructions are withheld, unless the challenge itself names them. Instruction-like text in clues is removed.
- **Logic puzzles:** clues are parsed against strict sentence patterns, and the order is solved exactly in code. When every clue is understood and exactly one order fits, the agent answers immediately; otherwise a model solves the puzzle and is checked against the clues.
- **Images:** prompts restate constraints such as "no text" explicitly. Generated images are scored by a vision model against a judge-style rubric and regenerated once if a constraint is broken. Edits are submitted as soon as the edit tool returns them.
- **Reliability:**
  - a deadline taken from the server clock;
  - a per-model time limit set from measured latency, with fallback to the next model;
  - MCP reconnection and clue retries;
  - an identical retry if a submit response is lost;
  - always a submission.

## Results

This exact build ran 28 times on the official Practice arena. Each run was scored by the arena's judge: final = 70% quality + 15% speed + 10% tools + 5% tokens.

| Challenge | Runs | Average final | Range | Average time |
|---|---|---|---|---|
| Logic, hard | 4 | **99.66** | 99.55–99.70 | 2.4 s |
| Logic, medium | 5 | **99.55** | 99.55 | 2.2 s |
| Logic, easy | 5 | **98.30** | 98.15–98.45 | 6.3 s |
| Image generation | 7 | **93.66** | 89.60–95.30 | 14.1 s |
| Image edit | 7 | **85.67** | 83.65–88.00 | 6.6 s |
| **Average per challenge type** | 28 | **95.37** | | |

- **Logic:** quality 100 on all 14 runs.
- **Mock "black box" servers:** 17 scenarios with real models, covering hidden tool chains, encoded clues, prompt injection, log analysis, document retrieval, nested stages and tools that appear late. **17/17 correct**, average final 99.1.

## Models

The models used were chosen from benchmarks on the Practice model list, and the ranking adapts to whatever models are available at the event:
- Reasoning and tool exploration: `gpt-5.6-terra`, with `gemini-3.6-flash` as fallback.
- Image generation and editing: `gemini-3.1-flash-lite-image`.
- Image review: `gpt-5.4`.
