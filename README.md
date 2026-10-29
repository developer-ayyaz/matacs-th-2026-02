# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** Fahad Ayyaz
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** 2h

## Files

- [WRITEUP.md](./WRITEUP.md): answers to the five questions
- [PROMPTS.md](./PROMPTS.md): extraction prompt and explanation prompt, as I would send them
- [ranking.json](./ranking.json): expected ranking for proposals A to E
- [diagram/](./diagram/): process flow from upload to ranked output

## How I used AI tools

I used Claude to draft the design, prompts and write-up, then read everything myself and edited the parts I thought were wrong or unclear after discussing those parts with claude. I disagreed with a suggestion: Claude added a human extraction review step in the middle of the pipeline. I removed it because the brief describes a single pass from upload to ranked list, so extraction problems now appear as flags in the output and the reviewer corrects them at the final review.
