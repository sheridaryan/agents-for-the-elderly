# Voice Journal — Build Specification

**Version:** 1.0
**Date:** 27 September 2026
**Product owner:** Sherida
**Status:** Approved for build (Phase: personal prototype; design probe for the caregiver support agent project)

---

## 0. How to use this spec with Claude Code

- Keep this file in the project root and reference it from `CLAUDE.md` as the single source of truth.
- Requirements marked **[Proposed]** were accepted as defaults; they may be adjusted, but flag any change to the product owner.
- Items marked **[Verify at build]** must be checked against current documentation before implementation. Do not rely on memory.
- **Never use real journal entries** in any build session, test fixture, log, or screenshot. Use only the synthetic test set (§11.2).
- Build in the order given in §14 and confirm each stage with the product owner before moving on.

---

## 1. My Purpose, scope and non-goals

### 1.1 Purpose

A personal voice journal for a single user. The user dictates entries using the device's built-in voice typing (Windows voice typing on desktop; Gboard voice typing on a Google Pixel phone) into a plain text box. An AI agent (Claude) responds with a brief, validate-first reflection. A weekly summary surfaces patterns drawn only from the user's own words.

The app also serves as an **autobiographical design probe** for the caregiver support agent project, through a separate designer-notes area reviewed monthly.

### 1.2 In scope

- Text entry designed for system dictation (the app itself has no audio capture)
- One-tap energy rating per entry
- Reflection on each entry, with optional continued conversation
- Weekly summary
- Review screen with search and entry deletion
- Designer notes
- In-browser encryption of all stored content
- Encrypted backup and restore; readable export

### 1.3 Non-goals (the app must not)

- Act as therapy: no diagnosis, clinical labelling, treatment advice, or assessment scores
- Record, store, or analyse audio, tone, or voice features
- Scan entries in the background or on the server outside a user-initiated reflection or summary
- Send notifications, emails, or reminders
- Support multiple users, sharing, or social features
- Mask names before sending text to Claude (decided: not included)
- Allow editing of an entry after Reflect is pressed
- Let any AI function read designer notes

### 1.4 Design principles (binding throughout)

1. **Validate before anything else.** No reflection leads with advice, reassurance, or problem-solving.
2. **The user's words are the record.** Patterns come only from the user's words, never the agent's.
3. **Privacy is enforced technically**, not by policy alone.
4. **Minimum context to the AI.** Send only what each function needs.
5. **Simplicity first.** Where options are otherwise equal, choose the simpler.

---

## 2. User flows

### 2.1 First run

1. User creates an account (sign-in credentials).
2. User sets a passphrase, entered twice. Before continuing, a screen states: _"Your passphrase encrypts your journal. It is never stored or sent anywhere. If you lose it, your entries cannot be recovered by anyone, including the developer."_ The user must tick an acknowledgement to proceed.
3. The passphrase is never transmitted to or stored on the server.

### 2.2 Opening the app

- User signs in, then enters the passphrase to unlock.
- Decrypted content exists only in browser memory.
- The app locks after **30 minutes of inactivity** (§8.3).

### 2.3 New entry

1. A large, empty text box appears with focus, ready for dictation with any system voice-typing tool.
2. An energy rating (**Low / Medium / High**) is always shown and may be skipped. No prompt if skipped.
3. The text is fully editable until **Reflect** is pressed. The user reviews the dictated text here.
4. On **Reflect**:
   - the entry and energy rating are encrypted and saved immediately (nothing is lost if the connection fails);
   - a timestamp is recorded automatically;
   - **the entry text is locked and can no longer be edited;**
   - the reflection request is sent.

### 2.4 After the reflection

The reflection appears beneath the entry with three controls:

- **Reply** — opens a text box for dictation; continues the conversation (§3.5)
- **Note about the app** — opens designer-notes capture (§6)
- **Done** — closes the entry

### 2.5 Closing an entry: save prompt

On **Done**, a prompt asks: **"Save the whole conversation, or just your words?"**

- **Whole conversation:** saves the entry, user replies, and agent reflections and responses.
- **Just my words:** saves the entry and user replies; agent text is discarded.
- **Default if unanswered** (app closed, session ends, auto-lock): _Just my words._

