# Assignment 5 — AI-Assisted Sprint Health Report via Jira MCP

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will connect Claude Code to your Jira board through an MCP server, the same way you connected it to GitHub in Week 2, and build a read-only `/sprint-health` skill. The skill reads your current sprint through Jira's API and reports sprint velocity, stories at risk of missing the sprint, and items missing an estimate — but it must never create, edit, comment on, or transition a single ticket itself. You will prove that boundary holds by making a real change on the board yourself and confirming the skill only ever reports, never acts.

---

# Task 1 — Create a Jira API Token

## Goal

Generate an API token from your Atlassian account that the MCP server will use to authenticate with your Jira site. Do not screenshot the token value itself.

### Evidence

#### Screenshot 1 — Jira API token creation confirmation page showing the token name, with the token value not visible

![image](screenshots/Week5-Assignment05-Screenshot01.jpg)

### Notes You Must Write (Very Important):

Why does the MCP server need your site URL and account email in addition to the token?

The MCP server may need all three because they serve different purposes:

Site URL – tells the MCP server which site/API instance to connect to. This is the destination for requests.
Account email – identifies which user/account is making the requests. It can also help the server determine account-level permissions or context.
Token – provides the authentication credential proving that the request is authorized.

Think of it like accessing an office:

Site URL = Office address
Email = Your identity
Token = Your access badge/password

The token alone may authenticate you, but the MCP server can still need the URL to know where to send the request and the email to know which account/context the token is associated with.

---

# Task 2 — Create .mcp.json at the Project Root

## Goal

Create or update `.mcp.json` at your project root with a Jira MCP server block, following the same shape as the GitHub MCP server you configured in Week 2.

### Evidence

#### Screenshot 2 — `.mcp.json` open in VS Code showing the Jira server configuration

![image](screenshots/Week5-Assignment05-Screenshot02.jpg)

### Notes You Must Write (Very Important):

Compare this jira block to the github block from Week 2 Assignment 5. The GitHub server ran via npx (a Node.js package); this one runs via uvx (a Python package) — what stays exactly the same shape despite that difference, and why doesn't Claude Code care which language a given MCP server is written in?

The overall configuration shape remains the same because Claude Code only needs to know how to start the server, what arguments to pass, and what environment variables it needs. It does not need to know or care whether the underlying MCP server is written in JavaScript or Python.

---

# Task 3 — Add Your Credentials to settings.local.json

## Goal

Add your Jira site URL, account email, and API token to `.claude/settings.local.json`, and confirm that file is listed in `.gitignore` so it is never committed.

### Evidence

#### Screenshot 3 — `settings.local.json` open in VS Code showing the `env` section, with the actual token value blurred or covered

![image](screenshots/Week5-Assignment05-Screenshot03.jpg)

### Notes You Must Write (Very Important):

Why must JIRA_API_TOKEN live in settings.local.json and never in .mcp.json?

Because JIRA_API_TOKEN is a secret credential and should be kept in the local, uncommitted settings file, not in the project-level MCP configuration.

.mcp.json → contains the MCP server configuration that can safely be shared with the project/team. It should describe how to connect to Jira, not contain personal credentials.
settings.local.json → is intended for machine/user-specific configuration, so your Jira token can remain local and private.

---

# Task 4 — Verify the Connection with /mcp

## Goal

Restart Claude Code and confirm the Jira MCP server shows as connected.

### Evidence

#### Screenshot 4 — `/mcp` output showing `jira: connected`

![image](screenshots/Week5-Assignment05-Screenshot04.jpg)

---

# Task 5 — Run a Live Query to Prove Real Board Data

## Goal

Ask Claude to list the issues in your current active sprint through the Jira MCP connection, and confirm the result matches what you see on your live board in the browser.

### Evidence

#### Screenshot 5 — Claude's response showing the live sprint issue list retrieved via Jira MCP

![image](screenshots/Week5-Assignment05-Screenshot05.jpg)

### Notes You Must Write (Very Important):

How did you confirm this was real board data and not something Claude guessed?

I knew the board is real because it brought out the Sprint Details

Sprint Details

