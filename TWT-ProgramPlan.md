# Program Plan — AI-Powered Documentation Mastery

## Training & Certification Program

---

## Program Overview

| Detail | Value |
|--------|-------|
| **Program name** | AI-Powered Documentation Mastery |
| **Host organization** | AI Documentation Program |
| **Total modules** | 8 |
| **Duration per module** | 2 hours |
| **Total classroom time** | 16 hours |
| **Mode** | Online, live (Zoom/Teams) |
| **Cadence** | Weekly (Saturdays recommended) |
| **Pre-work** | ~2 hours (software setup + introductory reading) |
| **Practice per module** | ~1–2 hours |

---

## Skill Progression Map

This program is built on a deliberate five-tier skill ladder. Participants enter wherever they are and exit with proof of where they've grown.

```
Tier       Modules    Milestone Deliverable
────────────────────────────────────────────────────────
🟢 Novice       1–2    Understand AI and MCP conceptually; complete setup
🔵 Beginner     3      Write your first Claude Skill; explain the five prompt elements
🟡 Intermediate 4–5    Build 3+ reusable skills using recognized patterns
🟠 Advanced     6      Extend a real MCP server; scaffold with MCPBuilder; publish to the official MCP registry
🟠 Advanced     7      Ship a live portfolio site from a résumé — skill + agent + about-me MCP, deployed to GitHub Pages
🔴 Expert       8      Present & package the portfolio for reuse; mentor others; earn certificate
```

---

## Certification Requirements

To earn the **AI Documentation Practitioner** certificate, participants must:

1. **Attend** at least 6 of 8 live sessions (or watch recordings within 1 week)
2. **Submit a portfolio** containing:
   - Minimum 4 Claude Skills built during Modules 3–5
   - A **live portfolio website** deployed to GitHub Pages, generated from `resume.json` (JSON Resume schema) — with the `/parse-resume` skill and `portfolio-builder` agent used to build it
   - An **`about-me` MCP server** that answers questions about you from your résumé data
   - A Personal AI Integration Plan
   - Optional: a published MCP server URL (registry.modelcontextprotocol.io, Smithery, or PulseMCP)
3. **Pass the skills assessment** (70% minimum):
   - Written scenario: choose the right primitive (skill / agent / MCP / plugin) for a given task
   - Live demo: run `/parse-resume` + the `portfolio-builder` agent on an unfamiliar résumé
4. **Complete peer review** for at least 2 classmates' skills (Module 5)

---

## Assessment Rubric

### Skills Assessment (70% to pass)

| Component | Weight | What's Evaluated |
|-----------|--------|-----------------|
| Correctness | 30% | Do the skill, agent, and site do what they claim? |
| Clarity | 20% | Are the skill/agent files readable and well-structured? |
| Output quality | 20% | Is the generated `resume.json` accurate and the site polished? |
| Reusability | 15% | Do `/parse-resume` and `portfolio-builder` work on an unfamiliar résumé? |
| Documentation | 15% | Is the repo's purpose and usage clear (README, CLAUDE.md)? |

### Portfolio Rubric

| Item | Required | Points |
|------|----------|--------|
| `/parse-resume` skill (résumé → `resume.json`) | Yes | 10 |
| `portfolio-builder` agent | Yes | 10 |
| `resume.json` populated with your real experience (JSON Resume schema) | Yes | 10 |
| Static site renders every section from `resume.json` (no hand-edited HTML) | Yes | 15 |
| **Live site deployed to GitHub Pages (public URL)** | Yes | 20 |
| `about-me` MCP server connected and answering queries | Yes | 15 |
| `CLAUDE.md` for the portfolio repo | Yes | 10 |
| Personal AI Integration Plan | Yes | 10 |
| Published `about-me` MCP server (npm / any public registry) | No — bonus | +10 |
| Skill + agent bundled as an installable plugin | No — bonus | +5 |
| **Total (base)** | | **100** |
| **Total (with bonus)** | | **115** |

Minimum to pass: **70 points**

> **Note on the capstone project:** Modules 7–8 build a personal **portfolio website from a résumé** — a tangible, public artifact every learner can put on their own résumé. The single source of truth is `resume.json` (the open [JSON Resume](https://jsonresume.org/schema/) schema); the site, the skills, the agent, and the `about-me` MCP server all revolve around that one file. The complete worked example is in `modules/07-expert-workflows-advanced/portfolio-example/`.

---

## Pre-Program Timeline

| When | Action |
|------|--------|
| 2 weeks before Module 1 | Send welcome email, setup guide, repo link |
| 1 week before Module 1 | Optional 30-min setup help session |
| Day before each module | Send reminder + homework prompt |
| After each module | Post recording + next module reading |

---

## Facilitator Notes

### Community Standards

All sessions follow the Code of Conduct:
- Inclusive, respectful language
- Questions are always welcome — "there are no dumb questions"
- Peer feedback is constructive, specific, and kind
- No one is left behind: help each other in the community channel

### Recommended Tools for Facilitators

- **Zoom** or **Teams** for live sessions
- **VS Code** for collaborative exercises
- **Slack** or similar for async Q&A
- **GitHub** for skill sharing and portfolio submissions

### Session Rhythm

Each 2-hour session follows this pattern:

| Time | Phase |
|------|-------|
| 0:00–0:15 | Recap + homework share |
| 0:15–1:00 | New concepts (lecture + demo) |
| 1:00–1:45 | Hands-on activity |
| 1:45–1:55 | Discussion + Q&A |
| 1:55–2:00 | Homework assignment |

---

*AI-Powered Documentation Mastery | Program v1.0*
