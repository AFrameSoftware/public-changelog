# AFrame Public API — Changelog

This page lists user-visible changes to the AFrame Public API (`/api-pub/v1`). New entries are added to the top; older entries remain below for reference. If you integrate directly against this API, skim the **Breaking Changes** of each entry before rolling out a new version of your client.

To be notified when this changelog is updated, [subscribe to changelog updates](https://github.com/AFrameSoftware/public-changelog) — release notifications via GitHub or RSS.

---

## 2026-10-06

Six colors have been added to the color palette.

### <span style="color: blue;">Non-Breaking Changes</span>

#### New `color` values

- `color` now also accepts and returns `MAROON`, `AMBER`, `SKY`, `VIOLET`, `PINK` and `GRAPHITE`. Existing values are unchanged.
- The full list is now `NONE`, `RED`, `MAROON`, `TANGERINE`, `AMBER`, `YELLOW`, `LIME`, `GREEN`, `TEAL`, `CYAN`, `SKY`, `COBALT`, `VIOLET`, `PURPLE`, `MAGENTA`, `PINK`, `TAUPE`, `GRAPHITE`.
- Applies to Tasks, Events, Transaction Attachments, Transaction Fields, Field choices, and Task, Event and Attachment Template Entries. Clients that reject unknown enum values should be updated to accept the new ones.

---

## 2026-10-05

Uploading a file to a Transaction Attachment placeholder now follows the same completion rule as assigning a file to one.

### <span style="color: blue;">Non-Breaking Changes</span>

#### Uploads leave a placeholder that still needs signatures incomplete

- `PATCH /xaction-attachments/{xactionAttachmentId}/file` with `completeMode` `DEFAULT` (or omitted) marks `completed = true` only when no further signatures are needed, matching `PATCH /xaction-attachments/{xactionAttachmentId}/file/assign`. When signature tracking is on and a signature is still not satisfied, `completed` is left as it was.
- `COMPLETE`, `INCOMPLETE` and `UNCHANGED` behave as before.

---

## 2026-10-01

A Field can now be shown in more than one Field Group. It still has one value per Transaction, so the value is the same in every Field Group it appears in. Contact and Transaction notes can now @mention team members.

### <span style="color: red;">Breaking Changes</span>

#### A Field can appear in more than one Field Group in tree responses

- In `GET /fields/tree` and `GET /xactions/{xactionId}/field-tree`, the same `fieldId` can now be listed under several Field Groups, each with the same value.
- `FieldDto.fieldGroupId` is still the Field's primary Field Group, so it can differ from the Field Group the entry is listed under.
- Likewise, `FieldDto.sort` is the Field's sort within its primary Field Group. Entries are returned in display order, so use their position in the list to order Fields within a Field Group.

`GET /fields` is unchanged; it still lists each Field once.

**Action required:** Clients that flatten the tree into a map or list keyed by `fieldId` must allow duplicates, or keep only the entry where `field.fieldGroupId` matches the enclosing Field Group's `fieldGroupId`.

### <span style="color: blue;">Non-Breaking Changes</span>

#### @mentions in notes

- A `note` sent to `POST /contact-notes`, `PATCH /contact-notes/{contactNoteId}`, `POST /xaction-activities` or `PATCH /xaction-activities/{xactionActivityId}` can mention a team member with `<span data-mention-app-user-id="{appUserId}">@Name</span>`. The `data-mention-app-user-id` attribute is now kept when the note is saved (it was previously removed).
- Each newly mentioned AppUser on your Team who can see the Contact or Transaction receives an email and an in-app notification, credited to the authenticated AppUser. Saving the note again does not notify the same AppUser twice.
- Notes returned by the API may contain these `<span>` tokens, including notes written in the AFrame app.

#### Zapier notes never notify

- In `POST /integrations/zapier/contacts` and `POST /integrations/zapier/xactions`, mention tokens in `note` are converted to plain `@Name` text, so no one is notified. Zapier notes often carry text from public forms.

---

## 2026-09-30

Transactions can now receive email at their own address. Received emails appear in email responses with `emailTransportDirection` `RECEIVE` and `emailQueueState` `RECEIVED`.

### <span style="color: red;">Breaking Changes</span>

#### `APIEmailQueueDto.scheduledSendDateTime` can be `null`

- `scheduledSendDateTime` *(date-time)* — now `null` for received (`RECEIVE`) emails, which are never scheduled to send. Sent emails always have a value, as before.

Affects `GET /xactions/{xactionId}/email-queues`.

**Action required:** Clients that read `scheduledSendDateTime` without a null check must handle `null`, or skip emails whose `emailTransportDirection` is `RECEIVE`.

### <span style="color: blue;">Non-Breaking Changes</span>

#### `APIEmailQueueDto.emailFrom` maximum length raised to 255

- `emailFrom` *(string)* — maximum length is now 255 (was 100). The From header of a received email can carry a long display name.

---

## 2026-09-26

This release adds read endpoints for the entries of Event, Task and Attachment Templates, so integrations can inspect which Events, Tasks and Attachment placeholders (and which date formulas) applying a template will create. It also brings Task responses in line with Event responses: the Date Calculator is now a nested object instead of a flat id + name pair. Participant Templates, which add a consistent set of Roles and Contacts to a Transaction, are now readable as well.

### <span style="color: green;">New Endpoints (Participant Templates)</span>

#### `GET /participant-templates` — List Participant Templates

Returns every ParticipantTemplate defined for the Team, ordered by Folder (the "[No Folder]" bucket first) then by each template's sort and name.

**Payload:** `ParticipantTemplateDto[]`
- `participantTemplateId` *(integer)*, `teamId` *(integer)*, `name` *(string)*, `description` *(string)*, `sort` *(integer)*
- `folder` *(FolderDto)* — the containing Folder, or `null` when the template is not in a Folder

#### `GET /participant-templates/{participantTemplateId}/participant-template-entries` — List Participant Template Entries

Returns every entry of the specified Participant Template, ordered by sort. The Participant Template must belong to the Team; otherwise the request returns `404`.

**Payload:** `APIParticipantTemplateEntryDto[]`
- `participantTemplateEntryId` *(integer)*, `participantTemplateId` *(integer)*
- `xactionParticipantRole` *(APIXactionParticipantRoleDto)* — the Transaction Participant Role the entry fills when applied (same shape as `GET /xaction-participant-roles`)
- `contact` *(APIContactDigestDto)* — the Contact who fills the Role when applied, or `null` when the entry applies as a placeholder
- `placeholder` *(boolean)* — true when the entry has no Contact, so applying it creates an empty Role placeholder
- `sort` *(integer)*
- `templateEntryApplySettings` *(object)* — `xactionSideBuyer`, `xactionSideSeller`, `xactionSideDual` *(boolean)*: which Transaction sides the entry applies to

#### `GET /folders/participant-templates` — Participant Template Folders

Returns the Participant Template Folders for the Team, in the same `FolderListDto` envelope as the other Template Folder endpoints.

### <span style="color: red;">Breaking Changes</span>

#### Date Calculator nested on `APITaskDto` (responses)

- `dueDateAdjustmentRulesId` *(integer)*, `dueDateAdjustmentRulesName` *(string)* → **`dueDateAdjustmentRules`** *(DateAdjustmentRulesDigestDto)* — the Date Calculator (same shape as `GET /date-adjustment-rules`), or `null` for calendar days

Affects every endpoint that returns `APITaskDto`:

- `GET /tasks/{taskId}`
- `POST /tasks` (response payload)

**Action required:** Clients reading `dueDateAdjustmentRulesId` or `dueDateAdjustmentRulesName` from a Task response must switch to `dueDateAdjustmentRules.dateAdjustmentRulesId` and `dueDateAdjustmentRules.name`, and treat a `null` `dueDateAdjustmentRules` as calendar days. `APITaskCreateDto` and `APITaskPatchDto` still take the flat `dueDateAdjustmentRulesId`. This matches how `APIEventDto.dateAdjustmentRules` has worked since 2026-09-21.

### <span style="color: blue;">Non-Breaking Changes</span>

#### Schema rename `APIDateAdjustmentRulesDigestDto` → `DateAdjustmentRulesDigestDto`

The OpenAPI schema for the Date Calculator is now named `DateAdjustmentRulesDigestDto`. It is used by `GET /date-adjustment-rules`, `APIEventDto.dateAdjustmentRules`, `APITaskDto.dueDateAdjustmentRules`, `EventTemplateEntryDto.dateAdjustmentRules`, and `TaskTemplateEntryDto.dueDateAdjustmentRules`. The JSON is unchanged; only generated client type names are affected.

### <span style="color: green;">New Endpoints</span>

#### `GET /event-templates/{eventTemplateId}/event-template-entries` — List Event Template Entries

Returns every entry of the specified Event Template, ordered by title. The Event Template must belong to the Team; otherwise the request returns `404`.

**Payload:** `EventTemplateEntryDto[]`
- `eventTemplateEntryId` *(integer)*, `eventTemplateId` *(integer)*
- `title` *(string)*, `location` *(string)*, `description` *(string)*, `folderName` *(string)* — the Event Folder the created Event is placed in, or `null`
- `color` *(enum)* — `NONE`, `RED`, `TANGERINE`, `TAUPE`, `YELLOW`, `LIME`, `GREEN`, `CYAN`, `TEAL`, `COBALT`, `PURPLE`, `MAGENTA`
- `allDayEvent` *(boolean)*, `startTimeMinutes` *(integer)*, `durationMinutes` *(integer)* — minutes after midnight and duration; ignored when `allDayEvent` is true
- `reminderSet` *(boolean)*, `reminderMinutes` *(integer)*
- `mergeFieldCode` *(string)* — the code assigned to the created Event; other entries reference it via `dateAdjustMergeFieldCode`
- `dateAdjustActive` *(boolean)*, `dateAdjustType` *(enum)*, `dateAdjustMergeFieldCode` *(string)*, `dateAdjustDelta` *(integer)* — the date formula, with the same semantics as on `APIEventDto`
- `dateAdjustmentRules` *(DateAdjustmentRulesDigestDto)* — the Date Calculator (same shape as `GET /date-adjustment-rules`), or `null` for calendar days
- `agentVisible` *(boolean)*, `buyerSellerVisible` *(boolean)* — portal visibility of the created Event
- `sort` *(integer)*
- `templateEntryApplySettings` *(object)* — `xactionSideBuyer`, `xactionSideSeller`, `xactionSideDual` *(boolean)*: which Transaction sides the entry applies to

#### `GET /task-templates/{taskTemplateId}/task-template-entries` — List Task Template Entries

Returns every entry of the specified Task Template, ordered by sort then subject. The Task Template must belong to the Team; otherwise the request returns `404`.

**Payload:** `TaskTemplateEntryDto[]`
- `taskTemplateEntryId` *(integer)*, `taskTemplateId` *(integer)*, `taskTemplateName` *(string)*
- `subject` *(string)*, `note` *(string)*
- `taskType` *(enum)* — `TODO`, `PHONE`, `LETTER`, `EMAIL`
- `color` *(enum)* — `NONE`, `RED`, `TANGERINE`, `TAUPE`, `YELLOW`, `LIME`, `GREEN`, `CYAN`, `TEAL`, `COBALT`, `PURPLE`, `MAGENTA`
- `autoFillWithAppUserId` *(integer)*, `autoFillWithRole` *(enum: `USER`, `PRIMARY`, `CO_AGENT`, `ASSISTANT1`, `ASSISTANT2`)* — who the created Task is assigned to; at most one is set
- `recurring` *(boolean)*, `recurringFrequency` *(enum: `DAILY`, `WEEKLY`, `MONTHLY`, `YEARLY`)*, `recurringSeparationCount` *(integer)*, `recurringCount` *(integer)*, `recurringDayOfWeek` *(enum)*, `recurringDayOfMonth` *(integer)*, `recurringMonthOfYear` *(enum)* — recurrence, with the same semantics as on `APITaskDto`
- `dueDateAdjustActive` *(boolean)*, `dueDateAdjustType` *(enum)*, `dueDateAdjustRefMergeFieldCode` *(string)*, `dueDateAdjustDelta` *(integer)* — the due-date formula, with the same semantics as on `APITaskDto`
- `dueDateAdjustRefTaskIdParent` *(integer)*, `dueDateAdjustRefTaskParentSubject` *(string)*, `dueDateAdjustRefTaskParentContingent` *(boolean)* — the sibling entry the due date chains from when `dueDateAdjustType` is `PARENT_TASK`
- `dateAdjustReferenceCode` *(string)* — convenience code for the formula's reference: `sys_` + `dueDateAdjustType` for system references, otherwise the referenced Merge Field Code
- `dueDateAdjustmentRules` *(DateAdjustmentRulesDigestDto)* — the Date Calculator (same shape as `GET /date-adjustment-rules`), or `null` for calendar days
- `dueTimeMinutes` *(integer)*, `dueTime` *(string, `HH:mm`)* — the due time in two forms; both `null` when no due time is set
- `agentVisible` *(boolean)*, `buyerSellerVisible` *(boolean)* — portal visibility of the created Task
- `prospecting` *(boolean)*, `onCalendar` *(boolean)*, `milestone` *(boolean)*
- `reminderSet` *(boolean)*, `reminderDelta` *(integer)*, `reminderTimeMinutes` *(integer)* — days before the due date and minutes after midnight the reminder fires
- `expense` *(number)*
- `sort` *(integer)*
- `templateEntryApplySettings` *(object)* — `xactionSideBuyer`, `xactionSideSeller`, `xactionSideDual` *(boolean)*: which Transaction sides the entry applies to

#### `GET /attachment-templates/{attachmentTemplateId}/attachment-template-entries` — List Attachment Template Entries

Returns every entry of the specified Attachment Template, ordered by sort then title. The Attachment Template must belong to the Team; otherwise the request returns `404`.

**Payload:** `AttachmentTemplateEntryDto[]`
- `attachmentTemplateEntryId` *(integer)*, `attachmentTemplateId` *(integer)*
- `attachmentType` *(enum)* — `FILE`, `URL`
- `title` *(string)*, `description` *(string)*, `webLink` *(string)* — `webLink` is only set when `attachmentType` is `URL`
- `color` *(enum)* — `NONE`, `RED`, `TANGERINE`, `TAUPE`, `YELLOW`, `LIME`, `GREEN`, `CYAN`, `TEAL`, `COBALT`, `PURPLE`, `MAGENTA`
- `required` *(boolean)*
- `folderName` *(string)* — the Attachment Folder the created Attachment is placed in, or `null`
- `mergeFieldCode` *(string)* — the code assigned to the created Attachment
- `signatureTrackingEnabled` *(boolean)*, `signatureStateSeller`, `signatureStateBuyer`, `signatureStateSellerAgent`, `signatureStateBuyerAgent` *(enum: `NOT_NEEDED`, `NOT_SATISFIED`, `SATISFIED`)*
- `agentVisible` *(boolean)*, `buyerSellerVisible` *(boolean)* — portal visibility of the created Attachment
- `sort` *(integer)*
- `templateEntryApplySettings` *(object)* — `xactionSideBuyer`, `xactionSideSeller`, `xactionSideDual` *(boolean)*: which Transaction sides the entry applies to

---

## 2026-09-23

This release tightens the status-dependent validation of a Transaction so the same rules apply on every write path. Three checks that previously came back as warnings (the request still succeeded) now reject the request, and one requirement is dropped.

### <span style="color: red;">Breaking Changes</span>

#### Status-dependent field rules on `POST /xactions` and `PATCH /xactions/{xactionId}`

The request is now rejected with the standard validation error response when the Transaction's status (after the write) lands in a stage whose rules are not met:

| Status stage | Required | Must be empty |
|---|---|---|
| `ACTIVE`, side `SELLER` or `DUAL` | `listPrice`, `listDate` | `closedDate` |
| `ACTIVE`, side `BUYER` | – | `closedDate` |
| `PRE_ACTIVE` | – | `closedDate` |
| `UNDER_CONTRACT` | `contractPrice`, `effectiveDate` | `closedDate` |
| `SOLD` | `contractPrice`, `closedDate` | – |
| `NOT_SOLD` | `closedDate` | – |
| `NOT_ACTIVE` | – | – |

Newly enforced (previously warnings): `contractPrice` on `UNDER_CONTRACT`/`SOLD`, `effectiveDate` on `UNDER_CONTRACT`, and a non-null `closedDate` on `PRE_ACTIVE`/`ACTIVE`/`UNDER_CONTRACT`. `closedDate` is now required on `NOT_SOLD` (a Transaction that fell apart is closed as of that date). When changing `xactionStatusId`, send the fields the new stage needs in the same request (a `PATCH` may set several paths at once).

### <span style="color: green;">Relaxed</span>

- `payoutActual` is no longer required when a Transaction is `SOLD`.
- `closedDate` is no longer required when a Transaction is `NOT_ACTIVE`.

## 2026-09-21

This release exposes an Event's **date formula** (auto-adjusting start date) on the Transaction Events endpoints, and adds a list endpoint for the Team's **Date Calculators** so their ids can be discovered. No existing fields or behaviors are removed; the Events list continues to return `APIEventDto`, which simply gains the fields below.

### <span style="color: green;">New Endpoints</span>

#### `GET /date-adjustment-rules` — List Date Calculators

Returns every Date Calculator (internally `DateAdjustmentRules`) defined for the Team, ordered by sort then name. A Date Calculator decides how days are counted when a date formula adds `dateAdjustDelta` days to its reference date.

**Payload:** `APIDateAdjustmentRulesDigestDto[]`
- `dateAdjustmentRulesId` *(integer)* — use as `dateAdjustmentRulesId` on Events
- `name` *(string)*, `description` *(string)*, `sort` *(integer)*
- `systemRule` *(boolean)* — system-provided calculators cannot be edited by the Team
- `countBy` *(enum)* — `CALENDAR_DAYS`, `BUSINESS_DAYS`, `CALENDAR_END_ON_BUSINESS_PREVIOUS`, `CALENDAR_END_ON_BUSINESS_NEXT`, `BUSINESS_DAYS_SATURDAY`, `BUS_5DAYS_CONVERT_TO_CAL_EOB_NEXT`

### <span style="color: green;">New Fields</span>

#### Event date formula on `GET`, `POST`, `PATCH /xactions/{xactionId}/events[/{eventId}]`

`APIEventDto` (returned by list, read, create, and patch) now includes:
- `dateAdjustActive` *(boolean)* — when true, `startDate` is computed by the server and kept in sync with the reference date
- `dateAdjustType` *(enum)* — `XACTION_LIST_DATE`, `XACTION_ON_MARKET_DATE`, `XACTION_EXPIRE_DATE`, `XACTION_EFFECTIVE_DATE`, or `EVENT_MERGE_FIELD_CODE` (another Event). Legacy rows may report `TEMPLATE_START_DATE`.
- `dateAdjustMergeFieldCode` *(string)* — the referenced Event's `mergeFieldCode`; only set for `EVENT_MERGE_FIELD_CODE`
- `dateAdjustReferenceCode` *(string)* — the reference in the write form below
- `dateAdjustDelta` *(integer)* — days added to the reference date (negative = before); `null` means the formula is incomplete
- `dateAdjustmentRules` *(APIDateAdjustmentRulesDigestDto)* — the Date Calculator (same shape as `GET /date-adjustment-rules`), or `null` for calendar days

`APIEventCreateDto` and `APIEventPatchDto` now accept:
- `dateAdjustActive` *(boolean)*
- `dateAdjustReferenceCode` *(string, max 100)* — one of the Transaction dates `sys_XACTION_LIST_DATE`, `sys_XACTION_ON_MARKET_DATE`, `sys_XACTION_EXPIRE_DATE`, `sys_XACTION_EFFECTIVE_DATE`, **or** the `mergeFieldCode` of another Event on the same Transaction (e.g. `d_ClosingDate`; case-sensitive)
- `dateAdjustDelta` *(integer)*
- `dateAdjustmentRulesId` *(integer)* — from `GET /date-adjustment-rules`; `null` counts calendar days

```
PATCH /xactions/123/events/456
Content-Type: application/json-patch+json

[
  {"op": "replace", "path": "/dateAdjustActive", "value": true},
  {"op": "replace", "path": "/dateAdjustReferenceCode", "value": "sys_XACTION_EFFECTIVE_DATE"},
  {"op": "replace", "path": "/dateAdjustDelta", "value": 10},
  {"op": "replace", "path": "/dateAdjustmentRulesId", "value": 12}
]
```

**Rules**
- When `dateAdjustActive` is true, `dateAdjustReferenceCode` is required, and any `startDate` you send is replaced by the computed date once the reference date is known. Patch `startDate` directly only on Events whose formula is off.
- If no non-omitted Event on the Transaction has the referenced `mergeFieldCode` yet, or `dateAdjustDelta` is `null`, `startDate` is left `null` until the formula can be evaluated.
- **`422 Unprocessable Content`**: unknown `sys_` reference type (e.g. `sys_PARENT_TASK`), a reference equal to the Event's own `mergeFieldCode`, or a `dateAdjustmentRulesId` that does not exist on the Team.

> **Note:** the 2026-07-18 entry stated that patching a date-adjust field returns `400`. That restriction is lifted; the four fields above are now patchable. Patching `omitted` still returns `400`.

### <span style="color: orange;">Bug Fixes</span>

- `PATCH /xactions/{xactionId}/events/{eventId}` validation failures (`422`) now include the field-level `validationErrors` messages in the error envelope; previously the messages were omitted.

---

## 2026-07-26

This release adds two new template-apply endpoints for Transactions: apply EventTemplates (Date Templates) and apply AttachmentTemplates — plus list endpoints for EventTemplates and AttachmentTemplates so their ids can be discovered. They complement the existing `POST /xactions/{xactionId}/apply-task-templates` and `GET /task-templates` endpoints, which are unchanged.

### <span style="color: green;">New Endpoints</span>

#### `GET /event-templates` — List Event Templates

Returns every EventTemplate (Date Template) defined for the Team, ordered by Folder (the "[No Folder]" bucket first) then by each template's sort and name.

**Payload:** `EventTemplateDto[]`
- `eventTemplateId` *(integer)*, `teamId` *(integer)*, `name` *(string)*, `description` *(string)*, `sort` *(integer)*
- `folder` *(FolderDto)* — the containing Folder, or `null` when the template is not in a Folder

#### `GET /attachment-templates` — List Attachment Templates

Returns every AttachmentTemplate defined for the Team, ordered by Folder (the "[No Folder]" bucket first) then by each template's sort and name.

**Payload:** `AttachmentTemplateDto[]`
- `attachmentTemplateId` *(integer)*, `teamId` *(integer)*, `name` *(string)*, `description` *(string)*, `sort` *(integer)*
- `folder` *(FolderDto)* — the containing Folder, or `null` when the template is not in a Folder

#### `POST /xactions/{xactionId}/apply-event-templates` — Apply EventTemplates to a Transaction

Applies one or more EventTemplates (Date Templates) to the specified Transaction. Each EventTemplate's entries are converted into individual Events attached to the Transaction (filtered by the Transaction's `xactionSide` via the template entries' apply settings, and placed in the entry's folder when one is named). The created Events are then dated automatically using each Event's date-adjust rules and the Transaction's key dates (list, on-market, expire, effective, closing) — no start date is supplied.

