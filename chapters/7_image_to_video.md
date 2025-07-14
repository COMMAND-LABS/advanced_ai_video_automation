# TLDR

Now we will convert selected segments of our story board into video clips

## Table of Contents

1. [TLDR](#tldr)
2. [Table of Contents](#table-of-contents)
3. [Jump into n8n](#jump-into-n8n)
  - [Add node - Fetch script from Airtable](#add-node---get-script)
  - [Add node - Search for segments](#add-node---get-segments)
  - [Add node - If](#add-node---if)
  - [Add node - Falai---Gen-video](#add-node---falai---gen-video)
  - [Add node - Airtable - Update record](#add-node---airtable---update-record)

## Jump into n8n

- Create a workflow called whatever you want but I suggest ie: `Image to Video`

### Add node - `Get script`

- copy from prior workflows

### Add node - `Get segments`

- copy from prior workflows

### Add node - `If`

For only generating the videos you want

### Add node - `Fal.ai - Gen video`

From https://fal.ai/models/fal-ai/kling-video/v2.1/master/image-to-video/api

```sh
curl -X POST "https://queue.fal.run/fal-ai/kling-video/v2.1/master/image-to-video" \
  --header "Content-Type: application/json" \
  --header "Authorization: Key <get access token>" \
  --data "{\"prompt\": \"prompt\",\"image_url\": \"your_image_url_here\",\"duration\": \"5 or 10\"}"
```

NOTE: Kling 2.1 can only generate 5 or 10 second videos
TIP: Check fal.ai for rendering progress at `https://fal.ai/models/fal-ai/kling-video/v2.1/master/image-to-video/requests`

### Add node - `Airtable - Update record`

Only execute if `Make Video` column of the `Segments` table is True

- Configure node
  - ID (using to match): `{{ $('Search for segments').item.json.ID }}`
  - Make Video: `{{ $('If').item.json['Make Video'] }}`
  - Video Render Request ID: `{{ $json.request_id }}`
