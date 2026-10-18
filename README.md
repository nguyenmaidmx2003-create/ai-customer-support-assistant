# AI Customer Support Workflow Assistant

An AI-assisted MVP for turning customer data-update requests into draft work packages for developer review and QC verification. The app helps organize a request; it does not perform system updates or contact customers.

## Business problem and intended users

Customer requests can be passed between team leads, developers, and QC, making it important to keep requested values, open questions, execution planning, and verification steps clear and traceable.

This prototype supports two roles:

- **Developer:** Reviews a request, its extracted changes, limitations, and a suggested work plan before taking any separately authorized action.
- **QC:** Uses the extracted requirements and checklist to independently verify any work performed outside this app.

## Workflow and features

1. Paste the customer email or request you received from the team mailbox. **Try Example Request** fills in the fictional Sarah Johnson email with three item status changes.
2. Select **Analyze Request** to send the request to Gemini and prepare a structured draft work package.
3. Review the summary, extracted information, requested changes, clarifications needed, suggested developer actions, and QC verification checklist.
4. Review the request classification, scope, stated authorization, explicit and inferred details, unresolved questions, and any referenced attachments that were not supplied.
5. Clarify ambiguous or missing item identifiers before any work. A stated group condition (for example, all items currently inactive) is represented as criteria, not as a missing item ID. Criteria-based and bulk updates remain blocked pending confirmation of the full scope and applicable authorization. The suggested actions separate immediate clarification from later identification of affected items, preparation of a proposed change list, and QC after approved and authorized execution.
6. Export the currently displayed work package as a Markdown (`.md`) file for review or record keeping.

The assistant classifies requests as specific-item updates, criteria-based updates, bulk data updates, report requests, issue/CAPA requests, incomplete requests, or unsupported requests. Request type describes the customer's intent; a separate review status says whether clarification is needed, the request is ready for human review, or the request is unsupported. Scope, authorization, and execution status are displayed separately. For example, a request to update one item with no ID remains a specific-item update, has a “Needs clarification” review status, and explains that the ID is missing. It is not reclassified as incomplete solely because the ID is absent. Item identifiers are preserved as written, including leading zeros (for example, `002`), and the backend rejects a non-empty change identifier unless it appears exactly in the request text. The app does not independently parse identifier boundaries, so a person must confirm that each extracted identifier is the intended one. The requested-changes table can show either an item ID or a stated criteria target, plus identifier status, field, requested value, and conditions.

Report requests and issue/CAPA requests are classified and routed at a draft level only. The assistant does not generate reports, investigate issues, determine root causes, or resolve CAPAs.

All generated content is a draft requiring human review. The app does not make data changes, verify live system records, send email, or integrate with Gmail or Jira.

In the current MVP, customer email content must be copied and pasted into the request box. The described group-mailbox and Jira-task workflow is not automated; Gmail and Jira integrations are future improvements.

## Architecture

- **Frontend:** A single `index.html` file with HTML, CSS, and browser JavaScript. It collects requests, displays work packages, handles table and blocked states, and creates Markdown downloads locally in the browser.
- **Backend:** `server.js` runs an Express server, serves the frontend, validates requests and Gemini responses, and exposes `POST /api/generate`.
- **AI:** The backend uses the Google GenAI SDK to call Gemini and request a structured JSON work package. Exporting does not make an AI API call.