A Transaction may not have two Events with the same Merge Field Code, so entries whose Merge Field Code already exists on one of the Transaction's Events (including Events created earlier in the same request) are **not** applied — they are returned in `skippedEntries` and reported as warnings in `error.validationErrors`.

**Request body:**
- `eventTemplateIds` *(integer[], required)* — IDs of the EventTemplates to apply. Invalid IDs are skipped and reported as warnings in `error.validationErrors`; if all IDs are invalid the request fails with `422`.

**Payload:** `APIXactionApplyEventTemplatesResponseDto`
- `events` *(APIEventBriefDto[])* — the Events that were created. Brief shape with IDs for associations and core scalar fields. Fetch `GET /xactions/{xactionId}/events/{eventId}` for full details.
- `skippedEntries` *(EventTemplateEntryBriefDto[])* — template entries skipped due to a duplicate Merge Field Code. Each entry contains `eventTemplateEntryId`, `eventTemplateId`, `title`, `mergeFieldCode`.

```json
{
  "eventTemplateIds": [123, 456]
}
```

#### `POST /xactions/{xactionId}/apply-attachment-templates` — Apply AttachmentTemplates to a Transaction

Applies one or more AttachmentTemplates to the specified Transaction. Each AttachmentTemplate's entries are converted into individual attachment placeholders on the Transaction (filtered by the Transaction's `xactionSide` via the template entries' apply settings, and placed in the entry's folder when one is named).

