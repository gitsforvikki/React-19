# Lesson 71 — React Security Fundamentals ⭐⭐⭐⭐⭐

## 1. The Most Important Security Rule

A React application runs in an environment controlled by the user.

Users can:

- inspect JavaScript
- modify browser state
- call APIs directly
- change request payloads
- manipulate client-side code
- bypass hidden/disabled UI

Therefore:

> Never treat the React client as a trusted security boundary.

```text
React UI
   │
   │ untrusted input/request
   ↓
SERVER / TRUST BOUNDARY
   │
   ├── authenticate
   ├── authorize
   ├── validate
   ├── enforce business rules
   └── return allowed data
```

This is the core mental model for frontend security.

---

# Part 1 — Authentication vs Authorization

## 2. Authentication

Authentication answers:

```text
Who are you?
```

Examples:

- session cookie
- access token
- server session

---

## 3. Authorization ⭐⭐⭐⭐⭐

Authorization answers:

```text
Are you allowed to do this?
```

Example:

```text
user is logged in
≠
user may delete every product
```

Authorization must be enforced by the server/API.

---

## 4. Hiding UI Is Not Authorization ⭐⭐⭐⭐⭐

React:

```jsx
{user.role === "admin" && (
  <DeleteProductButton />
)}
```

This is useful UX.

But an attacker can call:

```text
DELETE /api/products/123
```

without using your button.

Server must independently verify:

```text
authenticated?
+
authorized for this resource/action?
```

---

# Part 2 — Client Validation vs Server Validation

## 5. Client Validation

Useful for UX:

```jsx
if (!email.includes("@")) {
  setError(
    "Enter a valid email"
  );
}
```

It gives fast feedback.

---

## 6. Server Validation ⭐⭐⭐⭐⭐

Every untrusted value must still be validated server-side.

Attackers can bypass React entirely.

```text
React form validation
      ↓
good UX

Server validation
      ↓
security/correctness boundary
```

Both can exist, but they serve different purposes.

---

# Part 3 — XSS

## 7. What Is XSS? ⭐⭐⭐⭐⭐

Cross-Site Scripting occurs when attacker-controlled content is interpreted as executable script/content in a trusted page.

Potential impact:

- perform actions as user
- read data accessible to JavaScript
- manipulate UI
- redirect/phish users
- steal non-HttpOnly secrets

---

## 8. React Escapes Rendered Text by Default ⭐⭐⭐⭐⭐

Suppose:

```jsx
const name =
  '<script>alert("x")</script>';

return (
  <p>{name}</p>
);
```

React normally treats this as text rather than injecting it as executable HTML.

This default escaping is an important protection.

Do not defeat it unnecessarily.

---

# Part 4 — dangerouslySetInnerHTML

## 9. Why It Is Dangerous ⭐⭐⭐⭐⭐

```jsx
<div
  dangerouslySetInnerHTML={{
    __html: content,
  }}
/>
```

This tells React:

```text
Treat this string as HTML.
```

If `content` contains attacker-controlled unsafe HTML, XSS can occur.

---

## 10. Sanitization

If your product genuinely needs user-controlled rich HTML:

```text
untrusted HTML
      ↓
trusted, well-maintained
HTML sanitizer
      ↓
sanitized HTML
      ↓
render
```

Do not attempt to sanitize complex HTML using a few regular expressions.

Escaping and sanitizing are different operations.

---

# Part 5 — URLs

## 11. Untrusted URLs Need Care

Applications may render user-provided links:

```jsx
<a href={profileUrl}>
  Portfolio
</a>
```

Validate/allow expected URL schemes and destinations according to your product requirements.

Do not assume a string is safe merely because it came from your database.

Stored data may still be attacker-controlled.

---

# Part 6 — Secrets

## 12. Frontend Secrets Do Not Exist ⭐⭐⭐⭐⭐

Anything shipped to browser JavaScript can be inspected.

Never place true secrets in client bundles:

- database passwords
- payment secret keys
- private API credentials
- signing secrets
- webhook secrets

