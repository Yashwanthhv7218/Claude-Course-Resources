# Module 7: Capstone Build — Your Portfolio Site From a Résumé
## 🟠 Advanced Level (Upper)

**Duration:** 2 hours | **Prerequisite:** Modules 1–6

---

## What This Level Means

**Upper Advanced** means you stop practicing on sample files and build something that is *yours* and public. In this module you turn your résumé into a live, professional **portfolio website** — using every primitive from this program: a **skill** to structure your résumé as data, an **agent** to generate the site, a **resource** for the schema, and Git + GitHub Pages to ship it. You'll also wire in **Sign in with LinkedIn**.

By the end, you will have a real URL you can put on your résumé — like [amantalwar04.github.io/portfolio](https://amantalwar04.github.io/portfolio/).

### Advanced (Upper) Learning Objectives

By the end of Module 7, you will be able to:
- **Explain the complete mental model** — when to reach for a skill, an agent, a plugin, or an MCP server, and how they compose
- Turn a résumé (PDF/DOCX/text) into structured **`resume.json`** using the JSON Resume schema
- Generate a **static portfolio site** (HTML/CSS/JS) that renders from `resume.json`
- Build a **`portfolio-builder` agent** that regenerates the whole site from your data
- Add **Sign in with LinkedIn** and import your work experience from a LinkedIn data export
- **Deploy** to GitHub Pages and get a public URL
- **Tune the model** with `CLAUDE.md`, Cursor rules, and context/cost optimizations

---

## The Four Building Blocks — A Complete Mental Model (20 min)

Across this program you've met four things that all "extend the AI." By now they can blur together. Before you build, lock down what each one *is*, when to reach for it, and how they **compose** into one system — because your portfolio uses all four.

### The one-line definitions

| Block | What it is | Who invokes it | Lives in |
|-------|-----------|----------------|----------|
| **Skill** (slash command) | A reusable **prompt** — instructions the model follows. No new capabilities, just a repeatable *way of thinking* | You (`/name`) or the model auto-triggers by description | `.claude/commands/*.md` or `skills/*/SKILL.md` |
| **Agent** (subagent) | An **isolated worker** with its own context window, tool permissions, and model — for a delegated multi-step job | The model delegates to it, or you name it | `.claude/agents/*.md` |
| **MCP server** | A **program or URL** that gives the AI genuinely new *capabilities* — tools (actions), resources (data), prompts | The model calls its tools; you connect it | External process / remote URL (`.mcp.json`) |
| **Plugin** | A **distributable bundle** of the above — commands + skills + agents + hooks + MCP servers, shipped as one unit | You install it from a marketplace | A repo listed in a marketplace |

### The mental model in one picture

```
        ┌──────────────────── PLUGIN (the shipping container) ────────────────────┐
        │                                                                          │
        │   SKILLS            AGENTS              HOOKS            MCP SERVERS      │
        │   (how to think)    (who does the      (when things     (what's newly    │
        │                      delegated work)    auto-fire)        possible)       │
        │        │                  │                                  │           │
        │        ▼                  ▼                                  ▼           │
        │   a repeatable      an isolated       run on save /     tools (actions)  │
        │   prompt            context + model   commit / etc.     resources (data) │
        │                                                         prompts          │
        └──────────────────────────────────────────────────────────────────────────┘
                     ▲                                              ▲
         CLAUDE.md / Cursor rules                        Model settings & caching
         (always-on behavior & context)                 (which model, how cheap)
```

**Read it this way:** a *skill* changes **how** the model thinks. An *MCP server* changes **what** the model can do. An *agent* is **who** you hand a big job to. A *plugin* is **how you ship** it all. And `CLAUDE.md` / Cursor rules sit underneath as **always-on** context.

### The decision that trips everyone up

| Ask yourself… | Reach for a… | Because… |
|---------------|-------------|----------|
| "I keep typing the same complex instructions" | **Skill** | It's a prompt you want to reuse — no new capability needed |
| "This is a big, multi-step job I want to hand off" | **Agent** | It needs its own context so it doesn't clutter yours |
| "The AI *literally cannot do this* — read a file format, call an API" | **MCP server** | Only real code gives new capabilities |
| "My whole team should get all of this with one install" | **Plugin** | Bundling + a marketplace is the distribution mechanism |

> **The single most useful heuristic:** *If you can accomplish it by writing better instructions, it's a skill. If it requires touching a system outside the chat, it's an MCP server. If it's a large delegated job, it's an agent. If you need to distribute it, it's a plugin.*

### How they compose — your portfolio is proof

The site you build today is all four ideas working together:

- A **skill** (`/parse-resume`) turns your résumé into structured `resume.json`
- A **resource** (the JSON Resume schema) is the contract the skill fills
- An **agent** (`portfolio-builder`) regenerates the entire static site from `resume.json` as one delegated job
- A **plugin** could bundle the skill + agent + schema so a teammate builds *their* portfolio with one install

Deterministic vs. generative: your **`resume.json` is the ground truth** (it never changes on its own); the **skill/agent generate** the site around it. Keep facts in data; keep judgment in prompts.

---

## What You're Building (10 min)

A single-page, static portfolio site — no build step, no framework — that reads one data file and renders it. This is the same shape as the reference site [amantalwar04.github.io/portfolio](https://amantalwar04.github.io/portfolio/): a profile header, summary, career timeline, education, skills, and contact.

```
portfolio/
  index.html          ← the page shell + section containers
  styles.css          ← all styling (one file)
  app.js              ← fetches resume.json and renders every section
  resume.json         ← YOUR data — the single source of truth (JSON Resume schema)
  profile.jpg         ← your photo
  .claude/
    commands/
      parse-resume.md  ← skill: résumé → resume.json
    agents/
      portfolio-builder.md  ← agent: resume.json → full site
  CLAUDE.md           ← project conventions
```

**The core idea:** you never hand-edit HTML to update your portfolio. You update `resume.json` (or re-run the parse skill on a new résumé), and the site re-renders itself. **Résumé-as-data.**

> **📁 Worked example:** A complete, runnable version of everything in this module lives in [`portfolio-example/`](./portfolio-example/) — the site, the `/parse-resume` and `/import-linkedin` skills, the `portfolio-builder` agent, a `CLAUDE.md`, and the installable `about-me` MCP server. Its `resume.json` is **sample data — replace it with your own.** You'll need your real résumé to make the exercise yours.

---

## Step 1: Turn Your Résumé Into Data (25 min)

### Why a schema?

If every portfolio invents its own data shape, no tool can read it. **JSON Resume** ([jsonresume.org/schema](https://jsonresume.org/schema/)) is an open standard — a shared contract for what a résumé's fields are called. Using it means your data is portable and any JSON Resume theme could render it too.

Here's the subset you'll use (`resume.json`):

```json
{
  "basics": {
    "name": "Your Name",
    "label": "Technical Communication Leader",
    "image": "profile.jpg",
    "email": "you@example.com",
    "phone": "+91 90000 00000",
    "summary": "20+ years aligning technical strategy with business goals...",
    "location": { "city": "Hyderabad", "region": "India" },
    "profiles": [
      { "network": "LinkedIn", "url": "https://www.linkedin.com/in/your-handle" }
    ]
  },
  "work": [
    {
      "name": "Autodesk",
      "position": "Principal Technical Writer",
      "startDate": "2018-04",
      "endDate": "",
      "summary": "Lead documentation strategy for platform services.",
      "highlights": [
        "Cut release-doc turnaround from 3 days to 4 hours",
        "Built the team's first AI-assisted review workflow"
      ]
    }
  ],
  "education": [
    { "institution": "Kakatiya University", "studyType": "MCA", "area": "Computer Applications", "startDate": "2000", "endDate": "2003" }
  ],
  "skills": [
    { "name": "Technical Writing", "keywords": ["DITA", "Markdown", "API docs"] },
    { "name": "AI Tooling", "keywords": ["Claude Code", "MCP", "Prompt design"] }
  ],
  "certificates": [
    { "name": "AI Documentation Practitioner", "issuer": "Tech Writer's Tribe", "date": "2026-07" }
  ]
}
```

### The `/parse-resume` skill

You won't type this JSON by hand. Create a skill that reads your existing résumé and produces it. Create `.claude/commands/parse-resume.md`:

```markdown
You convert a résumé into a valid JSON Resume file.

## Input
The résumé to convert is at: $ARGUMENTS
(It may be .pdf, .docx, .md, or .txt.)

## Task
1. Read the résumé.
2. Extract every fact into the JSON Resume schema (jsonresume.org/schema):
   basics, work, education, skills, certificates, projects.
3. For each work entry, write 2–4 achievement-focused `highlights`
   (start with a verb, include a number where the résumé gives one).
4. Use "YYYY-MM" for dates. Leave `endDate` empty for the current role.
5. Do NOT invent facts. If a field is missing, omit it — never guess a date or employer.

## Output Contract
Output ONLY the JSON object — no preamble, no markdown fences, no commentary.
It must be valid JSON and parse on the first try.
```

Run it:

```
/parse-resume ~/Documents/my-resume.pdf > resume.json
```

Open `resume.json` and check it. This is the one file you'll maintain from now on.

> **🟠 Advanced Insight:** Notice the **output contract** — "output ONLY the JSON." A skill whose output another program reads (here, your website) must be strict. This is the same hardening you'll apply to every production skill.

---

## Step 2: Generate the Static Site (25 min)

Now the page that renders `resume.json`. Three files, no framework.

### `index.html` — the shell

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>My Portfolio</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <header id="hero"></header>
  <main>
    <section id="summary"></section>
    <section id="experience"><h2>Career Timeline</h2><div id="work-list"></div></section>
    <section id="education"><h2>Education</h2><div id="edu-list"></div></section>
    <section id="skills"><h2>Skills</h2><div id="skill-list"></div></section>
    <section id="contact"><h2>Get In Touch</h2><a id="contact-btn" class="btn">Email me</a></section>
  </main>
  <script src="app.js"></script>
</body>
</html>
```

### `app.js` — fetch data, render sections

```javascript
async function loadPortfolio() {
  const res = await fetch("resume.json");
  const data = await res.json();
  const b = data.basics || {};

  // Hero
  document.getElementById("hero").innerHTML = `
    <img src="${b.image || ""}" alt="${b.name}" class="avatar" />
    <h1>${b.name}</h1>
    <p class="label">${b.label || ""}</p>
    <p class="location">${b.location?.city || ""}, ${b.location?.region || ""}</p>
  `;

  // Summary
  document.getElementById("summary").innerHTML = `<p>${b.summary || ""}</p>`;

  // Experience (career timeline)
  document.getElementById("work-list").innerHTML = (data.work || []).map(w => `
    <article class="entry">
      <h3>${w.position} · ${w.name}</h3>
      <span class="dates">${w.startDate} – ${w.endDate || "Present"}</span>
      <p>${w.summary || ""}</p>
      <ul>${(w.highlights || []).map(h => `<li>${h}</li>`).join("")}</ul>
    </article>
  `).join("");

  // Education
  document.getElementById("edu-list").innerHTML = (data.education || []).map(e => `
    <article class="entry">
      <h3>${e.studyType}${e.area ? " · " + e.area : ""}</h3>
      <span class="dates">${e.startDate} – ${e.endDate || ""}</span>
      <p>${e.institution}</p>
    </article>
  `).join("");

  // Skills
  document.getElementById("skill-list").innerHTML = (data.skills || []).map(s => `
    <div class="skill"><strong>${s.name}</strong>: ${(s.keywords || []).join(", ")}</div>
  `).join("");

  // Contact
  const btn = document.getElementById("contact-btn");
  if (b.email) btn.href = `mailto:${b.email}`;
}
loadPortfolio();
```

### `styles.css` — starter (customize freely)

```css
:root { --accent: #0a66c2; --ink: #1a1a1a; --muted: #666; }
* { box-sizing: border-box; margin: 0; }
body { font-family: system-ui, sans-serif; color: var(--ink); line-height: 1.6; }
#hero { text-align: center; padding: 3rem 1rem; background: #f5f7fa; }
.avatar { width: 120px; height: 120px; border-radius: 50%; object-fit: cover; }
main { max-width: 760px; margin: 0 auto; padding: 2rem 1rem; }
section { margin-bottom: 3rem; }
h2 { border-bottom: 2px solid var(--accent); padding-bottom: .3rem; }
.entry { padding: 1rem 0; border-bottom: 1px solid #eee; }
.dates { color: var(--muted); font-size: .9rem; }
.btn { display: inline-block; background: var(--accent); color: #fff; padding: .6rem 1.2rem; border-radius: 6px; text-decoration: none; }
```

### Preview locally

A static site needs to be *served* (not opened as a `file://`) so `fetch` works:

```bash
cd portfolio
python3 -m http.server 8000
# open http://localhost:8000
```

Change one line in `resume.json`, reload — the page updates. You never touched HTML.

---

## Step 3: Build the `portfolio-builder` Agent (15 min)

You've built a skill (`/parse-resume`) and a hand-written renderer. The **agent** ties them together and does the multi-step job for you: read the résumé, produce `resume.json`, and (re)generate the site files — as one delegated task with its own context.

Create `.claude/agents/portfolio-builder.md`:

```markdown
---
name: portfolio-builder
description: >
  Builds or rebuilds a static portfolio website from a résumé or from resume.json.
  Use when someone wants to create, regenerate, or restyle their portfolio site.
tools: Read, Write, Glob
model: sonnet
---

You build a static, framework-free portfolio site (HTML/CSS/JS) driven by resume.json.

## Steps
1. If resume.json is missing, read the résumé the user names and create it using
   the JSON Resume schema. Never invent facts.
2. Ensure index.html, styles.css, and app.js exist and render every section present
   in resume.json (basics, work, education, skills, certificates, projects).
3. If a section has no data, hide it — never show an empty heading.
4. Keep it accessible: alt text on images, semantic headings, sufficient contrast.

## Return
A short summary: which files you wrote, which résumé sections were rendered,
and the exact command to preview locally. Do not paste full file contents back.
```

Run it:

```
Use the portfolio-builder agent to rebuild my site from resume.json
```

**Why an agent here?** Generating four files and reconciling them against your data is a multi-step job with a lot of intermediate reading. Delegating it keeps that churn out of your main conversation — you get back a clean summary, not 300 lines of generated code.

| Field | Why it matters |
|-------|----------------|
| `description` | *When* the model auto-delegates — phrase it as a trigger |
| `tools` | `Read, Write, Glob` — it needs to write files, but not run shell or delete |
| `model` | `sonnet` — site generation needs real reasoning; Haiku would be too shallow |

---

## Step 4: Connect LinkedIn (20 min)

Your portfolio should show two things from LinkedIn: **who you are** (live) and **your work experience** (imported). These come from two different mechanisms — and it's important to know why.

### Part A — "Sign in with LinkedIn" (live profile basics)

LinkedIn's official login uses **OpenID Connect (OIDC)**. It's the "Sign in with LinkedIn" button. It returns a small, fixed set of claims:

```
sub · name · given_name · family_name · picture · email · email_verified · locale
```

**That is the entire list.** OIDC sign-in does **not** return your positions, job history, skills, or education — LinkedIn only exposes those through approved Partner Programs, which aren't available for a personal project.

**Set it up:**

1. Go to [linkedin.com/developers](https://www.linkedin.com/developers/) → **Create app**.
2. On the app's **Products** tab, add **"Sign In with LinkedIn using OpenID Connect."**
3. Under **Auth**, add your **redirect URL** (e.g. `https://YOUR-USERNAME.github.io/portfolio/`) and copy your **Client ID**.

**The OAuth endpoints:**

| Step | Endpoint |
|------|----------|
| Authorize | `https://www.linkedin.com/oauth/v2/authorization` |
| Token exchange | `https://www.linkedin.com/oauth/v2/accessToken` |
| User info | `https://api.linkedin.com/v2/userinfo` |

Request scopes `openid profile email`.

> **⚠️ Static-site reality check:** The token exchange needs your **client secret**, which you must **never** put in `app.js` on a public site. A static GitHub Pages site can't keep a secret. So the token step must run on a tiny backend — a **serverless function** (Cloudflare Workers, Netlify, or Vercel — all have free tiers). The flow: your button → LinkedIn → back to your site with a `code` → your site calls *your* serverless function → the function exchanges the code (with the secret) and returns the profile. Treat the sign-in button as a "personalize the greeting" feature, not the source of your résumé data.

### Part B — Import your work experience (the data export)

Because OIDC can't give job history, you get it the official, ToS-safe way: **export your own data.**

1. On LinkedIn: **Me → Settings & Privacy → Data Privacy → Get a copy of your data.**
2. Select **Positions**, **Profile**, **Education**, and **Skills**. Request the archive.
3. LinkedIn emails you a `.zip` (usually within minutes to a day). It contains `Positions.csv`, `Profile.csv`, `Education.csv`, `Skills.csv`.

Then convert those CSVs to your schema with a small skill — create `.claude/commands/import-linkedin.md`:

```markdown
You merge a LinkedIn data export into resume.json.

## Input
A folder of LinkedIn CSV exports at: $ARGUMENTS
(Positions.csv, Profile.csv, Education.csv, Skills.csv)

## Task
1. Read the CSVs.
2. Map Positions.csv → work[] (Company Name→name, Title→position,
   Started On→startDate, Finished On→endDate, Description→summary).
3. Map Education.csv → education[] and Skills.csv → skills[].
4. Merge into the existing resume.json WITHOUT overwriting hand-written
   highlights or summary. Add new roles; update dates on existing ones.
5. Convert all dates to "YYYY-MM".

## Output Contract
Output ONLY the merged, valid JSON Resume object. No commentary.
```

Run it:

```
/import-linkedin ~/Downloads/linkedin-export/ > resume.json
```

**The honest summary:** Sign in with LinkedIn personalizes the page live; the data export is how your actual experience gets in. Re-running the export + import skill whenever you update LinkedIn keeps the two in sync — a manual sync, but a reliable and policy-safe one.

---

## Step 5: Create a GitHub Repo and Deploy (15 min)

Your site is on your laptop. To get a public URL, it needs to live in a **GitHub repository** with **GitHub Pages** turned on. There are two ways to create the repo — pick one.

### Prerequisites

- A free **GitHub account** ([github.com/signup](https://github.com/signup))
- **Git** installed (`git --version` should print a version)
- Your `portfolio/` folder with `index.html`, `styles.css`, `app.js`, `resume.json`

First, turn your folder into a Git repository:

```bash
cd portfolio
git init
git add .
git commit -m "Portfolio site from resume.json"
```

### Option A — Create the repo on GitHub.com (recommended, no extra tools)

1. Go to [github.com/new](https://github.com/new) (or click the **+** in the top-right → **New repository**).
2. **Repository name:** `portfolio`.
   - For a personal site at the *root* URL, name it `YOUR-USERNAME.github.io` instead.
3. **Visibility:** **Public** — GitHub Pages needs public on the free plan.
4. **Do NOT** check "Add a README", ".gitignore", or "license" — your folder already has files, and an empty repo avoids a merge conflict.
5. Click **Create repository.**
6. GitHub now shows a "…or push an existing repository" box. Copy the commands it gives you — they look like this (replace `YOUR-USERNAME`):

```bash
git remote add origin https://github.com/YOUR-USERNAME/portfolio.git
git branch -M main
git push -u origin main
```

> **First push authentication:** GitHub will ask you to sign in. Use a **Personal Access Token** as the password, not your account password — create one at [github.com/settings/tokens](https://github.com/settings/tokens) (Tokens (classic) → Generate new token → check the **`repo`** scope). Or install [GitHub Desktop](https://desktop.github.com/) / the `gh` CLI (Option B), which handle login for you.

### Option B — Create the repo from the terminal (GitHub CLI)

If you have the [GitHub CLI](https://cli.github.com/) installed and authenticated (`gh auth login`), it creates the repo and pushes in one command:

```bash
gh repo create portfolio --public --source=. --push
```

### Turn on GitHub Pages

However you created the repo:

1. On GitHub, open the repo → **Settings → Pages** (left sidebar).
2. **Source:** *Deploy from a branch.*
3. **Branch:** `main` · Folder: `/ (root)` → **Save.**
4. Wait ~1 minute, then refresh. Pages shows your live URL:
   `https://YOUR-USERNAME.github.io/portfolio/`

> **Troubleshooting:** a blank page usually means `resume.json` didn't load — open the browser console (F12). A **404** right after enabling Pages just means it hasn't finished the first build; wait a minute and refresh.

### Update it later

Edit `resume.json` (or re-run `/import-linkedin`), then push:

```bash
git add resume.json && git commit -m "Update experience" && git push
```

Pages redeploys automatically within a minute. **Your résumé is now a living, versioned website.**

---

## Bonus: An `about-me` MCP Server Others Can Install (15 min)

Your site is something people *look at*. An MCP server is something their *AI can query*. Ship both and your résumé becomes queryable: a recruiter who adds your server can ask their own Claude *"What's this person's experience with API documentation?"* and get a grounded, factual answer — straight from your `resume.json`.

The full server is in [`portfolio-example/about-me-mcp/`](./portfolio-example/about-me-mcp/). It reuses the exact MCP patterns from Module 6 — tools, a resource, and a prompt — but its data is *you*:

| It exposes | What a visitor's AI gets |
|------------|--------------------------|
| Tool `get_summary` | Your title, location, and professional summary |
| Tool `get_experience` (optional `keyword`) | Your work history, filterable ("API", "leadership") |
| Tool `get_skills` / `get_education` / `get_contact` | The rest of your background |
| Resource `about-me://resume` | Your full JSON Resume document |
| Prompt `introduce_me` | A factual intro drafted for a given audience |

The key design choice: the server reads a **bundled `data/resume.json`** by default (with a `RESUME_PATH` override), so when someone installs it from npm it serves *your* data with no setup.

**Build and connect it:**

```bash
cd portfolio-example/about-me-mcp
npm install && npm run build
claude mcp add about-me -- node "$(pwd)/dist/index.js"
```

Then in Claude Code:

```
/mcp                                          # confirm about-me is connected
What is this person's experience with APIs?   # calls get_experience
```

**Publish it** (Module 6 method) so anyone can install with one line:

```bash
npm publish --access public
# others then run:
claude mcp add about-me -- npx -y about-me-aman
```

> **Portfolio power move:** bundle the `/parse-resume` skill + `portfolio-builder` agent into a **plugin**, and publish the `about-me` server to the registry. Now you can hand a colleague *one install* that builds their portfolio — and your own résumé is discoverable by any MCP-capable AI. That's the whole program in one artifact.

---

## Tuning the Model: CLAUDE.md, Cursor Rules & Context Optimization (15 min)

These are the optimizations that make Claude behave consistently and cheaply on *any* project — including this one.

### 1. `CLAUDE.md` — the always-on instruction file

Claude Code **loads `CLAUDE.md` automatically into every session.** Put it in your portfolio repo so Claude always knows the rules:

```markdown
## This project
- Static portfolio site. Single source of truth is resume.json (JSON Resume schema).
- Never hand-edit index.html to change content — update resume.json and re-render.
- Never invent résumé facts. If data is missing, omit the field.

## Style
- Keep the site framework-free: plain HTML/CSS/JS, no build step.
- Accessible by default: alt text, semantic headings, WCAG-AA contrast.

## How to work with me
- No preambles. Show the diff, skip the summary.
```

Three levels stack (most specific wins): `~/.claude/CLAUDE.md` (global) → `./CLAUDE.md` (this repo) → `./subdir/CLAUDE.md` (one folder).

Build one with `/init` (Claude scans the repo and drafts it), then trim — every line is loaded into *every* prompt, so it costs tokens forever. Keep stable rules at the top for better caching.

### 2. Cursor rules — the same idea, in Cursor

If your team uses **Cursor**, the equivalent is `.cursor/rules/*.mdc` (the legacy `.cursorrules` still works but is deprecated). The difference: Cursor rules scope by **file glob**, `CLAUDE.md` by folder.

```markdown
---
description: Portfolio site conventions
globs: ["**/*.js", "**/*.html"]
alwaysApply: false
---
- resume.json is the source of truth; never hardcode résumé content in markup.
- Keep it framework-free and accessible.
```

Point both `CLAUDE.md` and a Cursor rule at the same conventions so standards match no matter which tool you open.

### 3. Model selection & cost habits

- **Model:** `/model` to switch — Sonnet for site generation, Haiku for cheap bulk passes, Opus for the hardest reasoning. Pin a model **per agent** (your `portfolio-builder` uses Sonnet).
- **Read once, reference thereafter** — re-reading files is the #1 token waster.
- **Batch independent actions** into one message.
- **Prompt caching** rewards stable-content-first — that's why `CLAUDE.md` rules stay put.
- **`/clear`** between unrelated tasks; **`/compact`** to summarize a long session.

---

## Production Hardening (10 min)

Before you call the site done:

1. **Validate the data.** A broken `resume.json` breaks the whole page. Add a guard in `app.js`:
   ```javascript
   try { data = await (await fetch("resume.json")).json(); }
   catch { document.body.innerHTML = "<p>Could not load resume.json — check it's valid JSON.</p>"; return; }
   ```
2. **Hide empty sections.** Never render a "Certifications" heading with nothing under it.
3. **Output contracts on your skills.** `/parse-resume` and `/import-linkedin` must output *only* JSON — already done above. That's what lets you pipe them to a file.
4. **Version your data.** `resume.json` is in Git — every edit is a diff you can revert. That *is* your version history.
5. **Don't commit secrets.** Your LinkedIn client secret lives in the serverless function's environment, never in the repo. Add a `.gitignore` for any local `.env`.

---

## Discussion and Wrap-Up (5 min)

### Reflection Questions

1. Which primitive did the most work in your build — the skill, the agent, or plain code? Why?
2. What's the trade-off you accepted by using OIDC sign-in *plus* a manual data export instead of a live API sync?
3. What would you bundle into a plugin so a colleague could build their portfolio in one step?

---

## Homework (Before Module 8)

1. **Ship your portfolio** to GitHub Pages and confirm the public URL loads on your phone.
2. **Populate `resume.json`** with your real experience (via `/parse-resume` and/or `/import-linkedin`).
3. **Write a `CLAUDE.md`** for the repo (run `/init`, then trim it).
4. **Prepare a 5-minute demo** for Module 8: show the site, then change `resume.json` live and watch it re-render.
5. **Read** Module 8 — prepare to present.

---

## 🟠 Advanced Checklist

Before moving to Module 8, confirm you can:

- [ ] Explain when to use a skill vs. an agent vs. an MCP server vs. a plugin, and how they compose
- [ ] Convert a résumé into valid `resume.json` (JSON Resume schema) with `/parse-resume`
- [ ] Render a static site from `resume.json` with no framework and no build step
- [ ] Build a `portfolio-builder` agent (`name`/`description`/`tools`/`model`) and run it
- [ ] Explain why Sign in with LinkedIn returns only basic profile fields, not job history
- [ ] Import work experience from a LinkedIn data export into `resume.json`
- [ ] Deploy to GitHub Pages and produce a public URL
- [ ] Build and connect the `about-me` MCP server so another AI can query your background
- [ ] Write a `CLAUDE.md`; explain Cursor rules' glob scoping
- [ ] Name three context/cost optimizations and why caching favors stable-content-first