**Request body:**
- `attachmentTemplateIds` *(integer[], required)* — IDs of the AttachmentTemplates to apply. Invalid IDs are skipped and reported as warnings in `error.validationErrors`; if all IDs are invalid the request fails with `422`.

**Payload:** `APIXactionAttachmentBriefDto[]` — the attachments that were created. Fetch `GET /xactions/{xactionId}/attachments` for full details.

```json
{
  "attachmentTemplateIds": [123, 456]
}
```

---

## 2026-07-18

This release adds write endpoints for Events — create, patch, and delete a calendar Event on a Transaction — and changes how a Transaction's **closing date** is managed: the closing date is now owned by its Event and is no longer patchable on the Transaction itself. It also tightens JSON Patch field validation across **all** `PATCH` endpoints.

### <span style="color: green;">New Endpoints</span>

Events are calendar dates and appointments attached to a Transaction (e.g. inspection, appraisal, closing). The read/list endpoints (`GET /xactions/{xactionId}/events`) shipped previously; this release adds the single-Event read and the full write surface. All endpoints are scoped to the Transaction in the path and return the standard envelope.

#### `GET /xactions/{xactionId}/events/{eventId}` — Read an Event

Returns the `APIEventDto` for the Event. Omitted Events, Events on another Transaction, and Transactions the caller cannot access are treated as not found (`404`) / forbidden (`403`).

