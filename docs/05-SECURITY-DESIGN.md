# Security Design — Let'sVote

- Status: MVP security baseline
- Threat model: public web app with authenticated voting and organization workspaces

## 1. Objectives
- Protect accounts and sessions.
- Prevent unauthorized organization access.
- Preserve vote integrity under retries/concurrency.
- Prevent identity spoofing in vote requests.
- Reduce abuse of authentication, voting, and uploads.
- Keep secrets out of logs and source control.

## 2. Authentication
- Use ASP.NET Core Identity for user management and password hashing.
- Enforce password policy, lockout, and rate limiting.
- Use generic login failure messages to reduce account enumeration.
- Require HTTPS.
- For same-site frontend/API, prefer secure HttpOnly SameSite cookies with CSRF protection.
- If cross-site deployment requires bearer tokens, define access-token lifetime, refresh rotation/revocation, and safe storage. Avoid long-lived credentials in localStorage.
- Revoke sessions/refresh tokens when account is disabled or credentials change, according to chosen design.

## 3. Authorization
Use role-based and resource-based authorization:
- Platform Admin: platform-wide moderation.
- Organization Owner: organization, membership, campaign control.
- Organizer: permitted campaign and entry operations.
- Voter: eligible votes and own history.
- Visitor: public read-only access.

Every management request verifies current user membership and role. UUID possession is not authorization. Verify child resources belong to the requested parent. Prevent IDOR and avoid leaking private records.

## 4. Vote integrity
For every vote:
1. Require authenticated active account.
2. Resolve voter from authenticated principal.
3. Verify campaign Active and StartAtUtc <= now < EndAtUtc.
4. Verify entry Published and belongs to same campaign.
5. Insert in a transaction.
6. Enforce unique (CampaignId, EntryId, UserId) in DB.
7. Map unique violation to HTTP 409.
8. Do not expose vote update/delete routes in MVP.

Application duplicate checks improve feedback but are not the integrity boundary.

## 5. Input and data protection
- Use DTOs and server-side validation.
- Use EF Core parameterized queries; avoid string-built SQL.
- Validate file size, extension, MIME type, and signature where practical.
- Generate storage keys server-side.
- Limit upload processing resources.
- Apply allowlist CORS.
- Add security headers.
- Protect cookie-authenticated state changes against CSRF.
- Avoid mass assignment; map only allowed fields.

## 6. Abuse prevention
- Rate limit login, registration, password reset, and vote endpoints.
- Consider per-account and per-IP limits, mindful of shared networks.
- Add CAPTCHA only if abuse signals justify friction.
- Limit request size.
- Monitor suspicious bursts and repeated failures.
- Do not claim account-based voting proves one account equals one real person.

## 7. Audit and logging
Audit campaign publish/pause/resume/end/archive, membership role changes, account suspension/reactivation, and high-impact entry/campaign changes.

Never log passwords, password hashes, cookies, access/refresh tokens, API keys, or unnecessary personal data. Use correlation IDs and restrict audit access.

## 8. Secrets and deployment
- Store secrets in environment variables or a secret manager.
- Commit only safe example config.
- Never commit production connection strings or signing keys.
- Use least-privilege database credentials.
- Update dependencies and scan for known vulnerabilities.
- Back up database and document restore steps before production use.

## 9. Security test checklist
- [ ] Anonymous user cannot vote.
- [ ] Suspended user cannot vote.
- [ ] Client cannot submit another UserId to vote as someone else.
- [ ] Same user cannot vote twice for same entry/campaign.
- [ ] Concurrent duplicate requests create exactly one vote.
- [ ] User may vote for another eligible entry.
- [ ] Voting outside time window is rejected.
- [ ] Paused/Ended/Archived campaigns reject votes.
- [ ] Draft/Unpublished entries reject votes.
- [ ] Entry from another campaign cannot be voted under this campaign.
- [ ] Non-member cannot access organization management.
- [ ] Organizer cannot perform Owner-only actions.
- [ ] Upload rejects unsupported/oversized content.
- [ ] Errors do not expose internal exception details.
