---
name: Upsert a contact into Hatch
description: >-
  Create or update a contact in Hatch and enroll it into a campaign for AI-driven
  voice, SMS, and email outreach, using the public REST API's upsert operation.
api: openapi/hatch-openapi-original.json
operations:
- HatchWeb.V1.Integrations.ContactController.upsert
---

# Upsert a contact into Hatch

Use this skill to push a lead or customer into Hatch so its AI CSRs can text,
email, and call them. Hatch deduplicates automatically, so the same call safely
creates or updates.

## Authentication
- HTTP bearer token. Send `Authorization: Bearer <api_key>`.
- Create/manage the API key in the **API Keys** tab of the Hatch App Marketplace.

## Endpoint
- `POST https://api.usehatchapp.com/v1/contacts` (or `PUT` — both perform the same upsert).
- Request and response are `application/json`.

## Required and key fields
- `source` (**required**) — must exactly match a custom source you configured in
  the App Marketplace, e.g. `custom:my-awesome-source`. Unknown sources are rejected.
- Provide `phoneNumber` and/or `email` — these are the dedup keys.
- Optional: `firstName`, `lastName`, `status`, `externalID`, `externalContactID`,
  `externalCreatedAt`, `externalUpdatedAt`, and a free-form `details` object for custom fields.

## Dedup / idempotency behavior
Matching is resolved in precedence order:
1. If `phoneNumber` matches an existing contact, it is updated.
2. Else if `email` matches, that contact is updated.
3. Else a new contact is created with a fresh `id`.

`externalID` is a unique record identifier from your system; ingesting data with a
matching `externalID` updates the corresponding opportunity. Because of this,
repeated identical upserts converge (natural-key idempotency) — there is no
`Idempotency-Key` header.

## Success and errors
- `200` returns `{ "data": { "id": "<contact-uuid>" } }`.
- `422 Unprocessable Entity` returns a JSON:API-style error envelope: an `errors[]`
  array where each item has `title`, `source.pointer` (the failing attribute), and
  `detail`. Read `source.pointer` to fix the specific field.

## Rate limits
100 requests per 10 seconds. Batch and back off accordingly.

## Example request body
```json
{
  "source": "custom:my-awesome-source",
  "firstName": "firstName",
  "lastName": "lastName",
  "email": "firstName.lastName@host.com",
  "phoneNumber": "6364618187",
  "status": "sold",
  "externalID": "03775ccb-ffd7-48a6-b1c6-0fac96289928",
  "details": { "address": "contact address", "favoriteColor": "Blue" }
}
```
