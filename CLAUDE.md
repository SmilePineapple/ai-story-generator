@AGENTS.md

## Claude-specific notes

- In cloud sessions, `node_modules` isn't present at start: run `npm ci --legacy-peer-deps` before you lint, build, or type-check.
- Keep AGENTS.md current. When you add env vars, scripts, tests, or the LLM pipeline, update the relevant section in the same change.
