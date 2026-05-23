# RSSFeedReader Project Constitution

## Core Purpose
Build a simple RSS/Atom feed reader demonstrating subscription management through a clean, secure, maintainable architecture that supports incremental growth from MVP to production-ready features.

---

## Security Principles

### 1. Secure By Default - No Validation Assumptions
**Principle:** While MVP assumes valid URLs, infrastructure must support validation before production.

**Actionable Requirements:**
- Never execute or display unsanitized feed content even in MVP
- Use `System.ServiceModel.Syndication` for all feed parsing (structured parsing, never regex-based)
- Implement CORS policy whitelist from day one; never use `AllowAnyOrigin()` in production code
- Store all configuration (API URLs, ports, secrets) in environment variables or `appsettings.json` (never hardcoded)
- Frontend must validate URLs against URI scheme (http/https) before submission; backend validates independently

**Security Checklist (Pre-MVP Release):**
- [ ] CORS policy explicitly configured for known ports
- [ ] No credentials or API keys in source code or launchSettings.json
- [ ] All external data (feed content) treated as untrusted input
- [ ] Frontend validates URL format; backend re-validates
- [ ] No eval/dynamic code execution for URL processing

### 2. Separation of Trust Boundaries
**Principle:** Frontend and backend are separate security domains; frontend validation is UX only, backend is security.

**Actionable Requirements:**
- Frontend validates URLs for UX feedback (disable submit button, show warnings)
- Backend independently validates all subscription URLs before accepting
- Backend assumes frontend inputs are malicious until proven otherwise
- API responses must not include system information, stack traces, or internal state
- Implement proper HTTP status codes (400 for bad input, 500 for server errors - never expose details)

### 3. Future-Proof HTTPS and Authentication
**Principle:** MVP runs locally; production requires encryption and user isolation.

**Actionable Requirements:**
- Use `https://localhost:7025` for HTTPS development (Blazor template default)
- Never log sensitive data (feed URLs should not include auth tokens)
- Prepare for user isolation: store subscriptions with user ID (even if MVP only supports one user)
- Database schema must support multi-user from day one (user_id foreign key)

---

## Maintainability Principles

### 4. Separation of Concerns - Rigid Boundary
**Principle:** Backend owns business logic; frontend owns UI logic. Cross-cutting concerns are centralized.

**Actionable Requirements:**

**Backend Responsibilities (Non-Negotiable):**
- Add/retrieve subscriptions
- Fetch and parse feeds (Extended-MVP+)
- Validate feed URLs
- Implement retry logic for feed fetching
- Error transformation (internal errors → API-safe responses)

**Frontend Responsibilities (Non-Negotiable):**
- Render subscription list UI
- Form validation and submission
- User feedback (success/error messages)
- Loading states and spinners
- Never implement business logic (calculations, decisions, filtering)

**Shared Code Restrictions:**
- Models/contracts only - no business logic in shared code
- Configuration constants only
- Exception hierarchies

### 5. Explicit Phase Boundaries - Version Control Strict Separation
**Principle:** Each development phase (MVP → Extended-MVP → Post-MVP) is isolated and verified before progression.

**Actionable Requirements:**

**Phase 1 (MVP: Subscription Management)**
- Storage: In-memory only (List<T>)
- API routes: `/api/subscriptions` (GET, POST)
- No feed fetching code
- No persistence layer
- No database dependencies
- Acceptance: User can add URL, see list

**Phase 2 (Extended-MVP: Feed Fetching)**
- Add HttpClient for feed fetching
- Add `System.ServiceModel.Syndication` dependency
- Add `/api/subscriptions/{id}/refresh` endpoint
- No persistence (still in-memory after restart)
- Acceptance: User can refresh feed, see items

**Phase 3+ (Post-MVP: Persistence)**
- Add EF Core + SQLite
- Implement `BackgroundService` for polling
- Add authentication layer
- Add delete subscription endpoint

**Implementation Rules:**
- Each phase has dedicated branch (e.g., `feature/extended-mvp`)
- Phase boundaries enforced by code review: no Extended-MVP code in MVP branch
- Schema changes documented in `Migrations/` folder
- Dependencies locked per phase (no unnecessary transitive deps)

### 6. Template Cleanup as Mandatory Foundation
**Principle:** Blazor template demo pages must be completely removed before feature implementation to prevent routing conflicts.