```text
build-time client variable
        ↓
browser bundle
        ↓
USER CAN INSPECT IT
```

A variable name containing:

```text
SECRET
PRIVATE
```

does not make it private.

---

## 13. Public Keys vs Secret Keys

Some services intentionally provide public/client keys.

Those are designed to be exposed.

Server secret keys must remain server-side.

Always follow the provider's security model.

---

# Part 7 — Tokens and Browser Storage

## 14. localStorage Security Tradeoff ⭐⭐⭐⭐⭐

Values in `localStorage` are accessible to JavaScript running on the page.

Therefore XSS can potentially read them.

Do not casually store high-value authentication credentials there.

---

## 15. HttpOnly Cookies

An `HttpOnly` cookie cannot be read through normal client-side JavaScript.

This reduces token theft through JavaScript access.

But cookies introduce other concerns such as:

- CSRF
- SameSite policy
- Secure transport
- session management

No storage mechanism removes the need for a complete security design.

---

# Part 8 — Cookie Attributes

## 16. Important Cookie Protections

Common attributes include:

### HttpOnly

```text
JavaScript cannot read cookie
```

### Secure

```text
send over HTTPS
```

### SameSite

Controls cross-site cookie sending behavior.

Exact settings depend on deployment/auth architecture.

Do not copy cookie configuration blindly.

---

# Part 9 — CSRF

## 17. What Is CSRF? ⭐⭐⭐⭐⭐

Cross-Site Request Forgery tricks a browser into sending an authenticated request to a site where the user is already logged in.

This is especially relevant when authentication credentials such as cookies are sent automatically by the browser.

Protections can include:

- appropriate SameSite cookies
- CSRF tokens where needed
- Origin/Referer validation as part of server strategy
- avoiding unsafe state changes through GET
- framework/platform protections

React itself does not solve CSRF.

---

# Part 10 — CORS

## 18. CORS Is Not Authorization ⭐⭐⭐⭐⭐

CORS controls which origins browser JavaScript is allowed to read/interact with under browser cross-origin rules.

It does **not** prove that a caller is authorized.

Attackers can call APIs outside your frontend/browser UI.

```text
CORS
≠ authentication
≠ authorization
```

Your API still needs proper security.

---

# Part 11 — HTTPS

## 19. Use HTTPS in Production

HTTPS protects data in transit and is required for many secure browser capabilities.

Do not send sensitive credentials over plaintext HTTP in production.

---

# Part 12 — IDOR / Broken Object-Level Authorization

## 20. Never Trust Resource IDs ⭐⭐⭐⭐⭐

Client sends:

```text
GET /api/applications/123
```

Attacker changes it:

```text
GET /api/applications/124
```

Server must verify that the authenticated user may access resource 124.

Never rely on:

```text
"The UI only shows IDs owned by this user."
```

Attackers can construct requests themselves.

---

# Part 13 — Mass Assignment

## 21. Do Not Trust Entire Client Objects

Dangerous conceptual server logic:

```js
updateUser(
  request.body
);
```

Attacker may add:

```json
{
  "name": "User",
  "role": "admin"
}
```

Server should allow only fields the operation is permitted to modify.

Client-side forms are not a whitelist.

---

# Part 14 — Payment Security

## 22. Never Trust Client Totals ⭐⭐⭐⭐⭐

Bad flow:

```text
React sends:
total = ₹1
      ↓
server charges ₹1
```

An attacker can modify the payload.

Correct mental model:

```text
client sends item IDs/quantities
          ↓
server loads authoritative products
          ↓
server validates stock/pricing/discounts
          ↓
server calculates final amount
          ↓
payment/order flow
```

The browser may display totals, but the server must make authoritative money decisions.

---

## 23. Payment Verification

A successful client redirect/message is not necessarily proof of payment.

Payment systems typically require server-side verification according to the provider's protocol, such as signed responses/webhooks or server API verification.

Secret verification credentials remain server-side.

---

# Part 15 — File Uploads

