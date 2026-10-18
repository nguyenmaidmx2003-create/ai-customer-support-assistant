# AI Customer Support Workflow Assistant

An AI-assisted MVP for turning customer data-update requests into draft work packages for developer review and QC verification. The app helps organize a request; it does not perform system updates or contact customers.

## Business problem and intended users

Customer requests can be passed between team leads, developers, and QC, making it important to keep requested values, open questions, execution planning, and verification steps clear and traceable.

This prototype supports two roles:

- **Developer:** Reviews a request, its extracted changes, limitations, and a suggested work plan before taking any separately authorized action.
- **QC:** Uses the extracted requirements and checklist to independently verify any work performed outside this app.

## Workflow and features

1. Enter or paste a customer request, including a copied email with its subject, sender, and body. The **Load Sample** button fills in the fictional Sarah Johnson email with three item status changes.
2. Generate a draft work package with Gemini.
3. Review the summary, extracted information, requested changes, missing information, developer plan, and QC checklist.
4. Clarify ambiguous or missing item identifiers before any work. When Gemini marks a requested change's identifier as ambiguous or missing, the backend marks the package as blocked and adds a do-not-execute instruction until clarification and required authorization are confirmed.
5. Export the currently displayed work package as a Markdown (`.md`) file for review or record keeping.

The prompt instructs Gemini to preserve item identifiers as written, including leading zeros (for example, `002`), and the backend rejects a non-empty identifier unless it appears exactly in the request text. The app does not independently parse identifier boundaries, so a person must confirm that each extracted identifier is the intended one. The requested-changes table displays the returned identifier, status, and any clarification note.

All generated content is a draft requiring human review. The app does not make data changes, verify live system records, send email, or integrate with Gmail or Jira.

In the current MVP, customer email content must be copied and pasted into the request box. The described group-mailbox and Jira-task workflow is not automated; Gmail and Jira integrations are future improvements.

## Architecture

- **Frontend:** A single `index.html` file with HTML, CSS, and browser JavaScript. It collects requests, displays work packages, handles table and blocked states, and creates Markdown downloads locally in the browser.
- **Backend:** `server.js` runs an Express server, serves the frontend, validates requests and Gemini responses, and exposes `POST /api/generate`.
- **AI:** The backend uses the Google GenAI SDK to call Gemini and request a structured JSON work package. Exporting does not make an AI API call.

The backend listens on port `3000`. The optional health endpoint is `GET /api/health`; it reports whether Gemini is configured without returning the API key.

## Prerequisites

- Node.js and npm
- A Gemini API key for generating work packages

## Install and run locally

From the project directory, install dependencies:

```bash
npm install
```

Create a local `.env` file in the project root and add your Gemini key as described below. Then start the server:

```bash
node server.js
```

Open [http://localhost:3000](http://localhost:3000) in a browser. Stop the local server with `Ctrl+C` in the terminal.

There is no `start` script or configured automated test suite in the current `package.json`.

## Configure `GEMINI_API_KEY` safely

1. Create `.env` in the project root. The `.gitignore` excludes `.env` and `node_modules/` from normal untracked-file additions.
2. Add your key locally using this format:

   ```dotenv
   GEMINI_API_KEY=your_gemini_api_key
   ```

3. Save the file and restart the server.

Use your own key in your local `.env`; do not paste it into source files, README examples, screenshots, or commits. Never share the key. Before pushing, you can check that Git ignores the file with `git check-ignore -v .env`; `.gitignore` does not untrack a file that was already committed, so make sure `.env` is not tracked. If a key is ever committed or exposed, revoke it and create a replacement. If the key is not configured, the app returns an error when asked to generate a work package. The backend uses the key to authenticate its Gemini API requests; it does not put the key in its API responses to the browser.

## Manual testing

Use synthetic requests only. Generation sends the request text to Gemini through the backend, so do not enter private customer information. These tests produce drafts only and do not modify data.

### Single-item request

1. Enter a single-item request:

   ```text
   Please update the status of item ITEM-1001 to Active.
   ```

2. Click **Generate work package**.
3. Review that the summary and extracted information describe the request, and that the requested-changes table shows the item, field, and value.
4. Confirm the plan and QC checklist remain drafts for human review.

### Multiple-item request

Click **Load Sample** to fill the request box with the fictional customer email, or open [`examples/sample_data_update.md`](./examples/sample_data_update.md) and copy its complete email into the box. Click **Generate work package** and check for three requested-change rows: `ITEM-1001` → `Active`, `002` → `Active`, and `ITEM-1003` → `Inactive`. Confirm that `002` is preserved exactly, and review any missing system/screen or approval information.

Alternatively, enter a short synthetic request such as:

```text
Please update the status of item ITEM-1001 to Active and set item 002 to Pending.
```

Generate the work package and check that each requested item has its own table row. Confirm that `002` is displayed with its leading zeros and is not expanded or normalized.

### Missing or ambiguous ID

Enter a request such as:

```text
Please update the status to Active, but I do not have the item ID.
```

Generate the work package. If Gemini identifies the item as missing, the table should indicate that it is missing, the package should show a blocked notice, and the plan should say not to execute any change until the ambiguity is clarified and required authorization is confirmed.

Gemini generates the extraction, so responses may vary. If a test does not clearly identify an item as missing or ambiguous, try rephrasing the synthetic request and review the output rather than treating it as authoritative.

## Markdown export

The **Export work package** button stays disabled until a work package is generated successfully. Click it to download a date-stamped `.md` file containing the customer request, summary, extracted information, requested-changes table, missing information and limitations, developer plan, and QC checklist.

The export uses the content currently displayed in the page and is created in the browser. It does not make another Gemini request and does not claim that any update or verification has been performed.

## Current limitations

- This is an MVP for preparing data-update work packages, not an execution or case-management system.
- Generation requires a configured Gemini API key and an available network connection to the Gemini API.
- AI-generated extraction can be incomplete or incorrect; a person must review it and resolve questions before taking any action.
- Identifier status and other extracted details depend on Gemini's interpretation of the request.
- The app does not connect to Gmail, Jira, customer databases, or production systems.
- It does not execute updates, verify live records, send customer emails, authenticate users, or save work packages in a database.
- The server uses a fixed local port (`3000`); production deployment and security configuration are not included.

## Possible future improvements

- Add automated tests for backend validation and frontend table/export behavior.
- Improve extraction review with clearer user confirmation of item IDs, fields, and requested values.
- Add optional local work-package saving and a completion/QC review workflow.
- Add configurable server settings and documented deployment/security guidance.
- Support additional request types such as reports or issue/CAPA requests after defining and testing their workflows.
