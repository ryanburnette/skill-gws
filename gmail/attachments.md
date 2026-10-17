# Message attachments

Helpers do not download inbound attachments. Use raw `messages.get` +
`messages.attachments.get`.

## Find parts with attachment ids

```bash
gws gmail users messages get --params '{
  "userId": "me",
  "id": "MESSAGE_ID",
  "format": "full"
}'
```

Walk `payload` (and nested `parts`). A downloadable part has:

- `filename` (may be empty for some inline parts)
- `mimeType`
- `body.attachmentId`
- `body.size` (bytes)

Inline images and multipart trees are normal; not every part is a “file.”

Search for candidates:

```bash
gws gmail +triage --max 20 --query 'has:attachment filename:pdf newer_than:30d'
# or
gws gmail users messages list --params '{
  "userId": "me",
  "q": "has:attachment larger:10000",
  "maxResults": 20
}'
```

## Download

```bash
gws gmail users messages attachments get --params '{
  "userId": "me",
  "messageId": "MESSAGE_ID",
  "id": "ATTACHMENT_ID"
}'
```

Response:

```json
{ "size": 32398, "data": "<base64url>" }
```

`data` is **base64url** (URL-safe): `-` / `_` instead of `+` / `/`, little or no
padding. Decode before writing a file:

```bash
# example: Node one-liner pattern
# Buffer.from(data.replace(/-/g,'+').replace(/_/g,'/'), 'base64')
```

PDF magic bytes after decode: `25 50 44 46` (`%PDF`).

Do not log or commit attachment bytes. Write under a host temp path outside git.

## Outbound attachments

Use helper `-a` / `--attach` on `+send` / `+reply` / `+forward`. Combined size
limit on helpers: **25MB**. Forward keeps original attachments by default;
`--no-original-attachments` drops them (inline images may still appear in HTML
mode).