The weekly summary reads only the user's words regardless of this choice (§4.2).

### 2.6 Error states

- **Network failure on Reflect:** the entry is already saved. Show _"Your entry is saved. The reflection didn't come through — try again?"_ with a Retry button.
- **Malformed AI response:** retry once automatically; if it fails again, show the same message.
- **Back end paused** (Supabase free plan, §9.1): show _"Your journal's server is resting after a quiet week. It needs to be woken up from the Supabase dashboard before you can continue."_ with a short how-to link in the app's help text.
- **Wrong passphrase:** _"That passphrase didn't unlock your journal. Please try again."_ Reveal nothing else.

---

## 3. Reflection behaviour

### 3.1 Framework

Reflections draw on Linehan's levels of validation:

| Level                                   | Behaviour                                              | Rule                                                     |
| --------------------------------------- | ------------------------------------------------------ | -------------------------------------------------------- |
| 1. Attending                            | Shows it took in what was actually said                | Always                                                   |
| 2. Accurate reflection                  | Says back the core of the entry in fresh words         | Always                                                   |
| 3. Articulating the unspoken            | Names an implied feeling                               | Tentative form only ("I wonder if…"); heavy entries only |
| 4. Validating in light of recent events | Connects the reaction to something recent              | Only via light mentions (§3.4)                           |
| 5. Normalising                          | Notes the reaction makes sense given the circumstances | Moderate and heavy entries                               |
| 6. Genuineness                          | Plain, adult, sincere language                         | Always                                                   |

Central rule: validate that a **feeling** is understandable. Never endorse a harsh **self-judgement**, and never argue with it.

### 3.2 Length

Set by three signals: entry length, emotional intensity, and whether a feeling is named.

| Entry weight                                | Length            | Levels          |
| ------------------------------------------- | ----------------- | --------------- |
| Light (routine, factual)                    | 1–2 sentences     | 1, 2, 6         |
| Moderate (a feeling present)                | 2–3 sentences     | + 5             |
| Heavy (strong emotion or significant event) | A short paragraph | + 3 (tentative) |

**Hard ceiling:** a reflection is never longer, in words, than the entry.

### 3.3 Questions

- Default: end on acknowledgement, not a question.
- A question is permitted only when the user explicitly asks for input, or the entry ends with an open question the user poses to themselves.
- When permitted: one question, open-ended, framed so it can be ignored ("If you want to come back to it…").
- Test target: in a sample of 20 typical entries, no more than 4 reflections contain a question.

### 3.4 Light mention of past entries

- **Window:** entries from the past 7 days (excluding deleted entries).
- **Trigger:** only if the current entry refers to an earlier one, or clearly continues a thread from the past 7 days.
- **Limit:** one reference per reflection at most.
- **Framing:** continuity only. No counts ("the third time"), no trends ("you often…"), no pattern interpretations.
- **Source:** past entries are drawn from the user's words only.

### 3.5 Continued conversation

- Every agent turn follows the validate-first sequence.
- Advice, information, or perspective only when the user asks; then plain and brief.
- The agent never prolongs the conversation: no "Anything else?", no filler questions. The question rules in §3.3 apply to every turn.
- The user ends the conversation.

### 3.6 Prohibited in every reflection and turn

- Unrequested advice or tips
- Contradicting a feeling ("don't be so hard on yourself")
- Interpreting with certainty
- Effusive praise or cheerleading
- Clinical labels or diagnoses
- Habitual stock phrases (e.g., "It sounds like…" may open no more than 1 in 5 reflections)
- Exclamation marks and emoji

### 3.7 Voice: warm and plain

Everyday words; short sentences; no jargon or therapy vocabulary; warmth shown through accuracy, not effusiveness; the user's name used rarely.

### 3.8 Safety floor

Safety detection happens only inside a reflection or conversation turn the user has initiated. There is no background scanning.

- **Risk of self-harm or suicide indicated:** suspend the question and length rules; respond directly and warmly; say plainly that the agent cares about the user's safety; name the **9-8-8 Suicide Crisis Helpline (call or text 9-8-8)**. Calm, not alarmist.
- **Medical emergency described:** say to call **911**.
- Safety responses are always shown regardless of the save choice.

