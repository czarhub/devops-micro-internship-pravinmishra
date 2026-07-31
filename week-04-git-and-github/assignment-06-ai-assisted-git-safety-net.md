# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch

![image](screenshots/Week04-Assignment06-Screenshot00.jpg)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

Creating a dedicated branch instead of working directly on main is a Git best practice because it protects the stable codebase and makes collaboration easier.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![image](screenshots/Week04-Assignment06-Screenshot01.jpg)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

This is used to simulate a common security issues that the pre-commit hook and Claude Code skill are expected to detect before the code is committed, demonstrating how automated checks can help prevent sensitive information and debugging code from being pushed to a repository.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

![image](screenshots/Week04-Assignment06-Screenshot02.jpg)

---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![image](screenshots/Week04-Assignment06-Screenshot03.jpg)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

This is because .git/hooks/ is never committed to Git, it's local only and invisible to everyone else on the team.

By tracking hooks/pre-commit in the repo, every developer who clones the project gets the same hook.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

Hook/pre-commit is used to secure the code based prevent sensitive information like Secret and keys that are exposed from getting to the codebase while PreToolUse prevents Claude from making use of some set of tools to perform actions and create new infrastructure.

What they have in common is that they are both gate keepers. They ask the question, should this actually happen?

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file


![image](screenshots/Week04-Assignment06-Screenshot04.jpg)

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

if git diff --cached -- "$file" | grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----'it matched because the fake key started with AKIA

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No it couldn't. The hook only scans for patterns it already knows — like AKIA (AWS key prefix) or specific regex rules.

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![image](screenshots/Week04-Assignment06-Screenshot05.jpg)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![image](screenshots/Week04-Assignment06-Screenshot06.jpg)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

This is because /pr-ready is a reviewer, not an editor. Its job is to:

Read — inspect your code files
Bash — run checks like tests, linters, and git diff

It doesn't need Write because it's not supposed to change anything — just evaluate whether the code is ready for a pull request and report back.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

Pre-commit hooks blocks a commit containing secret-like patterns or files over 1MB it uses fixed rules to scan the code before a commit. For example, it looks for a specific pattern such as: AKIA[0-9A-Z]{16} If it finds a match, it blocks the commit. 

The hook does not understand the code. It simply checks whether the text matches predefined patterns, while the /pr-ready skill works differently. Instead of searching for fixed patterns, it reads and analyzes the code changes like a human reviewer. In this assignment, it identified issues that the pre-commit hook could not.

For example, it recognized that:

The debug echo statement would expose the secret by printing it to the terminal.
The changes lacked supporting documentation.
The fake AWS key could still trigger automated secret scanners, even though it was only used for testing.
Unlike the hook, it explains why something is a problem instead of simply blocking the commit.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![image](screenshots/Week04-Assignment06-Screenshot07.jpg)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![image](screenshots/Week04-Assignment06-Screenshot08.jpg)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

I removed the fake AWS Secret Keys and it's correspoding Echo message

---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![image](screenshots/Week04-Assignment06-Screenshot09.jpg)

---

#### PR Link

https://github.com/pravinmishraaws/devops-micro-internship-interviews/pull/447

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

I edited it to be more meaniful and simple to understnd

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

This will confuse the reviewer and they might not understand what exactly the are reviewing

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

This is because I don't have write access to the upstream repository.If I targeted the upstream directly, GitHub would reject it — only maintainers can merge into the original repo.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

Claude reads the staged diff using Bash and Read tools — collecting all the changes, file names, and code context without modifying anything.

---

**2. Which step(s) represent Analyze?**

Claude intelligently reviews what it gathered — checking for secrets, code quality, logic gaps, naming issues, and whether the PR is genuinely ready for review

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

The Human act is making the changes by removing the exposed keys and the echo message, Claude did not run git commit, git push and open PR to avoid AI from making major changes to the infrastructure,this could be destructive

---

**4. Which step is Verify?**

Run /pr-ready again on the updated staged changes to confirm all flagged issues are resolved before the PR goes up for review.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

I need both because they are both doing different job, the pre-commit executes a bash script that prevents commits if some checks were not met while the AI skill is for reporting to inform you of what's going on before you proceed

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

https://www.linkedin.com/posts/diokparaphael_dmi-dmicohort3-devops-share-7488805850644533248-M4R4/?utm_source=share&utm_medium=member_desktop&rcm=ACoAABJFGsoB0Tj582Besj5R2uLnB6itVJv47yU

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

- Engineers/Developers are responsible for their code
- Using Hooks and AI assisted work flows helps you to identify errors or costly mistakes for pushing your code
- Anyone can make a mistake, so implementing this pre-hooks and checks save you costly mistakes
- There's no perfect response from AI, it still requires human review 


---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL

Paste your forked repository URL here:

https://github.com/czarhub/devops-micro-internship-interviews

---

# Completion Checklist

- [ ] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [ ] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [ ] `core.hooksPath` configured to point at `hooks/`
- [ ] Pre-commit hook shown blocking the risky commit
- [ ] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [ ] `/pr-ready` run against the risky diff and shown flagging issues
- [ ] Risky file fixed; `git commit` succeeds cleanly
- [ ] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [ ] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [ ] Agentic Loop mapping (Task 7) completed in your own words
- [ ] LinkedIn post published and URL submitted
- [ ] All required screenshots added
- [ ] GitHub repository URL provided

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
