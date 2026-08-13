# Interface Design System

## Direction

**Signal & Paper — an engineering research notebook.**

The site should feel scholarly, precise, and quietly technical. EB Garamond provides the editorial voice, while rules, nodes, numbering, and restrained technical labels borrow from signal paths, control loops, CPU pipelines, and agent workflows. The result should read as an engineer's working record rather than a generic portfolio or a literary magazine.

The primary audience is prospective research supervisors, collaborators, and technical recruiters. A visitor should understand Zehao Shen's research direction within 20 seconds, then be able to inspect concrete systems and open the CV or source repositories without searching.

## Domain Vocabulary

- Signal paths and communication links
- Closed-loop control
- CPU pipeline stages
- Agent workflows and tool calls
- System diagrams and schematics
- Laboratory notebooks and measurement records
- Research citations and technical annotations

## Color System

### Light

- `paper`: `#F7F3EA` — primary background
- `surface`: `#FCFAF5` — figures and restrained raised regions
- `ink`: `#26231F` — primary text
- `muted-ink`: `#6E685F` — metadata and captions
- `signal`: `#245B78` — links, active states, and signal-path nodes
- `copper`: `#9A6846` — rare secondary emphasis
- `trace`: `#D7D0C4` — rules and boundaries

### Dark

- `dark-paper`: `#171816`
- `dark-surface`: `#20211E`
- `dark-ink`: `#EAE4D8`
- `dark-muted-ink`: `#AAA399`
- `dark-signal`: `#7FA9BF`
- `dark-copper`: `#C18B68`
- `dark-trace`: `#3B3C37`

Use `signal` as the single primary accent. Use `copper` only where a second semantic distinction is necessary. Avoid pure black, pure white, gradients, and decorative accent colors.

## Typography

- Primary family: `"EB Garamond", Georgia, serif`
- Code, commands, and identifiers: the existing monospace stack
- Icons: their owning icon fonts
- Body: `18px / 1.65`, weight `400`
- Small body and metadata: `15px / 1.45`, weight `400`
- Display name: `52px / 1.02`, weight `600`
- Page title: `42px / 1.08`, weight `500`
- Section heading: `30px / 1.15`, weight `500`
- Subheading: `22px / 1.25`, weight `500`
- Strong emphasis: weight `700`
- Captions: `15px / 1.4`, italic

On screens below `640px`, use a `40px` display name, `34px` page title, and `17px` body. Avoid all-caps body text; short technical labels may use small caps or modest letter spacing.

## Spacing and Layout

- Base spacing unit: `8px`
- Main content width: `1040px` maximum
- Reading column: `38–42rem`
- Article figure width: up to `900px` when the diagram benefits from it
- Section spacing: `80px` desktop, `48px` mobile
- Component spacing: `24px` or `32px`
- Inline spacing: `8px` or `12px`

The homepage may keep its text-and-portrait split, but the text must remain the focal point. Project details use a narrow reading column with selected system figures extending beyond it.

## Depth Strategy

Use borders, background tone, and whitespace instead of elevation. Default surfaces have no shadow. The only permitted shadow is a very subtle image separation where a photograph would otherwise disappear into the background. Avoid stacked cards, glass effects, large rounded containers, and floating pills.

- Default border: `1px solid trace`
- Default corner radius: `2px`; photographs may use `4px`
- Default shadow: none
- Interactive transition: `150–180ms`, color or underline movement only

## Signature: The Signal Path

A thin line with small solid or hollow nodes is the site's identifying visual language. It should appear consistently in at least these five places:

1. The active navigation item uses a short signal-line underline with a terminal node.
2. Homepage section headings connect to a horizontal rule through a node.
3. Project entries carry a numbered node (`01`–`04`) at the start of their metadata line.
4. Project-detail section dividers use a short path that visually links the narrative to system diagrams.
5. The footer ends the page with a final node and short rule rather than a heavy solid band.

The motif must remain structural and quiet. Do not turn it into a circuit-board background, animated particle effect, or decorative illustration.

## Component Patterns

### Global navigation

`64px` desktop height · `56px` mobile height · transparent/paper background · `1px` bottom trace · text links with no button containers · active signal underline.

### Homepage introduction

Desktop uses an approximately `7:5` text-to-portrait split · `32px` gutter · portrait radius `4px` · introduction measure no wider than `42rem` · contact links remain compact and secondary.

### Project index entry

Editorial row rather than a floating card · `24px` vertical padding · `1px` bottom trace · project number and metadata at `15px` · title at `26px/500` · image ratio approximately `16:10` · title, image, and explicit link share one clear destination.

### Project article

Reading column `42rem` · body `18px/1.7` · section gap `56px` · figures may expand to `900px` · captions `15px` italic · blockquote text remains the same size as body text · code remains monospace.

### Repository presentation

Treat repository cards as supporting evidence, not the homepage focal point. Use flat bordered rows or restrained cards with no decorative shadows; prioritize repository name, description, language, and source link.

### Focus and hover

Text links gain a visible underline on hover. Keyboard focus uses a `2px` signal-colored outline with a `3px` offset. Do not rely on color alone to communicate state.

## Rejected Defaults

- Default white page with bright template blue → warm paper and muted signal blue
- Uniform rounded card grid → numbered editorial project index
- Repeated drop shadows and pills → rules, whitespace, and typographic hierarchy
- Oversized animated hero → compact research identity and immediate evidence
- Decorative circuit-board imagery → one disciplined signal-path motif

## Constraints

- Do not duplicate biography, education, or contact facts across sections.
- Do not add claims, metrics, affiliations, or project outcomes without source content.
- Preserve accessible contrast, keyboard focus, responsive behavior, and dark mode.
- Preserve monospace for code and specialist fonts for icons and mathematics.
- Respect the repository's thin-starter boundary; prefer the existing global stylesheet and content/front-matter hooks over local copies of gem-owned layouts or includes.

## Reference Status

No generated mood board or external visual reference has been adopted. This system is derived from the site's actual research domains and existing content.
