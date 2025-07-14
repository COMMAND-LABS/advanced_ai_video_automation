# TLDR

Now we will convert the script into audio using ElevenLabs

## Table of Contents

1. [TLDR](#tldr)
2. [Table of Contents](#table-of-contents)
3. [Useful tools](#useful-tools)
4. [Jump into n8n](#jump-into-n8n)
  - [Add a node - Fetch script from Airtable](#add-a-node---fetch-script-from-airtable)
  - [Add a node - Eleven Labs TTS](#add-a-node---eleven-labs-tts)
  - [Add a node - Upload audio to Dropbox](#add-a-node---upload-audio-to-dropbox)
  - [Add a node - Create a Dropbox Share Link](#add-a-node---create-a-dropbox-share-link)
  - [Add a node - Edit Field](#add-a-node---edit-fields-set)
  - [Add a node - Airtable: Update Record](#add-a-node---update-record-in-airtable)
5. [Free Workaround](#free-workaround)

## Useful tools

- Your mind
- Your voice
- A studio setup with mic included
- https://elevenlabs.io/

## Jump into n8n

Create a workflow called whatever you want but I suggest ie: `TTS`

### Add a node - `Get Script`

- Let's add an `Airtable - Search Records` node
  - Go to the Builder Hub in Airtable and create an access token
    - Name: `aaiva_n8n`
    - Scopes:
      - data.records:read
      - data.records:write
      - schema.bases:read
    - Access:
      - Add a Base: `AAIVA`
- Add the token into n8n
  - Call the token something that helps you know why you created it
    - ie: `Airtable AAIVA Auth`
- Configure the node
  - Filter by Formula: `{Active} = '1'`
    - FYI: https://support.airtable.com/docs/formula-field-reference
  - Limit: 1
- And "Test step" the node
- Rename the node: `Get Script`

### Add a node - `ElevenLabs TTS`

- Let's add an `HTTP Request` node
- REFERENCE: https://elevenlabs.io/docs/api-reference/text-to-speech/convert?explorer=true
- Configure the node
  - Method: POST
  - URL: https://api.elevenlabs.io/v1/text-to-speech/{VOICE_ID_HERE}?output_format=mp3_44100_128
    - https://elevenlabs.io/docs/api-reference/text-to-speech/convert?explorer=true
    - Copy the voice id from the Eleven Labs dashboard
      - https://elevenlabs.io/app/home
      - https://elevenlabs.io/app/voice-library
  - Authentication: Generic Credential Type
  - Generic Auth Type: Header Auth
    - Configure the auth
      - https://elevenlabs.io/app/settings/api-keys
      - `aaiva_n8n`
      - `ElevenLabs AAIVA Auth`
  - Enable `Send Body`
    - JSON
    - Specify Body
    ```json
{
  "text": "{{ $json.Script }}",
  "model_id": "eleven_multilingual_v2"
}
    ```

    - Find the `model_id` param here: https://elevenlabs.io/docs/models#models-overview
  - And "Test step" the node
- Rename the node: `ElevenLabs TTS`

### Add a node - `Upload Audio to Dropbox`

- Add the `Dropbox - Upload a file` node
- Configure the node
  - OAuth2
  - APP Access Type: `Full Dropbox`
  - Resource: File
  - Operation: Upload
  - File Path: `projects/aaiva_n8n/example_1/script_{{ $now.toMillis() }}`
  - Enable Binary File
  - Input Binary Field
    - `data` from previous node
- And "Test step" the node
- Rename the node: `Upload Audio to Dropbox`

### Add a node - `Create Dropbox Share Link`

- Peep the `additional_info/configure_dropbox_oauth2_token.md` file
- Configure the node
  - Import the curl from here into the node: https://www.dropbox.com/developers/documentation/http/documentation#sharing-create_shared_link_with_settings
```sh
curl -X POST https://api.dropboxapi.com/2/sharing/create_shared_link_with_settings \
    --header "Authorization: Bearer <get access token>" \
    --header "Content-Type: application/json" \
    --data "{\"path\":\"/Prime_Numbers.txt\",\"settings\":{\"access\":\"viewer\",\"allow_download\":true,\"audience\":\"public\",\"requested_visibility\":\"public\"}}"
```
  - Select the `Dropbox OAuth2 Refresh Token Cred AAIVA`
  - Enable `Send body`
```json
{
  "path": "{{ $json.path_lower }}",
  "settings": {
    "access": "viewer",
    "allow_download": true,
    "audience": "public",
    "requested_visibility": "public"
  }
}
```
- Rename the node: `Create Dropbox Share Link`

### Add a node - `Edit Fields (Set)`

- Mode: Manual Mapping
- Fields to Set
  - direct_dropbox_url: `{{$json["url"].replace("dl=0", "dl=1")}}`
- Rename the node: `Edit link to file`

### Add a node - `Update record in Airtable`

- Select the appropriate credential (ie: `Airtable AAIVA Auth`)
- Resource: Record
- Operation: Update
- Base: AAIVA
- Table: Scripts
- Mapping Column Mode: Map Each Column Manually
- Columns to match on: ID
  - {{ $('Fetch script from Airtable').item.json.ID }}
- Spoken Audio:
```json
[
    {
        "url": "{{ $json.direct_dropbox_url }}",
        "filename": "{{ $('Get link to file in Dropbox').item.json.name }}"
    }
]
```
- Spoken Audio URL: `{{ $json.direct_dropbox_url }}`

## FREE Workaround

If you have a microphone on your phone or laptop, try recording the script yourself.