The backend listens on the port supplied by the hosting service, with `3000` as the local fallback. The optional health endpoint is `GET /api/health`; it reports whether Gemini is configured without returning the API key. The generation endpoint has a basic in-memory limit of 10 requests per client IP in a 15-minute window. This is intended for a public demo, not as a distributed production rate limiter.

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
npm start
```

Open [http://localhost:3000](http://localhost:3000) in a browser. Stop the local server with `Ctrl+C` in the terminal.

The `start` script runs the existing `server.js`. The repository does not currently have a configured automated test suite.

## Configure `GEMINI_API_KEY` safely

1. Create `.env` in the project root. The `.gitignore` excludes `.env` and `node_modules/` from normal untracked-file additions.
2. Add your key locally using this format:

   ```dotenv
   GEMINI_API_KEY=your_gemini_api_key
   ```

3. Save the file and restart the server.

Use your own key in your local `.env`; do not paste it into source files, README examples, screenshots, or commits. Never share the key. Before pushing, you can check that Git ignores the file with `git check-ignore -v .env`; `.gitignore` does not untrack a file that was already committed, so make sure `.env` is not tracked. If a key is ever committed or exposed, revoke it and create a replacement. If the key is not configured, the app returns an error when asked to generate a work package. The backend uses the key to authenticate its Gemini API requests; it does not put the key in its API responses to the browser.

## Deploy to Render

Create a **Web Service** for this repository on Render. Use these settings:

- **Runtime:** Node
- **Build command:** `npm install`
- **Start command:** `npm start`

In the Render service's **Environment** settings, add `GEMINI_API_KEY` and set its value there. Do not add `.env` to the repository or put the key in the source code. Render supplies the listening port through `PORT`; the app uses `3000` only when running locally without a configured port.

After deployment, open the service's `onrender.com` URL and test `GET /api/health` and the UI with synthetic requests only. Render's free web service may sleep after inactivity; the first visit or request can take some time while the service wakes. The demo has a simple in-memory request limit that resets when the service restarts or sleeps. This app is a prototype: it prepares drafts and does not connect to company systems or execute updates.

## Manual testing

Use synthetic requests only. Generation sends the request text to Gemini through the backend, so do not enter private customer information. These tests produce drafts only and do not modify data.

### Single-item request

1. Paste a single-item request:

   ```text
   Please update the status of item ITEM-1001 to Active.
   ```

2. Click **Analyze Request**.
3. Review that the summary and extracted information describe the request, and that the requested-changes table shows the item, field, and value.
4. Confirm the suggested developer actions and QC verification checklist remain drafts for human review.

### Multiple-item request

Click **Try Example Request** to fill the request box with the fictional customer email, or open [`examples/sample_data_update.md`](./examples/sample_data_update.md) and copy its complete email into the box. Click **Analyze Request** and check for three requested-change rows: `ITEM-1001` → `Active`, `002` → `Active`, and `ITEM-1003` → `Inactive`. Confirm that `002` is preserved exactly, and review any clarifications needed, including system/screen or approval information.

Alternatively, enter a short synthetic request such as:

```text
Please update the status of item ITEM-1001 to Active and set item 002 to Pending.
```

Click **Analyze Request** and check that each requested item has its own table row. Confirm that `002` is displayed with its leading zeros and is not expanded or normalized.

### Missing or ambiguous ID

Enter a request such as:

```text
Please update the status to Active, but I do not have the item ID.
```

Click **Analyze Request**. If Gemini identifies the item as missing, the table should indicate that it is missing, the package should show a blocked notice, and suggested developer actions should say not to execute any change until the ambiguity is clarified and required authorization is confirmed.

The request type should remain **Specific item update**, while the separate review status should say **Needs clarification** and the clarifications should ask for the exact item ID.

Gemini generates the extraction, so responses may vary. If a test does not clearly identify an item as missing or ambiguous, try rephrasing the synthetic request and review the output rather than treating it as authoritative.

### Criteria-based update

Try: `Please update the status of all items currently set as inactive to active.`

Check that the assistant classifies this as a criteria-based or bulk update and shows the condition (current status is inactive), field (`status`), and requested value (`active`) in the change list without requiring individual IDs. The actual affected items and their current statuses must remain unidentified and unverified. Clarifications should ask about the complete scope and applicable authorization; the work package should be blocked. Suggested actions should first clarify, then conditionally identify items and prepare a proposed list, and reserve QC for after approved, authorized execution.

### Report or issue/CAPA request

Try: `Prepare a monthly report of service requests by facility for September.` The assistant should classify and route it as a report request, identify missing report specifications, and state that report generation is not supported.

Try: `Several customers report that item ITEM-1001 intermittently shows the wrong status. Please investigate and provide a CAPA.` The assistant should classify and route it as an issue/CAPA request without inventing a root cause, investigation result, or CAPA.

## Markdown export

The **Export Work Package** button stays disabled until a work package is generated successfully. Click it to download a date-stamped `.md` file containing the customer request, request type, review status, scope, authorization and execution statuses, summary, extracted information, requested-changes table, clarifications needed, suggested developer actions, and QC verification checklist.

The export uses the content currently displayed in the page and is created in the browser. It does not make another Gemini request and does not claim that any update or verification has been performed.

## Current limitations

- This is an MVP for preparing data-update work packages, not an execution or case-management system.
- Generation requires a configured Gemini API key and an available network connection to the Gemini API.
- AI-generated classification and extraction can be incomplete or incorrect; a person must review it and resolve questions before taking any action.
- Criteria-based and bulk changes cannot be scope-confirmed by this app; they remain blocked for human scope and authorization review.
- Report generation and issue investigation/CAPA resolution are not implemented; those request types are classified and routed only.
- Attachments cannot be uploaded or read separately. If a request references a file whose contents are not pasted into the request, the assistant should identify it as unavailable.
- Identifier status and other extracted details depend on Gemini's interpretation of the request.
- The app does not connect to Gmail, Jira, customer databases, or production systems.
- It does not execute updates, verify live records, send customer emails, authenticate users, or save work packages in a database.
- The app does not include production authentication, persistent rate limiting, or hardened deployment security. Its in-memory rate limit is intended only as a basic public-demo safeguard.

## Possible future improvements

- Add automated tests for backend validation and frontend table/export behavior.
- Improve extraction review with clearer user confirmation of item IDs, fields, and requested values.
- Add optional local work-package saving and a completion/QC review workflow.
- Add configurable server settings and documented deployment/security guidance.
- Support additional request types such as reports or issue/CAPA requests after defining and testing their workflows.
