# Case: proposing alternatives without noticing the subject was inverted

## Situation

An Android app UI that shows, for each component of a food package read by the camera (ingredients, additives, packaging material), "who supplies it". An outside designer had a proposal (hereafter "their design"), and we built an improved proposal on top of it (hereafter "our design") and presented it. After the presentation, the client-side lead sent it back, saying it "does not follow their design".

## Object model of their design (written down afterwards)

| Noun | Kind | Screens where it appears |
|---|---|---|
| Component (ingredient / additive / packaging material) | Object | List (the unit of the collection) |
| Business | Object | List (as an attribute of the component), detail (single) |
| Certification (by a third party) | Object | Detail |
| Product | Object (a separate layer) | Top of the list |
| Manufacturer / supplier | Relationship | List (heading labels) |
| Origin, type, logo | Attribute | List |

The subject of the list is the component. Headings are component types, with business cards under them. The list is cut so that its structure matches the structure of the package labeling.

## What happened in our design

- We made the business the subject of the list and turned the component type into a chip (an attribute) inside the card. The subject was the reverse of their design.
- This cut came from the original container (a home-screen widget), where each card had to be readable on its own. We carried it unchanged into a derivative that moved the container to a full-screen bottom sheet. We did not revisit a constraint-driven cut after the constraint was lifted.
- Before asking about intent, we replaced their structural component (expandable list items that show one level deeper) with a different structure (navigation to a detail screen).
- In presentation order, we put the design furthest from theirs first as "the current proposal", and lined up the ones closer to theirs behind it as derivatives.

## Why they read it as "not following their design"

The three cores of their design — sectioning with the component as the subject, expandable list items, and not showing highlights over the camera view by default — were all absent from our main proposal. Each change had its reason, but when all three are missing, people judge "it doesn't follow our design" before they read the reasons.

## Had this skill been used

Before making changes, writing down that the subject of their design is "the component" would have shown that making "the business" our subject is a subject change. The confirmation questions would have been:

1. I understood the subject of the list to be the component (ingredient / additive / packaging material), with the business shown as its attribute. Please tell me if that's wrong.
2. I understood the expandable list items to be a component for showing one level deeper without navigating, not meant to be replaced by navigation to a detail screen. Please tell me if that's wrong.
3. I understood the product (brand) to be treated as a layer separate from the components. Please tell me if that's wrong.

With answers to these three, our changes would have narrowed to "presentation changes" (origin flags to text, removing the certification badge, tidying symbols), and we could have presented them with the subject left as it was.

## Lessons

- A cut that came from a constraint is up for review once the constraint is lifted.
- Replacing the designer's structural component is a subject change. Do not treat it like a presentation change.
- In presentation order, put the proposal closest to theirs first, and talk in diffs.
