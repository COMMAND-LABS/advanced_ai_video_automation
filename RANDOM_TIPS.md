# TLDR

Some random but useful tips

## How to customize the subtitles (aka audio captions) with the JSON2Video API 

- https://json2video.com/docs/v2/api-reference/json-syntax/element/subtitles

```sh
export JSON2VIDEO_PROJECT_ID=ttQeEoXDQ94TfWbX
export JSON2VIDEO_API_KEY=<API_KEY_HERE>
curl --location --request GET "https://api.json2video.com/v2/movies?project=$JSON2VIDEO_PROJECT_ID" \
--header "x-api-key: $JSON2VIDEO_API_KEY"
```

## Chris Johnston's super advanced AI Video Automation resources

- frame2frame Video Generation
  - Good for connecting AI generated video clips
  - https://fal.ai/models/fal-ai/framepack/flf2v
  - https://fal.ai/models/fal-ai/wan-flf2v
- Finetune Flux on images of your choosing
  - https://fal.ai/models/fal-ai/flux-lora-fast-training
  - https://app.leonardo.ai/models-and-training
