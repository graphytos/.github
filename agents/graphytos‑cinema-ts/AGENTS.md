# Graphytos Cinema Workspace

Three related repos under active development:

- **graphytos-cinema-backend-ts** — https://github.com/graphytos/graphytos-cinema-backend-ts
- **graphytos-cinema-frontend** — https://github.com/graphytos/graphytos-cinema-frontend
- **graphytos-cinema-api-ts** — https://github.com/graphytos/graphytos-cinema-api-ts

The api-ts package is the typed HTTP client + shared types that sits
between the backend and the frontend. It is the **source of truth**
for the API contract — the backend mirrors its types, the frontend
imports the client. Backend and frontend are improved together;
changes in the api-ts package usually need matching updates in both
siblings (and vice versa).

## Obsidian Vaults

- `not-real-library/` — legacy Obsidian vault. Migrate notes from here into the new vault as the API stabilizes. (look here when testing the api and wondering what film or show to add to my new vault)
- `graphytos-notreallyfilms-vault/` — new vault for my film and show libary, what the backend and frontend are for manageing

## Dev Workflow

- Start the backend early and often to connect and test against the frontend.
- Keep frontend + backend changes in sync within each session.
- Prefer small, verifiable iterations over large refactors.