**Actionable Requirements:**
- MVP implementation BLOCKED until cleanup verified:
  - [ ] Home.razor deleted
  - [ ] Counter.razor deleted
  - [ ] Weather.razor deleted
  - [ ] NavMenu.razor updated (no dead links)
  - [ ] No ambiguous routes in `dotnet build` output
  - [ ] No routing errors on `http://localhost:5213`

**Why:** Routing conflicts are expensive to debug after feature implementation. Verify before building features.

### 7. Configuration Consistency - Single Source of Truth
**Principle:** Ports, URLs, and API configurations defined in one place and shared consistently.

**Actionable Requirements:**
- Backend port in `backend/RSSFeedReader.Api/Properties/launchSettings.json`
- Frontend port in `frontend/RSSFeedReader.UI/Properties/launchSettings.json`
- API URL in `frontend/RSSFeedReader.UI/wwwroot/appsettings.json`
- CORS policy in `backend/RSSFeedReader.Api/Program.cs`
- **All must match** before any feature testing
- Validation checklist in README.md for developers:
  ```
  - [ ] Backend runs on configured port
  - [ ] Frontend runs on configured port
  - [ ] Frontend appsettings.json → correct backend URL
  - [ ] CORS allows frontend origin
  - [ ] Browser DevTools shows no connection errors
  ```

### 8. Cross-Platform First - No Windows-Only Paths
**Principle:** Code must run on Windows, macOS, and Linux without modification.

**Actionable Requirements:**
- All file paths use forward slashes or Path.Combine() (never hardcoded backslashes)
- No Windows-only APIs (Windows.Foundation, etc.)
- Configuration paths relative to project root (not absolute C:\ paths)
- Scripts: provide both bash and PowerShell versions
- Local development guide documents setup for each OS

---

## Code Quality Principles

### 9. Minimal Viable Scope - No Gold Plating
**Principle:** MVP features are EXACTLY: add subscription URL, display list. Nothing more.

**Actionable Requirements:**
- No validation logic in MVP (assume valid URLs)
- No error handling in MVP (assume success)
- No persistence in MVP (in-memory only)
- No UI polish (functional, not beautiful)
- No feed fetching in MVP
- No logging beyond development debugging

**Why:** Speed and clarity. Extended-MVP adds everything deferred.

**Scope Enforcement:**
- PR checklist explicitly asks: "Is this MVP or Extended-MVP scope?"
- Out-of-scope PRs rejected with link to phase documentation
- Code review looks for scope creep

### 10. Explicit Testing Strategy - Per Phase
**Principle:** MVP tests are simple; complexity grows with scope.

**Actionable Requirements:**

**MVP Testing:**
- Manual testing documented in README
- Browser DevTools console: verify no errors
- Happy path: add URL, see it in list
- No unit tests required (code is trivial)
- No integration tests required

**Extended-MVP Testing:**
- Unit tests for feed parsing (`System.ServiceModel.Syndication` usage)
- Integration tests for HTTP client behavior
- Test with known-good feed: https://devblogs.microsoft.com/dotnet/feed/
- Error scenario tests: malformed XML, timeout, 404

**Post-MVP Testing:**
- xUnit + Moq test suite
- EF Core integration tests
- Background service scheduling tests
- Database migration tests

### 11. Dependency Management - Minimal and Documented
**Principle:** Add dependencies only when required by phase.

**Actionable Requirements:**

**MVP Dependencies:**
- ASP.NET Core Web API
- Blazor WebAssembly
- No additional NuGet packages

**Extended-MVP Adds:**
- `System.ServiceModel.Syndication` (feed parsing only)
- HttpClient (standard, no package needed)

**Post-MVP Adds:**
- `Microsoft.EntityFrameworkCore.Sqlite`
- `HtmlSanitizer` (if displaying rich content)
- `HtmlAgilityPack` (if discovering feeds from websites)

**Rule:** Every dependency must be approved in code review with rationale. Transitive dependencies documented.

### 12. Error Handling Strategy - Graduated Complexity
**Principle:** MVP has no error handling (assume success). Extended-MVP adds try-catch with user-facing messages.

**Actionable Requirements:**

**MVP Error Handling:**
- None. If it fails, app crashes (acceptable for local POC)