### 3.9 Context sent to Claude per reflection

Only:

- the current entry and its energy rating;
- the conversation so far (for replies);
- excerpts of the user's own words from past-7-day entries, _only_ when the §3.4 trigger is met.

Never sent: past agent reflections, summaries, designer notes, deleted entries.

### 3.10 Worked examples

**Light.** Entry: _"Lovely morning at the market. Came home with far too many tomatoes."_
Reflection: "A good morning out, and a tomato surplus to show for it."

**Moderate, with a light mention.** Entry: _"Couldn't sleep again. Kept going over that conversation from Tuesday."_

- ✓ "That conversation from Tuesday is still with you. It's hard to rest when something feels unfinished."
- ✗ "This is the third night this week you haven't slept." _(a count)_
- ✗ "You often seem troubled after talking to family." _(a pattern interpretation)_

**Heavy, with self-criticism and a self-posed question.** Entry: _"I snapped at someone who was only trying to help and I feel awful. I don't know why I do this."_
Reflection: "That lingering awful feeling says how much it matters to you to treat people well. Snapping when we're stretched thin is very human, even when we wish we hadn't. I wonder if something else was already weighing on you. If you want to come back to it: what was happening just before?"

**Request for advice in conversation.** User reply: _"What would you do?"_
Response: validates briefly first, offers one or two plain suggestions, and stops without a follow-up question.

---

## 4. Weekly summary

### 4.1 Timing

- Weeks run **Monday to Sunday**.
- Generated on the first unlock after a week closes; encrypted and stored with past summaries.

### 4.2 Input

- The user's own words from that week (entries and replies), energy ratings, and timestamps.
- Stored corrections (§4.5), as constraints.
- Never included: agent text, designer notes, deleted entries.

### 4.3 Structure (fixed order)

1. **The week in brief** — factual: number of entries and which days.
2. **What seemed to help** — activities, moments, or circumstances the user's words associate with things going better.
3. **Recurring feelings** — descriptive, not diagnostic; approximate language ("several entries", "mostly in the evenings"), not exact tallies.
4. **Rhythms of time and energy** — from timestamps and energy ratings.

Ends after section 4: no advice, no question, no closing encouragement.

### 4.4 Thresholds

- A feeling is "recurring" only if it appears in **at least 3 entries across at least 2 days**.
- If the week has **fewer than 4 entries**, show only "the week in brief" plus: _"There isn't enough this week to see patterns."_
- If energy ratings were skipped for most entries, the rhythms section says so rather than inferring energy from text.

### 4.5 Evidence and correction

- Every pattern links to the entries it came from.
- Every pattern has a **"Not quite right"** control; the user dictates or types a correction, which is encrypted and saved.
- Stored corrections are sent as constraints to future summary requests. The agent must not repeat a corrected framing.

### 4.6 Deleted entries

If an entry cited by a summary is later deleted, the pattern text remains, and the evidence link is replaced by: _"One entry this pattern drew on has been deleted."_

### 4.7 Prohibited in summaries

Advice; questions; clinical labels; causal claims ("X causes you to feel Y"); interpretation of relationships with named people; any pattern below threshold.

### 4.8 Example

> **Your week, 14–20 September**
> You wrote on five days, most often in the evening.
>
> **What seemed to help:** Time outdoors came up in two good-day entries — the market on Saturday and a walk on Wednesday. _[see entries]_
>
> **Recurring feelings:** Tiredness appeared in several entries, mostly written late in the day. _[see entries]_ · _Not quite right_
>
> **Rhythms:** Your energy ratings were higher on days you wrote in the morning. _[see entries]_ · _Not quite right_

---

## 5. Review screen

### 5.1 Entries

- Listed newest first, grouped by week (Monday–Sunday).
- Each item shows date and time, energy rating (if given), and the first line.
- Opening an entry shows the full text, plus the saved conversation if "whole conversation" was chosen.
- Entries are read-only (no editing after Reflect).

### 5.2 Deleting an entry

- One confirmation: _"Delete this entry permanently? Summaries that drew on it will be marked. A backup downloaded before today will still contain it."_
- Deletion removes the entry and its saved conversation permanently.
- Deleted entries are never used for light mentions or future summaries.

### 5.3 Search

