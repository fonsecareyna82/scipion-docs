# Tags

Tags help you organize protocols inside a ScipionWeb project using short, visual labels. They are useful when a workflow grows, when several users collaborate on the same project, or when you want to mark protocol groups by purpose, review status, data type, or priority.

Use this page when you need to create, manage, assign, or review tags in a project.

<img src="../../assets/images/screenshots/user-guide/tag_manager.png" alt="Tag manager" style="display:block;width:100%;max-width:980px;height:auto;margin:1.2rem auto 1.6rem;" />

*Tag manager used to review and maintain the available tags for a project.*

---

## What tags are for

Tags are project-level labels that can be attached to protocols. A tag usually combines:

- a short title
- an optional description
- a color that makes the label easy to recognize

They are intended to make large workflows easier to scan without changing the protocol execution itself.

Good tag examples include:

- `preprocessing`
- `review needed`
- `particles`
- `tomo`
- `final candidate`
- `deprecated`

---

## Typical tag workflow

A common workflow looks like this:

1. open the relevant project
2. open the tag management area
3. create the tags your project needs
4. assign tags to protocols from the workflow view
5. use the visible labels to understand the project structure faster
6. update or remove tags when the project convention changes

---

## Open the tag manager

The tag manager is where the available tags for a project are maintained.

Depending on the ScipionWeb deployment and your permissions, you may find it from the project-related settings area or from a tag management action in the interface.

Before editing tags, confirm:

- you are working in the intended project
- the tag convention is shared with collaborators if the project is shared
- the tag name is clear enough to be understood later
- the color is visually distinct from existing tags

!!! note "Tags organize the workflow"
    Tags help users read and group protocols visually. They do not replace protocol parameters, execution status, outputs, or project permissions.

---

## Create a new tag

1. Open the tag manager.
2. Select the action to create a new tag.
3. Enter a short title.
4. Add a description if the meaning may not be obvious to other users.
5. Choose a color.
6. Save the tag.
7. Confirm that the new tag appears in the tag list.

<img src="../../assets/images/screenshots/user-guide/new_tag.png" alt="New tag dialog" style="display:block;width:100%;max-width:980px;height:auto;margin:1.2rem auto 1.6rem;" />

*New tag dialog used to define the title, description, and color of a tag.*

A good tag title should be short enough to fit comfortably on protocol cards while still being meaningful.

Recommended naming habits:

- use clear names instead of abbreviations only one person understands
- avoid creating several tags that mean nearly the same thing
- reserve colors for meaning, not decoration
- keep descriptions focused on when the tag should be used

---

## Edit or delete existing tags

Use editing when the tag is still useful but its name, description, or color needs improvement.

Use deletion only when the tag is no longer part of the project convention.

Before deleting a tag, check whether it is already assigned to protocols. Removing a tag can make the workflow harder to interpret if collaborators were using it as part of their review process.

---

## Assign tags to protocols

Tags become useful when they are assigned to protocol cards in the project workflow.

A typical assignment flow is:

1. open the project workflow view
2. locate the protocol or protocols you want to classify
3. open the protocol action menu or tag assignment area
4. select one or more tags
5. confirm that the selected tags appear on the protocol card

<img src="../../assets/images/screenshots/user-guide/assign_tag.png" alt="Assign tag to protocol" style="display:block;width:100%;max-width:980px;height:auto;margin:1.2rem auto 1.6rem;" />

*Tag assignment from the protocol card or protocol action menu.*

When assigning tags, review whether the label describes the protocol itself, the current review status, or the role of the protocol inside the workflow. Mixing these meanings in the same tag set can make the workflow harder to read.

---

## Review tagged protocols

After assigning tags, use them as visual cues while navigating the workflow.

Tags can help you quickly identify:

- protocols that belong to the same processing stage
- protocols that need review
- protocols that generated important outputs
- protocols that should not be reused
- alternative branches or candidate results

In collaborative projects, tags are especially useful when combined with consistent project naming, comments, and clear communication about which protocols are considered final.

---

## Good practice

- define a small tag set first and expand only when needed
- use colors consistently across related projects when possible
- document team-specific meanings in tag descriptions
- keep tag names short enough to remain readable on protocol cards
- remove obsolete tags only after checking their current assignments
- avoid using tags as a substitute for protocol comments or execution status

---

## Common sources of confusion

Users often run into confusion when they:

- create too many tags with overlapping meanings
- use colors inconsistently
- assign review-status tags and workflow-stage tags without a clear convention
- assume a tag changes protocol execution behavior
- delete a tag that collaborators were using to track project progress

When a tag convention becomes unclear, simplify the tag list and agree on a small set of meanings before continuing.

---

## If tags do not appear as expected

Check:

- whether you are working in the correct project
- whether the tag was saved successfully
- whether the protocol card has refreshed after assignment
- whether your user role has permission to manage or assign tags
- whether the browser shows failed API requests
- whether another user changed the tag list while you were working

Useful recovery steps:

1. refresh the project view
2. reopen the tag manager and confirm the tag still exists
3. assign the tag again from the protocol menu
4. check backend logs if the API reports an error
