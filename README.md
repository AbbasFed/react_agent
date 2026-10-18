# SunCedar ReAct Agent · configurable personas

**EECE 503P/798S Agentic Systems · C4 Agent Personas & Thinking Mechanisms**
**Author:** Abbas Yassine

This extends the C3 SunCedar Energy chatbot into a **ReAct agent built in LangGraph**. The Thought → Action →
Observation loop is written by hand as graph nodes and a conditional edge; no `AgentExecutor`, no
`create_react_agent`, no prebuilt `ToolNode`. Three personas, each defined by **prompt traits and a matching LLM
configuration**, are compared in a logged experiment grid.

| | |
|---|---|
| **Framework** | LangGraph (`StateGraph`, manual nodes: think / act / repair / force / finish) |
| **LLM** | Gemini through `google-genai`: Flash-Lite (picked automatically) + Flash for the model-type test |
| **Tools** | `search_business_info`, `check_service_area`, `size_system`, `estimate_savings`, `record_customer_interest`, `record_feedback` |
| **Personas** | Abbas (Friendly Advisor, temp 0.8) · Rania (Strict Technical Expert, temp 0.1) · Lina (Cautious Support Helper, temp 0.3) |
| **Experiments** | 12 auto-scored test messages × up to 16 setups: personas, CoT, zero/one/few-shot, system vs in-message, no instructions, temperature, top_p/top_k, penalties, max tokens, persona/config mismatch, model type |
| **Tests** | 52 offline checks (no API key needed) |

## Repository layout

```
react_agent/
├── me/                          # SunCedar business files (reused from C3)
├── react_persona_agent.ipynb    # everything: use case, tools, personas, instructions, ReAct graph, experiments, reflection, UI
├── results/                     # experiment logs (created when you run section 11)
├── requirements.txt
├── .env.example                 # copy to .env, add your key (.env is git-ignored)
└── .gitignore
```

## Run it

### Google Colab
1. Upload `react_persona_agent.ipynb` to <https://colab.research.google.com>.
2. **Key icon** (Secrets) → add `GEMINI_API_KEY` → switch **Notebook access** on.
3. **Files** panel → upload the `me/` folder (both files inside it).
4. **Runtime → Run all.** The backend line should name a `gemini … flash-lite` model, not `mock`.

### Locally
```bash
pip install -r requirements.txt jupyter
cp .env.example .env        # add your key
jupyter notebook react_persona_agent.ipynb
```

## Running the experiments (section 11)

| Plan | Runs | API calls (approx.) | Time with pacing |
|---|---|---|---|
| `quick` | 7 setups × 6 tests = 42 | ~130 | ~10 min |
| `lite` (default) | 7 setups × 12 tests = 84 | ~250 | ~20 min |
| `core` | 11 setups × 12 tests = 132 | ~400 | ~35 min |
| `full` | 16 setups × 12 tests = 192 | ~600 | ~45 min |

- Results are saved to `results/experiments_gemini.jsonl` **as each run finishes**. Re-running the cell skips
  finished runs, so you can stop at any time, and if the free daily quota runs out, continue the next day.
- `MAX_CALLS` caps the calls per execution.
- Section 12 builds the tables and chart; section 13 writes the reflection answers **from your results**
  (also saved to `results/reflection_gemini.md`). Then add your own notes in the cell after it.
- Commit the `results/` folder: it is your experiment log.

## How the ReAct loop works (section 7)

```
START → think ──(action)──────────▶ act ──▶ think
            ├──(invalid format)────▶ repair ──▶ think
            ├──(final answer)──────▶ finish ──▶ END
            └──(step limit)────────▶ force ──▶ END
```

- **think**: one model call with persona + tools + instructions + history + scratchpad. The reply is parsed into
  `action` / `final` / `invalid` (section 6).
- **act**: runs the tool and appends `Observation: {...}` to the scratchpad, then loops back to think.
- **repair**: sends the format error back to the model instead of crashing.
- **force**: after 6 model calls, demands a Final Answer.
- `Observation:` is a stop sequence, so the model can't invent observations.

## Demo video: what to show (about 4-5 min)

1. Section 2 (use case table) and section 4 (persona table with the config reasons).
2. Section 5: the instructions block (single-purpose rules, I/O format, worked example).
3. Section 7: the compiled LangGraph diagram, and the three persona traces in section 8 (Thought → Action → Observation).
4. Section 12: the summary table, the chart, the persona table, and the side-by-side answers.
5. Section 13: the reflection answers.
6. The Gradio app: ask the same question with **Abbas**, then **Rania**, then turn Chain-of-Thought off, and show
   the trace panel each time. Good question: *"I have an inverter AC, a fridge, lights and a TV, and pay $120 a month
   to the generator. Which package and what's the payback?"*

## Troubleshooting

| Problem | Fix |
|---|---|
| `provider=mock` although you added a key | Turn **Notebook access** on for the secret, then **Runtime → Restart session** and run again. |
| `HTTP 429, retrying` | Free-tier per-minute limit; the code waits. Raise `MIN_SECONDS_BETWEEN_CALLS` if it happens a lot. |
| `daily quota used up -> switching to …` | Normal: it moves to another model. If all are used up, the run stops cleanly; re-run tomorrow. |
| `rejected ['presence_penalty', …]` | That model doesn't accept the parameter. Runs continue without it, and it is noted in the results and the reflection. |
| `FileNotFoundError … me/` | Upload the `me/` folder next to the notebook. |

## Limitations

- Sizing and savings use planning assumptions (typical appliance loads, an assumed share of the generator bill),
  not real quotes.
- The automatic scores check tools, key facts and invented numbers, not everything that makes a reply good. Tone is
  rated by an LLM judge from the same model family, so treat it as indicative.
- Each setup runs each test once; differences of a few points can be noise.
- SunCedar Energy, its staff, prices and contact details are fictional.