#### `POST /xactions/{xactionId}/events` — Create an Event

Creates an Event on the Transaction. Returns `201` with the new `APIEventDto`.

**Request body:** `APIEventCreateDto`
- `title` *(string, max 300)*, `description` *(string, max 65535)*, `location` *(string, max 300)*
- `allDayEvent` *(boolean)*, `startDate` *(date, `yyyy-MM-dd`)*, `startTime` *(time, `HH:mm`)*, `durationMinutes` *(integer)*, `eventTZ` *(string, IANA time zone)*
- `reminderSet` *(boolean)*, `reminderMinutes` *(integer)*
- `completed` *(boolean)*, `color` *(enum, default `NONE`)*
- `agentVisible` *(boolean)*, `buyerSellerVisible` *(boolean)*
- `mergeFieldCode` *(string)* — see the closing-date note below
- `folderId` *(integer)* / `newFolderName` *(string)* — mutually exclusive folder placement
- `sort` *(integer)*

#### `PATCH /xactions/{xactionId}/events/{eventId}` — Patch an Event

Updates the Event using JSON Patch (RFC 6902). See `APIEventPatchDto` for the patchable fields (the create fields above, minus folder-creation nuances). Patching a field outside this set — for example `omitted` or a date-adjust field — returns `400`. Returns the updated `APIEventDto`.

```
PATCH /xactions/123/events/456
Content-Type: application/json-patch+json

[
  {"op": "replace", "path": "/title", "value": "Final Walkthrough"}
]
```

#### `DELETE /xactions/{xactionId}/events/{eventId}` — Delete an Event

Deletes the Event. Returns `"success"`.

#### `mergeFieldCode` — uniqueness and the special `d_ClosingDate` code

- **Uniqueness:** a Transaction may not have two Events with the same `mergeFieldCode`. A create or patch that would produce a duplicate returns **`422 Unprocessable Content`**.
- **`d_ClosingDate`:** an Event whose `mergeFieldCode` is `d_ClosingDate` is bound to the Transaction's closing date — the Event's `startDate` **is** the closing date. Creating or patching such an Event sets/updates `Xaction.closingDate`; deleting it clears the closing date. This is the supported way to set or change a Transaction's closing date after creation.
- **Other Transaction dates are not Events:** `listDate`, `onMarketDate`, `expireDate`, `effectiveDate`, `closedDate`, and `listPrice` live on the Transaction — update them via `PATCH /xactions/{xactionId}`.

> **Note:** the read `APIEventDto` exposes the start time as `startTimeMinutes` (minutes since midnight), while the create/patch DTOs accept `startTime` (`HH:mm`). This asymmetry is intentional.

### <span style="color: red;">Breaking Changes</span>

#### `closingDate` removed from `PATCH /xactions/{xactionId}`

`closingDate` is no longer a patchable field on the Transaction. A JSON Patch op targeting `/closingDate` now returns **`400 Bad Request`**, and `closingDate` has been removed from `APIXactionPatchDto`.

**Action required:** set or change a Transaction's closing date by creating or patching its `d_ClosingDate` Event (its `startDate` is the closing date), and clear it by deleting that Event — see the Events endpoints above. `POST /xactions` **still accepts** `closingDate` on create (unchanged), which creates the closing-date Event automatically.

#### Stricter JSON Patch field validation on all `PATCH` endpoints

Every patch operation's `path` (and `from`, for `move`/`copy` ops) is now validated against the endpoint's documented patch DTO before anything is applied. An op targeting an unknown or undocumented field returns **`400 Bad Request`** naming the offending field. Previously, a `replace` on an unknown field failed with a generic error while an `add` on an unknown field was **silently ignored**.

**Action required:** remove any patch ops your client sends on undocumented paths — they were no-ops before and are rejected now. Clients that only patch documented fields see no change.

#### `editDateTime` removed from `APIContactNotePatchDto`

`editDateTime` is server-managed. A patch op targeting `/editDateTime` on `PATCH /contact-notes/{contactNoteId}` now returns `400 Bad Request`; previously it was accepted but silently ignored.

### <span style="color: orange;">Bug Fixes</span>

#### Team Member login email now syncs on Contact patches

