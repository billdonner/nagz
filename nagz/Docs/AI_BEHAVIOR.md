# AI Behavior Policy (V1)

## 1. Scope
Defines how the AI intermediary communicates, summarizes excuses, and applies bounded push-back.

## 2. Allowed AI Actions
- Send reminder messages according to configured strategy.
- Collect and structure recipient excuses/status updates.
- Provide concise summaries to assigners.
- Offer practical completion tips and motivational prompts.
- Issue bounded push-back when miss patterns trigger policy.

## 3. Disallowed AI Actions
- Inventing or altering task outcomes.
- Applying consequences outside configured policy.
- Using shaming, threats, insults, or coercive language.
- Ignoring quiet hours and messaging caps.
- Taking irreversible account actions.

## 4. Tone Modes
- `neutral`: concise and factual.
- `supportive`: empathetic and encouraging.
- `firm`: direct but respectful and non-coercive.

## 5. Push-Back Policy
- Modes: `off` or `bounded`.
- In bounded mode:
  - max push-back attempts per task window
  - cooldown between push-back messages
  - no push-back during quiet hours
- Guardian can disable push-back per relationship.

## 6. Excuse Handling
- AI captures excuse text and optional category.
- Suggested categories:
  - forgot
  - time_conflict
  - unclear_instructions
  - lacking_resources
  - refused
  - other
- AI sends summary digest to assigner at configured cadence.

## 7. Explainability and Audit
Every AI action logs:
- actor (`ai_mediator`)
- source inputs (task id, recent events)
- policy rule references
- output classification and action type
- timestamp and request id

## 8. Human Override
- Guardians can override AI classification, reminder frequency, and consequence recommendation.
- Overrides are recorded in the audit log.
