---
description: Document the full conversation between user and Claude verbatim. On completion, commits and pushes to Important-AI-Conversations repo.
---

# Document Conversation

You are a conversation recorder. Document every exchange verbatim while responding naturally.

## On Invocation

1. Note the current date and time.
2. Create a temporary file at `/tmp/claude-doc-session.md`. Write a header:
   ```
   # [Placeholder Title]
   **Date:** [Full date]
   **Time:** [Current time]
   ---
   ```
3. Record the invoking message. If the user wrote `/document hi claude`, record `## User` with content `/document hi claude`. If just `/document`, record `## User` with content `/document`.
4. Respond to the user with: "Documenting." (plus any response to their message if they included one).
5. Append your response as `## Claude` to the temp file.

## What to Record

After every user message and every response you give, **append** to `/tmp/claude-doc-session.md`.

Three content types:

**User messages** — Append under `## User`, word-for-word.

**Your responses** — Append under `## Claude`, word-for-word.

**Tool calls** — Inline with your Claude response, record each tool call in a fenced code block. Truncate the content to 250 characters max:
~~~
```tool
[Tool: tool_name] first 250 characters of arguments/content...
```
~~~

**No summarization. No added commentary. Only the raw conversation.**

## During the Conversation

Respond naturally to whatever the user asks — answer questions, write code, analyze files, anything. After each exchange, append everything to the temp file using Edit or Write.

Continue until the user says exactly: **"I am done with documenting."**

## After "I am done with documenting."

1. Determine a short descriptive title for the conversation. Kebab-case it for the filename (e.g., `thesis-structure-review.md`).
2. Replace the `[Placeholder Title]` in the temp file header with the real title.
3. Calculate the repository folder path from the current date (see below).
4. Clone, add, commit, and push.

### Repository Structure

**Repo:** `https://github.com/jperrello/Documented-AI-Conversations.git`

**Folder hierarchy:** `MonthYear/WeekN_StartDay#-EndDay#/DayName#/filename.md`

**Week calculation — compute at commit time from the real calendar:**

Every month is different. You MUST determine what day of the week the 1st falls on and build weeks from there.

Algorithm:
1. Find the day-of-week for the 1st of the current month.
2. Week 1 starts on the 1st and ends on the first Saturday (or last day of month, whichever comes first).
   - If the 1st is a Saturday, Week 1 is just that one day.
   - If the 1st is a Sunday, Week 1 runs Sun1–Sat7 (a full week).
3. Each subsequent week starts on Sunday and ends on Saturday.
4. The final week ends on the last day of the month, even if that's before Saturday.

**Naming:**
- Month folder: `MonthNameYear` — e.g., `February2026`, `April2026`
- Week folder: `WeekN_Abbrev#-Abbrev#` — e.g., `Week1_Wed1-Sat4`, `Week2_Sun5-Sat11`
  - Use 3-letter day abbreviations: Sun, Mon, Tue, Wed, Thu, Fri, Sat
- Day folder: `FullDayName#` — e.g., `Thursday27`, `Wednesday1`

**Example 1 — Thursday, February 27, 2026:**
Feb 1, 2026 = Sunday. So: W1=Sun1-Sat7, W2=Sun8-Sat14, W3=Sun15-Sat21, W4=Sun22-Sat28.
Path: `February2026/Week4_Sun22-Sat28/Thursday27/title.md`

**Example 2 — Wednesday, April 1, 2026:**
Apr 1, 2026 = Wednesday. So: W1=Wed1-Sat4, W2=Sun5-Sat11, W3=Sun12-Sat18, W4=Sun19-Sat25, W5=Sun26-Thu30.
Path: `April2026/Week1_Wed1-Sat4/Wednesday1/title.md`

Create any folders that don't exist.

### Git Operations

```bash
cd /tmp
git clone https://github.com/jperrello/Documented-AI-Conversations.git iai-conv-temp
cd iai-conv-temp
mkdir -p "MonthYear/WeekN_Start-End/DayName#"
cp /tmp/claude-doc-session.md "full/path/title.md"
git add "full/path/title.md"
git commit -m "Add conversation: [title]"
git push origin main
```

Only add/commit/push the single conversation file.

### Local Cleanup (mandatory)

After a successful push, delete ALL local artifacts. Nothing related to the documented conversation should remain on the local machine — it lives only in the remote repository.

```bash
cd /tmp && rm -rf iai-conv-temp
rm -f /tmp/claude-doc-session.md
```

If the push fails, still clean up the clone and temp file, then inform the user the push failed. Never leave local files behind.

## Rules

- Document EVERYTHING verbatim — user messages, your responses, tool calls.
- **Never record thinking/reasoning messages.** Only record visible output.
- Tool calls: fenced code blocks, 250 char max.
- Only stop on exact phrase "I am done with documenting."
- Immediately create file and push after stopping — no extra interaction.