## 24. Client accept Is Not Security

```jsx
<input
  type="file"
  accept="image/*"
/>
```

This helps the file picker UX.

It does not securely validate the uploaded file.

Server should enforce appropriate:

- authentication/authorization
- size limits
- type/content validation
- safe storage/naming
- processing policies

depending on the application.

---

# Part 16 — Dependency Security

## 25. Third-Party Code Is Part of Your Attack Surface

Packages execute in your application/build environment.

Good practices:

- minimize unnecessary dependencies
- keep dependencies maintained
- review security advisories
- use lockfiles
- investigate audit findings
- remove abandoned/unneeded packages
- avoid blindly running unknown install scripts/code

Do not assume every package is trustworthy because it is popular.

---

# Part 17 — Supply Chain

## 26. Package Names Matter

Attackers may publish similarly named packages.

Before installing:

```text
verify exact package
verify official documentation
verify maintainer/source
```

Do not copy installation commands blindly from random posts.

---

# Part 18 — Environment Variables

## 27. Environment Variables Are Not Automatically Secret

Server-only environment variables can hold secrets.

But variables injected into client code become public.

Mental model:

```text
Does this value reach
browser JavaScript?
      │
     YES
      ↓
assume user can see it
```

Framework-specific public-variable naming conventions belong in the relevant framework repository.

---

# Part 19 — Source Maps and Client Code

## 28. Never Depend on Obscurity

Minification does not secure business logic.

Users can inspect network requests and reverse engineer client behavior.

Do not put security decisions in React and assume:

```text
"Nobody will find this code."
```

Security must survive full knowledge of client implementation.

---

# Part 20 — Sensitive Data Exposure

## 29. Only Send Data the Client Is Allowed to See ⭐⭐⭐⭐⭐

Bad:

```text
API returns all user fields
React hides sensitive fields
```

The hidden values are still present in the network response.

Better:

```text
server selects allowed fields
      ↓
client receives only
what it may access
```

CSS hiding is not data protection.

---

# Part 21 — Logging

## 30. Do Not Leak Secrets Through Logs

Avoid logging:

- passwords
- tokens
- payment secrets
- sensitive personal data unnecessarily

Client console logs are visible to users.

Production logs also require secure handling.

---

# Part 22 — Open Redirects

## 31. Validate Redirect Destinations

If an application accepts:

```text
?next=...
```

and redirects without validation, attackers may abuse trusted domains for phishing flows.

Allow only expected destinations according to application requirements.

---

# Part 23 — Tabnabbing / External Links

## 32. New Tabs

Modern browsers provide protections around `target="_blank"`, but explicitly using an appropriate `rel` policy can communicate intent and support compatibility/security requirements.

Example:

```jsx
<a
  href={safeUrl}
  target="_blank"
  rel="noopener noreferrer"
>
  Portfolio
</a>
```

The more important rule remains validating untrusted destinations.

---

# Part 24 — Content Security Policy

## 33. CSP as Defense in Depth

Content Security Policy can restrict which scripts/resources the browser may execute/load.

A strong CSP can reduce XSS impact.

But CSP does not replace:

- safe rendering
- sanitization
- server validation
- authorization

It is defense in depth.

Configuration is deployment-specific.

---

# Part 25 — Security Headers

## 34. Browser Security Controls

Production deployments may use headers/policies for concerns such as:

- content security
- framing/clickjacking
- MIME handling
- referrer information
- transport security

These are server/deployment responsibilities, not JSX features.

React developers should still understand that frontend security extends beyond components.

---

# Part 26 — Race Conditions and Business Security

## 35. Client Button Disabling Is Not Enough

Suppose checkout disables:

```jsx
<button
  disabled={isPending}
>
  Buy
</button>
```

An attacker can still send concurrent requests.

The server/database must enforce invariants such as:

- stock cannot go below zero
- coupon limits
- one-time actions
- idempotency where required

Frontend pending state is UX, not concurrency security.

---

# Part 27 — Optimistic UI

## 36. Optimistic UI Is Not Authorization