`PATCH /contacts/{contactId}` and `PATCH /xaction-participants/{id}/linked-contact` now update the associated Team Member's login email when the patch changes the primary email of a Contact linked to a Team Member. Previously this sync only ran when the email was changed through the AFrame web app.

---

## 2026-07-17

This release adds read endpoints for listing Folders — the containers AFrame uses to organize Templates, Transaction Attachments, Tasks, and Events — and a single-call endpoint for creating a Transaction Attachment together with its file, so callers no longer need to create the attachment and upload the file in two separate requests. Additive — no breaking changes.

### <span style="color: green;">New Endpoints</span>

All Folder endpoints below return the same envelope: a `FolderListDto` with a `folders` array of `FolderDto`. Each `FolderDto` carries `folderId`, `teamId`, `contactId`, `xactionId`, `name`, `folderType`, `renderClosed`, and `sort`.

#### Template Folders (team-scoped)

The Template Folders are readable by any authenticated Team Member — no per-entity access checks apply.

- `GET /folders/letter-templates` — Letter Template Folders for the Team.
- `GET /folders/task-templates` — Task Template Folders for the Team.
- `GET /folders/event-templates` — Event Template Folders for the Team.
- `GET /folders/attachment-templates` — Attachment Template Folders for the Team.

#### `GET /folders/xaction-attachments` — Transaction Attachment Folders

Returns the attachment Folders for a Transaction.

**Query parameters:**
- `xactionId` *(integer, required)* — omitting it returns `400 Bad Request`.

**Access:** the authenticated user must have access to the Transaction. An unknown `xactionId` returns `404 Not Found`; a Transaction the user cannot access returns `403 Forbidden`.

#### `GET /folders/tasks` — Task Folders for a Contact or Transaction

Returns the Task Folders scoped to a Contact or a Transaction.

**Query parameters:**
- `xactionId` *(integer, optional)* — mutually exclusive with `contactId`.
- `contactId` *(integer, optional)* — mutually exclusive with `xactionId`.

Exactly one of `xactionId` or `contactId` must be supplied — supplying neither, or both, returns `400 Bad Request`.

**Access:** each supplied id is checked — the user must have access to the referenced Contact and/or Transaction. An unknown id returns `404 Not Found`; an id the user cannot access returns `403 Forbidden`.

#### `GET /folders/events` — Event Folders for a Contact or Transaction

Same request shape and access rules as `GET /folders/tasks`, returning Event Folders instead.

#### `POST /xaction-attachments/file` — Create a Transaction Attachment with a file

A new `POST /xaction-attachments/file` endpoint accepts `multipart/form-data`, letting you create the attachment and store its file in one request. The existing `POST /xaction-attachments` (metadata only, file uploaded separately via `PATCH /xaction-attachments/{id}/file`) is unchanged.

**Request (`multipart/form-data`):**

- **`data`** *(part, `application/json`, required)* — an `APIXactionAttachmentCreateDto` (same body as the JSON create). For a file attachment, set `attachmentType` to `FILE`; `title` is optional because the uploaded file supplies the name.
- **`file`** *(part, binary, required)* — the file to store. Omitting it returns `400 Bad Request`.
- **`newFileName`** *(query, optional)* — override the stored file's name. The extension must match the uploaded file's extension.
- **`completeMode`** *(query, optional)* — override the implicit `completed` mutation, using the same values as the `/file` endpoints: `DEFAULT` (or omitted) marks `completed = true` after the upload; `COMPLETE` forces `true`; `INCOMPLETE` forces `false`; `UNCHANGED` leaves the value from the `data` part alone.

```
POST /xaction-attachments/file?completeMode=COMPLETE
Content-Type: multipart/form-data

--boundary
Content-Disposition: form-data; name="data"
Content-Type: application/json

{"xactionId": 123, "attachmentType": "FILE", "title": "Purchase Agreement"}
--boundary
Content-Disposition: form-data; name="file"; filename="purchase-agreement.pdf"
Content-Type: application/pdf

<binary>
--boundary--
```

**Payload:** `APIXactionAttachmentDto` — the newly created Attachment, with its file stored.

Additive — existing JSON `POST /xaction-attachments` clients see no change.

---

## 2026-07-06

This release lets callers reassign a Transaction's agents (primary agent, co-agent, and the two Transaction Coordinator assistants) through the `PATCH /xactions/{xactionId}` endpoint. Additive — no breaking changes.

### <span style="color: green;">New Capabilities</span>

#### New patchable agent fields on `APIXactionPatchDto` (`PATCH /xactions/{xactionId}`)

Four Transaction agent-assignment fields are now patchable via JSON Patch (RFC 6902):

- **`appUserIdPrimaryAgent`** *(integer)* — the primary agent.
- **`appUserIdCoAgent`** *(integer, nullable)* — the co-agent.
- **`appUserIdAssistantTC1`** *(integer, nullable)* — Transaction Coordinator / Assistant 1.
- **`appUserIdAssistantTC2`** *(integer, nullable)* — Transaction Coordinator / Assistant 2.

Each value must be the ID of a Team Member (AppUser) on the authenticated user's Team.

```
PATCH /xactions/123
Content-Type: application/json-patch+json

[
  {"op": "replace", "path": "/appUserIdPrimaryAgent", "value": 456},
  {"op": "replace", "path": "/appUserIdCoAgent",      "value": null}
]
```

**Behavior and constraints:**

- **Unknown ID** — an ID that does not resolve to a Team Member on the caller's Team returns `404 Not Found`.
- **Access required** — the authenticated user must have edit access to the Team Member being assigned, or be assigning themselves; otherwise `403 Forbidden`.
- **Clearing** — `appUserIdCoAgent`, `appUserIdAssistantTC1`, and `appUserIdAssistantTC2` may be cleared by patching them to `null`. `appUserIdPrimaryAgent` is required and **cannot** be cleared — patching it to `null` returns `422 Unprocessable Content`.
- **Isolation mode** — users in isolation mode may not change any of these four fields (`403 Forbidden`); all other patchable fields remain available to them.

Additive — clients that don't patch these paths see no behavior change.

---

## 2026-05-29

This release removes public-API access to **"omitted"** Tasks, Events, and Transaction Attachments, finalizes the Transaction Attachment uploader field rename, adds the Team Member's profile picture URL, adds a `completeMode` query parameter on the Transaction Attachment file endpoints that lets callers opt out of the implicit `completed` flag mutation, and refreshes the Swagger tag descriptions for clarity.

### <span style="color: red;">Breaking Changes</span>

#### "Omitted" Tasks, Events, and Transaction Attachments are no longer exposed

Entities flagged as **omitted** in AFrame are now treated as non-existent by the public API, matching the AFrame web app's behavior of hiding omitted items.

- **Tasks** — `GET /tasks/{taskId}` returns `404 Not Found` for an omitted Task, and `PATCH /tasks/{taskId}` rejects an omitted Task with `404 Not Found`. (`POST /tasks/search` already excluded omitted Tasks by default — unchanged.)
- **Events** — omitted Events are filtered out of the nested `events` collection on `APIXactionDto` / `APIXactionDigestDto`.
- **Transaction Attachments** — `GET /xaction-attachments/{xactionAttachmentId}` returns `404 Not Found` for an omitted attachment, and the nested `xactionAttachments` collection on `APIXactionDto` / `APIXactionDigestDto` excludes them. All mutating attachment endpoints (`PATCH /xaction-attachments/{id}`, `PATCH` / `DELETE .../file`, `PATCH .../file/assign`, `PATCH .../file/unassign`) reject an omitted attachment (source or target) with `404 Not Found`. (`GET /xactions/{xactionId}/xaction-attachments` already excluded omitted attachments — unchanged.)

