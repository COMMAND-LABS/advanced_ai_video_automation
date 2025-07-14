# TLDR

Now we will break the full audio into segments using Whisper

## Table of Contents

1. [TLDR](#tldr)
2. [Whisper GitHub repo](#whisper-github-repo)
3. [Jump into n8n](#jump-into-n8n)
  - [Add a node - Get script](#add-a-node---get-script)
  - [Add a node - Download Spoken Audio](#add-a-node---download-spoken-audio)
  - [Add a node - Generate transcription](#add-a-node---generate-transcription)
  - [Add a node - Get segments](#add-a-node---get-segments)
  - [Add a node - Code](#add-a-node---code)
  - [Add node group - If](#add-node-group---if)
4. [Helper - Validate Segment Durations](#helper)
  - [a](#add-node-for-helper---get-script)
  - [b](#add-node-for-helper---get-segments)
  - [c](#add-node-for-helper---code)
  - [d](#add-node-for-helper---update-segments)

## Whisper GitHub repo

https://github.com/openai/whisper

## Jump into n8n

- Create a workflow called whatever you want but I suggest ie: `Segment Spoken Audio`

### Add a node - `Get script`

- Let's add an `Airtable - Search Records` node
- Configure the node
  - Filter by Formula: `{Active} = '1'`
    - FYI: https://support.airtable.com/docs/formula-field-reference
  - Limit: 1
- And "Test step" the node
- Rename the node: `Get Script`

### Add a node - `Download Spoken Audio`

- Let's add an `HTTP Request` node
- Configure the node
  - Method: GET
  - URL: `{{ $json['Spoken Audio URL'] }}`
- Add option:
  - Response
    - Response Format: File
    - Put Output in Field: data
- Rename the node: `Download Spoken Audio`

### Add a node - `Generate transcription`

- Let's add a node - `HTTP Request node`
- import curl based on these docs (https://platform.openai.com/docs/guides/speech-to-text)
```sh
curl --request POST \
  --url https://api.openai.com/v1/audio/transcriptions \
  --header "Authorization: Bearer $OPENAI_API_KEY" \
  --header 'Content-Type: multipart/form-data' \
  --form file=@/path/to/file/audio.mp3 \
  --form model=whisper-1 \
  --form response_format=verbose_json \
  --form prompt="Generate a transcription that has 3 segments"
```
- Configure the node
  - Authentication: Generic Credential Type
  - Generic Auth Type: Header Auth
  - Header Auth: Create new credential
    - Rename credential: `Open AI API AAIVA cred`
    - Generate API key here: https://platform.openai.com/api-keys
    - Create the n8n credential
      - Name: Authorization
      - Value: Bearer {OpenAI API key here}
  - Edit the `file` field
    - Parameter type: n8n Binary File
    - Name: file
    - Input Data Field Name: data
- Rename the node: `Generate transcription`

### Add a node - `Get segments`

- Let's add an `Airtable: Search Records` node
- Configure the node
  - Resource: Record
  - Operation: Search
  - Filter By Formula: `{Full Script} = {{ $('Fetch script from Airtable').item.json.ID }}`
- Rename the node: `Get segments`

### Add a node - `Code`

Check if there are segments sitting in the `Segments` table

```js
if (Object.keys($input.last().json).length === 0) {
  return {
    "count": 0,
    "priorSegments": [],
    "segments": $('Generate transcription').first().json.segments
  }
} else {
  return {
    "count": $input.all().length,
    "priorSegments": $input,
    "segments": $('Generate transcription').first().json.segments
  }
}
```

### Add node group - `If`

- FALSE
  - Store segments
    - Use `Merge` node to precede the `Split Out` node
      - `Get segments` node has the segments
    - Create records in Airtable
- TRUE
  - Delete existing segments
    - Use `Merge` node to precede the `Split Out` node
      - `Get segments` node has the segments
    - Add an `Airtable - Delete a record` node
  - Store segments
    - Use `Merge` node to precede the `Split Out` node
      - `Generate transcription` node has the segments 
    - Loop over `Whisper segments` from "If" node
    - Create records in Airtable

## HELPER

Quick lil' helper to "validate" the segments

### Add node for helper - `Get script`

Copy from prior workflows

### Add node for helper - `Get segments`

Copy from prior workflows

### Add node for helper - `Code`

```js
let segments = $input.all()
for (let i = 0; i < (segments.length-1); i++) {
  if (segments[i].json['End Time'] !== segments[i+1].json['Start Time']) {
    segments[i].json['End Time'] = segments[i+1].json['Start Time']
  }
}
return segments
```

### Add node for helper - `Update segments`

- Update the following fields in the `Segments` table
  - Start Time
  - End Time
  - Make Video
  