React may optimistically display:

```text
Connected
```

before server confirmation.

Server remains authoritative.

On rejection:

```text
optimistic state
→ reconcile with server result
```

Never assume optimistic state proves permission or success.

---

# Part 28 — Error Messages

## 37. Avoid Excessive Internal Detail

User-facing error:

```text
Unable to complete request.
```

Internal diagnostic systems may record appropriate details.

Do not expose:

- stack traces
- database queries
- filesystem paths
- secret values
- internal service credentials

through production UI.

---

# Part 29 — CodeBuddy Example

## 38. Connection Request Security

Client:

```text
click Connect
→ POST targetUserId
```

Server must validate:

```text
session valid?
target exists?
not connecting to self?
already connected/requested?
user allowed?
rate/abuse rules?
```

React disabling the button is only UX.

---

# Part 30 — ShopHub Example

## 39. Checkout Security ⭐⭐⭐⭐⭐

```text
React cart
   ↓
item IDs + quantities
   ↓
SERVER
├── authenticate
├── load products
├── validate inventory
├── calculate prices
├── calculate discount/tax/shipping
├── enforce authorization
└── create trusted payment/order amount
   ↓
payment provider
   ↓
server-side verification
   ↓
finalize order
```

Never trust a client-supplied final amount.

---

# Part 31 — CareerLoop Example

## 40. Application Ownership

Client requests:

```text
PATCH /applications/123
```

Server must verify:

```text
application 123
belongs to / is editable by
current authenticated user
```

A hidden edit button is not sufficient.

---

# Part 32 — Security Checklist

## 41. Before Shipping ⭐⭐⭐⭐⭐

Ask:

```text
1. Are all privileged actions authorized server-side?
2. Is all untrusted input validated server-side?
3. Are true secrets absent from client bundles?
4. Is untrusted HTML avoided or safely sanitized?
5. Are user-provided URLs handled safely?
6. Are cookies/tokens stored with a deliberate security model?
7. Is CSRF addressed where relevant?
8. Am I mistakenly treating CORS as authorization?
9. Does the server verify ownership for resource IDs?
10. Does the server whitelist writable fields?
11. Are money and inventory authoritative server-side?
12. Are uploads validated server-side?
13. Are sensitive fields omitted from API responses?
14. Are dependencies maintained/reviewed?
15. Are errors/logs free of unnecessary secrets?
16. Are concurrent requests safe server-side?
17. Are security headers/policies configured appropriately?
```

---

# Part 33 — Common Mistakes

## 42. Common Security Mistakes ⭐⭐⭐⭐⭐

1. Treating hidden React UI as authorization.
2. Trusting client validation.
3. Exposing secret environment variables to the browser.
4. Storing sensitive tokens casually in localStorage.
5. Using `dangerouslySetInnerHTML` with unsanitized content.
6. Writing homemade HTML sanitization with regex.
7. Treating CORS as authentication/authorization.
8. Returning sensitive data and merely hiding it in UI.
9. Trusting client-provided prices.
10. Trusting client-provided roles/ownership.
11. Accepting arbitrary update fields.
12. Trusting file input `accept`.
13. Treating successful client payment UI as payment verification.
14. Relying on minification/obscurity.
15. Logging secrets.
16. Ignoring dependency/supply-chain risk.
17. Assuming disabled buttons prevent duplicate malicious requests.
18. Forgetting server-side object ownership checks.
19. Showing detailed internal errors in production.
20. Assuming React alone solves web security.

---

# Part 34 — Interview Questions

## 43. Is React Secure Against XSS? ⭐⭐⭐⭐⭐

React escapes normal interpolated text by default, which provides important XSS protection.

However, XSS is still possible through unsafe patterns such as rendering unsanitized HTML, unsafe third-party code, vulnerable dependencies, or other browser injection paths.

---

## 44. Why Is dangerouslySetInnerHTML Dangerous?

Because it renders a string as HTML. If attacker-controlled HTML is not properly sanitized, executable content may be introduced.