#### `omitted` field removed from request and response DTOs

The `omitted` boolean has been removed from the following DTOs:

- Responses: `APIEventDto`, `APIEventBriefDto`, `APITaskDto`, `APITaskDigestDto`, `APIXactionAttachmentBriefDto`
- Create / Patch: `APITaskCreateDto`, `APITaskPatchDto`, `APIXactionAttachmentCreateDto`, `APIXactionAttachmentPatchDto`

**Action required:** Stop reading `omitted` from any Task, Event, or Transaction Attachment response — the field is gone. Remove `omitted` from create payloads and drop any patch ops targeting `/omitted`; the omit state can no longer be set or changed through the public API.

#### Field rename on `APIXactionAttachmentDto` (responses)

- `appUserIdUploader` → **`appUserId`**

Affects every endpoint that returns `APIXactionAttachmentDto`, including:

- `GET /xaction-attachments/{xactionAttachmentId}`
- `GET /xactions/{xactionId}/xaction-attachments` (each item)
- `POST /xaction-attachments`, `PATCH /xaction-attachments/{id}`, `PATCH /xaction-attachments/{id}/file`, `PATCH /xaction-attachments/{id}/file/unassign` (response payload)

**Action required:** Clients reading `appUserIdUploader` from any Transaction Attachment response must switch to `appUserId`. The value semantics are unchanged — it's still the AppUser who uploaded the attachment.

### <span style="color: green;">New Capabilities</span>

#### New field on `APIAppUserDigestDto` (responses)

- **`profileUrl`** *(string, URI, nullable)* — URL to the Team Member's profile picture.

Affects every endpoint that returns `APIAppUserDigestDto`, including:

- `GET /app-users/me`
- `GET /app-users/{appUserId}`
- `POST /app-users/search` (each item in the paged result)
- Nested on `APIXactionDto` and `APIXactionDigestDto` under `appUserPrimaryAgent`, `appUserCoAgent`, `appUserAssistant1`, `appUserAssistant2` — and the same nested fields on the `APIXactionDto` payload delivered by `XACTION_CREATED` and `XACTION_UPDATED` webhooks.

Existing clients can ignore the new field; no action is required.

#### New `completeMode` query parameter on Transaction Attachment file endpoints

The following endpoints implicitly mutate the Transaction Attachment's `completed` flag as a side effect of their primary file operation. Callers can now override that side effect via a new optional `completeMode` query parameter:

- `PATCH /xaction-attachments/{xactionAttachmentId}/file` — default behavior: marks `completed = true` after a successful upload.
- `DELETE /xaction-attachments/{xactionAttachmentId}/file` — default behavior on the placeholder branch (Title non-blank): marks `completed = false`. The non-placeholder branch deletes the entire Attachment, and `completeMode` has no effect there.
- `PATCH /xaction-attachments/{xactionAttachmentId}/file/assign` — default behavior on the target: marks `completed = true` if no further signatures are needed. `completeMode` controls the **target only**; the source's completion behavior is unchanged.

**Allowed values:**

- `DEFAULT` (or omitting the parameter) — applies the endpoint's implicit behavior described above. Preserves existing semantics.
- `COMPLETE` — forces `completed = true`.
- `INCOMPLETE` — forces `completed = false`.
- `UNCHANGED` — leaves the existing `completed` value alone.

```
PATCH /xaction-attachments/123/file?completeMode=INCOMPLETE
```

Additive — clients that don't pass `completeMode` see no behavior change.

### <span style="color: blue;">Non-Breaking Changes</span>

#### Swagger tag description refresh

The `@Tag` descriptions shown as section headers in the Swagger UI were rewritten to lead with what each entity *is* in real-estate / CRM terms, with concrete examples, instead of enumerating the available CRUD verbs. Affects every tag in the public API (Transactions, Transaction Participants, Transaction Activities, Transaction Attachments, Transaction Data, Events, Email Queues, Tasks, Contacts, Contact Notes, Files, Team Members, and the five `Admin :: *` lookups). Wire formats (URLs, verbs, field names, field types) are unchanged; no client action required.

---

## 2026-05-25

This release restructures the Transaction Participant endpoints to clearly separate three distinct operations: editing the participant's denormalized contact-info snapshot, editing the underlying linked Contact entity, and managing the link itself. Includes field renames across the DTOs for naming consistency.

### <span style="color: red;">Breaking Changes</span>

#### Renames on `APIXactionParticipantCreateDto` (`POST /xaction-participants`)

- `existingContactId` → **`linkedContactId`**
- `contact` → **`contactInfo`**
- Added: `agentVisible`, `buyerSellerVisible` (optional, defaults applied by server)

#### Renames on `APIXactionParticipantDto` and `APIXactionParticipantDigestDto` (responses)

- `contact` → **`contactInfo`**
- Added top-level: **`linkedContactId`** (nullable Integer) — caller uses this to choose between `/linked-contact` and `/contact-info` update endpoints.

#### `PATCH /xaction-participants/{id}` — patchable scope narrowed

The endpoint now only patches participant-level fields: `xactionParticipantRoleId`, `agentVisible`, `buyerSellerVisible`. Contact-info fields are **no longer patchable** here — patches targeting the previously-supported nested paths (`/contact/firstName`, `/contact/phone1`, etc.) will be rejected. Use the new sub-routes below to update contact-info:

- `PATCH /xaction-participants/{id}/linked-contact` — when the participant has a linked Contact (updates the Contact entity and re-syncs the snapshot)
- `PATCH /xaction-participants/{id}/contact-info` — when the participant has no linked Contact (updates the snapshot only)

#### Webhook payload — embedded participant field rename

Webhooks for `XACTION_CREATED` and `XACTION_UPDATED` carry an `APIXactionDto` whose `participants[*]` entries are `APIXactionParticipantDigestDto`. The same DTO rename applied to the REST responses also applies to the webhook payload:

- `participants[*].contact` → **`participants[*].contactInfo`**
- `participants[*].linkedContactId` is now present on each participant entry (nullable Integer, additive).

If you consume `XACTION_*` webhooks, update your payload parsers to read `participants[*].contactInfo` instead of `participants[*].contact`. Reading `linkedContactId` is optional.

### <span style="color: green;">New Endpoints</span>

#### `PATCH /xaction-participants/{id}/linked-contact` — Edit the linked Contact

Applies JSON Patch (RFC 6902) ops to the **Contact entity** linked to the participant. The participant's snapshot is re-synced from the saved Contact. Returns **409 Conflict** if the participant has no linked Contact. Address ops route to the Contact's primary address (HOME or WORK).

**Request body:** `APIContactInfoPatchDto` (root-level paths: `/firstName`, `/phone1`, `/addressLine1`, etc.)

#### `PATCH /xaction-participants/{id}/contact-info` — Edit the snapshot only

Applies JSON Patch ops to the participant's denormalized contact-info snapshot. Does NOT touch any Contact entity. Returns **409 Conflict** if the participant has a linked Contact.

**Request body:** same `APIContactInfoPatchDto` shape as `/linked-contact`.

#### `PUT /xaction-participants/{id}/linked-contact/{linkedContactId}` — Set or replace the link

Sets the participant's linked Contact to the Contact identified by `{linkedContactId}`. The snapshot is re-synced from the resolved Contact. If the participant was previously linked to a different Contact, the prior link is replaced. **No request body.**

To inline-create a new Contact and link it in one call, use `POST /xaction-participants` with `contactInfo` set and `onlySaveContactInTransaction=false`.

#### `DELETE /xaction-participants/{id}/linked-contact` — Unlink

Removes the Contact link; snapshot fields are preserved on the participant. Returns **409 Conflict** if not currently linked.

