# CodeWhy AI

CodeWhy AI answers "why does this code exist?" from repository evidence: commits, pull requests, issues, incidents, design documents, and discussions. Every claim cites the records it relies on. When the evidence is missing, the app says so instead of guessing. It also shows when a decision was later replaced.

This is a **frontend-only prototype** for a hackathon demo. It runs on a **fictional** repository, "Northstar Checkout", bundled as local fixture data. It does not read GitHub, call an LLM, or need an API key.

## Run it

Requirements: Node.js 20 or newer, and npm.

```bash
npm install
npm run dev        # open the printed local URL
```

On Windows PowerShell, use the same commands. If PowerShell blocks `npm` with an execution-policy error, run this once in a terminal you control:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Other scripts:

```bash
npm test           # Vitest: retrieval engine, fixture integrity, URL validation
npm run build      # type-check, then produce a static site in dist/
npm run preview    # serve dist/ locally
```

Deploy: `dist/` is static and uses relative asset paths, so any static host works (Netlify, Vercel, GitHub Pages, or a plain folder).

## What is implemented

- **Overview.** Repository URL validation for github.com. Live analysis is explained as unavailable in this build, and the bundled demo is offered instead. Metrics are computed from the fixture. Evidence coverage and gaps are shown, with a recent-events list.
- **Ask.** Deterministic retrieval over the fixture. Each answer has a headline, claims, and a citation chip for each record. It also shows the searched terms, a demo confidence level with its reasons, follow-up questions, copy, and clear. The app asks for clarification when two candidates tie. It refuses when there is no rationale record. A loading state is shown while the search runs.
- **Evidence.** Search, type, relationship, and date filters. A detail panel shows each record's summary, author, date, relationships, and a source link. The fixture has no public URLs, and the UI says so. Simulated discussions and design documents are labelled.
- **Knowledge graph.** React Flow. Only explicit fixture relationships are drawn. Click a node to see its records and highlight its connections. Type filters, fit to screen, reset view, and a legend are included.
- **Decision timeline.** Decisions in date order, with status (active, superseded, uncertain), owner, affected code, original reason, and supporting records. Supersession comes only from an explicit SUPERSEDES relationship. Earlier records stay visible.
- **Settings.** Light and dark theme, saved locally. Reset demo.

## Demo script (about three minutes)

1. **Overview.** Point out that coverage is calculated from the data. Note the evidence gap for `LEGACY_TAX_ROUNDING`.
2. **Ask:** "Why was the retry limit set to 3?" Show the issue, PR, incident, and commit citations, and the high confidence reasons.
3. **Ask:** "Was the original payment library decision replaced?" The answer is yes. DEC-01 is superseded by DEC-03, and the old record is kept.
4. **Ask:** "Why does the undocumented legacy flag exist?" The app refuses and lists what is missing.
5. **Ask:** "Why was the timeout changed?" The app asks which of two decisions is meant.
6. **Graph.** Select DEC-04 and show the highlighted connections.
7. **Timeline.** Show the superseded and uncertain decisions.

## Fictional dataset

"Northstar Checkout" is a teaching dataset. It is not a real GitHub repository. Its identifiers (C-, PR-, ISS-, INC-, DD-, DISC-, DEC-) are internal demo IDs.

| Record type | Count |
| --- | --- |
| Commits | 9 |
| Pull requests | 3 |
| Issues | 3 |
| Incidents | 2 |
| Design documents (simulated) | 2 |
| Discussions (simulated) | 2 |
| Decisions | 5 (1 superseded, 1 uncertain) |
| Code entities | 6 |
| People | 2 |
| Explicit relationships | 37 |

Relationship types: `CHANGED` (commit to code), `INCLUDES` (PR to commit), `RESOLVES` (PR to issue), `CAUSED_BY`, `DISCUSSED_IN`, `DECIDED_BY`, `AFFECTS`, `SUPERSEDES` (newer decision to older one), and `RELATED_TO`.