- Runs entirely in the browser over decrypted content in memory; nothing is sent to the server.
- **[Proposed]** Searches the user's own words only.

### 5.4 Summaries

- Past weekly summaries listed by week; evidence links open source entries; corrections shown beneath the pattern they apply to.

### 5.5 Backup, restore and export

- **Download backup:** an encrypted file that requires the passphrase to restore.
- **Restore from backup** **[Proposed]**.
- **Readable export:** a plain Markdown file of entries, saved conversations, summaries, and corrections (designer notes excluded; they have their own export). Before download, a warning: _"This file is not protected. Anyone who opens it can read your journal."_
- **[Proposed]** A quiet in-app banner once a month: _"It's been a month since your last backup."_ (Not a notification.)

---

## 6. Designer notes

### 6.1 Separation

- A separate tab, distinguished by label and colour (never colour alone).
- Never read by any AI function; not included in search, summaries, or reflections.

### 6.2 Capture

- A **Note about the app** button on every reflection, conversation turn, and summary.
- Auto-attached context metadata: source screen (reflection / conversation / summary), date, entry weight, whether a light mention was used, whether a question was asked, whether the safety floor was triggered.
- **The entry's text is never attached.**
- Notes can also be written directly in the tab.
- Optional tags: _pattern surfacing, memory registers, emotion naming, detection and safety, voice and tone, friction._

### 6.3 Monthly review

- On the first unlock of each month, a banner offers the review.
- **Export notes** produces a readable Markdown file of that month's notes, containing no journal text, for review against the caregiver project's open questions.

---

## 7. Data model

### 7.1 Principle

The server stores only **record type, record ID, user ID, and creation time** in readable form. Everything else is one encrypted block per record. Creation times remain visible to the server by decision (simplicity); the server can see _when_ the user writes, never _what_.

### 7.2 Records

| Record          | Visible to server                                      | Encrypted contents                                                                                                          |
| --------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `key_params`    | user ID; salt, iteration count, algorithm (not secret) | passphrase check value                                                                                                      |
| `entry`         | ID, user ID, created time                              | text, energy rating, local timestamp, user replies, agent turns (whole-conversation only), save choice, reflection metadata |
| `summary`       | ID, user ID, week start                                | four sections; pattern items with evidence entry IDs; deleted-evidence markers                                              |
| `correction`    | ID, user ID, created time                              | week, pattern text, user's correction                                                                                       |
| `designer_note` | ID, user ID, created time                              | text, tags, context metadata                                                                                                |
| `settings`      | user ID                                                | last summary week, last backup date, last notes-review month, text-size setting                                             |

### 7.3 Reflection metadata

Each reflection response is structured output (§10.1). The app displays `reflection`; stores `weight`, `used_mention`, `asked_question`, `safety_triggered` in the encrypted entry, including under "just my words" (metadata contains no agent wording). Used for designer notes and for testing §3 rules.

---

## 8. Security and privacy architecture

### 8.1 What is and isn't protected

**Protected:** database breach; hosting provider reading content; lost or stolen device while locked.
**Not protected:** a compromised device while unlocked; entry text held by Anthropic during its retention window (§8.6); speech processing by Microsoft and Google (depends on voice-typing settings, §8.8); anyone who knows the passphrase. State these limits openly in the app's help text.

### 8.2 Encryption

- **Key derivation:** PBKDF2 with HMAC-SHA-256, **at least 600,000 iterations** (OWASP Password Storage guidance).
- **Salt:** random, at least 16 bytes (128 bits) per user (NIST SP 800-132).
- **Cipher:** AES-GCM, 256-bit key; fresh random 12-byte IV per encryption.
- **Key handling:** non-extractable Web Crypto key, held in memory only; never written to browser storage.
- **Passphrase check:** on unlock, decrypt the check value; failure means wrong passphrase.
- Web Crypto requires a secure context (HTTPS).

### 8.3 Auto-lock

After **30 minutes** of inactivity, and on sign-out: wipe the key and all decrypted content from memory; show the unlock screen.

### 8.4 Server function (AI relay)

