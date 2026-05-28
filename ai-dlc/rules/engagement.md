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
