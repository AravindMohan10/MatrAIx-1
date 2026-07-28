# Depression Support Chatbot

MatrAIx chatbot task for a synthetic depression support assistant with SAMHSA
safe-messaging guardrails.

Product under test: multi-LLM depression support sidecar (Qwen default; OpenAI
and Anthropic optional). The persona agent acts as a simulated user, has a
multi-turn conversation, and saves transcript plus self-report artifacts.

Harbor runtime:

- Persona agent: `environment/task-environments/application/shared-chat-persona`
- Local endpoint: `environment/task-environments/application/chatbot-api-sidecar_depression`
  (`depression-chatbot`, host port **8906**)

## Evaluation goal

Measure whether the depression support chatbot delivers **empathetic, safe, clinically
appropriate support** for high-neuroticism personas, and surface **who it works for**
(age, trust, safety sensitivity) via batch persona insights.

## Sidecar smoke

```bash
cd environment/task-environments/application/chatbot-api-sidecar_depression
export QWEN_API_KEY=your-key
docker compose -f standalone-compose.yaml up --build
curl http://127.0.0.1:8906/health
```

## Harbor smoke

```bash
uv run python application/scripts/generate_application_job.py \
  --task application/tasks/chat_synthetic-depression-support \
  --execution-mode auto \
  --persona-ids 0042

export QWEN_API_KEY=your-key
export CHATBOT_UPSTREAM_DEPRESSION=http://127.0.0.1:8906
export ANTHROPIC_API_KEY=sk-ant-...
uv run harbor run -c configs/jobs/application-task-job-recipe/chat-synthetic-depression-support-n1.yaml
```

## Persona pool

Generate a strategy-aligned pool before batch runs (required for persona-insight
cross-tabs in Playground):

```bash
uv run python persona/scripts/generate_dev_personas.py \
  --strategy application/tasks/chat_synthetic-depression-support/persona_strategy.json
```

Default batch cohort: **stratified** on `age_bracket` × `trust_level` (6 trials),
filtered to high-neuroticism personas.

## Expected artifacts

- `/app/output/transcript.json`
- `/app/output/user_feedback.json`

See `input/protocol.md` for the HTTP contract and `input/self_report_schema.yaml`
for the feedback schema.

## Batch reporting

Policy in `reporting.json`:

- **Persona insights** — outcome, conversation path, PHQ-9 coverage, felt
  understood, trust score, safe messaging, etc. cross-tabbed by `age_bracket`,
  `trust_level`, `safety_sensitivity`
- **Custom analysis** — LLM summaries of persona feedback by segment (age, trust,
  safety sensitivity) and by outcome/PHQ-9/clarification buckets

Roll up with:

```bash
uv run python application/scripts/report_job.py jobs/<job-name>
```

View in Playground: `http://127.0.0.1:8765/?view=runs&harborJob=<job-name>`

Metric templates: `application/task-spec/chatbot/README.md`
