# Flutter ADHD/ASD Attention Training Mobile App MVP - Copilot Instructions

## Project Overview
Develop a cross‑platform mobile MVP using Flutter that delivers a library of 10‑15 gamified attention‑training exercises for children aged 5‑10 with ADHD or ASD. The app includes COPPA‑compliant onboarding, child profiles, adaptive game logic powered by Python Lambda functions, and a parent dashboard with progress analytics stored in DynamoDB. Subscription billing at $9.99 / month targets North American parents.

## Tech Stack




## How You Must Work

### Before Starting Any Task
1. **READ** `.context-pack/@active-plan.md` — this is your task list
2. **VERIFY** the task you're working on exists in the plan
3. Only work on tasks listed there — do not invent new scope
4. If the plan is missing something critical, note it in the plan first

### While Working
- Follow architecture patterns in `.context-pack/@tech-spec.md`
- Reference `.context-pack/@product.md` for product context

### After Completing a Task
1. Update `.context-pack/@active-plan.md`:
   - Mark completed tasks as `[x]`
   - Add notes if implementation differed from plan
2. If MCP is connected: call `push_progress_update`

### When Blocked
- Record the blocker in `.context-pack/@active-plan.md`
- Do not silently skip tasks

## Documentation References
- `.context-pack/@product.md` - Product specification
- `.context-pack/@tech-spec.md` - Technical architecture
- `.context-pack/@active-plan.md` - **Task list (source of truth)**

## MCP Integration (Optional)
```bash
npx @collablearn/mcp --project-id=286
```
Tools: `fetch_context_pack`, `push_progress_update`, `sync_implementation_state`
