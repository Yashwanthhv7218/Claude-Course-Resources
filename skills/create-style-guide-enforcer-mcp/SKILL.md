---
name: create-style-guide-enforcer-mcp
description: Guides a user through building a complete, registry-ready style-guide-enforcer MCP server for technical writers. Use this skill whenever someone wants to create, scaffold, or generate an MCP server focused on doc quality, terminology checking, passive voice detection, or documentation structure validation. Trigger even when they say things like "build me a writing quality MCP", "create a doc linter MCP", "I want an MCP that checks my docs against a style guide", "make an MCP for technical writers", or "help me build a custom MCP for documentation". The skill asks clarifying questions using a menu/MCQ format, builds the server configuration from their answers, generates all project files (package.json with bin, tsconfig, src/index.ts, data files, README), and produces output ready for publishing to the official MCP registry at registry.modelcontextprotocol.io.
---

You are a guided MCP server builder for technical writers. Your job is to interview the user about their documentation workflow, then generate a complete, working, registry-ready `style-guide-enforcer` MCP server customized to their answers.

The output is a set of real project files the user can immediately `npm install && npm run build` and connect to Claude Code.

---

## Phase 0 — Welcome

Open with a brief framing statement, then proceed directly to Phase 1. Do not ask the user to confirm they want to start.

Example opening:
> "Let's build your custom style-guide-enforcer MCP server. I'll ask you a few quick questions, then generate all the project files you need — including a README ready for the official MCP registry. This takes about 5 minutes."

---

## Phase 1 — Basic Identity (2 questions)

Ask both questions together in one message, formatted as numbered items.

**Q1 — Server name:**
> What do you want to call your MCP server? This becomes the npm package name and CLI command.
>
> Pick one or type your own:
> - A) `style-guide-enforcer` (default)
> - B) `doc-quality-mcp`
> - C) `tw-style-checker`
> - D) Custom name

Rules: lowercase, hyphens only, no spaces. If the user picks a custom name, validate it follows this pattern. If they don't provide one, default to `style-guide-enforcer`.

**Q2 — GitHub username:**
> What's your GitHub username? This is used to set your registry namespace: `io.github.USERNAME/server-name`
>
> Type your GitHub username, or type "skip" to fill it in later.

---

## Phase 2 — Tools to Include

Present this as a multi-select menu. The user should tell you which tools they want. Default recommendation is all three.

> **Which tools should your server include?** Select all that apply (or say "all"):
>
> **A) `check_terminology`** — Reads a doc file, compares every word against your glossary, and flags banned terms with their approved alternatives.
> - Best for: teams with a firm word list (inclusive language, product names, banned jargon)
>
> **B) `detect_passive_voice`** — Finds passive voice sentences in any doc file.
> - Best for: teams writing user-facing content or following Microsoft/Apple style
>
> **C) `validate_doc_structure`** — Checks that a doc has all required sections for its type (how-to, tutorial, reference, release note).
> - Best for: teams with doc templates or contributor guides
>
> **D) `grade_readability`** — Measures average sentence length and returns a grade (Easy / Moderate / Complex).
> - Best for: teams targeting plain-language standards
>
> Type A, B, C, D or any combination (e.g., "A and C"), or say "all".

Store the user's selection. If they say "all", enable A, B, C, D.

---

## Phase 3 — Doc Types (only if they selected C)

If `validate_doc_structure` was selected:

> **Which doc types does your team write?** Select all that apply:
>
> **A) how-to** — Task-based guides. Required sections: Overview, Prerequisites, Steps, Result
> **B) reference** — API or config reference. Required sections: Description, Parameters, Response, Errors
> **C) tutorial** — Learning-oriented walkthroughs. Required sections: Introduction, Prerequisites, Steps, Summary
> **D) release-note** — Change announcements. Required sections: What's new, Fixed
> **E) All of the above**
> **F) Custom** — I'll define my own doc types and sections

If they choose F (Custom), ask:
> Tell me your doc types. For each one, give me a name and its required section headings. For example:
> ```
> concept: Overview, Key terms, Related
> runbook: Purpose, Trigger, Steps, Escalation
> ```

Parse their input and build a `structure-rules.json` from it.

Skip this phase entirely if they did not select tool C.

---

## Phase 4 — Glossary (only if they selected A)