- Holds the Claude API key (Edge Function secret); the key never reaches the browser.
- Accepts requests only from the authenticated user.
- Forwards to Claude and returns the response.
- **Never logs request or response content** — only status, timing, errors.
- **[Proposed]** Daily request cap (default 100 requests/day, configurable) to guard against runaway cost.

### 8.5 Minimum context

Only the context defined in §3.9 (reflections) and §4.2 (summaries) is sent.

### 8.6 Data handling at Anthropic [Verify at build]

As verified September 2026: API inputs and outputs are automatically deleted within 30 days, with exceptions including Usage Policy enforcement; data is not used for model training without express permission. Zero-data-retention is an organisational agreement and not assumed here.

### 8.7 Privacy during the build

- Real entries must never appear in Claude Code sessions, fixtures, logs, or screenshots.
- All testing uses synthetic entries (§11.2).
- The product owner should check their own Claude privacy settings for build sessions.

### 8.8 User-side checklist: voice-typing settings

**Windows voice typing (Windows key + H)**

- Choose not to contribute voice clips (set from within voice typing).
- Note: voice typing requires online speech recognition; turning it off disables voice typing.

**Gboard on Google Pixel (Pixel 6 or later)**

- Turn **Audio donations** off.
- Turn on faster / offline voice typing (on-device on Pixel 6 and later).
- **Personalize for you** may stay on (learns from the user's corrections for their own use).

**Fallbacks if accuracy or privacy disappoints (desktop):** Windows Voice Access (on-device) or Spokenly (third-party, local).

### 8.9 Account deletion

**[Proposed]** "Delete account and all data," with two confirmations, permanently removes every record.

---

## 9. Technology stack

### 9.1 Back-end plan

Start on the **Supabase free plan**; upgrade to Pro ($25 USD/month) only if pauses become a nuisance. Free-plan constraints [Verify at build]:

- Projects pause after one week of inactivity; data is preserved and the project is restored manually from the dashboard.
- No automatic backups — covered by the app's own backup export (§5.5).

### 9.2 Components

| Layer                | Choice                                                                                                             | Notes                                                                                                                    |
| -------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| App                  | Progressive web app (installable on the Pixel home screen), TypeScript, mainstream framework chosen by Claude Code | One codebase for phone and Windows desktop                                                                               |
| App hosting          | A free static-site host                                                                                            | [Verify at build]                                                                                                        |
| Sign-in and database | Supabase Auth and Postgres                                                                                         | Row-level security on every table (`auth.uid() = user_id`)                                                               |
| AI relay             | Supabase Edge Function                                                                                             | Claude API key stored as an Edge Function secret. Supabase secret keys bypass RLS and must never be used in the browser. |
| Encryption           | Web Crypto API                                                                                                     | §8.2                                                                                                                     |
| AI model             | `claude-sonnet-5`                                                                                                  | Set thinking to the minimum for reflections to limit cost and latency [Verify at build]                                  |

### 9.3 Estimated running cost

Sonnet 5: $2 per million input tokens, $10 per million output tokens (Anthropic, September 2026). Assuming one ~200-word entry a day, two replies, and a weekly summary: **about $1 USD/month**; heavier use **under about $4/month**. Supabase free plan and static hosting: $0. Estimates, not measurements.

---

## 10. Prompts

### 10.1 Reflection prompt (system)

```
You are a reflective listener inside a private journal belonging to one adult.
Respond to their entry with a brief reflection. You are not a therapist, coach,
or advisor.

ORDER: Always acknowledge and validate first.

WHAT TO VALIDATE: That feelings are understandable. Never endorse a harsh
self-judgement, and never argue with it.

VALIDATION LEVELS:
- Always: show you took in what was said; reflect its core in fresh words.
- Moderate and heavy entries: note the reaction makes sense given the
  circumstances.
- Heavy entries only: you may name an implied feeling, tentatively
  ("I wonder if...").

LENGTH: Match the entry. Light: 1-2 sentences. Moderate: 2-3. Heavy: a short
paragraph. Never write more words than the entry contains.

QUESTIONS: Rarely. End on acknowledgement by default. Ask one open question only
if the user asked for input or ended with an open question to themselves, and
phrase it so it can be ignored.

PAST ENTRIES: You may receive excerpts from the past 7 days. Refer to one only if
the current entry points to it or clearly continues it. At most one reference,
framed as continuity. Never count, never describe trends, never interpret
patterns.

NEVER: give unrequested advice; reassure against a feeling; interpret with
certainty; praise effusively; use clinical labels; use exclamation marks or
emoji; open with "It sounds like" habitually.

VOICE: Warm and plain. Everyday words, short sentences, no therapy vocabulary.

CONVERSATION: If the user replies, keep the same order. Give advice only if
asked, plainly and briefly. Never prolong the conversation.

SAFETY: If the entry suggests risk of self-harm or suicide, set the length and
question rules aside. Respond directly and warmly, say you care about their
safety, and name the 9-8-8 Suicide Crisis Helpline (call or text 9-8-8). If it
describes a medical emergency, say to call 911.

OUTPUT: Only JSON:
{"reflection": string,
 "weight": "light" | "moderate" | "heavy",
 "used_mention": boolean,
 "asked_question": boolean,
 "safety_triggered": boolean}
```

### 10.2 Summary prompt (system)

```
You write a weekly summary of one person's private journal, using only their own
words. It contains no text written by an AI.

SECTIONS, in this order:
1. The week in brief: how many entries, and on which days.
2. What seemed to help.
3. Recurring feelings.
4. Rhythms of time and energy.

RULES:
- A feeling counts as recurring only if it appears in at least 3 entries across
  at least 2 days.
- If there are fewer than 4 entries, write only section 1 and the sentence:
  "There isn't enough this week to see patterns."
- Use approximate language ("several", "mostly"), not exact tallies.
- Every pattern must list the IDs of the entries it came from.
- If energy ratings are missing for most entries, say so. Do not infer energy
  from the text.
- Obey every correction provided. Never repeat a framing the user has corrected.

NEVER INCLUDE: advice, questions, clinical labels, causal claims, interpretations
of relationships with named people, or closing encouragement.

OUTPUT: Only JSON:
{"week_in_brief": string,
 "helped": [{"text": string, "evidence": [entry_id]}],
 "feelings": [{"text": string, "evidence": [entry_id]}],
 "rhythms": [{"text": string, "evidence": [entry_id]}],
 "insufficient_data": boolean}
```

---

## 11. Acceptance criteria and synthetic test set

### 11.1 Acceptance criteria

**Reflections (§3)**

1. No reflection exceeds the entry's word count.
2. Across the 20-entry test set, ≤ 4 reflections contain a question.
3. No reflection contains advice unless the entry asks for it.
4. "It sounds like" opens ≤ 1 in 5 reflections; no exclamation marks or emoji.
5. Light mentions appear only for entries designed to trigger them, and never contain counts or trends.
6. Safety test entries always produce 9-8-8 or 911 wording.
7. Malformed AI output triggers one retry, then the §2.6 message.

**Summaries (§4)** 8. A test week with 3 entries produces the "not enough" summary. 9. A corrected framing does not reappear the following week. 10. Every pattern has evidence links; no pattern below threshold appears. 11. A week with mostly skipped energy ratings says so and infers nothing.

**Data and privacy (§§5, 7, 8)** 12. The database contains no readable text (verified by inspecting stored records). 13. The app locks after 30 minutes' inactivity; no decrypted content remains in memory. 14. An entry cannot be edited after Reflect. 15. A deleted entry is marked in the summaries that cited it and never appears in later context. 16. Relay logs contain no entry text. 17. Designer notes are never included in any AI request. 18. Encrypted backup restores correctly with the passphrase and fails without it. 19. Readable export shows the warning before download.

**Readability (§12)** 20. All interactive controls measure at least 48 × 48 CSS pixels on phone and desktop layouts. 21. Body text contrast is at least 7:1 in light and dark modes. 22. Layout works at 200% zoom without horizontal scrolling.

### 11.2 Synthetic test set (written for testing; never real entries)

**Entries (20):** 4 light; 4 moderate; 4 heavy (2 with self-criticism); 2 requests for advice; 2 referring to a specific earlier entry; 1 that tempts a pattern observation without referring back; 1 very short ("Tired."); 1 long and rambling with dictation errors; 1 self-harm safety test; 1 medical-emergency test.

**Weeks (5):** a sparse week (3 entries); a full week; a week with energy ratings mostly skipped; a week following a correction; a week containing a deleted entry.

---

## 12. Readability and interaction

Aim for WCAG 2.2 Level AAA where practical.

1. **Contrast:** body text at least 7:1 against its background; no light-grey text; no text over images.
2. **Text size:** **[Proposed]** default body text 18 px, in relative units so the Pixel's font-size setting and browser zoom are respected; a setting offers standard, large, and extra large.
3. **Layout:** lines no wider than 80 characters; left-aligned (not justified); line spacing 1.5; works at 200% zoom without horizontal scrolling.
4. **Controls:** every button at least **48 × 48 CSS pixels**, with at least 8 px of clear space between adjacent controls (Reflect, energy buttons, Reply, Done, "Not quite right", etc.). Exceeds the WCAG 2.2 AAA target of 44 × 44.
5. **Energy rating:** three large buttons labelled in words (Low, Medium, High), never colour or icon alone.
6. **Entry text box:** large, comfortable padding, clearly visible cursor.
7. **Destructive actions** (delete entry, readable export, delete account): plain-language dialogs; confirm button visually distinct from and separated from cancel.
8. **Designer notes tab:** distinguished by label and colour, never colour alone.
9. **Light and dark mode:** **[Proposed]** follows the device setting; both meet the contrast rule.
10. **Language:** all app wording in the same warm, plain voice as reflections.

---

## 13. Open questions and out of scope

### 13.1 Items to verify at build

- Static-site host choice
- Current Supabase free-plan terms and feature details
- Current Anthropic API data-handling terms and Sonnet 5 thinking settings

### 13.2 Out of scope for version 1

- Adding a note to a locked entry later ("addendum")
- Monthly or longer-range patterns
- Any audio features
- Any sharing
- Name masking

### 13.3 Links to the caregiver project (for monthly review)

The designer notes feed these caregiver-project questions: how the longitudinal emotional record should surface patterns; the dual-register memory model; how far emotion naming should extend toward clinical territory; in-session-only detection as a resolution of the passive-detection tension. Lessons are treated as **hypotheses** for the caregiver design, never defaults.

---

## 14. Suggested build sequence for Claude Code

Confirm each stage with the product owner before starting the next.

1. Project scaffolding, PWA shell, Supabase project, Auth, RLS on all tables
2. Encryption module (passphrase setup, unlock, check value, auto-lock)
3. New-entry flow (text box, energy rating, lock on Reflect, save)
4. AI relay Edge Function (secret key, auth check, no content logging, daily cap)
5. Reflection (prompt §10.1, structured output, retry, display)
6. Conversation, Done, and save prompt with default
7. Review screen, search, deletion with summary marking
8. Weekly summary (prompt §10.2, thresholds, evidence links, corrections)
9. Designer notes and monthly export
10. Backup, restore, readable export, backup banner
11. Readability pass (§12)
12. Full acceptance testing against §11 using the synthetic test set only

---

## 15. References

- Linehan, M. M. — levels of validation (Dialectical Behavior Therapy).
- Shenk, C. E., & Fruzzetti, A. E. (2011). The impact of validating and invalidating responses on emotional reactivity. _Journal of Social and Clinical Psychology._
- Sharma, M., et al. (2023). Towards understanding sycophancy in language models. Anthropic.
- Neustaedter, C., & Sengers, P. (2012). Autobiographical design in HCI research. _DIS 2012._
- Desjardins, A., & Ball, A. (2018). Revealing tensions in autobiographical design in HCI. _DIS 2018._
- OWASP Password Storage Cheat Sheet (PBKDF2-HMAC-SHA256, 600,000 iterations).
- NIST SP 800-132 (salt length ≥ 128 bits).
- W3C WCAG 2.2 (SC 1.4.6, 1.4.8, 1.4.4, 2.5.5, 2.5.8).
- Anthropic Privacy Center and Claude Platform documentation (API data retention), accessed September 2026.
- Anthropic, "Introducing Claude Sonnet 5" (pricing update 10 August 2026).
- Supabase documentation and pricing (Edge Function secrets; free-plan limits), accessed September 2026.
- Microsoft Support (Windows voice typing; online speech recognition); Google Gboard Help (voice typing settings), accessed September 2026.
