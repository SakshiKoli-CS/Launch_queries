QUESTION

A customer (`www.monclergroup.com`, Launch UID `68090b64187732a6f60c27a5`) wanted to enable **HTTP/2 on Akamai** for their Launch-hosted site. Akamai stated that a prerequisite for enabling HTTP/2 is that the **origin supports Chunked Transfer Encoding (CTE)**. The question was whether Launch origins support CTE.

ANSWER

**Yes — Launch origins support Chunked Transfer Encoding.**

Launch deployments run on a **Node.js/Next.js runtime** that supports **HTTP/1.1 streaming responses**. When a response is streamed without a `Content-Length` header, the runtime automatically uses `Transfer-Encoding: chunked`. This has been verified by testing the Launch origin endpoint directly over HTTP/1.1.

**How HTTP/2 + CTE works in this architecture**

In a typical CDN setup with Akamai in front of Launch:

- **Client → Akamai**: HTTP/2 (this is the connection HTTP/2 is being enabled for)
- **Akamai → Launch origin**: HTTP/1.1 with `Transfer-Encoding: chunked` (CTE) for streamed responses

Akamai handles the protocol translation — it receives chunked HTTP/1.1 responses from the Launch origin and converts them into HTTP/2 frames for delivery to the client. The CTE requirement applies to the **origin → Akamai** leg, which Launch satisfies natively.

**No additional configuration is required on the Launch side** to enable HTTP/2 delivery through Akamai. The origin meets Akamai's prerequisite out of the box.

**Related**

- For general Akamai + Launch CDN compatibility, see @networking-connectivity/launch-akamai-cdn.md
- Note: a `Transfer-Encoding: chunked` response without a chunk terminator can cause proxy timeouts (524/520). See @networking-connectivity/launch-524-520-login-redirect-chunked-no-terminator-content-length.md for that separate issue.

**Resolution**

Confirmed and communicated to customer. Issue resolved; case closed.
