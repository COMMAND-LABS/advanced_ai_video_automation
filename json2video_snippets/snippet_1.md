# TLDR

A JSON2Video scene snippets that support videos & images (zoomed out via `"zoom": -1`). On movie level, highlighted subtitles based on full length audio are shown.

## Scenes

```js
{
  "comment": "Scene",
  "elements": [
    {{ $json['Make Video'] ? {
      "type": "video",
      "src": $json.Video[0].url,
      "resize": "cover",
      "position": "center-center",
      "duration": $json['End Time'] - $json['Start Time'] 
    } : {
      "type": "image",
      "src": $json.Image[0].url,
      "zoom": -1,
      "resize": "cover",
      "position": "center-center",
      "duration": $json['End Time'] - $json['Start Time'] 
    }
    }}
  ]
}
```

## Movie

```js
{
  "comment": "Video",
  "resolution": "instagram-story",
  "quality": "high",
  "scenes": {{ JSON.stringify($json.scenes) }},
  "elements": [
    {
      "type": "subtitles",
      "settings": {
        "font-family": "Boldonse-Regular",
        "font-url": "https://www.dropbox.com/scl/fi/9x84netesf7acx9d6qml4/Boldonse-Regular.ttf?rlkey=bps6ymww8oe67vcg1f4bl1uqq&st=fp5e1jic&raw=1",
        "box-color": "#8a112d",
        "font-size": 160,
        "word-color": "#FFFFFF",
        "position": "center-center",
        "style": "boxed-word",
        "outline-color": "#000000",
        "outline-width": 2,
        "shadow-color": "#000000",
        "shadow-offset": 4,
        "max-words-per-line": 4
      }
    },
    {
      "type": "audio",
      "src": "{{ $('Fetch script from Airtable').item.json['Narrated Audio URL'] }}"
    }
  ]
}
```