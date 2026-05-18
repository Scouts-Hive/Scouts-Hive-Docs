# Contributing Guide

## Repos

| Repo | Language | What to contribute |
|---|---|---|
| `Scouts-Hive-Contract` | Rust | Smart contract logic, tests |
| `Scouts-Hive-Backend` | TypeScript | API endpoints, event listener, services |
| `Scouts-Hive-Frontend` | TypeScript / Next.js | UI components, pages, hooks |
| `Scouts-Hive-Docs` | Markdown | Documentation, diagrams, guides |

## Workflow

1. Fork the relevant repo and create a branch:
   ```
   feat/<repo-prefix>/<short-description>
   fix/<repo-prefix>/<short-description>
   ```
   Examples: `feat/contract/spotlight-payment`, `fix/frontend/wallet-connect`

2. Make your changes. Follow the conventions below.

3. Run tests before pushing:
   ```bash
   # Contract
   cargo test

   # Backend / Frontend
   npm test
   ```

4. Open a pull request against `main`. Keep the PR title under 70 characters. Use the description for context.

5. A maintainer will review within 48 hours.

## Conventions

### Contract (Rust)
- Every public function must have a corresponding test in `src/test.rs`
- Use `#[contracttype]` for all on-chain data structures
- Emit an event for every state-changing operation
- After any contract change, regenerate TS bindings — see [`ai.md`](../ai.md)

### Backend (TypeScript)
- All routes live under `src/api/routes/`
- Validate request bodies with Zod before touching the database
- Never call contract functions directly from route handlers — use the service layer

### Frontend (TypeScript / Next.js)
- Do not edit files under `src/contracts/` — they are auto-generated
- Use the `useWallet` hook for all wallet interactions
- Keep components in `src/components/`, pages in `src/app/`

### Docs
- Write in plain English, present tense
- Prefer tables and code blocks over long prose
- Update `contracts/deployment.md` whenever contract IDs change

## Cross-Repo Changes

When a change spans multiple repos, follow this order:

1. Contract → deploy to testnet
2. Regenerate TS bindings → copy to Frontend
3. Update Backend if events or functions changed
4. Update Docs

See [`ai.md`](../ai.md) for the full cross-repo checklist.

## Reporting Issues

Open an issue in the relevant repo. Include:
- What you expected to happen
- What actually happened
- Steps to reproduce
- Network (testnet / mainnet) and relevant contract IDs
