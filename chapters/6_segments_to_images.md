# TLDR

We will now convert these segments into images aka we sort of storyboard our script

## Table of Contents

1. [TLDR](#tldr)
2. [Table of Contents](#table-of-contents)
3. [Jump into n8n](#jump-into-n8n)
  - [Add a node - Get script](#add-a-node---get-script)
  - [Add a node - Get segments](#add-a-node---get-segments)
  - [Add a node - OpenAI: Generate Image Prompts](#add-a-node---openai-generate-image-prompts)
  - [Add several nodes for doing the image gen](#add-several-nodes-for-doing-the-image-gen)
  - [After polling loop add node - `HTTP Request`](#after-polling-loop-add-node---http-request)
  - [Download image](#after-polling-loop-add-node---http-request)
  - [Add node - `Dropbox: Upload a file`](#add-node---dropbox-upload-a-file)
  - [Add node - `Get link to file in Dropbox`](#add-node---get-link-to-file-in-dropbox)
  - [Add node - `Edit link to file`](#add-node---edit-link-to-file)
  - [Add node - `Airtable: Update record`](#add-node---airtable-update-record)
4. [A](#helper-a)
  - [i](#helper-a-reference-urls)
  - [ii](#get-the-record-id-for-the-miscellaneous-table-row-from-the-record-url)
  - [iii](#add-a-node---get-miscellaneous-init-image)
  - [iv](#add-a-node---get-presigned-url)
  - [v](#add-a-node---download-init-image-from-airtable)
  - [vi](#add-a-node-http-request-node---upload-init-image)
  - [vii](#finally)
5. [B](#helper-b)
  - [i](#http-request---gen-image-with-leonardoai)


## Jump into n8n

- Create a workflow called whatever you want but I suggest ie: `Segments to Images`

### Add a node - `Get script`

Copy from prior workflows

### Add a node - `Get segments`

Copy from prior workflows

### Add a node - `OpenAI: Generate Image Prompts`

- Let's add an `OpenAI: Message Model` node
- Configure the node
  - Credential to connect with: Create a credential
  - Model: GPT-4O
  - Customize Prompt
```txt
# TASK

Create an image prompt for an #short form viral video created using AI Video Automation techniques. Your task is to create an image prompt that will produce a beautiful and aesthetically pleasing image for a section of the overall script. The image prompt text should contain no special chars and be ready for use by a image-to-image model.

## CURRENT SECTION OF SCRIPT

{{ $json['Script Segment'] }}

## OVERALL SCRIPT

{{ $('Get script').item.json.Script }}

## JSON OUTPUT RESPONSE FORMAT

{
  "imagePrompt": string;
}
```
  - Output Content as JSON: Enable

### Add several nodes for doing the image gen

- Nodes are: 
  - HTTP Request node ie: `Gen image with Leonardo.ai`
    - Method: POST
    - URL: `https://cloud.leonardo.ai/api/rest/v1/generations`
    - Headers:
      - accept: application/json
      - authorization: Bearer <API_TOKEN_HERE>
    - Body:
      - modelId: `b2614463-296c-462a-9586-aafdb8f00e36` (https://docs.leonardo.ai/docs/commonly-used-api-values)
      - contrast: `{{Number.parseFloat(3.5)}}`
      - prompt: `{{ $json.message.content.imagePrompt }}`
      - num_images: `{{Number.parseInt(1)}}`
      - width: `{{Number.parseInt(864)}}`
      - height: `{{Number.parseInt(1536)}}`
      - styleUUID: `111dc692-d470-4eec-b791-3475abac4c46` (https://docs.leonardo.ai/docs/generate-images-using-leonardo-phoenix-model)
      - enhancePrompt: `{{Boolean(false)}}`
  - HTTP Request node ie: `Check image gen status`
  - If node
    - If status == COMPLETE continue
    - Else Wait and call the `Gen image with Leonardo.ai` again
  - Wait node

- REFERENCE: https://docs.leonardo.ai/docs/getting-started
- REFERENCE: https://docs.leonardo.ai/docs/generate-images-using-flux
- REFERENCE: https://docs.leonardo.ai/reference/creategeneration
- REFERENCE: https://docs.leonardo.ai/docs/commonly-used-api-values

#### LOG

- Simple test first: https://docs.leonardo.ai/docs/generate-images-using-flux
- Get an API key from the Leonardo.ai dashboard
- Test this curl to do an image generation (https://docs.leonardo.ai/docs/generate-images-using-flux)
  - Will kick off a generation
  - TIP: Needed to cast the non-string json body types to work
  - Use Leonardo dashboard for verifying things are working
- Call this to check on the image generation (https://docs.leonardo.ai/reference/getgenerationbyid)
  - Implements the "polling loop"

### After polling loop add node - `HTTP Request`

- Downloads gen'd img from Leonardo.ai
- Method: GET
- URL: Link to img provided by Leonardo.ai

### Add node - `Dropbox: Upload a file`

- Resource: File
- Operation: Upload
- File Path: Where in Dropbox you want the file to sit
- Input Binary Field: data

### Add node - `Get link to file in Dropbox`

- Use the `Dropbox OAuth2 Refresh Token Cred AAIVA` header
  - Can always reconnect if needed
- URL: `https://api.dropboxapi.com/2/sharing/create_shared_link_with_settings`
- JSON
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

### Add node - `Edit link to file`

- direct_dropbox_url: `{{$json["url"].replace("dl=0", "dl=1")}}`

### Add node - `Airtable: Update record`

- Base: AAIVA
- Table: Segments
- Columns to match on: ID
- Image
```js
[
  {
    "url": "{{ $json.direct_dropbox_url }}",
    "filename": "{{ $('Get link to file in Dropbox').item.json.name }}"
  }
]
```
- Full Script: `["{{ $('Get script').item.json.id }}"]`
- Remaining fields are straightforward

## HELPER A

Exploring simple fine-tuning techniques

### Helper A Reference URLs

- https://docs.leonardo.ai/reference/uploadinitimage

### Get the record id for the `Miscellaneous` table row from the Record URL

ie: `recb44l2A77iPHrYv`

### Add a node - `Get Miscellaneous Init Image`

### Add a node - `Get presigned URL`

```sh
curl -X POST https://cloud.leonardo.ai/api/rest/v1/init-image \
  -H "accept: application/json" \
  -H "content-type: application/json" \
  -H "authorization: Bearer <YOUR_API_KEY>" \
  -d '{"extension": "jpg"}'
```

###  Add a node - `Download Init Image from Airtable`

### Add a node `HTTP Request Node - Upload Init Image`

- Configure the node
  - Method: POST
  - URL: `{{ $json.uploadInitImage.url }}`
  - Body Content Type: Form-Data
    - Content-Type: `image/jpeg` (Form Data)
    - bucket: `{{$json.uploadInitImage.fields.parseJson()['bucket']}}` (Form Data)
    - X-Amz-Algorithm: `{{$json.uploadInitImage.fields.parseJson()['X-Amz-Algorithm']}}` (Form Data)
    - X-Amz-Credential: `{{$json.uploadInitImage.fields.parseJson()['X-Amz-Credential']}}` (Form Data)
    - X-Amz-Date: `{{$json.uploadInitImage.fields.parseJson()['X-Amz-Date']}}` (Form Data)
    - X-Amz-Security-Token: `{{$json.uploadInitImage.fields.parseJson()['X-Amz-Security-Token']}}` (Form Data)
    - key: `{{$json.uploadInitImage.fields.parseJson()['key']}}` (Form Data)
    - Policy: `{{$json.uploadInitImage.fields.parseJson()['Policy']}}` (Form Data)
    - X-Amz-Signature: `{{$json.uploadInitImage.fields.parseJson()['X-Amz-Signature']}}` (Form Data)
    - file: data (n8n Binary File)

### Finally

Get image id from the `Get presigned URL`

## HELPER B

Gen images (image-2-image)

### HTTP Request - `Gen image with Leonardo.ai`

- Configure the node
  - Method: POST
  - URL: https://cloud.leonardo.ai/api/rest/v1/generations
  - Headers:
    - accept: application/json
    - authorization: Bearer <API_KEY_HERE>
  - Body:
    - modelId: de7d3faf-762f-48e0-b3b7-9d0ac3a3fcf3 (https://docs.leonardo.ai/docs/commonly-used-api-values)
    - contrast: {{Number.parseFloat(3.5)}}
    - prompt: Man smiling
    - num_images: {{Number.parseInt(1)}}
    - width: {{Number.parseInt(864)}}
    - height: {{Number.parseInt(1536)}}
    - styleUUID: 111dc692-d470-4eec-b791-3475abac4c46 (https://docs.leonardo.ai/docs/generate-images-using-leonardo-phoenix-model)
    - enhancePrompt: {{Boolean(true)}}
    - init_image_id: cc63ed03-c710-462e-a981-c50333c7edc3 (from step_3_HELPER.md)
    - init_strength: 0.9 (Must float between 0.1 and 0.9)
