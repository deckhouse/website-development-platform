---
title: Entity status
weight: 50
---

The widget displays the entity status and status check results.

## Displayed data

The widget displays the following information.

### Overall status

- **Progress bar** — a visual representation of the overall entity status, including the percentage of successful checks.
- **Successful check count** — the number of successful checks out of all configured status checks.

### Check list

The following information is displayed for each status check:

- **Check name** — the name of the check rule.
- **Status** — the check result:
  - **Passed** — the check completed successfully.
  - **Failed** — the check did not pass, but no execution error occurred.
  - **Error** — an error occurred while running the check.
- **Last check time** — the date and time when the check last ran.
- **Error message** — the error text, displayed if the check ended with an error.

### Statistics

The bottom of the widget displays summary check statistics:

- **Passed** — the number of successful checks.
- **Failed** — the number of checks that did not pass without execution errors.
- **Error** — the number of checks that ended with an error.

### Blocked actions

The widget automatically identifies and displays actions that are unavailable for the entity's current status.

Display conditions:

- The action must be available for the resource associated with the entity.
- The action must have allowed statuses configured.
- The entity's current status must not be in the action's list of allowed statuses.

The widget displays the following information:

- Action name.
- Action description, if specified.

## Configuration

The widget requires no additional configuration.

To use the widget, configure status checks for the resource associated with the entity.
For details, refer to [status check configuration](../../healthchecks/overview/).

## Notes

The widget has the following behavior:

- If no status checks are configured for the entity, the widget indicates that no checks are available.
- If status check data is unavailable, the widget indicates that no data is available.
