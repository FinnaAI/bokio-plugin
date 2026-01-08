---
description: Upload a receipt or invoice to Bokio
---

# Bokio Upload

Upload a receipt or invoice to Bokio accounting.

If the user provided a file path as an argument, use that. Otherwise, ask what they want to upload.

Use the bokio-upload skill instructions to:
1. Load credentials from `~/dev/finna/bokio-finna/.env`
2. Upload the file via the Bokio API
3. Confirm the upload with the returned ID

Supported formats: PDF, JPEG, PNG only.
