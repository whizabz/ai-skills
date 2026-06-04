---
name: how-i-built-this
version: 1.0.0
description: |
  Generate or update a how-i-built-this.html build journey page for a project.
  Triggers: any request to document a build, create a project journey page,
  or update an existing how-i-built-this.html. Supports both new file creation
  and in-place updates when the file already exists. Output uses the Kami design
  system (bg #f5f4ed, accent #1B365D, Newsreader + Inter) with a timeline/card
  layout showing phases, prompts, decisions, and a live "Last updated" timestamp.
license: MIT
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
---

# How I Built This — Skill

You are producing a **living build journey document** — an HTML file that documents how a project was built, using a card/timeline layout styled in the Kami design system. The file should feel like a personal dev log, not a case study.

---

## 1. Trigger conditions

Use this skill when the user asks to:
- Create a `how-i-built-this.html` for a project
- Document their build journey / process
- Update an existing build journey file with new progress
- Add phases, prompts, or decisions to an existing file

---

## 2. Check for existing file FIRST

Before writing anything, check if `how-i-built-this.html` already exists in the project directory or outputs:

```bash
find . -name "how-i-built-this.html" 2>/dev/null | head -5
```

**If file exists → UPDATE mode:** Read the file, extract existing content, append/merge new content. Preserve existing phases and prompts. Update the `data-updated` timestamp attribute on the `<body>` tag and the visible "Last updated" line.

**If file doesn't exist → CREATE mode:** Generate the full file from scratch.

---

## 3. What to ask before building

If the user hasn't provided the following, ask before generating:

- **Project name** — what's the app/tool called?
- **Live URL** (if deployed) — where can people see it?
- **GitHub URL** (if public) — link to the repo
- **Stack** — main technologies, frameworks, services used
- **One-liner** — how would you describe it in a sentence?
- **Phases or milestones** — rough stages of the build (can be inferred from a BUILD_JOURNEY.md or similar if provided)
- **Prompt log or notes** — do they have a list of what they asked/did? (optional but useful)

If they've uploaded a BUILD_JOURNEY.md, design.md, or similar — read those first and infer what you can before asking.

---

## 4. File structure

The output HTML follows this structure exactly:

```
<header>         — Project name, one-liner, live link, stat strip, intro note
<nav>            — Phase navigation (sticky, scrolls to sections)
<main>
  <section>      — One per phase, with phase label + description
    <article>    — One per prompt/step/decision within the phase
  </section>
</main>
<footer>         — Last updated timestamp + project links
```

---

## 5. Kami design system — mandatory constraints

These are non-negotiable. Every output must conform:

```css
/* Palette */
--bg: #f5f4ed;            /* warm parchment, page background */
--accent: #1B365D;        /* ink blue, primary accent */
--accent-mid: #2a4f87;
--accent-light: #c8d6e8;
--accent-pale: #e8eef5;
--warm-100: #f0ede3;
--warm-200: #e4dfc8;
--warm-300: #c8c2aa;
--warm-400: #9c9480;
--warm-600: #5a5244;
--warm-800: #2c2820;      /* body text */

/* Typography */
font-family (headings/body): 'Newsreader', serif   — weight 500, never bold
font-family (UI/labels): 'Inter', sans-serif
Google Fonts import: Newsreader (opsz 6–72, weights 300/400/500/600 + italic) + Inter (300/400/500/600/700)

/* Line heights */
headlines: 1.1–1.3
dense body: 1.4–1.45
reading body: 1.5–1.55   — never exceed 1.6

/* Shadows */
--shadow-ring: 0 0 0 1px rgba(27,54,93,0.08);
--shadow-whisper: 0 2px 12px rgba(27,54,93,0.06), 0 1px 3px rgba(27,54,93,0.04);
--shadow-lift: 0 8px 32px rgba(27,54,93,0.10), 0 2px 8px rgba(27,54,93,0.06);

/* Rules */
- No hard drop shadows
- Tag backgrounds: solid hex only (never gradients, no rgba tags)
- Warm neutrals only — no grey (#6b7280, #888, etc.)
- No bold headlines — use font-weight 500 for Newsreader
```

---

## 6. Timestamp behaviour (critical for updates)

The file must carry an updatable timestamp in two places:

**In the `<body>` tag:**
```html
<body data-updated="2026-06-04T14:30:00">
```

**In the footer:**
```html
<div class="last-updated">
  Last updated — <time id="last-updated" datetime="2026-06-04T14:30:00">4 June 2026</time>
</div>
```

**JavaScript to auto-format the date:**
```javascript
const t = document.getElementById('last-updated');
if (t) {
  const d = new Date(t.getAttribute('datetime'));
  t.textContent = d.toLocaleDateString('en-GB', { day: 'numeric', month: 'long', year: 'numeric' });
}
```

When updating an existing file, find these two attributes and update both to the current date/time. Use ISO 8601 format for `datetime` and `data-updated`.

---

## 7. Phase card structure

Each phase is a `<section>` with a header and one or more step cards:

```html
<section class="phase" id="phase-1">
  <div class="phase-header">
    <span class="phase-label">Phase 1</span>
    <h2 class="phase-title">Design & Spec</h2>
    <p class="phase-desc">What this phase was about, in 1–2 sentences.</p>
  </div>

  <div class="steps">
    <article class="step">
      <div class="step-meta">
        <span class="step-number">01</span>
        <span class="step-type">Prompt</span>   <!-- or: Decision / Pivot / Debug / Deploy -->
      </div>
      <h3 class="step-title">Short title for this step</h3>
      <p class="step-body">What happened. Paraphrased, conversational. What was asked, what came back, what decision was made. Written in first person.</p>
      <div class="step-tags">
        <span class="tag">Firebase</span>
        <span class="tag">Auth</span>
      </div>
    </article>
  </div>
</section>
```

**Step type labels** — choose the most accurate one:
- `Prompt` — something typed into the AI / a specific ask
- `Decision` — a product or design choice made
- `Pivot` — a direction change or rethink
- `Debug` — something that broke and how it was fixed
- `Deploy` — shipping, publishing, or going live
- `Idea` — the origin spark or a new feature concept

---

## 8. Header / hero section

```html
<header class="site-header">
  <div class="header-inner">
    <div class="project-eyebrow">How I built this</div>
    <h1 class="project-name">[Project Name]</h1>
    <p class="project-tagline">[One-liner description]</p>

    <div class="project-links">
      <a href="[live-url]" class="link-pill live">↗ Live app</a>
      <a href="[github-url]" class="link-pill github">GitHub</a>
    </div>

    <div class="stat-strip">
      <div class="stat-cell">
        <div class="stat-value">[N]</div>
        <div class="stat-label">Steps documented</div>
      </div>
      <div class="stat-cell">
        <div class="stat-value">[N]</div>
        <div class="stat-label">Phases</div>
      </div>
      <div class="stat-cell">
        <div class="stat-value">[Stack item]</div>
        <div class="stat-label">Primary stack</div>
      </div>
    </div>

    <div class="intro-note">
      <p>This is an overview, not a transcript. The steps below are paraphrased summaries — written to be easy to follow for someone who wasn't there. The actual prompts were often shorter, messier, and more back-and-forth.</p>
    </div>
  </div>
</header>
```

---

## 9. Navigation

Generate a sticky `<nav>` that links to each phase section. Label should be "Phase N — [Title]". Keep it compact — Inter 12px, horizontal scroll on mobile.

---

## 10. UPDATE mode — how to merge new content

When the file already exists and the user wants to add new progress:

1. **Read the file** — identify the existing phases and the highest step number used
2. **Determine where to add** — are we adding to an existing phase, or creating a new phase?
3. **Append new `<article>` cards** inside the relevant `<section>`, with step numbers continuing from the last
4. **If new phase** — add a new `<section>` after the last existing one; add a new `<a>` to the `<nav>`
5. **Update timestamps** — both `data-updated` on `<body>` and `datetime` + text in the footer
6. **Update stat strip** — increment "Steps documented" count
7. Do NOT change existing content unless the user explicitly asks

---

## 11. Tone and voice

- Written in **first person** ("I asked...", "I decided...", "The first thing I did...")
- **Conversational, not polished** — sounds like a dev log, not a case study
- **Specific** — name the actual tool, service, or decision (Firebase, not "a database service")
- **Honest about the messy parts** — pivots, bugs, and confusion are features, not bugs
- Don't over-explain — a sentence or two per step is often enough
- No role labels ("Senior Designer Abhi did X") — work speaks

---

## 12. What NOT to do

- Don't add a dark mode toggle (not part of Kami — stick to the warm parchment background)
- Don't use `#6b7280` or any CSS grey — warm neutrals only
- Don't use `font-weight: 700` on Newsreader headings — use 500
- Don't add a print stylesheet
- Don't use `rgba()` for tag backgrounds — solid hex only
- Don't bold the phase title with `<strong>` inside headings
- Don't add animations beyond subtle hover transitions (0.15s ease)
- Don't embed images unless the user specifically provides them
- Don't use `border-radius > 12px` on cards
- Don't make the layout wider than `72ch` for reading text

---

## 13. Full CSS reference

Below is the canonical CSS skeleton. Always start from this and extend — don't deviate from the core variables:

```css
@import url('https://fonts.googleapis.com/css2?family=Newsreader:ital,opsz,wght@0,6..72,300;0,6..72,400;0,6..72,500;0,6..72,600;1,6..72,300;1,6..72,400&family=Inter:wght@300;400;500;600;700&display=swap');

:root {
  --bg: #f5f4ed;
  --accent: #1B365D;
  --accent-mid: #2a4f87;
  --accent-light: #c8d6e8;
  --accent-pale: #e8eef5;
  --warm-100: #f0ede3;
  --warm-200: #e4dfc8;
  --warm-300: #c8c2aa;
  --warm-400: #9c9480;
  --warm-600: #5a5244;
  --warm-800: #2c2820;
  --shadow-ring: 0 0 0 1px rgba(27,54,93,0.08);
  --shadow-whisper: 0 2px 12px rgba(27,54,93,0.06), 0 1px 3px rgba(27,54,93,0.04);
  --shadow-lift: 0 8px 32px rgba(27,54,93,0.10), 0 2px 8px rgba(27,54,93,0.06);
}

* { margin: 0; padding: 0; box-sizing: border-box; }
html { scroll-behavior: smooth; }

body {
  background: var(--bg);
  color: var(--warm-800);
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  line-height: 1.5;
}

/* Headings always use Newsreader */
h1, h2, h3, h4 {
  font-family: 'Newsreader', serif;
  font-weight: 500;
}

/* Phase header */
.phase-header {
  padding: 3rem 0 1.5rem;
  border-bottom: 1px solid var(--warm-200);
  margin-bottom: 2rem;
}
.phase-label {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--warm-400);
}
.phase-title {
  font-size: clamp(1.5rem, 3vw, 2.25rem);
  line-height: 1.2;
  color: var(--accent);
  margin: 0.25rem 0 0.5rem;
}
.phase-desc {
  font-family: 'Inter', sans-serif;
  font-size: 0.9375rem;
  color: var(--warm-600);
  line-height: 1.5;
  max-width: 60ch;
}

/* Step cards */
.step {
  background: #fff;
  border-radius: 10px;
  padding: 1.5rem;
  margin-bottom: 1rem;
  box-shadow: var(--shadow-whisper);
  border: 1px solid var(--warm-200);
  transition: box-shadow 0.15s ease;
}
.step:hover {
  box-shadow: var(--shadow-lift);
}
.step-meta {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  margin-bottom: 0.75rem;
}
.step-number {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 700;
  color: var(--warm-300);
  letter-spacing: 0.05em;
}
.step-type {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 500;
  color: var(--accent);
  background: var(--accent-pale);
  padding: 2px 8px;
  border-radius: 4px;
}
.step-title {
  font-size: 1.0625rem;
  line-height: 1.3;
  color: var(--warm-800);
  margin-bottom: 0.5rem;
}
.step-body {
  font-family: 'Inter', sans-serif;
  font-size: 0.9375rem;
  line-height: 1.55;
  color: var(--warm-600);
}
.step-tags {
  margin-top: 1rem;
  display: flex;
  flex-wrap: wrap;
  gap: 0.375rem;
}
.tag {
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 500;
  color: var(--warm-600);
  background: var(--warm-100);
  border: 1px solid var(--warm-200);
  padding: 2px 8px;
  border-radius: 4px;
}

/* Stat strip */
.stat-strip {
  display: flex;
  gap: 2rem;
  margin: 2rem 0 1.5rem;
  padding: 1.25rem 0;
  border-top: 1px solid var(--warm-200);
  border-bottom: 1px solid var(--warm-200);
}
.stat-value {
  font-family: 'Newsreader', serif;
  font-size: 2rem;
  font-weight: 500;
  color: var(--accent);
  line-height: 1.1;
}
.stat-label {
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  color: var(--warm-400);
  margin-top: 0.25rem;
}

/* Intro note */
.intro-note {
  background: var(--warm-100);
  border-left: 3px solid var(--accent-light);
  padding: 1rem 1.25rem;
  border-radius: 0 6px 6px 0;
  font-family: 'Inter', sans-serif;
  font-size: 0.875rem;
  line-height: 1.55;
  color: var(--warm-600);
  margin-top: 1.5rem;
}
.intro-note p + p { margin-top: 0.5rem; }

/* Nav */
.phase-nav {
  position: sticky;
  top: 0;
  background: var(--bg);
  border-bottom: 1px solid var(--warm-200);
  padding: 0.75rem 0;
  z-index: 100;
  overflow-x: auto;
  white-space: nowrap;
}
.phase-nav a {
  display: inline-block;
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  font-weight: 500;
  color: var(--warm-400);
  text-decoration: none;
  margin-right: 1.5rem;
  padding-bottom: 2px;
  border-bottom: 2px solid transparent;
  transition: color 0.15s, border-color 0.15s;
}
.phase-nav a:hover,
.phase-nav a.active {
  color: var(--accent);
  border-bottom-color: var(--accent);
}

/* Footer */
footer {
  padding: 3rem 0 2rem;
  border-top: 1px solid var(--warm-200);
  margin-top: 4rem;
}
.last-updated {
  font-family: 'Inter', sans-serif;
  font-size: 0.8125rem;
  color: var(--warm-400);
}

/* Layout wrapper */
.container {
  max-width: 720px;
  margin: 0 auto;
  padding: 0 1.5rem;
}
```

---

## 14. Checklist before outputting

Before writing the final file, confirm:

- [ ] Kami CSS variables present and correct
- [ ] Google Fonts `@import` at top of `<style>` block
- [ ] `data-updated` attribute on `<body>` with today's date
- [ ] `<time id="last-updated">` in footer with matching `datetime`
- [ ] JS timestamp formatter included in `<script>` at bottom
- [ ] At least one `<section class="phase">` with at least one `<article class="step">`
- [ ] Nav links match section IDs
- [ ] Stat strip counts are accurate
- [ ] No grey colours (#6b7280, #888, etc.)
- [ ] No bold (700) Newsreader headings
- [ ] File named `how-i-built-this.html` unless user specifies otherwise
