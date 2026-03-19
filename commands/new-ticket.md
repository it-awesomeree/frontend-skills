---
description: Create a new Jira ticket — asks for parent, type, title, description, assignee, status, then creates and registers on board
argument-hint: [optional: rough description of what the ticket is about]
---

# New Jira Ticket Creator

## Step 1: Gather Information

Ask the user for the following (one message, let them answer all at once or piece by piece):

1. **Parent Epic** — show this table for reference:

### Web App Epics
| Key | Epic | Key | Epic |
|-----|------|-----|------|
| AW-323 | Inventory | AW-332 | Comp. Analysis MY |
| AW-324 | Costing | AW-333 | Comp. Analysis SG |
| AW-325 | Parcel Claim MY | AW-334 | Comp. Analysis TT |
| AW-326 | Parcel Claim SG | AW-335 | Design Requests |
| AW-327 | Parcel Claim TT | AW-336 | User Management |
| AW-328 | Returns | AW-343 | New Item MY |
| AW-329 | Order Management MY | AW-344 | New Item SG |
| AW-330 | Order Management SG | AW-345 | SB Variation |
| AW-331 | Sales Velocity | | |

### Non-Web App Epics
| Key | Epic | Key | Epic |
|-----|------|-----|------|
| AW-337 | Shopee Chatbot | AW-340 | Shopee Listing |
| AW-338 | TikTok Chatbot | AW-341 | Infrastructure |
| AW-339 | Bot Automation | AW-342 | Awesomeree Website |

2. **Issue Type** — Bug, Task, or Story (default: Task)
3. **Title** — ticket summary
4. **Description** — user gives rough details, Claude writes it up properly with clear structure (Issue / Recommended Fix / Files Changed as needed)
5. **Assignee** — default: Kelly Ee. Options: Kelly Ee, Melinda, anfal, eten, Janarthan Nair, Pang Yu Hang, nisha, Agnes Foong
6. **Status** — default: TO DO. Options: TO DO, IN PROGRESS, BLOCKED
7. **Labels** — pick from: `frontend`, `backend`, `database`, `devops`, `ai`, `tech-debt` (can be multiple)

## Step 2: Create the Ticket

Use Jira REST API from the browser (user is authenticated on Jira tab):

```
POST https://awesomeree.atlassian.net/rest/api/3/issue
```

Payload structure:
```json
{
  "fields": {
    "project": { "key": "AW" },
    "summary": "<title>",
    "issuetype": { "name": "<Bug|Task|Story>" },
    "parent": { "key": "<epic key>" },
    "labels": ["<label1>", "<label2>"],
    "description": { "type": "doc", "version": 1, "content": [...ADF content...] }
  }
}
```

## Step 3: Assign the Ticket

Look up assignee account ID:
```
GET https://awesomeree.atlassian.net/rest/api/3/user/search?query=<name>
```

Then assign:
```
PUT https://awesomeree.atlassian.net/rest/api/3/issue/<key>/assignee
Body: { "accountId": "<id>" }
```

## Step 4: Set Status

Get available transitions:
```
GET https://awesomeree.atlassian.net/rest/api/3/issue/<key>/transitions
```

Known transition IDs for AW project:
| ID | Status |
|----|--------|
| 11 | To Do |
| 21 | In Progress |
| 3  | BLOCKED |
| 2  | TO REVIEW |
| 4  | APPROVAL |
| 31 | Done |

Then transition:
```
POST https://awesomeree.atlassian.net/rest/api/3/issue/<key>/transitions
Body: { "transition": { "id": "<id>" } }
```

## Step 5: Register on Board

CRITICAL — API-created tickets don't auto-appear on the Kanban board. Always register:

```
POST https://awesomeree.atlassian.net/rest/agile/1.0/board/34/issue
Body: { "issues": ["<key>"] }
```

## Step 6: Navigate to Ticket

Open the ticket in the browser so the user can see it:
```
https://awesomeree.atlassian.net/browse/<key>
```

## Step 7: Confirm & Follow-up

Tell the user:
- Ticket key and link
- Parent, assignee, status, labels
- Ask if they want to update anything (add comments, change description, etc.)

## Follow-up Actions

When the user asks to update the ticket:

### Add a comment
```
POST https://awesomeree.atlassian.net/rest/api/3/issue/<key>/comment
Body: { "body": { "type": "doc", "version": 1, "content": [...ADF...] } }
```

### Update description
```
PUT https://awesomeree.atlassian.net/rest/api/3/issue/<key>
Body: { "fields": { "description": { ...ADF... } } }
```

### Change status
Use the transition API from Step 4.

## Notes
- Always run from a Jira-authenticated browser tab (use existing Jira tab or navigate to any Jira page first)
- Use ADF (Atlassian Document Format) for all rich text fields
- Wrap API calls in `(async () => { ... })()` for the JavaScript tool
- Labels must be from the approved list: `frontend`, `backend`, `database`, `devops`, `ai`, `tech-debt`
- NEVER move tickets to TO REVIEW or APPROVAL — those are Agnes's columns only
