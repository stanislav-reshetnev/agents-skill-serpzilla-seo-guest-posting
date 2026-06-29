# SEO Actions for Managing Placements (Serpzilla)

> **When to read this:** whenever you are about to call `perform_placement_action`. Every action here has a precondition (the placement's current status) and a financial consequence, so confirm intent with the user first — see *Financial guardrails* in `SKILL.md`.

All actions are invoked through one tool:

```
perform_placement_action(placement_ids=[PLACEMENT_ID, ...], action="ACTION_NAME")
```

`placement_ids` is a **list**; you can act on several placements at once. Replace `ACTION_NAME` with one of the values below.

## Available actions

| Action                          | Description                                                                                                | From status(es)                                                                                | To status                       | Notes                                                                                                                       |
|---------------------------------|------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| `approve_seo`                   | Approve a placement after reviewing the live page. Marks the placement as successfully completed.         | `STATUS_PLACEMENT_CHECK`                                                                       | `STATUS_APPROVED`               | Only possible when the placement is in the final check stage.                                                               |
| `approve_content_seo`           | Accept the content provided by the webmaster (for formats that require content approval).                 | `STATUS_ON_CONTENT_APPROVAL`, `STATUS_PLACEMENT_ARBITRATION`                                   | `STATUS_ON_PLACEMENT_GUARANTEED`| After approval the placement moves to a guaranteed state. Usually used for `review`, `link`, or `archive` formats.          |
| `approve_from_arbitration_seo`  | Approve a placement that was sent to arbitration, resolving the dispute in the advertiser's favor.        | `STATUS_PLACEMENT_ARBITRATION`                                                                 | `STATUS_APPROVED`               | Use when the moderator or system decides in the advertiser's favor.                                                         |
| `cancel_from_on_placement_seo`  | Cancel a placement currently in the "on placement" state (before the webmaster has published).            | `STATUS_ON_PLACEMENT`                                                                          | `STATUS_REJECTED`               | Available only 1–72 hours after the placement entered `STATUS_ON_PLACEMENT`. After 72 hours it becomes unavailable.         |
| `cancel_from_improve_seo`       | Cancel a placement in the "improvement needed" state (the webmaster is reworking the content).            | `STATUS_PLACEMENT_IMPROVE`                                                                     | `STATUS_REJECTED`               | Allowed only after the placement has been in `STATUS_PLACEMENT_IMPROVE` for more than 15 days.                              |
| `cancel_seo`                    | Generic cancellation for placements in various states where the process has stalled or you are unsatisfied. | `STATUS_MODER`, `STATUS_CLARIFICATION`, `STATUS_ON_PLACEMENT_GUARANTEED`, `STATUS_NEED_APPROVE` | `STATUS_REJECTED`               | Requires the placement to be guaranteed and a timeout condition.                                                            |
| `terminate_seo`                 | Request permanent removal (teardown) of an already-approved backlink.                                     | `STATUS_APPROVED`                                                                              | `STATUS_TO_TERMINATE`           | The link is removed after the webmaster acts on the request.                                                                |

## Workflow notes

- **Guaranteed placements** (`STATUS_ON_PLACEMENT_GUARANTEED`) are those where the webmaster has confirmed they will complete the work. Most actions on guaranteed placements have stricter timeouts — check the *Notes* column before acting.
- **Arbitration** (`STATUS_PLACEMENT_ARBITRATION`) is the dispute state. Try to resolve with the webmaster first; if unresolved, use `approve_from_arbitration_seo` or let the moderator decide.

## Example

```
# Cancel a placement stuck in "on placement"
perform_placement_action(placement_ids=[12345], action="cancel_from_on_placement_seo")
```

## Common statuses

| Status                          | Meaning                                    |
|---------------------------------|--------------------------------------------|
| `STATUS_PHANTOM`                | Initial ghost state before any processing. |
| `STATUS_CREATING`               | Placement is being created in subsystems.  |
| `STATUS_NEED_APPROVE`           | Awaiting the advertiser's approval (auto-buyer mode). |
| `STATUS_ON_PLACEMENT`           | Webmaster is expected to place the link.   |
| `STATUS_ON_PLACEMENT_GUARANTEED`| Webmaster guaranteed the placement.        |
| `STATUS_ON_CONTENT_APPROVAL`    | Content is ready for the advertiser's review. |
| `STATUS_PLACEMENT_CHECK`        | Live page is being checked by the system.  |
| `STATUS_PLACEMENT_IMPROVE`      | Webmaster must improve the placement.      |
| `STATUS_PLACEMENT_ARBITRATION`  | Dispute resolution in progress.            |
| `STATUS_APPROVED`               | Placement successfully completed.          |
| `STATUS_TO_TERMINATE`           | Link removal requested.                    |
| `STATUS_TERMINATED`             | Link removed.                              |
| `STATUS_CHANGE_REQUESTED`       | Link change requested.                     |
| `STATUS_BUY_FAIL`               | Purchase attempt failed.                   |
| `STATUS_WAIT_WITHDRAWAL`        | Draft waiting for payment.                 |
| `STATUS_REJECTED`               | Placement rejected / cancelled.            |
| `STATUS_DELETED`                | Permanently deleted.                       |
