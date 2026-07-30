# Anxiety Support Chatbot

MatrAIx chatbot task for a synthetic anxiety support assistant.

Product under test: Qwen-powered anxiety support sidecar (OpenAI and Anthropic
optional). The persona agent acts as a simulated user, has a multi-turn
conversation, and saves transcript plus self-report artifacts.

Harbor runtime:

- Persona agent: `environment/task-environments/application/shared-chat-persona`
- Local endpoint: `environment/task-environments/application/chatbot-api-sidecar_anxiety`
  (`anxiety-chatbot`, host port **8907**)

## Evaluation goal

Measure whether the anxiety support chatbot delivers **empathetic, safe, useful
support** for personas who are anxious and high-neuroticism, and surface **who
it works for** (age, trust, safety sensitivity) via batch persona insights.

## Sidecar smoke

```bash
cd environment/task-environments/application/chatbot-api-sidecar_anxiety
export QWEN_API_KEY=your-key
docker compose -f standalone-compose.yaml up --build
curl http://127.0.0.1:8907/health
```

## Harbor smoke

```bash
uv run python application/scripts/generate_application_job.py \
  --task application/tasks/chat_synthetic-anxiety-support \
  --execution-mode auto \
  --persona-ids 0042

export QWEN_API_KEY=your-key
export CHATBOT_UPSTREAM_ANXIETY=http://127.0.0.1:8907
export ANTHROPIC_API_KEY=sk-ant-...
uv run harbor run -c configs/jobs/application-task-job-recipe/chat-synthetic-anxiety-support-n1.yaml
```

## Persona pool

Generate a strategy-aligned pool before batch runs (required for persona-insight
cross-tabs in Playground):

```bash
uv run python persona/scripts/generate_dev_personas.py \
  --strategy application/tasks/chat_synthetic-anxiety-support/persona_strategy.json
```

Default batch cohort: **stratified** on `age_bracket` × `trust_level` (6 trials),
filtered to anxious / high-neuroticism personas.

## Expected artifacts

- `/app/output/transcript.json`
- `/app/output/user_feedback.json`

See `input/protocol.md` for the HTTP contract and `input/self_report_schema.yaml`
for the feedback schema.

## Batch reporting

Policy in `reporting.json`:

- **Persona insights** — outcome, conversation path, coping helpfulness, felt
  understood, trust score, etc. cross-tabbed by `age_bracket`, `trust_level`,
  `safety_sensitivity`
- **Custom analysis** — LLM summaries of persona feedback by segment (age, trust,
  safety sensitivity) and by outcome/coping/clarification buckets

Roll up with:

```bash
uv run python application/scripts/report_job.py jobs/<job-name>
```

View in Playground: `http://127.0.0.1:8765/?view=runs&harborJob=<job-name>`

Metric templates: `application/task-spec/chatbot/README.md`