**Extended-MVP Error Handling:**
- Feed fetch failure → show "Failed to load feed" (no details)
- Malformed XML → show "Invalid feed format"
- Network timeout → show "Connection timeout"
- Never log or display stack traces to users
- Backend logs exceptions internally (stdout for local dev)

**Post-MVP Error Handling:**
- Retry logic with exponential backoff
- Detailed error categorization (feed moved, access denied, etc.)
- User notifications for retriable errors

### 13. Documentation Standards - Link Theory to Implementation
**Principle:** Every principle has a corresponding code artifact. Docs reference code; code references docs.

**Actionable Requirements:**

**Mandatory Documentation:**
- README.md: Links to ProjectGoals, AppFeatures, TechStack
- README.md: Local development checklist (ports, CORS, routing)
- README.md: Phase progression (MVP → Extended-MVP → Post-MVP)
- TechStack.md: Cleanup and template removal steps (already done ✓)
- ProjectGoals.md: MVP definition and Extended-MVP deferral (already done ✓)
- AppFeatures.md: Feature scoping rules (already done ✓)

**Code Documentation:**
- API endpoints documented with phase they appear in
- Configuration classes include comments on which phase introduced them
- Deferred features marked with `// Extended-MVP: [feature name]` comments
- No orphaned comments (all reference a doc or issue)

### 14. Incremental Architecture - No Premature Optimization
**Principle:** MVP is deliberately minimal. Architecture grows as scope grows.

**Actionable Requirements:**

**MVP Architecture:**
```
RSSFeedReader.Api/
├── Controllers/SubscriptionsController.cs
├── Models/Subscription.cs
└── Program.cs (in-memory List<Subscription>)

RSSFeedReader.UI/
├── Pages/Subscriptions.razor
├── Services/SubscriptionService.cs
└── appsettings.json (API base URL)
```

**Extended-MVP Architecture Additions:**
```
RSSFeedReader.Api/
├── Services/FeedFetcherService.cs
├── Models/FeedItem.cs
└── Models/FeedParseError.cs
```

**Post-MVP Architecture Additions:**
```
RSSFeedReader.Data/
├── ApplicationDbContext.cs
├── Migrations/
└── Repositories/SubscriptionRepository.cs

RSSFeedReader.Services/
├── BackgroundServices/FeedPollingService.cs
└── HtmlSanitizer configuration
```

**Rule:** No layer added until needed. No interfaces for single implementations (MVP). No DI container complexity until Post-MVP.

---

## Governance & Enforcement

### 15. Code Review Checklist - Phase-Aware
Every pull request must include:
- [ ] Phase scope stated (MVP / Extended-MVP / Post-MVP)
- [ ] Scope stays within declared phase
- [ ] No dependencies added without justification
- [ ] Configuration consistency verified (ports, URLs)
- [ ] Cross-platform compatibility (no Windows-only paths)
- [ ] Security: no hardcoded secrets, no unsafe parsing
- [ ] Documentation updated to match code

### 16. Phase Progression Criteria - Explicit Checkpoints
**MVP → Extended-MVP:**
- [ ] Subscription add/list working end-to-end
- [ ] Manual testing completed and documented
- [ ] Browser console shows no errors
- [ ] All stakeholders confirm MVP scope complete
- [ ] Branch: `main` (MVP stable)

**Extended-MVP → Post-MVP:**
- [ ] Feed fetching and item display working
- [ ] Error handling for network failures implemented
- [ ] Testing with known-good feed completed
- [ ] Persistence strategy finalized (SQLite + EF Core)
- [ ] Branch: `main` (Extended-MVP stable)

### 17. Success Metrics - Observable Outcomes
- **MVP Success:** User adds URL, sees it in list (documented manual test)
- **Extended-MVP Success:** User adds URL, clicks refresh, sees items (documented manual test with real feed)
- **Post-MVP Success:** Subscriptions persist across app restarts, auto-refresh on schedule

---

## Summary: Constitution in Action

This constitution ensures:
1. **Security:** Validates input, isolates domains, prepares for HTTPS/auth
2. **Maintainability:** Clear phase boundaries, separation of concerns, configuration consistency
3. **Code Quality:** Minimal scope, graduated testing, explicit architecture growth

Every decision must trace back to one of these 17 principles. Questions not addressed here should be escalated to stakeholders, not decided in code.
