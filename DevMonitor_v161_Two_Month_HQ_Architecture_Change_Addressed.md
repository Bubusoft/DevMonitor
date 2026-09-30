# v161 --- Change Addressed: Two-Month HQ Data Architecture

## Change Addressed

The previous DevMonitor architecture used multiple physical databases
for Month-on-Month processing, including a historical design based on
four physical monthly databases.

That architecture has now been **superseded and addressed**.

### New Physical Month Architecture

DevMonitor now works with only **two physical HQ month stores**:

1.  **Current calendar month**
2.  **Immediately previous calendar month**

These are the only two physical months the application needs to
maintain.

The application must not depend on retaining a rolling collection of
older physical HQ databases.

Historical references in earlier development records to four physical
Month-on-Month databases remain valid as historical documentation of the
previous design. They should not be interpreted as the current
architecture.

------------------------------------------------------------------------

## HQ Report Authority

HQ reports are authoritative exports of the company's database and are
treated as verified company snapshots.

An HQ report:

-   contains only devices with activity for the covered period;
-   does not contain inactive/zero-activity devices;
-   does not span more than one calendar month;
-   does not change after it has been generated.

Therefore, the absence of a device from a newly imported HQ report is
meaningful. The device may have had no activity or may have been
removed/expunged from the company database.

The application must therefore **not merge an incoming HQ report with
the previous snapshot for the same month**.

Instead, the incoming report becomes the complete authoritative snapshot
for that month.

------------------------------------------------------------------------

## Replace-All Import Rule

For a given calendar month:

-   First valid HQ report → import it.
-   Newer HQ report covering the same month → replace the existing
    month's snapshot completely.
-   Report with the same coverage/end date → treat as duplicate and
    reject/ignore.
-   Older report when a newer report already exists → reject.
-   Report for a different month → allow import, regardless of
    chronological import order.

Example:

September data may already exist, and the user may subsequently import
August data.

This is valid and must work correctly.

The application must determine the correct physical month store from the
report's calendar month rather than assuming that reports will always be
imported chronologically.

------------------------------------------------------------------------

## Atomic Replacement

Replacing a month's HQ snapshot must be atomic.

The application should:

1.  Validate the incoming report.
2.  Determine its calendar month and coverage.
3.  Validate that it is acceptable according to the
    newer/duplicate/older-report rules.
4.  Replace the complete existing snapshot for that month within a
    controlled transaction.
5.  Commit only after the replacement succeeds.

If the operation fails, the previous valid month's data must remain
intact.

------------------------------------------------------------------------

## Import Must Not Trigger Downstream Processing

Importing an HQ report is deliberately an **import-only operation**.

After a successful import, DevMonitor must **not automatically**:

-   refresh Device Monitor;
-   recalculate payment progression;
-   recalculate payment estimates;
-   refresh Month-on-Month;
-   recalculate dashboards;
-   run other expensive downstream calculations.

The user-initiated refresh buttons already provided throughout the
application remain responsible for those operations.

This separation is intentional so that importing data does not
unnecessarily consume processing power, battery, memory, or time on the
Android device.

------------------------------------------------------------------------

## Calculation and Payment Logic Protection

This architecture change does **not** constitute a change to the
established payment formulas, payment progression rules, payment-cycle
logic, or payment estimator behavior.

Existing payment calculation and progression logic must remain
protected.

The required change is limited to making the data-access layer aware of
the new monthly storage architecture.

Where a calculation requires:

-   current-month data → read the current-month store;
-   previous-month data → read the previous-month store.

The calculation itself must continue to use the established rules.

In particular:

> **Do not modify payment formulas or payment/progression logic merely
> because the underlying monthly storage architecture has changed.**

The distinction between **Active activation counts** and **MTD/display
values** remains unchanged. MTD values must not be substituted for
Active values in payment calculations.

------------------------------------------------------------------------

## Month-on-Month Architecture

Month-on-Month processing must use the same two-month HQ architecture
rather than maintaining a separate set of four physical Month-on-Month
databases.

The Month-on-Month feature therefore reads:

-   the current month's physical HQ store; and
-   the immediately previous month's physical HQ store.

Its existing calculation rules, ownership scope, cutoff handling, and
user-specific filtering remain unchanged unless separately documented.

The change is architectural: **Month-on-Month now consumes the shared
monthly HQ stores instead of maintaining separate physical monthly
databases.**

------------------------------------------------------------------------

## Historical Architecture Status

The former four-physical-database design is retained in the development
history for traceability.

**Status: SUPERSEDED / CHANGE ADDRESSED**

The current requirement is:

> DevMonitor physically maintains only the current calendar month and
> the immediately previous calendar month.

This entry documents the transition rather than rewriting earlier
historical records.

------------------------------------------------------------------------

## Implementation Boundary

This change affects:

-   monthly HQ data storage;
-   HQ report replacement;
-   report ordering/acceptance rules;
-   out-of-order month imports;
-   Month-on-Month data access;
-   calculation data retrieval;
-   documentation of the current architecture.

This change does **not** authorize unrelated changes to:

-   payment formulas;
-   payment progression;
-   payment-cycle rules;
-   Device Monitor business rules;
-   established Active/MTD definitions;
-   user ownership rules;
-   report authority;
-   existing refresh-button behavior.

Any future modification to those areas must be separately identified,
documented, and approved as a distinct change.
