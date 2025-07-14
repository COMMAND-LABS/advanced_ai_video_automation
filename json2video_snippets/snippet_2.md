# TLDR

JSON2Video scene snippets that support videos & images (zoomed out via `"zoom": -1`) with text overlay based on segment. 

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
    }},
    {
      "type": "text",
      "style": "002",
      "text": "{{ $json['Script Segment'] }}",
      "settings": {
        "font-family": "Roboto",
        "font-size": "80px",
        "font-weight": "700",
        "font-color": "#000000",
        "text-align": "center"
      }
    }
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
      "type": "audio",
      "src": "{{ $('Fetch script from Airtable').item.json['Narrated Audio URL'] }}"
    }
  ]
}
```

## Reference links

- https://web.json2video.com/docs/v2/api-reference/json-syntax/element/text
- https://json2video.com/docs/resources/text/