---
name: generate-image
description: Generate an image using Google's Gemini image generation API based on a text prompt.
user-invocable: true
argument-hint: [prompt]
allowed-tools: Bash, Read
---

# Generate Image with Gemini

The user wants to generate an image. Their prompt is: $ARGUMENTS

Follow these steps:

1. First, load the API key from the `.env` file by running:

```bash
export GEMINI_API_KEY=$(grep GEMINI_API_KEY .env | cut -d'=' -f2)
```

If the key is empty or the file doesn't exist, tell the user to create a `.env` file with `GEMINI_API_KEY=<their_key>` and stop.

2. Generate a short, lowercase, hyphenated filename from the user's prompt (e.g., `sunset-over-ocean.png`). Do not include spaces or special characters.

3. Use `curl` to call the Gemini image generation API:

```bash
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3-pro-image-preview:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "role": "user",
      "parts": [{"text": "<USER_PROMPT>"}]
    }],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"]
    }
  }'
```

Replace `<USER_PROMPT>` with the user's actual prompt from $ARGUMENTS.

4. The response JSON contains `candidates[0].content.parts[]`. Parts can contain either `text` or `inlineData` (with `mimeType` and `data` fields).

5. If any part has a `text` field, display it to the user.

6. For the part with `inlineData`, extract the base64 `data` field and decode it:

```bash
echo "<BASE64_DATA>" | base64 --decode > "<filename>.png"
```

7. Tell the user the saved file path.

8. If the API returns an error, display the error message clearly to the user.
