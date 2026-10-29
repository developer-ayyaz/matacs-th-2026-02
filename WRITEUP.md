# Questions to answer

1. AI vs. rules. Which steps in your flow use AI, and which are plain code or rules? Why did you
draw the line there?

**AI is used for two things only:**
- Extracting facts from proposal text into a fixed structure, with a short evidence quote for each fact.
- Writing a plain-language explanation of a result after the numbers are already computed.

**Plain code or rules handle everything else:** the mandatory gates, converting values to points, applying weights, ranking, tie-breaks, and assigning the status (ranked, flagged, ineligible).

**Why the line is here:** Language is messy, so AI helps there. Awarding money should be repeatable and checkable, and models are weak at consistent arithmetic.

---

2. Consistency and explainability. How do you make scores repeatable and explainable? If the same proposals were ranked twice, would the result be the same? How would a reviewer see why a proposal ranked where it did?

- Scores come from code applied to extracted facts, so the same facts always give the same score.
- Extraction runs at temperature 0 with a versioned prompt. The result is cached by file hash, prompt version and criteria version. Ranking the same proposals twice reads the stored extraction, so the result is identical.
- If a proposal is re-extracted fresh, wording could shift slightly. To stop that affecting the result, code checks that every evidence quote really appears in the source text, and any mismatch is flagged.
- The explanation text is generated once from the fixed numbers and stored, so it does not change between views.
- A reviewer sees a per-criterion table for each proposal: extracted value, evidence quote, points earned, weight, and the final score. Missing information is shown as "not stated", never guessed.

---

3. Human in the loop. Where does a person review, confirm or override the result? What gets logged for audit?

The reviewer uploads, selects a category and gets the ranked list back in one pass. People step in after that, at three points:

1. **Result review:** the reviewer reads the ranking and the per-criterion table instead of rereading every proposal. Flagged and ineligible proposals are listed separately and need a decision before the ranking is final. If an extracted value looks wrong next to its evidence quote, the reviewer can correct it, and only the rules rerun.
2. **Override:** the reviewer can change any rank or status, with a required written reason.
3. **Award decision:** always made by a person, never by the tool.

Problems found during processing (missing facts, injection attempts, ambiguous dates) never stop the run. They become flags in the output, so the reviewer sees them at the result review.

**Audit log records:** who did what and when, file hash, prompt version, model name and settings, criteria version, raw extraction output, computed scores, flags raised, every correction and override with its reason, and the final decision.

---

4. Changing criteria. What happens if an admin edits the criteria or weights after some proposals have already been scored?

- Criteria are versioned and never edited in place. An edit creates a new version, and weights are validated to total 100% on save.
- Every scoring run records which version it used, so old results stay accurate and labeled.
- The tool never mixes versions in one ranking. Stale results show "scored under v2, current is v3" with a re-score button.
- Re-scoring is cheap because extraction is cached and only the rules rerun. If a new criterion needs a fact that was never extracted, only that field is re-extracted. Re-scoring is logged, and the old ranking is kept for comparison.

---

5. Risks. What could go wrong with this tool (for example wrong extraction, made-up facts, bias, manipulation, cost), and how does your design guard against each one?

- **Wrong extraction:** every value needs an evidence quote that code checks against the source. Unclear values return "not stated" and are shown as flags in the ranked output.
- **Made-up facts:** the prompt forbids inference, and values without quotes are rejected. Scanned PDFs with poor text quality are flagged.
- **Bias:** the AI never ranks and never sees names for scoring decisions beyond what extraction needs. Scores use only admin criteria. Company size, brand and wording style are not criteria.
- **Manipulation (prompt injection):** proposal text is treated as untrusted data and never as instructions. The extraction prompt reports any text that tries to instruct a reviewer or system. Code then flags that proposal and ignores the instruction. Proposal C is the example.
- **Missing or vague information:** a proposal that does not state a required fact is flagged, not guessed, as with D's missing schedule.
- **Weak mandatory evidence:** a licence promised after award does not count as held, so B is ineligible.
- **Cost:** one extraction call per proposal, cached, and no scoring calls. Re-scoring reuses cached extractions. Page and file size limits apply.
- **Privacy:** only upload data the organization is allowed to send to the model provider, and keep audit logs access-controlled.
- **Overtrust:** the tool is labeled as decision support, and the ranking always shows the evidence.

## How the sample set is handled

- **B:** ineligible. The licence is held by an unnamed subcontractor to be confirmed later, so the mandatory gate fails despite the lowest price.
- **C:** flagged. It contains an instruction aimed at automated systems. Ignored, and scored only on its real content (provisional 53.7), but a person must review it.
- **D:** flagged. No completion time is stated, so the 16-week limit cannot be verified. Provisional score is 59.9 with timeline at zero.
- **A and E:** both eligible and fully verifiable. A ranks first on experience, price and timeline. E has the best warranty and monitoring but fewer installs and a higher price.

---