### <span style="color: orange;">Bug Fixes</span>

#### `POST /xactions` — post-create side effects now run

Creating an Xaction with a `closingDate` set now correctly creates the corresponding closing-date Event, queues calendar invite resync to participants, and runs auto-adjust so template-driven Tasks/Events anchor their dates against the new Xaction. Previously these side effects were skipped on the API path and only fired through the legacy edit UI. Behavior when `closingDate` is omitted is unchanged.

#### `PATCH /xactions/{xactionId}` — post-update side effects now consistent

Patching an Xaction now consistently triggers the same set of side effects that the legacy edit UI has always run:

- **Closing-date Event sync** — when the patch changes `closingDate`, the corresponding Event entity (mergeFieldCode `closingDate`) is created/updated/cleared to match. Previously the Event would drift out of sync.
- **PRICE_CHANGE activity auto-log** — when the patch changes `listPrice`, a PRICE_CHANGE entry is automatically added to the Xaction's activity log with the old and new prices. Previously not triggered on PATCH.
- **Calendar invite resync** — calendar invites to participants are queued for refresh. Previously not triggered on PATCH.
- **Geo-locator refresh** — when the patch changes any of the address fields, the Xaction is flagged so the geo-locator job recomputes latitude/longitude. Previously the flag was not set on PATCH and lat/lng would not refresh.

Auto-adjust on changes to adjust-date inputs (`listDate`, `onMarketDate`, `expireDate`, `effectiveDate`, `closingDate`, `closedDate`) was already running and continues to run unchanged.

#### `PATCH /tasks/{taskId}` — auto-adjust now runs

Patching a Task now runs auto-adjust against the Task's Xaction/Contact tree, so date changes propagate to descendant Tasks and Events. Previously `POST /tasks` ran auto-adjust but `PATCH /tasks/{taskId}` did not, causing the two paths to behave inconsistently.

#### `POST /xaction-participants` — `onlySaveContactInTransaction` now honored

The `onlySaveContactInTransaction` field has been documented on the create payload since the endpoint shipped but was previously ignored — a Contact entity was always created when `contactInfo` was supplied. As of this release the flag works as documented: when `true`, the inline `contactInfo` is materialized only on the participant's snapshot, no Contact entity is created, and `linkedContactId` on the response is `null`. Behavior when omitted or set to `false` is unchanged.

---

## 2026-04-24

This release adds full CRUD endpoints for Transaction Attachments, including dedicated sub-routes for managing the underlying file (upload/replace, remove, assign, unassign). It also refreshes `@Schema` documentation across many existing DTOs for accuracy and consistency — no wire format changes. No breaking changes.

### <span style="color: green;">New Endpoints</span>

#### `GET /xaction-attachments/{xactionAttachmentId}` — Read a Transaction Attachment

Returns the full `APIXactionAttachmentDto` for the supplied ID.

#### `POST /xaction-attachments` — Create a Transaction Attachment

Creates a new Transaction Attachment record (metadata only). To attach a file, first create the attachment with this endpoint, then upload the file to `PATCH /xaction-attachments/{id}/file`.

**Request body:** `APIXactionAttachmentCreateDto`
- `xactionId` *(integer, required)* — ID of the Transaction the attachment belongs to.
- `attachmentType` *(`FILE` | `URL`, required)*
- `folderId` *(integer, optional)* — target Folder; mutually exclusive with `newFolderName`.
- `newFolderName` *(string, optional)* — create a new Folder with this name and place the attachment in it; mutually exclusive with `folderId`.
- `title` *(string, max 300)*
- `description` *(string, max 65535)*
- `webLink` *(string, URI — required when `attachmentType` is `URL`)*
- `required`, `completed`, `omitted`, `agentVisible`, `buyerSellerVisible` *(boolean)*
- `color` *(enum, default `NONE`)*
- `mergeFieldCode` *(string)*
- `sort` *(integer)*

**Payload:** `APIXactionAttachmentDto` — the newly created Attachment.

#### `PATCH /xaction-attachments/{xactionAttachmentId}` — Patch a Transaction Attachment

Updates the Transaction Attachment using JSON Patch (RFC 6902) operations. See `APIXactionAttachmentPatchDto` for the set of patchable fields (`attachmentType`, `title`, `description`, `webLink`, `fileName`, `required`, `completed`, `omitted`, `color`, `agentVisible`, `buyerSellerVisible`, `mergeFieldCode`, `sort`, `folderId`, `newFolderName`). Attempts to patch fields outside this set return `400`.

> **Note:** To upload, replace, remove, or reassign the file itself, use the dedicated `/file` sub-routes below — not a JSON Patch on this endpoint.

#### `PATCH /xaction-attachments/{xactionAttachmentId}/file` — Upload or replace the file

Multipart upload with a single `file` part. If a file is already present on the Attachment it is replaced.

**Query parameters:**
- `newFileName` *(optional)* — override the uploaded file's name. The extension must match the uploaded file's extension.

**Payload:** `APIXactionAttachmentDto` — the updated Attachment.

#### `DELETE /xaction-attachments/{xactionAttachmentId}/file` — Remove the file

Removes the file from the Transaction Attachment. If the Attachment is a placeholder (has a non-blank title), the file is deleted and the Attachment is marked incomplete. If the Attachment is not a placeholder, the entire Attachment is deleted.

#### `PATCH /xaction-attachments/{xactionAttachmentId}/file/assign` — Move a file to another Attachment

Moves a Transaction Attachment's file (source) onto another Transaction Attachment that has no file (target). After assignment, if the source is not a placeholder (title is blank), the source is deleted. The target is marked complete; the source (if not deleted) is marked incomplete.

**Query parameters:**
- `targetXactionAttachmentId` *(integer, required)* — ID of the target Attachment that will receive the file.

#### `PATCH /xaction-attachments/{xactionAttachmentId}/file/unassign` — Unassign a file

Removes the file from the Transaction Attachment and creates a new, untitled Transaction Attachment that owns the file. The original Attachment is marked incomplete. Only applies when the original has a file and is a placeholder (title is non-blank).

**Payload:** `APIXactionAttachmentDto` — the newly created Attachment that now owns the file.

### <span style="color: blue;">Non-Breaking Changes</span>

#### DTO `@Schema` documentation refresh

`@Schema` annotations were revised across many existing request/response DTOs (AppUser, Contact, ContactNote, Email, Event, Folder, Task, Xaction, XactionActivity, XactionParticipant, Zapier, etc.) for accuracy and consistency — descriptions, examples, `allowableValues`, `requiredMode`, and `maxLength` values. Wire formats (URLs, verbs, field names, field types) are unchanged; only the OpenAPI documentation has been updated. Regenerate your client SDK if you rely on generated doc comments or validation metadata.

---

## 2026-04-23

This release adds new read endpoints for fetching data related to a Transaction, three endpoints for downloading file attachments, a new email preview endpoint, a new endpoint for applying TaskTemplates to a Transaction, renames DTOs and operation IDs across the spec for consistency, and makes two breaking changes to existing endpoint response shapes. URL paths for previously existing endpoints are unchanged.

### <span style="color: green;">New Endpoints</span>
All paged list endpoints below support the `page` (0-based, default `0`) and `pageSize` (1–100, default `50`) query parameters, and return a response envelope of the form:

```json
{
  "payload": {
    "<items>": [ ... ],
    "pageMetadata": {
      "page": 0,
      "pageSize": 50,
      "totalElementsOnPage": 12,
      "totalElements": 12,
      "hasNextPage": false,
      "lastPage": 0
    }
  }
}
```

#### `GET /xaction-participant-roles` — List Transaction Participant Roles

