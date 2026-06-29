QUESTION

An internal service (`GROWTH-ONECT`, org `blt3a74003aa26c445e`) creating Launch projects via GraphQL (`importProject`) on **azure-stag-na** received a `200` status with `data: null` and the following error:

```json
{
  "errors": [{
    "message": "Bad request",
    "extensions": {
      "exception": {
        "errorObject": { "uploadUid": [{ "code": "launch.DEPLOYMENT.INVALID_FILE_TYPE" }] }
      }
    }
  }],
  "data": null
}
```

The ZIP file upload step returned `201` successfully, but project creation failed. The **same service and commit worked on aws-stag** without changes.

ANSWER

**Root cause — `Content-Type` header mismatch on Azure/GCP when uploading via form-data**

When uploading the ZIP file using `multipart/form-data`, the request carries:

```
Content-Type: multipart/form-data; boundary=<...>
```

On **AWS**, this works because the platform strips or ignores the multipart Content-Type before passing the file to the project creation API. On **Azure (and GCP)**, the `multipart/form-data` Content-Type is forwarded as-is, causing the API to read the file type as `multipart/form-data` rather than `application/zip` — triggering:

> *"File type is not allowed. Only zip files are allowed. Found: multipart/form-data"*

This is why the ZIP upload returns 201 (it was accepted at the upload layer) but project creation fails (the creation API then rejects the uploaded file's reported type).

**Fix — upload as raw binary instead of form-data (Azure and GCP only)**

Change the ZIP upload method to send the file as **raw binary** rather than `multipart/form-data`. This ensures the request reaches the project creation API with `Content-Type: application/zip`, which the API accepts.

```
# form-data (fails on Azure/GCP)
Content-Type: multipart/form-data; boundary=...

# raw binary (correct for Azure/GCP)
Content-Type: application/zip
```

No changes are required for AWS regions where form-data upload works.

**Diagnostic pattern**

- ZIP upload step: `201 Created` ✓
- GraphQL `importProject`: `200` with `data: null` + `launch.DEPLOYMENT.INVALID_FILE_TYPE` ✗
- Same code works on AWS but fails on Azure/GCP → Content-Type propagation difference between clouds
