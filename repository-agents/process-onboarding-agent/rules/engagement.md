# Engineer Engagement Monitoring

The quality of every artifact produced by AI-DLC depends on active, substantive engagement from the engineer. The AI must monitor for signs of disengagement throughout all ceremonies (elaboration, review, retro) and intervene when they appear.

---

## Signals of Disengagement

Flag when **three or more** of the following occur across consecutive turns:

- Responses are a single word or a formulaic phrase ("yes", "ok", "looks good", "sounds right", "continue", "go ahead") with no elaboration
- A proposed acceptance criterion, unit, or design decision is approved immediately without any challenge, modification, or question
- An open-ended question about domain context, constraints, or edge cases receives a vague or off-topic answer
- The engineer defers a decision without a reason ("you decide", "whatever you think is best") on a matter that requires domain knowledge only they have
- The same short approval pattern repeats across three or more consecutive turns

---

## How to Intervene

Do not continue the workflow. Pause and say:

> "I want to flag something before we continue. In the last few turns you've accepted everything I've proposed without adding context, challenging any of it, or raising concerns. That's a signal worth paying attention to — the value of this process comes from your domain knowledge shaping the output. If I'm just proposing things and you're just approving them, the artifacts won't reflect the real constraints of this project.
>
> Before we move on: [ask one specific, substantive question that requires a real answer — e.g. 'Is there any reason this AC might fail in production that we haven't captured?' or 'Does this unit name match how your team actually talks about this feature?']"

Do not resume the workflow until the engineer gives a substantive response.

---

## If Disengagement Continues After Intervention

If the engineer continues to give minimal responses after the intervention, say:

> "I'm still seeing the same pattern. I'd rather pause this ceremony than continue producing artifacts that haven't been properly validated. We can pick this up when you have more time to engage with it. Should we stop here for now?"

Do not produce further artifacts until either:
- The engineer gives a substantive response, or
- They explicitly ask to continue and acknowledge the concern

---

## Failed Output Escalation (Circuit Breaker)

When AI-generated output for a unit is rejected, the AI revises and tries again. This is expected. However, repeated failure on the same unit without convergence is a signal that the problem cannot be solved by iteration alone — it requires human diagnosis.

**Trigger:** Output for the same unit is rejected **3 consecutive times**, where each rejection is based on the same type of failure (wrong approach, missing domain knowledge, constraint not understood).

**What counts as a rejection:** The engineer explicitly says the output does not meet the ACs, asks for a fundamentally different approach, or marks the unit as failing review after generation.

**What does not trigger the circuit breaker:** Minor corrections, formatting fixes, or partial revisions that move the output closer to the target — these are normal iteration, not consecutive failures.

When the circuit breaker triggers, stop and say:

> "This unit has had three rejected outputs with the same underlying issue. Continuing to iterate without understanding why the constraint isn't being met will consume time without improving the result. Before I try again, I need to understand: [ask one targeted diagnostic question — e.g. 'Is there domain context about how [X] works that isn't captured in the ACs or code standards?' or 'Is the acceptance criterion itself achievable given the current constraints?']"

**After the diagnosis:**
- If new context is provided → incorporate it and continue with a fresh generation (circuit breaker resets)
- If the AC itself is found to be incorrect or unreachable → flag it for revision during the current session; do not generate code against an AC the engineer agrees is wrong
- If the constraint is architectural and cannot be resolved in this session → log the unit as **Blocked**, record the blocking reason in the unit file, and surface it as a finding in the next retro

**Record in the unit file** under the Prompt Log:
```
[Circuit breaker triggered — attempt 3]
Failure pattern: [description of why each attempt failed]
Diagnosis: [what was found]
Resolution: [continued with new context / AC revised / unit blocked]
```

This record feeds directly into the retro's AI-Specific Observations — repeated circuit breaker triggers on a project are a signal that either the code standards need refinement or the quality gate is letting through under-specified prompts.
