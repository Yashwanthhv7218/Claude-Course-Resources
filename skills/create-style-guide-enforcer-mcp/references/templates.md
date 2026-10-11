# File Templates for style-guide-enforcer MCP

Use these templates verbatim during Phase 7. Substitute `{{SERVER_NAME}}`, `{{GITHUB_USERNAME}}`, and `{{TOOL_DESCRIPTIONS}}` with user-confirmed values.

---

## package.json

```json
{
  "name": "{{SERVER_NAME}}",
  "version": "1.0.0",
  "description": "{{DESCRIPTION}}",
  "bin": {
    "{{SERVER_NAME}}": "./dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.12.1"
  },
  "devDependencies": {
    "typescript": "^5.4.0",
    "@types/node": "^20.0.0"
  },
  "keywords": ["mcp", "technical-writing", "style-guide", "documentation"],
  "license": "MIT"
}
```

`{{DESCRIPTION}}` — generate a one-sentence description from the tools selected. Examples:
- All tools: "MCP server that checks documentation for terminology violations, passive voice, structure completeness, and readability."
- Terminology only: "MCP server that flags banned terms in documentation and suggests approved alternatives."
- Structure only: "MCP server that validates documentation structure against required sections by doc type."

---

## tsconfig.json

Use verbatim — no substitutions needed.

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

## src/index.ts

### File header (always include)

```typescript
#!/usr/bin/env node
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import * as fs from "fs";
import * as path from "path";

const server = new McpServer({
  name: "{{SERVER_NAME}}",
  version: "1.0.0",
});
```

### Tool block: check_terminology (include if user selected A)

```typescript
// ── Tool: check_terminology ───────────────────────────────────────────────────

server.tool(
  "check_terminology",
  "Flag banned terms in a document and suggest approved alternatives from the team glossary",
  {
    filePath: {
      type: "string",
      description: "Path to the documentation file to check",
    },
    glossaryPath: {
      type: "string",
      description: "Path to the glossary JSON file (defaults to data/glossary.json)",
    },
  },
  async ({ filePath, glossaryPath }) => {
    const resolvedGlossary = glossaryPath || path.join(__dirname, "../data/glossary.json");
    const content = fs.readFileSync(filePath, "utf-8");

    type GlossaryEntry = { banned: string; approved: string; note?: string };
    const glossary: GlossaryEntry[] = JSON.parse(fs.readFileSync(resolvedGlossary, "utf-8"));

    const violations = glossary
      .filter(({ banned }) => new RegExp(`\\b${banned}\\b`, "gi").test(content))
      .map(({ banned, approved, note }) => ({ found: banned, use_instead: approved, note }));

    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          file: filePath,
          violations_found: violations.length,
          violations,
          status: violations.length === 0 ? "PASS" : "NEEDS_REVISION"
        }, null, 2)
      }]
    };
  }
);
```

### Tool block: detect_passive_voice (include if user selected B)

```typescript
// ── Tool: detect_passive_voice ────────────────────────────────────────────────

server.tool(
  "detect_passive_voice",
  "Find passive voice sentences in a documentation file and suggest revisions",
  {
    filePath: {
      type: "string",
      description: "Path to the documentation file to check",
    },
  },
  async ({ filePath }) => {
    const content = fs.readFileSync(filePath, "utf-8");

    const plainText = content
      .replace(/```[\s\S]*?```/g, "")
      .replace(/`[^`]+`/g, "")
      .replace(/#+\s/g, "")
      .replace(/\[([^\]]+)\]\([^)]+\)/g, "$1");

    const passivePattern = /\b(is|are|was|were|has been|have been|will be|can be|should be|must be)\s+\w+ed\b/gi;
    const sentences = plainText.split(/[.!?]+/).filter(s => s.trim().length > 10);

    const passiveSentences = sentences
      .filter(s => passivePattern.test(s))
      .map(s => s.trim().replace(/\n/g, " "));

    passivePattern.lastIndex = 0;

    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          file: filePath,
          passive_sentences_found: passiveSentences.length,
          sentences: passiveSentences,
          tip: "Rewrite using active voice: name the subject doing the action.",
          status: passiveSentences.length === 0 ? "PASS" : "REVIEW_RECOMMENDED"
        }, null, 2)
      }]
    };
  }
);
```

### Tool block: validate_doc_structure (include if user selected C)

```typescript
// ── Tool: validate_doc_structure ──────────────────────────────────────────────

