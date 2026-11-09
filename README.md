# 🔬 Glassbox Audit — GitHub Action

Audit an LLM verdict for **faithfulness** in your CI pipeline — catch
hallucinated evidence before it ships. Wraps
[glassbox](https://github.com/winxtxrgit/glassbox): grounding (does the cited
evidence exist?), counterfactual faithfulness (erase it — does the verdict flip?),
and consistency (reword it — does the decision hold?).

The step's exit code is `0` only when the verdict clears all three, so it gates a
job the same way a failing test does.

## Usage

```yaml
- uses: winxtxrgit/glassbox-action@v1
  with:
    file: output/summary.txt
    backend: openrouter        # or ollama (self-hosted runner) / groq
  env:
    OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
```

## Example — gate a PR on LLM faithfulness

```yaml
name: LLM Audit
on: pull_request

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate the LLM output under test
        run: python generate_summary.py   # writes output/summary.txt

      - name: Audit it for hallucinated evidence
        uses: winxtxrgit/glassbox-action@v1
        with:
          file: output/summary.txt
          backend: openrouter
        env:
          OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
```

If the model cited evidence that isn't in the source, or the verdict doesn't
actually depend on what it quoted, the step fails and the PR is blocked.

## Inputs

| Input | Default | Description |
|---|---|---|
| `file` | — (required) | Document to audit. |
| `backend` | `openrouter` | `ollama`, `openrouter`, or `groq`. The API backends read `<PROVIDER>_API_KEY` from the environment. |
| `model` | backend default | Model name, e.g. `qwen2.5:3b`. |
| `glassbox-version` | `0.8.0` | [`glassbox-audit`](https://pypi.org/project/glassbox-audit/) version to install from PyPI. Pinned to a tested release. |
| `glassbox-ref` | — | Install from this glassbox git ref (tag, branch or SHA) instead of PyPI, e.g. `main` to track the latest. |
| `python-version` | `3.12` | Python to set up. |

## Outputs

| Output | Description |
|---|---|
| `trustworthy` | `true` if the verdict passed all three checks. |
| `report` | The full JSON audit report. |

---

Powered by [glassbox](https://github.com/winxtxrgit/glassbox) · MIT
