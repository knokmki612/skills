---
name: ui-object-model
description: Before changing, porting, or comparing a UI design (Figma, wireframes, specs, existing screens), extract its object model (subjects, attributes, relationships, actions, views) onto one page, split the changes you plan into "subject changes" and "presentation changes", and write confirmation questions for the designer. Use for requests that start from an existing design — "apply", "port", "propose improvements", "compare", "alternatives", "based on this design". Also use right after a hearing with the designer, and when you decide to replace one of the designer's structural components (accordion, tabs, expandable list items, etc.) with a different structure. Do not use when designing from scratch.
license: CC-BY-4.0
---

# UI Object Model Analysis

## Purpose

Before changing, porting, or comparing a UI design, write down what that design treats as its subject (the object), and what it treats as attributes, relationships, and actions — then confirm it. This skill does not create designs; it is the preprocessing step before you touch one.

If you keep stacking changes while misreading the subject, each change may have its own reason, yet the designer will read the result as "not following our design". Writing the subject down first shows which of your changes alter the subject, and narrows what you need to confirm to a few questions.

## When to use

- **Requests that start from an existing design**: "apply", "port", "propose improvements", "compare", "alternatives", "based on this design".
- **Right after a hearing with the designer**: while the memory is fresh, write down whether what you heard bears on subjects, attributes, or relationships. Use the hearing notes as the "material showing the designer's intent" in step 1.
- **When you decide to replace one of the designer's structural components**: turning an accordion into navigation to a detail screen, tabs into sections, and so on. Stop at the moment you decide and use this skill. Replacing a structural component is a subject change, so classify it as one in step 5 and turn it into a question in step 6.

Do not use when designing from scratch.

## Procedure

1. **Fix the inputs.** Identify the target design (the list of screens) and any material that shows the designer's intent (comments, hearing notes). If there is none, state "intent unconfirmed" and continue. If you need to read Figma, use the caller's Figma tools.
2. **Extract the objects.** List the nouns that appear on screen and sort them into four kinds:
   - **Object**: something the user can point at as "this" (a piece of content, an organization, a verification)
   - **Attribute**: a value attached to an object (country, type, logo)
   - **Relationship**: a link between objects (author, publisher, the organization that verified it)
   - **View**: whether it is shown as a collection or as a single item
3. **Decide the subject.** For each screen, answer "what is the unit of the collection?", "whose name is the heading?", and "what does each component represent?", and settle on one subject. Record a screen whose subject cannot be settled as "mixed".
4. **Find constraint-driven choices.** Identify where the subject or the layout comes from constraints of the display target (layout, viewport, device), and mark them "revisit when the constraint is lifted".
5. **Classify your planned changes.** Split the changes you intend to make into:
   - **Subject changes**: changing the subject, turning a relationship into an attribute, replacing the designer's structural component with a different structure
   - **Presentation changes**: symbols, color, wording, order

   Confirm subject changes with the designer before proceeding. Proceed with presentation changes, stating your reasons.
6. **Write the confirmation questions.** Ask only about what bears on the subject. Use the forms "I understood the subject of this screen to be X. Please tell me if that's wrong." and "I understood X to be treated as an attribute of Y. Please tell me if that's wrong.", and keep it to three questions or fewer. Do not ask about presentation changes.
7. **Output.** Put everything on one page using the template in [references/template.md](references/template.md).

## Guardrails

- Do not put a change of subject into a deliverable before it is confirmed.
- This is not the place to list the design's weaknesses. Under "what remains", write only what serves as a reason for a change.
- Write questions as negative confirmations ("Please tell me if that's wrong.").
- One page of output, three questions or fewer. Do not turn it into a long document.

## References

- [references/template.md](references/template.md) — output template
- [references/example.md](references/example.md) — a case: the subject inverted, a constraint-driven cut carried over, a structural component replaced, and example confirmation questions