---

## 45. Can You Store Secrets in React Environment Variables?

Not if the value is included in browser-delivered code.

Anything delivered to the client should be considered visible to the user.

---

## 46. Authentication vs Authorization?

Authentication verifies identity.

Authorization verifies whether that identity may perform a specific action or access a resource.

---

## 47. Is Hiding an Admin Button Enough?

No.

The server must authorize the corresponding API operation independently.

---

## 48. Is CORS a Security Boundary for API Authorization?

No.

CORS is a browser cross-origin policy, not a replacement for authentication or authorization.

---

## 49. localStorage vs HttpOnly Cookie?

`localStorage` is accessible to page JavaScript, so XSS can potentially read its values.

HttpOnly cookies are not readable through normal JavaScript, but cookie-based authentication must also address CSRF, SameSite, Secure transport and session design.

---

## 50. Why Recalculate Prices on the Server?

The client is attacker-controlled and request values can be modified. The server must use authoritative product/pricing data.

---

# Part 35 — Interview-Ready Answer

## 51. Final Answer ⭐⭐⭐⭐⭐

```text
I treat the React client as untrusted. Client-side
validation, hidden buttons, disabled controls and route
guards improve UX, but they are not security boundaries.

The server must authenticate, authorize, validate input,
verify resource ownership and enforce business rules.
Sensitive data and secret keys must never be shipped to
the browser.

For XSS, I rely on React's default text escaping and
avoid rendering raw HTML. If rich untrusted HTML is
required, I use a trusted sanitizer rather than custom
regex logic.

I also consider token/cookie storage, CSRF, unsafe URLs,
CORS misconceptions, file uploads, dependency security,
data exposure and server-authoritative payment and
inventory calculations.

My core rule is that an attacker may bypass the React
UI completely, so every privileged operation must remain
secure when called directly.
```

---

# Part 36 — Complete Mental Model

## 52. React Security ⭐⭐⭐⭐⭐

```text
                  USER / BROWSER
                        │
              fully attacker-controlled
                        │
                        ↓
                   React Client
           ┌────────────┼────────────┐
           ↓            ↓            ↓
       validation     UI guards   local state
          UX             UX           UX
           └────────────┼────────────┘
                        │
                 untrusted request
                        ↓
              ═══ TRUST BOUNDARY ═══
                        ↓
                     SERVER
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
     authenticate    authorize      validate
          │             │             │
          └─────────────┼─────────────┘
                        ↓
              business invariants
                        ↓
              database / services


Browser safety:
normal React text → escaped
raw HTML → dangerous unless sanitized

Secrets:
server only → can be secret
browser bundle → public

Core principle:
NEVER TRUST THE CLIENT
```

---

## 53. Key Takeaways

- Treat the browser/React client as untrusted.
- Authentication and authorization are different.
- Authorization belongs on the server.
- Hiding UI is UX, not security.
- Client validation improves UX; server validation enforces trust.
- React escapes ordinary text rendering by default.
- Unsanitized `dangerouslySetInnerHTML` can create XSS risk.
- Use trusted sanitization for genuinely required untrusted rich HTML.
- Validate untrusted URLs according to application policy.
- True secrets must never be shipped to browser code.
- localStorage is readable by JavaScript.
- HttpOnly cookies reduce JavaScript token access but require a complete cookie/CSRF strategy.
- CORS is not authorization.
- Verify object/resource ownership server-side.
- Whitelist fields users are permitted to modify.
- Prices, discounts, inventory and payment decisions must be authoritative server-side.
- File picker restrictions are not server validation.
- Dependencies are part of the attack surface.
- Environment variables are secret only if they stay outside client code.
- Send only data the user is allowed to receive.
- Client pending states do not enforce server concurrency rules.
- Optimistic UI never proves authorization or success.
- Keep internal diagnostics/secrets out of user-facing errors.
- React security is one layer of broader web/application security.

---

## Next Lesson

➡️ [Lesson 72 — Testing React Components](./72-testing.md)
