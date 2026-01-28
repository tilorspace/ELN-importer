# ELN-importer

A small browser-based tool to preview and import ELN (Electronic Lab Notebook) export packages (RO‑Crate / `.eln`) into RSpace. It parses exports, normalises metadata, lets users review items in a preview session, and then maps and sends selected items to an RSpace instance.

## Table of contents
- [What it does](#what-it-does)
- [Key features](#key-features)
- [Architecture (high level)](#architecture-high-level)
- [Quick start](#quick-start)
- [How to use](#how-to-use)
- [Testing & verification](#testing--verification)
- [Development notes & recent fixes](#development-notes--recent-fixes)
- [Contributing](#contributing)
- [Troubleshooting](#troubleshooting)
- [License & contact](#license--contact)

## What it does
- Accepts `.eln` export files (RO‑Crate style ZIP containing JSON metadata + attachments).
- Parses the package and normalises metadata into internal types.
- Validates and classifies items, extracts custom fields, and shows a preview UI.
- Maps selected items into RSpace import payloads and posts them to an RSpace server via the API.

## Key features
- Browser-only app (no backend required): runs with Vite/React + TypeScript.
- Parser normalises real-world variations (e.g., `contentSize` as number or string).
- Preview UI with item cards and detail modal so users can review before importing.
- Mapping layer to convert ELN metadata into RSpace-compatible payloads.
- Basic input validation and path sanitisation.

## Architecture (high level)
- Frontend SPA (TypeScript + React, Vite) — `src/`
- Parser/types — `src/utils/eln-parser.ts`, `src/types/eln.ts`
- Validation & classification — `src/utils/ValidationEngine.ts`, `src/utils/ClassificationEngine.ts`
- Preview/session — `src/components/*`, `src/services/preview-session.ts`
- RSpace mapping & API services — `src/services/rspace-mapper.ts`, `src/services/rspace-importer.ts`, `src/services/rspace-api.ts`

## Quick start

Prerequisites
- Node.js (14+ recommended) and npm installed.

Install and run locally
```bash
npm install
npm run dev
```
Open the app in your browser (Vite default: http://localhost:5173).

Build for production
```bash
npm run build
# optionally preview build
npm run preview
```

## How to use
1. Open the web app in your browser.
2. Upload an `.eln` file (a zipped RO‑Crate containing metadata + files).
3. The parser reads the package and shows a preview list of items with metadata.
4. Inspect items in the preview UI (use filters, open detail modal).
5. When ready, choose to import selected items. The app will map data and send it to RSpace via the configured API.
6. Review import results shown in the app.

Note: The app communicates with RSpace via the RSpace API. In many setups you will need a server URL and API token/credentials. Check project configuration or ask an engineer for how to supply these credentials securely.

## Testing & verification
Recommended manual/integration checks:
- Run `npm run build` to verify the project builds.
- Test with at least two `.eln` samples:
  - A real-world eLabFTW export (where `contentSize` may be a number).
  - A spec-compliant RO‑Crate `.eln` (where `contentSize` may be a string).
- Verify the preview shows correct metadata (check `@id`, `@type`, file names, sizes).
- Perform an import to a staging RSpace instance and confirm contents arrive as expected.

Recommended unit tests to add (examples)
```typescript
test('handles contentSize as string', () => {
  const item = { contentSize: "85530" };
  // should be parsed/normalised to number 85530
});

test('FileMetadata uses @id and @type', () => {
  const metadata = parseFileMetadata(item);
  expect(metadata).toHaveProperty('@id');
  expect(metadata).toHaveProperty('@type');
  expect(metadata).not.toHaveProperty('id'); // old bug
});
```

## Development notes & recent fixes
- The codebase was refactored to be ELN-generic (removed eLabFTW-specific naming).
- `contentSize` is now normalised to accept both string (spec) and number (real exports).
- File metadata properties were fixed to use `@id` and `@type`.
- Parser class renamed to `ELNParser`; types moved to `src/types/eln.ts`.
- Old files `src/types/elabftw.ts` and `src/utils/elabftw-parser.ts` were made obsolete — consider removing them after verification:
```bash
rm src/types/elabftw.ts
rm src/utils/elabftw-parser.ts
```

## Contributing
We welcome contributions from the community. Suggested workflow:
1. Fork the repo and create a feature branch: `git checkout -b feat/some-change`
2. Implement changes and run the app locally to test.
3. Commit and open a pull request with a clear title and description.
4. Include tests where appropriate (unit + integration if it touches parsing/importing).

Suggested issues to tackle:
- Add automated unit tests for `contentSize` and `FileMetadata`.
- Add CI pipeline to run `npm ci && npm run build` on PRs.
- Add sample `.eln` export files for different ELN systems (eLabFTW, Chemotion, Kadi4Mat) to `tests/samples/` for integration testing.

## Troubleshooting
- Parser errors on upload: check the `.eln` package is a valid ZIP and contains RO‑Crate JSON metadata.
- Missing `@id` / `@type`: make sure you are using the latest refactored parser (ELNParser) and types.
- Import failures to RSpace: confirm the target RSpace endpoint and API token are correct and that the user has permission to create content in the target workspace.

If you hit a bug, please open a GitHub issue with:
- Repro steps
- A sample `.eln` file (if possible) or minimal reproduction
- Browser console output and any stack traces

## License & contact
- License: (add your license here, e.g. MIT)
- Maintainers / Contact: open issues in this repository or contact the RSpace product team.

---