# obsidian-vault

Personal knowledge base. Plain Markdown; Obsidian is only the editor.

## Top-level folders describe a note's *state*; inside `10-notes/`, one folder per area

| Folder | What lives here |
|---|---|
| `00-inbox/` | Drafts: raw captures, mine or Claude's. New notes land here by default. |
| `10-notes/<area>/` | Approved atomic notes. `<area>` is the map just below the root map, e.g. `spring`, `database`, `ops-and-cloud`. |
| `20-moc/` | Approved Maps of Content. |
| `30-sources/` | One note per book / course / article. |
| `40-blog-drafts/` | Notes mature enough to become a post. |
| `90-attachments/` | Images and files. |
| `templates/` | Note templates. |

Topic is expressed by `[[links]]`, tags and MOCs, never by folders.

## Two kinds of node

| Kind | Title | Example |
|---|---|---|
| Atomic note | A complete claim: readable as a sentence on its own. | *A checked exception commits a Spring @Transactional method by default* |
| MOC (map of content) | `<Topic> MOC`. The suffix marks it as a map, keeps it from clashing with a future note named after the concept, and is the only cue the graph shows (it hides folders). | *@Transactional MOC* |

Every note points to its map with `up:`; every MOC points to its parent map with `up:`, up to
Home. Maps form a network: a note may sit under several MOCs. A topic gets its own MOC once it
has about five notes; below that, its notes sit in a section of the broader MOC.

## Status

- `status: draft`: captured, possibly written by Claude. Not yet in my own words.
- `status: verified`: I rewrote it and checked it. Only these may be published or fed to any AI index.

Fields the `obsidian-note` skill adds to notes it captures (all in `00-inbox/`):

| Field | Meaning |
|---|---|
| `author: claude` | Written by Claude, not by me. |
| `up` | The MOC(s) this note sits under. Structural: it needs no reason and is not shown to the judge. |
| `type: moc` | This note is a map, not an idea. Maps are linted, never scored. |
| `review` | `conflict` (contradicts a verified note), `ready` (every checked criterion passed), `borderline` (the judge was unsure on one), `parked` (one failed), `pending` / `unjudged` (not scored: quick capture, or Jev unavailable). |
| `score` | Weighted mean of Jev's per-criterion answers, 0–1. It orders the queue; it never promotes a note. |
| `score_reasons` | Failed or unsure criteria with their values, conflicts first. |
| `judged_by`, `judge_rounds` | Jev model version and how many rounds it took. |
| `proposes_for` | The note an addition is proposed for; the original is never edited. |

Only I set `status: verified`, and only after rewriting the note in my own words.

## Approving a note

1. Rewrite it in my own words, then set `status: verified` in its Properties.
2. Ask Claude to "file my approved notes". `vault.py promote` moves each verified note from
   `00-inbox/` (or loose in `10-notes/`) to `10-notes/<area>/`, and each verified MOC to `20-moc/`.
   It never renames, overwrites or deletes; links keep working because they resolve by name.
3. A note is skipped, with the reason, when it sits under no MOC, under maps of two areas
   (set `up:` to choose), or when a same-named file already exists at the target.

Area folders are coarse on purpose: a note belongs to one folder but may sit under several
MOCs, so the fine structure lives in the maps, not in the folders.

## Rules

1. One idea per note. The title is a claim ("asyncio suits I/O-bound work"), not a topic ("asyncio").
2. Every link says why it is there.
3. Every note gets a "Why choose / why not" section; that is what interviews ask.
4. **No client or employer data in this vault.** It is the personal vault and may be published.
