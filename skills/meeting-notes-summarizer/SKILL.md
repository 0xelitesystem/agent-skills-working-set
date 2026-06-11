---
name: meeting-notes-summarizer
description: Use when the user provides a meeting transcript, recording transcript, or raw meeting notes and wants a structured summary. Triggers on "summarize this meeting", "extract action items from this transcript", "what did we decide in this call", "TL;DR this Zoom recording", or when the user pastes a transcript longer than ~500 words from a meeting context.
---

# Meeting Notes Summarizer

Turn raw transcript into a structured, scannable summary.

## Process

1. **Read the entire input first.** Don't summarize as you go. Meeting context shifts; early statements get reversed later.

2. **Output sections in this order:**

   ### TL;DR
   
   2-4 sentences. The headline of the meeting. A reader who only reads this should know what the meeting was about and whether it concerns them.

   ### Decisions
   
   Bulleted. Each decision: what was decided, in past-tense, action voice. No hedging.
   - "Decided to ship v2 on Friday." OK
   - "Discussed shipping v2." FAIL (a discussion isn't a decision)
   
   If something was raised but not decided, put it in "Open questions" instead.

   ### Action items
   
   Bulleted. Each: WHO, WHAT, by WHEN.
   - "Alice will publish the API spec by EOD Wednesday."
   - "Bob to follow up with legal on the contract; no deadline given."
   
   If the owner wasn't named, write "[unowned]" so it's obvious. If no deadline was set, omit the deadline rather than fabricating one.

   ### Open questions
   
   Bulleted. Things raised that didn't get resolved, with whatever context exists about who's blocking or what's missing.

   ### Discussed (skim-friendly recap)
   
   Brief paragraphs by topic. Use this for stakeholders who want context but didn't attend.

3. **Tone:** factual, neutral, no editorializing. You weren't there. Don't add sentiment ("the team was excited about...") unless the transcript explicitly says it.

4. **Speaker attribution:** include speakers' names when they own a decision or action item. Drop names for general discussion to keep it readable.

## What NOT to do

- **MUST NOT** invent action items. If nobody committed to doing something, don't list it as an action item.
- **MUST NOT** invent deadlines. If the discussion was vague about timing, the action item is vague about timing.
- **MUST NOT** include filler: "we then discussed", "the meeting started with", "moving on to the next topic". Strip transcription noise.
- **MUST NOT** include personal small talk, off-topic asides, or interruptions. Skip them silently.
- **MUST NOT** quote large chunks verbatim. Summarize. Quote only when a specific phrasing matters (e.g. exact commitment language for a contract conversation).

## Privacy considerations

- Mention names that appear in the transcript only in their roles (decisions, action items). Don't add titles, departments, or other identifying info that wasn't said.
- If the transcript clearly contains sensitive info (compensation discussions, performance reviews of named individuals, legal disputes), flag this at the top: "This transcript contains sensitive content. Confirm you want a summary written down before I produce one." Wait for confirmation.

## Edge cases

- **Transcript with low-quality auto-transcription**: if you see frequent garbled phrases ("the [unintelligible] is going to [unintelligible]"), note that the summary may have gaps and suggest the user spot-check.
- **Transcript with multiple meetings concatenated**: ask whether to summarize each separately or combine.
- **Very short transcript (<200 words)**: the structured format may be overkill. Ask whether they want full structure or just a 1-paragraph summary.
- **Transcript in a language other than English**: confirm what language to write the summary in. Default to the transcript's language.

## Length calibration

- 30-min meeting transcript → ~200-400 word summary
- 1-hour meeting transcript → ~400-700 word summary
- 2+ hours → consider asking if a tiered summary (exec / detailed) is wanted