server.tool(
  "validate_doc_structure",
  "Check that a document has all required sections for its type ({{DOC_TYPE_LIST}})",
  {
    filePath: {
      type: "string",
      description: "Path to the documentation file to validate",
    },
    docType: {
      type: "string",
      description: "The type of document: {{DOC_TYPE_LIST}}",
    },
    rulesPath: {
      type: "string",
      description: "Path to the structure rules JSON file (defaults to data/structure-rules.json)",
    },
  },
  async ({ filePath, docType, rulesPath }) => {
    const resolvedRules = rulesPath || path.join(__dirname, "../data/structure-rules.json");
    const content = fs.readFileSync(filePath, "utf-8");

    type StructureRules = Record<string, { required: string[]; optional: string[] }>;
    const rules: StructureRules = JSON.parse(fs.readFileSync(resolvedRules, "utf-8"));

    if (!rules[docType]) {
      return {
        content: [{
          type: "text",
          text: JSON.stringify({
            error: `Unknown doc type: "${docType}". Valid types: ${Object.keys(rules).join(", ")}`
          }, null, 2)
        }]
      };
    }

    const { required, optional } = rules[docType];
    const headings = (content.match(/^#{1,3}\s+.+/gm) || []).map(h => h.replace(/^#+\s+/, "").toLowerCase());

    const missingRequired = required.filter(
      section => !headings.some(h => h.includes(section.toLowerCase()))
    );
    const presentOptional = optional.filter(
      section => headings.some(h => h.includes(section.toLowerCase()))
    );

    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          file: filePath,
          doc_type: docType,
          missing_required_sections: missingRequired,
          present_optional_sections: presentOptional,
          required_score: `${required.length - missingRequired.length}/${required.length}`,
          status: missingRequired.length === 0 ? "COMPLETE" : "INCOMPLETE"
        }, null, 2)
      }]
    };
  }
);
```

Replace `{{DOC_TYPE_LIST}}` with a pipe-separated list of the user's selected doc types, e.g. `how-to | reference | release-note`.

### Tool block: grade_readability (include if user selected D)

```typescript
// ── Tool: grade_readability ───────────────────────────────────────────────────

