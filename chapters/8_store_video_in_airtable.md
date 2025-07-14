# TLDR

We will now store the generated video clips into Airtable

## Table of Contents

1. [TLDR](#tldr)
2. [Table of Contents](#table-of-contents)
3. [Jump into n8n](#jump-into-n8n)
    - [Add a node - Get script](#add-a-node---get-script)
    - [Add a node - Search for segments](#add-a-node---get-segments)
    - [Add a node - If (Make Video is TRUE)](#add-a-node---if-make-video-is-true)
    - [Add a node - HTTP Request - Download gen'd video metadata](#add-a-node---http-request---download-gend-video-metadata)
    - [Add a node - Airtable - Update a record](#add-a-node---airtable---update-a-record)

## Jump into n8n

- Create a workflow called whatever you want but I suggest ie: `Store Video in Airtable`

### Add a node - `Get script`

Copy from prior workflows

### Add a node - `Get segments`

Copy from prior workflows

### Add a node - `If (Make Video is TRUE)`

Copy from prior workflows

### Add a node - `HTTP Request - Download gen'd video metadata`

```sh
curl -X GET "https://queue.fal.run/fal-ai/kling-video/requests/$VIDEO_RENDER_REQUEST_ID"
```

### Add a node - `Airtable - Update a record`

- Configure the node
  - ID (using to match): `{{ $('If').item.json.ID }}`
  - Start Time: `{{ $('If').item.json['Start Time'] }}`
  - End Time: `{{ $('If').item.json['End Time'] }}`
  - Video:
```js
[
  {
    "url": "{{ $json.video.url }}",
    "filename": "{{ $json.video.file_name }}"    
  }
]
```
  - Video URL: `{{ $json.video.url }}`
  - Make Video: `{{ $('If').item.json['Make Video'] }}`