- Sprint Name: DMIWDR Sprint 1
- Status: Active
- Start Date: 2026-08-11
- End Date: 2026-08-14
- Sprint Goal: Ship a visible Pravin Mishra Portfolio footer (version + deploy date + author) to EC2 and document progress daily in Jira

---

# Task 6 — Build the /sprint-health Skill

## Goal

Create a `/sprint-health` skill restricted to read-only Jira tools plus `Read`, with no issue-mutating tools and no `Write`. Run it and confirm it produces a report covering sprint velocity, at-risk stories, and items missing an estimate.

### Evidence

#### Screenshot 6 — `SKILL.md` frontmatter showing `allowed-tools` limited to read-only Jira tools plus `Read`, with `disable-model-invocation: true`

![image](screenshots/Week5-Assignment05-Screenshot06.jpg)

#### Screenshot 7 — `/sprint-health` output showing the full triage report against your real sprint

![image](screenshots/Week5-Assignment05-Screenshot07.jpg)

### Notes You Must Write (Very Important):

1. Which Jira MCP tools does this skill's allowed-tools list include, and which mutating tools (create issue, update issue, transition issue, add comment) does it deliberately exclude?

allowed-tools: mcp__jira__jira_search, mcp__jira__jira_get_issue, mcp__jira__jira_get_sprint, mcp__jira__jira_get_board, Read

disable-model-invocation: true


2. Why does a Scrum Master need this restriction more than almost any other role in this course?

This restriction is required to prevent the agent from making changes to the boards as this will mislead the entire team and affect the project

---

# Task 7 — Prove the Skill Never Mutates the Board

## Goal

Manually update one ticket on your board in the browser (for example, move a story to "Done" or add a missing estimate), then run `/sprint-health` again and confirm the new report reflects your change — proving the skill only ever reads live state and never wrote to the board itself.

### Evidence

#### Screenshot 8 — Second `/sprint-health` run showing the report now reflects your manual board change

![image](screenshots/Week5-Assignment05-Screenshot08.jpg)

### Notes You Must Write (Very Important):

Map this assignment to Gather → Analyze → Human Act → Verify from Week 3 Assignment 6. Which step did you perform manually in the browser, and why must that step stay human?

Gather: The /sprint-health skill used the Jira MCP read-only tools to collect live sprint information such as issue status, assignees, story points, and last-updated dates.

Analyze: Claude analyzed the retrieved data to calculate sprint velocity, identify at-risk stories, and find issues missing estimates or acceptance criteria.

Human Act: This was the manual browser step. I opened Jira in the browser and manually changed an issue—for example, moving a story, assigning an issue, adding story points, or adding acceptance criteria.

Verify: I returned to Claude Code and ran /sprint-health again. The skill retrieved the updated Jira data and showed that the change was reflected in its report.


The /sprint-health skill was deliberately designed as read-only. Its permitted Jira tools can retrieve and analyze information but cannot create, edit, assign, comment on, or transition Jira issues. The step performed in the browser was to update and change the status of work items, this step must stay human to ensure that Claude is not making unecessary changes to the project board
---

# Submission Instructions

Complete all tasks in sequence.

Your submission must include:
- All 8 required screenshots
- All the required notes

---

# Completion Checklist

- [ ] Task 1: Jira API token created, value never screenshotted (Screenshot 1)
- [ ] Task 2: `.mcp.json` has the Jira server block (Screenshot 2)
- [ ] Task 3: Credentials stored in `settings.local.json`, token blurred, file gitignored (Screenshot 3)
- [ ] Task 4: `/mcp` shows the Jira server connected (Screenshot 4)
- [ ] Task 5: Live query returned real sprint data, verified against the browser (Screenshot 5)
- [ ] Task 6: `/sprint-health` skill created with correct read-only `allowed-tools`, and produced a full report (Screenshots 6–7)
- [ ] Task 7: A manual board change was reflected in a second `/sprint-health` run (Screenshot 8)
- [ ] Skill never created, edited, transitioned, or commented on any issue
- [ ] Reflection answered (Notes)
- [ ] No API token value exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
