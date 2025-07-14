# TLDR

Now we will compile all the media we have created with JSON2Video

## Table of Contents

1. [TLDR](#tldr)
2. [Table of Contents](#table-of-contents)
3. [Jump into n8n](#jump-into-n8n)
  - [Add a node - Get script](#add-a-node---get-script)
  - [Add a node - Get segments](#add-a-node---get-segments)
  - [Add a node - Configure JSON2Video Scene](#add-a-node---configure-json2video-scene)
    - [Additional JSON2Video snippets](#additional-json2video-snippets)
  - [Add a node - Aggregate](#add-a-node---aggregate)
  - [Add a node - Stitch Video Together](#add-a-node---stitch-video-together)

## Jump into n8n

- Create a workflow called whatever you want but I suggest ie: `JSON2Video`

### Add a node - `Get script`

Copy/Tweak from prior workflows

### Add a node - `Get segments`

Copy/Tweak from prior workflows

### Add a node - `Configure JSON2Video Scene`

```js
{
  "comment": "Scene",
  "elements": [
    {{ $json['Make Video'] ? {
      "type": "video",
      "src": $json.Video[0].url,
      "position": "center-center",
      "duration": $json['End Time'] - $json['Start Time'] 
    } : {
      "type": "image",
      "src": $json.Image[0].url,
      "zoom": -1,
      "position": "center-center",
      "duration": $json['End Time'] - $json['Start Time'] 
    }
    }}
  ]
}
```

#### Additional JSON2Video snippets

- `json2video_snippets/snippet_1.md`
- `json2video_snippets/snippet_2.md`

### Add a node - `Aggregate`

- Configure the node:
  - Aggregate: All Item Data (Into a Single List)
  - Put Output in Field: scenes
  - Include: All Fields

### Add a node - `Stitch Video Together`

```sh
curl --location --request POST 'https://api.json2video.com/v2/movies' \
--header 'x-api-key: YOUR_API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "scenes": [
        {
            "elements": [
                {
                    "type": "video",
                    "src": "https://example.com/path/to/my/video.mp4"
                }
            ]
        }
    ]
}'
```

TIP: Check JSON2Video for rendering progress at `https://json2video.com/dashboard/renders/`
