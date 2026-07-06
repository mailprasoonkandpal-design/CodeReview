# Apex Review Agent Runbook

This runbook explains how to use the Apex Code Review Agent across many Salesforce orgs without opening every org in VS Code.

## Recommended Workflow

1. Authenticate each org in Salesforce CLI.
2. Retrieve Apex metadata into a temporary folder per org.
3. Run Salesforce Code Analyzer or PMD.
4. Give the analyzer output and the most relevant Apex files to the agent.
5. Save one report per org.
6. Create a combined summary across all orgs.

## Folder Layout

Use a simple structure like this:

```text
apex-review-workspace/
  orgs.txt
  retrieved/
    org-a/
    org-b/
  analyzer-results/
    org-a.json
    org-b.json
  reports/
    org-a-review.md
    org-b-review.md
    combined-summary.md
```

## Example Org List

Create `orgs.txt` with one Salesforce CLI alias per line:

```text
client-prod-a
client-prod-b
client-sandbox-c
```

## Retrieval Command

For each org, retrieve Apex classes and triggers:

```bash
sf project retrieve start \
  --metadata ApexClass,ApexTrigger \
  --target-org ORG_ALIAS
```

If you also need LWC, Visualforce, permissions, or flows, add metadata types:

```bash
sf project retrieve start \
  --metadata ApexClass,ApexTrigger,LightningComponentBundle,ApexPage,PermissionSet,Flow \
  --target-org ORG_ALIAS
```

## Analyzer Command

Run Salesforce Code Analyzer from the folder containing the retrieved project:

```bash
sf code-analyzer run \
  --target force-app \
  --output-file analyzer-results/ORG_ALIAS.json
```

If your installed CLI uses a different command name, check:

```bash
sf code-analyzer --help
```

## Prompt To Use With The Agent

Paste this with the relevant files and analyzer output:

```text
Review this Salesforce Apex code for production risk.

Context:
- Org: ORG_ALIAS
- Scope: Apex classes and triggers
- Business context: [briefly describe if known]
- Analyzer output is included below.

Focus on:
1. Security: sharing, CRUD/FLS, unsafe dynamic SOQL, exposed methods
2. Governor limits and bulk safety
3. Trigger design and recursion
4. Data correctness
5. Error handling
6. Test quality
7. Maintainability

Return findings using the Apex Code Review Agent format.
Prioritize Critical and High issues first.
Avoid generic style feedback unless it affects real maintainability.
```

## What To Send The Agent

For each org, send:

- Analyzer JSON/SARIF results
- The affected Apex files for Critical/High findings
- All triggers
- Trigger handler classes
- Classes with `without sharing`
- Classes using `Database.query`, `@AuraEnabled`, callouts, Batch, Queueable, or Scheduled Apex
- Relevant test classes when asking for test review

## Combined Summary Prompt

After you have per-org reports, ask:

```text
Create a combined summary across these Apex review reports.

Output:
- Highest-risk orgs
- Top recurring issue patterns
- Critical/High issue counts by org
- Recommended remediation roadmap
- Quick wins
- Issues that require architecture decisions

Use this table:

| Org | Critical | High | Medium | Top Risk | Recommended Action |
| --- | ---: | ---: | ---: | --- | --- |
```

## Practical Advice

Do not ask AI to review every file in every org at once. Use scanners to narrow the scope, then use the agent for judgment, prioritization, and fix guidance.

Start with:

1. All triggers
2. All `without sharing` classes
3. All dynamic SOQL
4. All public/global service endpoints
5. All async/integration code
6. Analyzer Critical and High findings