Returns every Transaction Participant Role defined for the authenticated user's Team (e.g. Buyer, Listing Agent, Lender). Returned as a flat array (not paged).

**Payload:** `List<APIXactionParticipantRoleDto>`
- `xactionParticipantRoleId` *(integer)*
- `name` *(string)*
- `sort` *(integer)*
- `mergeFieldCodePrefix` *(string)*

#### `GET /xactions/{xactionId}/events` — List Events for a Transaction

Paged list of calendar Events attached to the Transaction. Only non-omitted Events are returned.

**Response wrapper field:** `events` → `APIEventDto[]`
Key fields: `eventId`, `xactionId`, `title`, `description`, `location`, `allDayEvent`, `startDate`, `startTimeMinutes`, `durationMinutes`, `eventTZ`, `reminderSet`, `reminderMinutes`, `completed`, `omitted`, `color`, `visibility`, `showAs`, `mergeFieldCode`, `sort`, `editDateTime`, `createDateTime`, `folder`.

#### `GET /xactions/{xactionId}/xaction-activities` — List Transaction Notes

Paged list of Transaction Notes (a.k.a. XactionActivities) on the Transaction.

**Response wrapper field:** `xactionActivities` → `APIXactionActivityDto[]`

#### `GET /xactions/{xactionId}/xaction-attachments` — List Transaction Attachments

Paged list of file and URL attachments on the Transaction. Only non-omitted attachments are returned.

**Response wrapper field:** `xactionAttachments` → `APIXactionAttachmentDto[]`
Key fields: `xactionAttachmentId`, `xactionId`, `appUserIdUploader`, `attachmentType` (`FILE` | `URL`), `title`, `description`, `fileName`, `contentType`, `fileSizeBytes`, `fileUploadDateTime`, `webLink`, `required`, `completed`, `color`, `mergeFieldCode`, `sort`, `createDateTime`, `folder`.

#### `GET /xactions/{xactionId}/email-queues` — List Email Queues for a Transaction

Paged list of Email Queue records (both queued and already-sent emails) attached to the Transaction.

**Response wrapper field:** `emailQueues` → `APIEmailQueueDto[]`
Key fields: `emailQueueId`, `xactionId`, `contactId`, `taskId`, `appUserIdCreatedBy`, `appUserIdFrom`, `emailQueueState`, `scheduledSendDateTime`, `emailFrom`, `emailTo`, `emailCC`, `emailBCC`, `emailSubject`, `hasAttachment`, `emailTransportDirection` (`INBOUND` | `OUTBOUND`), `emailTransportDateTime`, `createDateTime`.

#### `GET /email-queues/{emailQueueId}/preview` — Fetch rendered email preview

Returns the rendered content of a queued or sent email.

**Payload:** `APIEmailDto`
- `from` *(string)*
- `to`, `cc`, `bcc` *(string[])*
- `subject` *(string)*
- `body` *(string, HTML)* — combined email body and signature
- `emailFileAttachments` *(APIEmailFileAttachmentDto[])* — each entry contains `emailQueueAttachmentId`, `fileName`, `contentType`, `fileSizeBytes`. Pass `emailQueueAttachmentId` to `GET /files/email-queue-attachments/{id}` to download the file bytes.

#### `GET /files/xaction-attachments/{xactionAttachmentId}` — Download a Transaction Attachment file

Streams the file bytes with `Content-Type` and `Content-Disposition` headers set from the stored attachment metadata.

**Query parameters:**
- `disposition` *(optional, `INLINE` | `ATTACHMENT`, default `INLINE`)* — `INLINE` renders the file in the browser when possible; `ATTACHMENT` triggers a download.

**Response:** `200` with `Content-Type: <stored contentType>` and binary body.

#### `GET /files/contact-notes/{contactNoteId}` — Download a Contact Note file

Streams the file bytes for a file attached to a Contact Note. Same `disposition` query parameter as above.

#### `GET /files/email-queue-attachments/{emailQueueAttachmentId}` — Download an Email Queue Attachment file

Streams the file bytes for an attachment on a queued or sent email. Same `disposition` query parameter as above. Typically used in combination with `GET /email-queues/{emailQueueId}/preview` — take each `emailFileAttachments[i].emailQueueAttachmentId` from that response and call this endpoint to fetch the file.

#### `POST /xactions/{xactionId}/apply-task-templates` — Apply TaskTemplates to a Transaction

Applies one or more TaskTemplates to the specified Transaction. Each TaskTemplate's entries are converted into individual Tasks attached to the Transaction (filtered by the Transaction's `xactionSide` via the template entries' apply settings, and placed in the template's task folder).

**Request body:**
- `taskTemplateIds` *(integer[], required)* — IDs of the TaskTemplates to apply. Invalid IDs are skipped and reported as warnings in `error.validationErrors`; if all IDs are invalid the request fails with `422`.
- `startDate` *(string, optional, ISO date)* — start date used to compute Task due dates. Defaults to today in the authenticated user's time zone if omitted.

**Payload:** `APITaskBriefDto[]` — the Tasks that were created. Brief shape with IDs for associations (`contactId`, `xactionId`, `appUserId`, `folderId`, `appUserIdCompletedBy`) and core scalar fields (`taskId`, `taskType`, `status`, `subject`, `color`, `dueDate`, `dueTime`, `timeZone`, `completeDate`, `editDateTime`, `createDateTime`). Fetch `GET /tasks/{taskId}` for full details.

```json
{
  "taskTemplateIds": [123, 456],
  "startDate": "2026-04-23"
}
```

### <span style="color: red;">Breaking Changes</span>

#### 1. `POST /tasks/search` — response payload field renamed

The response wrapper DTO was renamed from `APISearchResultsTaskDto` to `APITaskPagedResultDto`, which aligns Tasks with the rest of the paged endpoints. The nested field `tasks` was renamed to `items`.

**Before**
```json
{
  "payload": {
    "tasks": [ { "taskId": 1 } ],
    "pageMetadata": { }
  }
}
```

**After**
```json
{
  "payload": {
    "items": [ { "taskId": 1 } ],
    "pageMetadata": { }
  }
}
```

**Action required:** Clients reading `response.payload.tasks` must change to `response.payload.items`. The element shape (`APITaskDigestDto`) is unchanged.

#### 2. `GET /xactions/{xactionId}/xaction-participants` — paged response + new query params

This endpoint previously returned a bare list and now returns a paged envelope consistent with the rest of the API. It also accepts new optional `page` and `pageSize` query parameters.

**Before**
```json
{
  "payload": [
    { "xactionParticipantId": 1 },
    { "xactionParticipantId": 2 }
  ]
}
```

**After**
```json
{
  "payload": {
    "xactionParticipants": [
      { "xactionParticipantId": 1 },
      { "xactionParticipantId": 2 }
    ],
    "pageMetadata": { "page": 0, "pageSize": 50, "totalElements": 2 }
  }
}
```

**Action required:** Clients reading `response.payload` as an array must now read `response.payload.xactionParticipants`. To preserve previous "fetch all at once" behavior, pass `?pageSize=100`. The element shape (`APIXactionParticipantDigestDto`) is unchanged.

### <span style="color: blue;">Non-Breaking Changes</span>

These changes do **not** affect HTTP requests or response bodies — URL paths, verbs, request parameters, and JSON fields all remain the same. However, if your integration is built from a generated client SDK (OpenAPI Generator, NSwag, etc.), regenerate your client so that class names and method names line up with the new spec.

#### Swagger tag renames

| Before | After |
|---|---|
| `AppUsers` | `Team Members` |
| `Xactions` | `Transactions` |
| `Xaction Statuses` | `Transaction Statuses` |

