OpenRouter is supported as a first-class provider.

- **Usage:** `pip install openai`; `export OPENROUTER_API_KEY=...`; `inspect eval arc.py --model openrouter/<provider>/<model>` (e.g., `openrouter/gryphe/mythomax-l2-13b`).
- **Custom args:** Pass via `-M` / `model_args`: `models` (routing list), `provider` filters, `transforms`, `reasoning_enabled`, plus standard GenerateConfig options.
- **Endpoints:** `OPENROUTER_BASE_URL` overrides default `https://openrouter.ai/api/v1`.
- **Tools:** Supports tool calling; tool emulation available via `emulate_tools=true` model arg.
- **Conclusion:** Inspect can run identical tasks across OpenRouter-routed models and mix them with other providers in eval sets; OpenRouter works for multi-provider comparisons without extra code.