server.tool(
  "grade_readability",
  "Grade the readability of a documentation file based on average sentence length",
  {
    filePath: {
      type: "string",
      description: "Path to the documentation file to grade",
    },
  },
  async ({ filePath }) => {
    const content = fs.readFileSync(filePath, "utf-8");

    const plainText = content
      .replace(/```[\s\S]*?```/g, "")
      .replace(/`[^`]+`/g, "code")
      .replace(/#+\s/g, "")
      .replace(/\[([^\]]+)\]\([^)]+\)/g, "$1");

    const sentences = plainText.split(/[.!?]+/).filter(s => s.trim().length > 10);
    const words = plainText.split(/\s+/).filter(w => w.length > 0);
    const avgSentenceLength = sentences.length > 0 ? words.length / sentences.length : 0;

    let grade: string;
    let recommendation: string;
    if (avgSentenceLength < 15) {
      grade = "Easy";
      recommendation = "Good readability for developer audiences.";
    } else if (avgSentenceLength < 25) {
      grade = "Moderate";
      recommendation = "Acceptable. Consider shortening the longest sentences.";
    } else {
      grade = "Complex";
      recommendation = "Too complex. Target sentences under 25 words for developer docs.";
    }

    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          file: filePath,
          avg_sentence_length: Math.round(avgSentenceLength),
          grade,
          recommendation,
          sentence_count: sentences.length,
          word_count: words.length
        }, null, 2)
      }]
    };
  }
);
```

### Resource blocks (include both if user selected "Yes" in Phase 5)

Only include the resource for glossary if `check_terminology` was selected.
Only include the resource for structure-rules if `validate_doc_structure` was selected.

```typescript
// ── Resource: Glossary ────────────────────────────────────────────────────────
// (include only if check_terminology selected)

server.resource(
  "glossary",
  "style-guide://glossary",
  async (uri) => {
    const glossaryPath = path.join(__dirname, "../data/glossary.json");
    const content = fs.readFileSync(glossaryPath, "utf-8");
    return {
      contents: [{ uri: uri.href, mimeType: "application/json", text: content }]
    };
  }
);

// ── Resource: Structure rules ─────────────────────────────────────────────────
// (include only if validate_doc_structure selected)

server.resource(
  "structure-rules",
  "style-guide://structure-rules",
  async (uri) => {
    const rulesPath = path.join(__dirname, "../data/structure-rules.json");
    const content = fs.readFileSync(rulesPath, "utf-8");
    return {
      contents: [{ uri: uri.href, mimeType: "application/json", text: content }]
    };
  }
);
```

### File footer (always include)

```typescript
// ── Start the server ──────────────────────────────────────────────────────────

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

---

## data/glossary.json — Starter set

Use this when the user selects A or B in Phase 4.

```json
[
  { "banned": "whitelist",  "approved": "allowlist",  "note": "Inclusive language standard" },
  { "banned": "blacklist",  "approved": "blocklist",  "note": "Inclusive language standard" },
  { "banned": "master",     "approved": "primary",    "note": "Inclusive language standard (Git context: use 'main')" },
  { "banned": "slave",      "approved": "replica",    "note": "Inclusive language standard" },
  { "banned": "abort",      "approved": "cancel",     "note": "Less alarming in user-facing copy" },
  { "banned": "login",      "approved": "sign in",    "note": "Use 'login' only as a noun; 'sign in' as a verb" },
  { "banned": "execute",    "approved": "run",        "note": "Simpler verb for developer audiences" },
  { "banned": "utilize",    "approved": "use",        "note": "Simpler — 'utilize' rarely adds meaning" },
  { "banned": "leverage",   "approved": "use",        "note": "Business jargon — prefer plain language" },
  { "banned": "simply",     "approved": "",           "note": "Avoid confidence-assuming words; delete or rephrase" },
  { "banned": "just",       "approved": "",           "note": "Avoid confidence-assuming words; delete or rephrase" },
  { "banned": "easy",       "approved": "",           "note": "Avoid confidence-assuming words; delete or rephrase" },
  { "banned": "obviously",  "approved": "",           "note": "Condescending; delete" }
]
```

Append custom user entries after the last entry (before the closing `]`).

---

## data/structure-rules.json — Default rules

Use only the doc types the user selected. If they selected "how-to" and "reference" only, output only those two keys.

```json
{
  "how-to": {
    "required": ["Overview", "Prerequisites", "Steps", "Result"],
    "optional": ["Troubleshooting", "Next steps", "Related"]
  },
  "reference": {
    "required": ["Description", "Parameters", "Response", "Errors"],
    "optional": ["Authentication", "Example", "Notes"]
  },
  "tutorial": {
    "required": ["Introduction", "Prerequisites", "Steps", "Summary"],
    "optional": ["Next steps", "Resources"]
  },
  "release-note": {
    "required": ["What's new", "Fixed"],
    "optional": ["Known issues", "Deprecated", "Breaking changes", "Upgrade guide"]
  }
}
```

For custom doc types (Phase 3 option F), map the user's provided section names to `required` and treat any sections they marked as optional as `optional`. If they didn't distinguish, put all sections in `required`.

---

## README.md

```markdown
# {{SERVER_NAME}}

{{DESCRIPTION}}

## Tools

| Tool | Description |
|------|-------------|
{{TOOLS_TABLE_ROWS}}

{{RESOURCES_SECTION}}

## Installation

```bash
npm install
npm run build
claude mcp add {{SERVER_NAME}} -- node /absolute/path/to/dist/index.js
```

## Usage

```
{{USAGE_EXAMPLES}}
```

{{CUSTOMIZATION_SECTION}}

## Publishing

```bash
npm login
npm publish --access public
```

Registry namespace: `io.github.{{GITHUB_USERNAME}}/{{SERVER_NAME}}`

## License

MIT
```

### Substitution guide

**`{{TOOLS_TABLE_ROWS}}`** — one row per selected tool:
```
| `check_terminology` | Flags banned terms and suggests approved alternatives from `data/glossary.json` |
| `detect_passive_voice` | Finds passive voice sentences and suggests active rewrites |
| `validate_doc_structure` | Checks required sections by doc type ({{DOC_TYPE_LIST}}) |
| `grade_readability` | Returns Easy / Moderate / Complex grade based on sentence length |
```
Include only the rows for tools the user selected.

**`{{RESOURCES_SECTION}}`** — include only if user selected Yes in Phase 5:
```markdown
## Resources

| Resource URI | Description |
|---|---|
| `style-guide://glossary` | The approved terms list as JSON |
| `style-guide://structure-rules` | Required and optional sections per doc type as JSON |
```
Omit the Resources section entirely if the user said No.

**`{{USAGE_EXAMPLES}}`** — one example per selected tool:
```
Use check_terminology on docs/my-article.md
Use detect_passive_voice on docs/my-article.md
Use validate_doc_structure on docs/my-article.md with docType "how-to"
Use grade_readability on docs/my-article.md
```

**`{{CUSTOMIZATION_SECTION}}`** — include only if `check_terminology` or `validate_doc_structure` selected:
```markdown
## Customizing

### Glossary (`data/glossary.json`)
Each entry: `{ "banned": "word", "approved": "replacement", "note": "reason" }`.
Set `approved` to `""` to flag the word for deletion without a replacement.

### Structure rules (`data/structure-rules.json`)
Each doc type maps to `required` (must have) and `optional` (nice to have) section name arrays.
Matching is case-insensitive and partial — a heading "Next Steps" matches rule "Next steps".
```
