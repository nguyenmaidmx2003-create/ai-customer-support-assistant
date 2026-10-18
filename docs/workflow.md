# Customer Support Workflow

## 1. Current Workflow

1. Customer sends an email through Gmail.
2. The team lead forwards the email to the group mailbox.
3. The email automatically creates a Jira task.
4. The team lead assigns one developer and one QC to the task.
5. QC reads the request and communicates the requirements to the developer.
6. The developer performs the requested work.
7. The developer provides the output to QC.
8. QC verifies the output against the request.
9. If the output is correct, the team sends the deliverable and a response email to the customer.

## 2. Request Types

### Data Update
- Customer request: Update existing system data.
- Expected deliverables: Screenshot and customer notification email.

### Report Request
- Customer request: Prepare a report based on specified requirements.
- Expected deliverable: Data sheet.

### Issue Report
- Customer request: Report a system issue.
- Expected deliverable: Corrective and Preventive Action (CAPA).

## 3. Proposed AI Assistance

### Developer Assistant
- Extract the requested data changes and input values.
- Prepare a work plan using a validated procedure.
- Identify missing or conflicting information.
- Prepare a completion summary and supporting-evidence checklist.

### QC Assistant
- Review extracted requirements.
- Generate a verification checklist.
- Help compare actual system data with approved requirements.
- Draft a customer email after QC verification.

## 4. Human Responsibilities

- The team lead assigns the developer and QC.
- QC confirms the requirements and verifies the output.
- The developer executes the requested work.
- For issue reports, the developer investigates and proposes the root cause and CAPA; QC reviews the findings.
- An authorized team member reviews and sends the customer email.

## 5. Safety Boundaries

- Do not invent missing requirements or input values.
- Do not allow the initial prototype to modify production data.
- Do not send customer emails automatically.
- Use synthetic examples unless company data use is explicitly approved.