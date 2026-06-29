# Contribution #1: Fix URL encoding for `claims` parameter in self-registration redirect

**Contribution Number:** 1
**Student:** Rafeek Hanna
**Issue:** https://github.com/wso2/product-is/issues/23288
**Status:** Phase IV — Submitted & In Review

## Why I Chose This Issue

The "Back to sign in" link on the WSO2 Identity Server self-registration page fails with a 400 error because the `claims` JSON parameter in the callback URL is not URL-encoded. I chose this issue because it has clear reproduction steps, a bounded scope (a single JSP file), and involves frontend/URL handling which matches my web development background. It has active maintainer engagement and a well-described root cause, making it a solid first contribution.

## Understanding the Issue

### Problem Description
When a user reaches the self-registration page (e.g. via an organization's My Account) and clicks "Back to sign in," the server returns 400 Bad Request. The `callback` URL carries an OIDC `claims` request object — raw JSON with characters like `{`, `}`, `"`, `:` — that are illegal in a URL query string and were never percent-encoded.

### Expected Behavior
Clicking "Back to sign in" should navigate to the login page successfully (HTTP 200), with the `claims` parameter properly percent-encoded.

### Current Behavior
The link points to `.../login.do?claims={"userinfo":...}` with raw braces, so the server rejects the malformed request with HTTP 400.

### Affected Components
- Repo: `wso2/identity-apps` → module `identity-apps-core` → recovery portal (`accountrecoveryendpoint` webapp).
- File: `identity-apps-core/apps/recovery-portal/src/main/webapp/self-registration-with-verification.jsp` — line 107 (reads the callback), line ~918 (renders the "Sign in" link), lines 194–199 (callback validation).

## Reproduction Process

### Environment Setup
The fix lives in a JSP served by WSO2 Identity Server, so I ran the IS locally (not the React apps). I downloaded released IS packs (7.1.0 — the version on the issue — and 7.3.0, the latest), ran them directly, and replaced the single JSP under `repository/deployment/server/webapps/accountrecoveryendpoint/`. Self-registration was enabled via the Identity Governance API and `deployment.toml`.

Challenges I faced and solved:
1. **JDK mismatch** — the project README says "Install JDK 11," but IS 7.3.0 refuses to start below JDK 21. I installed the right JDK per pack (17 for 7.1.0, 21 for 7.3.0) and set `JAVA_HOME`. (This is itself a doc bug in the repo README.)
2. **JSP edits don't hot-reload** — Tomcat caches the compiled servlet in memory, so a dropped-in `.jsp` change only takes effect after a server restart.
3. **Version-dependent flow** — on 7.3.0 the normal flow renders the already-fixed `username-request.jsp`; the buggy `with-verification.jsp` is reached via `signup.do`/the org flow, which I used to render the exact PR file directly.

### Steps to Reproduce
1. Start WSO2 IS 7.1.0 with the correct JDK.
2. Enable self-registration (Console → Login & Registration → Self Registration), reCAPTCHA off, account verification on.
3. Reach the "with verification" sign-up page with a callback carrying a `claims` JSON value.
4. Click "Already have an account? Sign in."
5. Observed result: **HTTP 400** — the URL contains raw `claims={"userinfo":...}`.

### Reproduction Evidence
- **Working branch:** https://github.com/iRAFEEK/identity-apps/tree/fix-url-encoding-claims
- **Pull Request:** https://github.com/wso2/identity-apps/pull/10382
- **Screenshots/recordings:** before/after screen recordings (400 → 200) in the PR description.
- **My findings:** verified at the HTTP level that raw `claims={...}` → 400 while percent-encoded `claims=%7B...` → 200, confirming the root cause is the missing URL-encoding.

## Solution Approach

### Analysis
`Encode.forHtmlAttribute()` makes a string safe inside HTML but does not URL-encode, so the `claims` JSON's `{ } " :` stay raw in the query string. When that callback is rendered into the "Sign in" `href` and clicked, the browser sends an illegal URL → 400. Root cause: the callback's query parameters are never URL-encoded.

### Proposed Solution
Wrap `request.getParameter("callback")` with `IdentityManagementEndpointUtil.encodeURL()` before HTML-attribute encoding, so query values are percent-encoded while the URL structure is preserved. This matches the existing pattern in `self-registration-username-request.jsp:116`.

### Implementation Plan
Using UMPIRE framework (adapted):

**Understand:** A `claims` JSON object ends up raw in the callback URL → illegal query → 400.

**Match:** `self-registration-username-request.jsp:116` already solves the identical problem with `Encode.forHtmlAttribute(IdentityManagementEndpointUtil.encodeURL(...))`. Reuse it.

**Plan:**
- Line 107 — wrap the callback read in `encodeURL()`.
- Lines 194–199 — remove the now-redundant `getURLEncodedCallback()` (it would double-encode, `%7B`→`%257B`); validate the already-encoded value directly (safe — `isValidMultiOptionURI()` checks only scheme/host/path).

**Implement:** final commit `8cd7ff6` (via `ea88714` → `2c7b15d` → `b19a3fd` → `8cd7ff6`).

**Review:** matches the project's existing pattern; added a changeset; CLA + CodeRabbit pass.

**Evaluate:** manual/behavioral verification — HTTP 400→200, rendered-href diff, before/after video.

## Testing Strategy

### Unit Tests
- [ ] N/A — the recovery-portal JSPs have no automated unit-test harness in the repo (server-rendered templates). Verification was manual/behavioral, the appropriate method here (noted honestly rather than inventing tests).

### Integration Tests
- [x] Existing build checks (CLA, CodeRabbit) pass on the PR.

### Manual Testing
- HTTP-level: raw `claims={...}` → 400; encoded `claims=%7B...` → 200.
- Rendered the buggy vs. fixed JSP and confirmed the emitted `<a href>` changed from raw braces to `%7B`/`%22`.
- Recorded a before/after screen recording (400 → 200) on the actual PR file, attached to the PR.

## Implementation Notes

### Week — Build Progress
Implemented the fix in `self-registration-with-verification.jsp` (line 107 + the lines 194–199 cleanup) and added the changeset. CodeRabbit flagged a double-encoding risk in the validation block; I verified `isValidMultiOptionURI()` ignores the query, removed the redundant `getURLEncodedCallback()`, and CodeRabbit re-reviewed and confirmed. An intermediate commit accidentally dropped the line-107 change — I caught it by diffing the committed file against intent and restored it.

### Week — Submit & Iterate Progress
Opened PR #10382, addressed maintainer feedback (concise changeset, before/after recording), and responded to a maintainer question about whether the bug is already fixed on master (see feedback log).

### Code Changes
- Files modified: `identity-apps-core/apps/recovery-portal/src/main/webapp/self-registration-with-verification.jsp`; added `.changeset/fix-self-registration-callback-encoding.md`.
- Key commits: `8cd7ff6` (final; +9/−4 across 2 files).
- Approach decisions: reused the existing `encodeURL` pattern; validated the encoded value directly to avoid double-encoding.

## Pull Request

**PR Link:** https://github.com/wso2/identity-apps/pull/10382

**PR Description:** Purpose → Problem → Solution → Changed File → Verification (with before/after videos). References the issue with `Fixes wso2/product-is#23288`.

**Maintainer Feedback:**
- 2026-05-23: CodeRabbit — validation block could double-encode the callback. → Verified `isValidMultiOptionURI` ignores the query; removed the redundant `getURLEncodedCallback()` (`8cd7ff6`); bot confirmed resolved.
- 2026-05-23: @pavinduLakshan — keep the changeset description concise. → Trimmed to a one-line release note (`8cd7ff6`).
- 2026-05-23: @pavinduLakshan — add a screen recording; address the CodeRabbit comments. → Added before/after recording (400 → 200); CodeRabbit resolved.
- 2026-05-30: @pavinduLakshan — did you test on latest master? the bug may already be fixed there. → Replied honestly: tested released packs (7.1.0/7.3.0); on 7.3.0 the normal flow routes through the already-fixed `username-request.jsp`, so the user-facing 400 doesn't reproduce — the buggy line in `with-verification.jsp` is reachable only via direct render. Offered to close if that page is unused; asked him to confirm.

**Status:** Review-ready; in review (awaiting maintainer approval). Last maintainer activity ~2 weeks ago (2026-05-30); no reply since my response.
s unused; asked him to confirm.

**Status:** Review-ready; in review (awaiting maintainer approval).

## Learnings & Reflections

### Technical Skills Gained
Stood up an unfamiliar enterprise server (WSO2 IS) from scratch — twice (7.1.0 and 7.3.0) — including JDK/runtime setup and `deployment.toml`/governance-API configuration. Learned how a real self-registration/OIDC flow routes across multiple JSPs, and the difference between HTML-attribute encoding and URL encoding. Practiced reproducing a production bug at the HTTP level and diffing rendered output to confirm a fix.

### Challenges Overcome
JSP changes requiring a full server restart (compiled-servlet caching) and a documentation-wrong JDK requirement; a CodeRabbit-flagged double-encoding risk that I verified against the source and resolved; tracking down why the bug reproduces on one version/flow but appears masked on another.

### What I'd Do Differently Next Time
Verify reproducibility against the latest `master` before writing code. My fix is correct for the buggy line, but the maintainer noted the user-facing bug may already be resolved on master because the reachable page was fixed separately. Confirming on the latest default branch first would have surfaced the "is this still needed?" question up front — the biggest takeaway I'm carrying into my next contribution cycle.

## Resources Used
- WSO2 Identity Server docs — self-registration & organization management (is.docs.wso2.com).
- The sibling fix pattern in the codebase: `self-registration-username-request.jsp:116` (`IdentityManagementEndpointUtil.encodeURL`).
- `wso2/product-is` issue #23288 and the PR review thread (#10382).
- WSO2 source for `IdentityManagementEndpointUtil.encodeURL` and `AuthenticationEndpointUtil.isValidMultiOptionURI` (carbon-identity-framework).