## How the answers are produced

1. **Terms.** The question is lowercased, camelCase is split, plurals are stemmed, and stop words are removed. Words such as "replaced" are read as supersession intent.
2. **Scoring.** Decisions and code entities are scored by field. An identifier match is worth 5, a title or symbol match 3, an alias match 2, and a description match 1. A candidate that a higher-scoring decision already covers is folded into that decision.
3. **Ambiguity.** If several candidates score within 80% of the best, the app asks instead of guessing.
4. **Tracing.** A breadth-first trace runs up to three hops over explicit relationships. Owners are shown separately. A decision reached from elsewhere is never expanded. Entities expand only to their commits. `RELATED_TO` is weak and never extends the trace.
5. **Rationale check.** A rationale record is an issue, incident, design document, or discussion. With no rationale record and no supersession link, the answer is "insufficient evidence".
6. **Claims.** Each claim is built from fixture fields and cites its records. Tests confirm that every citation is a real record the trace reached.
7. **Confidence.** This is a transparent demo rule, not a calibrated probability. It counts matched terms, rationale records, and supersession links. An uncertain decision is capped at low.

## Feature checklist

| Area | Status |
| --- | --- |
| Engine: supported answers, supersession, insufficient evidence, ambiguity, no match | Implemented. Verified by execution in the build sandbox (see below). |
| Engine: citation correctness and bounded traversal | Implemented. Verified by execution. |
| URL validation | Implemented. Verified by execution. |
| Fixture integrity and minimum sizes | Implemented. Verified by execution. |
| TypeScript strictness of engine, fixture, types, and URL module | Verified with TypeScript 6.0.3. The project pins ~5.6, so run `npm run build` to confirm. |
| Overview, Ask, Evidence, Graph, Timeline screens | Implemented. Not compiled or run in a browser in the build sandbox. Run `npm run build` and `npm run dev` to confirm. |
| `npm install`, `npm test` (Vitest), `npm run build` (Vite) | Not run in the build sandbox, which has no network access. Run them locally. |

## Not included in this build

- Live GitHub analysis and any LLM. The engine is a deterministic local module, so a backend can replace it later.
- Recharts charts, a separate repository-history explorer (the Evidence screen provides search and filters), a dedicated settings page (theme and reset live in the sidebar), the guided tour, and the PPT outline.

## Project layout

```
src/
  types.ts            shared domain types (records, decisions, relationships, answers)
  data/northstar.ts   fictional fixture and its explicit relationships
  lib/engine.ts       deterministic retrieval, tracing, supersession, claims, confidence
  lib/repoUrl.ts      GitHub URL validation (no network calls)
  app/                shell, navigation, theme, routing, and shared workspace state
  features/           overview, ask, evidence, graph, and timeline screens
  styles/index.css    theme variables and component styles
tests/                Vitest suites for the engine and URL validation
```

## Limits and future backend

- Retrieval is keyword and graph based. It will miss paraphrases that the fixture terms do not cover, and it can only answer from records it holds.
- Confidence is rule based and not a probability.
- The fixture is fictional and has no public links.

A backend would fetch commits, pull requests, issues, and discussions from the GitHub API and store them in the same shapes as `src/types.ts`. It would keep the rules that matter most: every citation must be a retrieved record, and missing rationale must still produce "insufficient evidence".

## Environment

Nothing needs to be configured for the demo. `.env.example` lists the one variable a future backend might read. Do not put secrets in `VITE_` variables, because Vite bundles them into public JavaScript.

## UI refresh

This version includes a refreshed visual system: a dark gradient navigation rail, glass-style panels, subtle 3D depth on repository/stat cards, luminous accent gradients, animated hover states, responsive layouts, and reduced-motion support. The visual refresh does not change the prototype's frontend-only data/integration boundary: the bundled Northstar repository remains demo data, and live GitHub analysis still requires a backend.
