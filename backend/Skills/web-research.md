# Skill: Web Research

## Purpose

Provide a structured protocol for capturing web content into the workspace research folder with full provenance tracking.

## When This Skill Applies

- The user asks to research, look up, or investigate a topic online
- The user asks to capture or save a web page for reference
- The user provides a URL and asks for it to be recorded or filed
- The agent needs background information from the web to support a task (e.g. checking a company, looking up a regulation, verifying a property record)

## Relationship to workspace-skill.md

This skill **extends** the Deterministic Filing Table, which routes "policy/research docs" to `operations/Research/`.
workspace-skill.md defines the destination but not the capture procedure. This skill defines the procedure.

## Protocol

### Step 1: Clarify Research Scope

Before fetching anything, confirm:
- What topic or entity the user wants researched
- Whether specific URLs are provided or the agent should search
- Which project(s) this research relates to (for tagging)

If the user provides a URL directly, skip to Step 3.

### Step 2: Search and Identify Sources

When no URL is given:
- Use available search tools to find relevant pages
- Prioritise official sources (company websites, government registries, professional directories)
- Identify 1-3 most relevant pages to capture

Present the sources to the user before capturing, unless PermitAll is active.

### Step 3: Capture Web Content

For each URL:
- Fetch the page content
- Extract the meaningful text, stripping navigation, ads, sidebars, and boilerplate
- Preserve headings, lists, tables, and links from the original
- Note any content that could not be captured (e.g. behind login, JavaScript-only rendering)

### Step 4: Save Research File

Save to: `operations/Research/<YYYY-MM-DD>_<Topic_Slug>.md`

Topic slug rules:
- Use 2-4 descriptive words separated by underscores
- No spaces, lowercase
- Examples: `corby_surveyors_background`, `savills_property_services`, `uk_planning_policy`

If a research file for the same topic already exists (same slug, different date), append new content as a new dated section rather than creating a duplicate file.

### Step 5: Metadata Header

Every research file must start with a metadata header:

```
---
CapturedDate: YYYY-MM-DD
Topic: <descriptive topic name>
Sources:
  - URL: <url1>
    AccessedDate: YYYY-MM-DD
  - URL: <url2>
    AccessedDate: YYYY-MM-DD
SearchTerms: <terms used to find these sources, if applicable>
Project: <project tag(s)>
Relevance: <one-sentence summary of why this research matters>
---
```

### Step 6: Content Formatting

Below the metadata header, structure the captured content as:

```
## <Source Title or Domain>
**URL:** <url>
**Accessed:** YYYY-MM-DD

<captured content in clean markdown>

---
```

Repeat for each source. Use `---` separators between sources.

### Step 7: Update Workspace Records

After saving the research file:
1. Create or update a `FileAnalysisCurrent.md` entry:
   - Use `DOC-###` ID scheme
   - Include file path, topic, sources, and relevance
   - Tag with project(s)
2. Add a `ChronologyCurrent.md` entry:
   - `YYYY-MM-DD — Web research captured: <topic> — <file path>`

### Step 8: Follow-Up Actions

If the research reveals information that affects strategy or active tasks:
- Note the finding in the relevant FileAnalysis entry
- Suggest whether StrategyCurrent.md should be updated
- Do not update strategy automatically -- ask the user first

## Multi-Session Research

When the user researches the same topic across multiple sessions:
- Append to the existing research file rather than creating a new one
- Add a new dated section header within the file
- Update the metadata header's Sources list
- Update the FileAnalysis entry to reflect the additional sources

## Output

- Research file: `operations/Research/<YYYY-MM-DD>_<Topic_Slug>.md`
- FileAnalysis entry with `DOC-###` ID
- Chronology entry recording the research event