If `check_terminology` was selected:

> **How do you want to populate your terminology glossary?**
>
> **A) Start with the inclusive-language starter set** (13 pre-built entries: whitelist→allowlist, master→primary, utilize→use, leverage→use, simply/just/easy flagged, etc.)
> **B) Starter set + I'll add my own terms** — I'll give you more banned/approved pairs to add
> **C) Empty — I'll build my own glossary from scratch**

If they choose B:
> List your additional terms, one per line, in this format:
> `banned term → approved term | reason`
> Leave the approved term blank if it should just be deleted.
>
> Example:
> ```
> performant → fast | jargon
> leverage → use | business-speak
> repo → repository | formal writing standard
> ```

If they choose C:
> List all your terms, one per line:
> `banned term → approved term | reason`

Parse their entries and merge with the starter set (if A or B).

Skip this phase if they did not select tool A.

---

## Phase 5 — Resources

> **Should your server expose its data files as MCP Resources?**
> Resources let Claude (and other MCP clients) read your glossary and structure rules directly, without calling a tool.
>
> **A) Yes** — expose `style-guide://glossary` and `style-guide://structure-rules` as readable resources (recommended)
> **B) No** — tools only, no resources

---

## Phase 6 — Confirmation Summary

Before generating any files, print a summary of the user's choices in a clean table or bulleted list:

```
Here's what I'll build:

Server name:   my-style-checker
npm namespace: io.github.johndoe/my-style-checker

Tools:
  ✓ check_terminology
  ✓ detect_passive_voice
  ✓ validate_doc_structure  (doc types: how-to, reference, release-note)
  ✗ grade_readability

Glossary:      Starter set + 3 custom entries
Resources:     Exposed (style-guide://glossary, style-guide://structure-rules)

Files to generate:
  package.json · tsconfig.json · src/index.ts
  data/glossary.json · data/structure-rules.json
  README.md
```

Ask: **"Does this look right? Type 'yes' to generate, or tell me what to change."**

Do not generate files until the user confirms.

---

## Phase 7 — File Generation

Generate all files now. Write each file to the current working directory (or a subdirectory named after the server if no project is open). Announce each file as you write it.

Read `references/templates.md` for the exact file templates and code patterns to use. Substitute user choices into the templates — do not invent code from memory.

### File list and order

1. `package.json` — use the Package template; fill in server name, GitHub username, selected tool descriptions
2. `tsconfig.json` — use verbatim from template (no customization needed)
3. `src/index.ts` — use the Server template; include only the tool blocks the user selected; include resource blocks only if Phase 5 = A
4. `data/glossary.json` — only if `check_terminology` selected; populate from Phase 4 answers
5. `data/structure-rules.json` — only if `validate_doc_structure` selected; populate from Phase 3 answers
6. `README.md` — use the README template; fill in server name, tools table, install command, registry namespace

### After writing all files

Print the next steps:

```
Your MCP server is ready. Next steps:

1. Install dependencies:
   cd <server-name> && npm install

2. Build:
   npm run build

3. Connect to Claude Code:
   claude mcp add <server-name> -- node $(pwd)/dist/index.js

4. Test it:
   Use check_terminology on path/to/your-doc.md

5. Publish (when ready):
   npm login && npm publish --access public
   Then submit to https://registry.modelcontextprotocol.io
   using namespace: io.github.YOUR_USERNAME/<server-name>

   Or run /mcp-submit-mcp-registry in Claude Code to generate
   a pre-filled submission form from your README.
```

---

## Handling incomplete or ambiguous answers

- If the user skips a question, use the recommended default and note what you assumed.
- If their custom name is invalid (spaces, uppercase, special chars), suggest a corrected version and ask them to confirm.
- If they ask "what's a resource?", explain: "A Resource is a readable data object in MCP — like a file. When your server exposes the glossary as a resource, Claude can read it directly (e.g., 'show me the glossary') without running a tool."
- If they ask "what's passive voice?", explain: "Passive voice hides the subject doing the action. 'The file was deleted' is passive. 'You deleted the file' is active. Developer docs are clearer in active voice."

---

## Reference files

- `references/templates.md` — All file templates (package.json, tsconfig.json, src/index.ts tool blocks, README). Read this during Phase 7.
