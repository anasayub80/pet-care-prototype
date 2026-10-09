# Pet Care App — Complete Product and Engineering Requirements

Version: 1.0 · Prepared: 9 October 2026 · Status: implementation baseline for review

**App name is undecided.** “Pet care” and “Pet care Plus” are working labels. This document describes the complete intended product behind the existing 66-screen design prototype, plus the engineering work needed to ship it. Recommended defaults and release priorities below are proposals, not decisions already approved by the founder.

Source baseline: repository commit `5ef1a4681f059f095172954a7fccc7deae2fc265`; [screen inventory](screens.json); [design handoff](design-handoff.md); [assets](assets.md); `dist/app.js` and `dist/design.js`. The redesigned violet/lime presentation takes precedence over older base styles still present in the source.

## Contents

1. [Product purpose and scope](#1-product-purpose-and-scope)
2. [Users, roles, and permissions](#2-users-roles-and-permissions)
3. [Navigation and user journeys](#3-navigation-and-user-journeys)
4. [Functional requirements and acceptance criteria](#4-functional-requirements-and-acceptance-criteria)
5. [Design, branding, assets, and motion](#5-design-branding-assets-and-motion)
6. [Data model and business rules](#6-data-model-and-business-rules)
7. [Backend and integration requirements](#7-backend-and-integration-requirements)
8. [Offline operation and notifications](#8-offline-operation-and-notifications)
9. [Security, privacy, and data lifecycle](#9-security-privacy-and-data-lifecycle)
10. [Quality and nonfunctional requirements](#10-quality-and-nonfunctional-requirements)
11. [Delivery phases and launch strategy](#11-delivery-phases-and-launch-strategy)
12. [Analytics and success measures](#12-analytics-and-success-measures)
13. [Validation and release checklist](#13-validation-and-release-checklist)
14. [Prototype limitations and implementation gaps](#14-prototype-limitations-and-implementation-gaps)
15. [Decisions still needed](#15-decisions-still-needed)
16. [Complete screen-by-screen specification](#16-complete-screen-by-screen-specification)

## 1. Product purpose and scope

### Problem

Pet-care information is scattered across messages, paper records, calendars, and individual memories. Family members cannot reliably tell whether a meal, walk, or prescribed dose has already been given. A sitter needs a practical care guide without unrestricted or permanent access to private records.

### Product promise

One place to organize everyday care, keep owner-entered health information, and coordinate the people caring for each pet. The distinguishing experience is an attributed shared timeline paired with a scoped, time-bound sitter handoff.

### Outcomes

- Help a household answer who did what, for which pet, and when.
- Make logging common care actions quick and understandable.
- Keep records and emergency contacts accessible during normal use and temporary disconnection.
- Let an owner grant, review, expire, and revoke access deliberately.
- Support multiple pets, different units, currencies, and time zones without mixing their data.
- Provide a professional, warm visual identity with useful illustrations and restrained motion.

### Full product scope

Account and onboarding; pet profiles; routines and calendars; meal, walk, wellbeing, water/other-task and prescribed-dose logging; history; vaccinations; medications; weight; document vault; appointment reminders; vet and emergency contacts; care circles; invitations; sitter guides; insights; editorial content; expenses; reminders; activity inbox; preferences; privacy controls; optional ad-free subscription; support; export and deletion; offline and recoverable-error states. The review website, screen gallery, design system, and original assets are also project deliverables.

### Scope boundaries

The current design is a care organizer, not a sitter marketplace or clinic booking service. Sitter discovery, payments to caregivers, provider reviews, grooming bookings, insurance sales, live telemedicine, and clinic integrations are optional future products requiring their own requirements. An appointment reminder records an appointment already arranged elsewhere. A health passport is a personal care summary, not an official travel certificate. Medication and wellbeing features organize observations and vet-provided instructions; they do not diagnose or calculate treatment. GPS walk tracking is not required by the current timer-based design.

## 2. Users, roles, and permissions

Roles apply per pet; a person can have different roles for different pets. The following is the proposed production permission model. The prototype previews roles without enforcing real access.

| Capability | Owner | Co-owner | Caregiver | Viewer | Sitter link |
|---|---|---|---|---|---|
| View permitted profile, routine, and timeline | Yes | Yes | Yes | Yes | Selected guide sections only |
| Log care and complete assigned tasks | Yes | Yes | Yes | No | No |
| Correct own care event with audit trail | Yes | Yes | Yes | No | No |
| Manage routines and reminder assignments | Yes | Yes | No by default | No | No |
| Edit pet details and health records | Yes | Yes | No by default | No | No |
| View health documents | Yes | Yes | Only if expressly granted | Only if expressly granted | Selected documents only |
| Create/revoke sitter guides | Yes | Yes | No | No | No |
| Invite or manage caregivers/viewers | Yes | Yes | No | No | No |
| Appoint co-owner, transfer ownership, delete pet | Yes | No | No | No | No |
| Export pet data | Yes | Yes | No by default | No by default | Approved guide download only |
| Change own account, consent, or notifications | Yes | Yes | Yes | Yes | No account settings |

Each pet has one primary owner. Co-owner does not imply ownership transfer or access to another person's account or billing. A full-account export includes the requester's authorized data, with other people's unnecessary identifiers minimized. Platform support/admin access is a separate audited operational role, not a care-circle role.

An “All pets” invitation should resolve to the currently selected explicit pet IDs, not automatically grant access to future pets. Explain this in the UI. Future automatic household membership is a separate decision. Revocation blocks subsequent online access immediately; already downloaded documents and offline copies cannot be recalled remotely, and sharing copy must say so.

## 3. Navigation and user journeys

Five persistent destinations: **Today, Pets, Log, Circle, More**. Focused forms use a clear back action and a single primary save action. The active pet is visible wherever input or a summary is pet-specific. Returning from a form should restore relevant filters and scroll position.

| Journey | Sequence | Completion condition |
|---|---|---|
| New household | Welcome → introductions → account → preferences/permission → pet → optional details → routine → ready → Today | Pet saved and a routine created or deliberately skipped |
| Returning owner | Sign in → Today → selected pet | Only authorized pets are loaded |
| Daily care | Today or Log → meal/walk/check-in → save → confirmation → history | One durable event with actor, pet, and timestamp |
| Scheduled task | Today/calendar → task → complete or undo | Shared occurrence status is consistent across members |
| Health record | Pet → health area → add/edit item → saved list/detail | Structured record persists and appears for the correct pet |
| Document upload | Pet → documents → select → metadata → upload → details | Only a completed, validated upload is available to others |
| Shared care | Circle → invite → recipient accepts → assigned tasks/history | Verified recipient receives only intended role and pet scope |
| Sitter handoff | Circle → choose pet, dates, sections → preview → create link → share | Guide matches preview and enforces start, expiry, and revocation |
| Emergency | Pet or offline state → emergency card → contact dialer | Critical cached information available without an ad/paywall |
| Account exit | Settings → export/transfer if needed → typed deletion confirmation | Sessions, links, and account access revoked; data lifecycle applied |

Invitation acceptance, expired-link, upload-progress, ownership-transfer, and subscription-result states need additional production UI beyond the 66 existing gallery screens. They may be dialogs or screens; do not imply they already exist in the prototype.

## 4. Functional requirements and acceptance criteria

**Priority:** P0 = recommended first-release requirement; P1 = full product after the core pilot; P2 = optional expansion. Priorities do not remove any existing screens from the design package.

### FR-01 — Accounts and recovery · P0

Support email registration, email verification, sign-in, sign-out, and password reset. Apple/Google entry points are represented in the design; enable only providers actually integrated. Use the provider's secure identity flow and deliberate account linking to prevent duplicate or taken-over accounts. Persist profile name, email, region, locale, and preferences. Password strength and validation must be consistent with the chosen auth service.

Acceptance: recovery responses do not reveal account existence; expired/used recovery links have a recoverable state; sign-out revokes the current session and clears private local data; sensitive email/password/deletion changes require recent authentication. Social entry points are hidden or clearly unavailable until implemented.

### FR-02 — Onboarding, choices, permissions · P0

Explain daily care, shared care, and the health summary in four illustrated screens. Allow returning users to sign in directly. Optional pet details and notifications may be skipped. Ad and analytics choices are separate, editable, and versioned; apply production consent requirements where relevant. Request OS permissions at the moment their purpose is clear. Demo exploration must use isolated sample data.

Acceptance: denying notifications does not block setup; continuing without optional analytics/advertising consent works; back navigation preserves form drafts; a returning verified user does not have to repeat completed setup. The permission screen reports denied status and a settings shortcut where supported.

### FR-03 — Multiple pets and profiles · P0

Create a pet with required name (maximum 40 characters in the design) and species: dog, cat, rabbit, bird, or other. Optional data: photo, breed, birthday, sex, spayed/neutered status, initial weight, microchip, and notes. Unknown values remain unknown, never replaced by sample defaults. Allow profile editing, pet switching, archive/delete, and ownership transfer through deliberate controls. Archive/delete/transfer require production states beyond the current edit form.

Acceptance: optional details can be skipped; future birthday is rejected; positive measurements retain their unit; upload failure does not erase profile text; each pet has its own image and records; archived pets retain authorized history and do not generate new routine occurrences. Non-dog routines must not automatically include walks.

### FR-04 — Routine setup and calendar · P0

Offer editable starter meals, walks where relevant, and water tasks. Let owners add custom tasks, recurrence, time, instructions, and assignees; all starters can be deselected. Show weekly calendar, task status, instructions, and completed actor. Distinguish routine template from each scheduled occurrence. Editing a template affects future occurrences by default, not past history.

Acceptance: empty routines show an add-reminder action, not division-by-zero progress; pet/date filters update actual data; a weekly task is not generated daily; assignees must still have logging access; missed tasks remain missed rather than auto-completed. A one-off edit does not silently change the whole series.

### FR-05 — Today dashboard and task completion · P0

Show the selected pet, progress for the chosen day, upcoming tasks, quick logging, and recent attributed events. Completion and undo must synchronize with the care circle. A completion records the task occurrence ID, actor, actual event time, and synchronization state.

Acceptance: two caregivers completing the same occurrence produce one canonical completion; the second user sees who completed it; undo leaves an audit trail and recalculates progress; completing one pet's task never marks another pet's task done. Show pending local entries separately from server-confirmed success.

### FR-06 — Meal logging · P0

Capture pet, meal type (breakfast/lunch/dinner/snack), food, positive amount, unit (g/oz/cups), date and time, and optional notes. The prototype has time-only input; production requires a date/time or an explicit today default. Offer a link to the matching scheduled occurrence rather than silently completing any meal task.

Acceptance: saved values and notes persist across restart; past entries retain their actual date; ounce/gram conversion is valid but cups must not be converted to mass without a food-specific density; duplicate retries do not duplicate entries. Logging a snack does not complete dinner.

### FR-07 — Walk timer and manual entry · P0

Start, pause, resume, stop, and review a timer; provide manual duration entry. Save duration, date/time, pet, and notes. Base elapsed time on timestamps/monotonic elapsed time rather than an assumption that a UI timer runs continuously in the background. No location permission is needed for this scope.

Acceptance: backgrounding/locking/relaunch does not lose an active draft; paused time is excluded; saving twice is idempotent; timer duration and manual overrides are explicit. Proposed manual bounds match the design: 1–1,440 minutes; request confirmation for unusually long values.

### FR-08 — Wellbeing and other care · P0

Record owner observations: happy/calm/tired/unwell, appetite, energy, notes, and event time. Permit completion of water, grooming, and custom routine tasks. Do not convert observations into a diagnostic score or automatically prescribe care.

Acceptance: all observations persist, not just the mood label; “unwell” offers a neutral route to saved vet contacts; custom care uses the same pet/actor/time model as meals and walks. Health-related text does not imply clinical assessment.

### FR-09 — Shared history and corrections · P0

Provide a chronological timeline with pet, actor, care type, event time, optional notes, and sync state. Filter by pet, date/range, and care type; paginate longer histories. Permit role-authorized corrections or voiding with audit metadata. Preserve attribution when membership changes.

Acceptance: records are stably ordered by event time with an ID tie-breaker; corrected events indicate editing; voided entries do not count in insights; access removal does not delete legitimate shared events; one user's optimistic local entries cannot appear in another account's cache.

### FR-10 — Vaccinations · P0

Store name, administered date, next due date if supplied, clinic/vet, notes, and optional linked document. Clearly distinguish administered doses from upcoming reminders. Follow-up dates come from the owner/vet; the app does not invent a vaccination course.

Acceptance: an administered record cannot be dated in the future; a scheduled dose uses a reminder instead; a due date earlier than the administered date requires correction; editing/deleting a due date updates or cancels its reminder. No species-wide medical scheduling is assumed.

### FR-11 — Medications and dose events · P0

Store medication name, verbatim prescribed instructions, prescribing vet, start/next-dose date, reminder time, and recurrence (once/daily/weekly/monthly/custom). Custom schedules need their own production editor. Confirm that a dose has already been administered before logging it; identify the corresponding scheduled dose where possible. Medication instruction fields are required. Provide active/stopped states.

Acceptance: the last dose and actor are visible; simultaneous logs of the same scheduled dose warn of existing completion and do not create duplicates; stopping a medication cancels future reminders; no automatic dose calculation or silent instruction alteration; no ads or upgrade blockers interrupt dose logging.

### FR-12 — Weight tracking · P1

Save value, kg/lb unit, measured date, notes, and actor; display date-labelled trends and a text/table alternative. Preserve source precision and convert only for display using a canonical measurement representation. Chart ranges come from actual measurements.

Acceptance: kg → lb → kg display changes do not alter stored values; mixed-unit entries plot correctly; empty and single-point charts are useful; no health diagnosis is inferred from the trend. Proposed input bounds follow the prototype (positive, up to 1,000 in the entered unit) with unusual-value confirmation rather than species-specific medical judgments.

### FR-13 — Health documents · P0

Allow PDF/JPG/JPEG/PNG upload up to a proposed 10 MB per file; title, type, date, notes, and pet required/contextual as appropriate. Categories: vet report, vaccination certificate, test result, insurance, receipt, other. Show upload progress, processing status, retry, searchable list, detail, preview, download, and deletion. Receipt attachment in expenses is a planned extension; it is not an existing expense-form upload.

Acceptance: server validates file bytes/type/size rather than filename alone; partial/quarantined uploads cannot be shared; retry preserves metadata and avoids duplicate documents; private downloads require current authorization; deleting a document removes shared guide references and eventually its storage object. A filename is not proof of completed upload.

### FR-14 — Health passport, visits, and contacts · P0

Generate a reviewable pet-specific health summary including selected identity, vaccines, prescribed instructions, records, and contacts. Export a readable PDF or equivalent portable report with generation date and data timestamp. Store upcoming and past vet visits, clinic, reason, date/time, reminder lead time, questions, and visit notes. Allow owner editing of regular/emergency clinic and owner contact details; the current contact display needs a production editor.

Acceptance: export includes only selected pet/scope and is not described as an official certificate; appointments do not imply confirmed clinic bookings; changing/cancelling a visit updates reminders; actual contact numbers open the dialer only after user action. Missing contacts get a helpful add action instead of a fictional clinic.

### FR-15 — Emergency card · P0

Show selected pet identity, critical owner-entered notes, prescribed-care instructions, owner and emergency-vet contacts. Store an authorized minimum locally for offline viewing, visibly dated. Emergency access means fast access for authorized users; it does not mean a publicly searchable record or an automatic lock-screen feature.

Acceptance: emergency content has no ads, subscription gates, or unrelated popups; cached information shows its last updated time; missing numbers cannot trigger calls; revoked users' cached access is cleared at the next authorized synchronization. Do not promise offline content is always current.

### FR-16 — Care circle and invitations · P0

Owners/co-owners invite an email recipient with role and explicit pet scope, review members and responsibilities, change permitted roles, and remove access. Invite acceptance requires the intended verified identity. Invitations have pending/accepted/expired/revoked states, finite expiry, resend, and cancellation. Proposed expiry: seven days.

Acceptance: recipients cannot substitute their own pet IDs/role; a viewer cannot log; a caregiver cannot grant access; expired/revoked invitations fail safely; member removal stops future online access and cancels/reassigns affected tasks with owner feedback. The only owner cannot leave without transfer or explicit pet deletion.

### FR-17 — Time-bound sitter handoff · P0

Choose specific pets, sitter label, start/end, routine, health summary, selected documents, and personal notes. Review the exact guide before publishing. Use a read-only opaque sharing token and expose copy/share, preview, expiry, and stop-sharing controls. Prefer a frozen reviewed snapshot at creation; updating its contents requires a new preview/version rather than silently adding newly uploaded private documents.

Acceptance: end cannot precede start; date-only end is displayed as an exact expiry time in the guide's chosen time zone; token works only within the configured period; sections toggled off are absent from responses, not merely hidden in the UI; linked downloads enforce the same grant; revocation blocks the guide and uncached downloads server-side. Link possession grants only the approved guide, never general account access. An optional recipient verification/PIN is an open product decision.

### FR-18 — Reminders and activity inbox · P0

Support one-off/daily/weekly/monthly recurring reminders for meal/walk/medication/grooming/appointment/other care. Capture pet IDs, start date, time, time zone, recurrence, assignee, and instructions. Enable/disable and edit series. Inbox includes due reminders, logged care, shared access changes, and document updates; support read/unread/read-all and authorized deep links.

Acceptance: no duplicate due notices after sync; completion before a due send suppresses it where technically possible; denied push permission does not remove the in-app calendar; tapping an outdated notification cannot expose revoked data. Explain any notification already delivered before cancellation.

### FR-19 — Care insights · P1

Compute weekly/monthly care counts, meals, logged walk minutes, and caregiver contributions from actual authorized, non-voided records. Show pet, date range, time zone, units, and incomplete periods. These are logging summaries, not health or caregiver-quality scores.

Acceptance: changing range recomputes values; future days are visibly distinct from zero completed days; counts reconcile with filtered history; edited/voided events update aggregates; a no-data state suggests logging without guilt.

### FR-20 — Expenses · P1

Store pet, title, amount, currency, category (food/health/grooming/toys/insurance/other), date, notes, and optional receipt. Use currency minor units or precise decimal handling, never floating-point accumulation. Summarize by month, pet, category, and currency separately.

Acceptance: GBP/EUR/USD totals never mix without an explicit conversion feature; editing currency moves the entry to its own group; positive amount and currency-specific precision are validated; date filters affect totals. PKR and local payment methods are proposed localization additions, not present in the current expense UI.

### FR-21 — Editorial library · P1

Browse/search categorized articles, view readable article details, save/unsave, and show related content. Publish only approved content with author/reviewer, published/updated date, and sources where relevant. Separate sponsored placements from editorial content. Admin content management is a production requirement, not an existing screen.

Acceptance: selected search/category remains active together; saved articles survive restart; unavailable content has a recovery state; medical claims receive appropriate editorial review; sponsored entries are explicitly labelled. Sample prototype articles are not automatically a launch-ready content library.

### FR-22 — Account preferences and localization · P0

Persist weight unit, currency, 12/24-hour format, language, region, notification categories, and quiet hours. Separate care/medication/appointment reminders, shared-care updates, and optional marketing. Define quiet hours as suppressing optional updates; time-critical selected care schedules remain explicit and separately controllable.

Acceptance: all screens use the saved preferences; a displayed language must have translated strings rather than an untranslated selector; names and notes remain Unicode-safe; dates/currency formats match locale; changing units does not mutate measurements. English is the recommended initial content language; other languages require translation and QA.

### FR-23 — Ads, consent, and Plus · P1

Native-style, labelled sponsored placements may appear in browsing areas such as More and library. Emergency, medication, consent, and care-logging journeys remain ad-free. All essential care stays available free. Optional Plus removes sponsored placements and may offer report themes. `$2.99/month` is a prototype concept price, not approved pricing. Production pricing, store product IDs, regional offers, and entitlement scope remain decisions.

Acceptance: declining personalized ads is respected by SDK configuration; optional analytics has its own control; ad SDKs receive no care records, medication text, documents, or pet identifiers; SDKs initialize only when permitted. Server-verified store purchase/restore, expiry, cancellation, refunds/revocation, and grace periods produce correct entitlements; screens use store-supplied price/terms. Define whether Plus is account-wide before implementation; recommended default is purchaser-account scope.

### FR-24 — Help, support, data export, deletion · P0

Offer searchable help and a real support submission with confirmation/reference and retry. Export authorized account/pet data in machine-readable form plus a readable summary. Deletion requires recent authentication, typed DELETE, consequence acknowledgement, and ownership-transfer handling. Provide production privacy/terms documents and reachable support contacts.

Acceptance: support failures are not reported as sent; exports exclude sharing secrets and other accounts' private data; deleting the sole owner prompts transfer or explicit deletion of owned pet data; transferred shared histories retain necessary attribution with minimized personal data. Revoke sessions/invitations/links, stop notifications, delete files and caches under the documented lifecycle, and tell users that store subscriptions may need separate cancellation.

## 5. Design, branding, assets, and motion

### Visual identity

Working direction: electric violet `#7549EF`, lime `#DDF688`, ink `#262139`, cloud `#F8F7FC`, warm supporting accents, tactile pet illustrations, and selective serif italic editorial emphasis. Final name, logo, icon, and trademark/domain review are open. Do not present “Pawday” or any other name as selected.

Use shared color/type/spacing/radius tokens and reusable cards, persistent-label inputs, pet chips, task rows, tags, primary/secondary/destructive actions, dialogs, bottom navigation, and skeletons. Spacing follows 4/8/12/16/20/24/32; mobile content margin is approximately 22 px in the prototype. Control corners 12–16 px, cards 18–24 px, hero containers 26–32 px. Native units and platform adaptations must be tested rather than copying CSS pixels blindly.

Native implementation should use scalable platform typography with a licensed editorial accent if retained. Keep practical instructions readable; never embed actionable medical or contact text into illustration assets. Error/status meaning must use text or symbols alongside color. Validate contrast against the final palette and each state.

### Asset deliverables

| Asset stem | Purpose | Original | Runtime |
|---|---|---|---|
| `cozy-friends` | Main onboarding/shared warmth | `assets/originals/cozy-friends.png` | `dist/assets/cozy-friends.webp` |
| `friends` | Additional pet-pair composition | `assets/originals/friends.png` | `dist/assets/friends.webp` |
| `health-scene` | Health introduction/summary | `assets/originals/health-scene.png` | `dist/assets/health-scene.webp` |
| `milo` | Dog profile/editorial imagery | `assets/originals/milo.png` | `dist/assets/milo.webp` |
| `luna` | Cat profile/editorial imagery | `assets/originals/luna.png` | `dist/assets/luna.webp` |
| `playful-dog` | Daily-care/walk introduction | `assets/originals/playful-dog.png` | `dist/assets/playful-dog.webp` |

All six originals and their optimized variants are in the repository. Distinguish sample illustrated profiles from the user's uploaded pet photos. Preserve transparency where supplied, appropriate crops, alt text for meaningful images, and decorative hiding for ornaments. Native adaptive app icons, splash assets, store screenshots, final brand logo, and localization-specific marketing assets still need production packaging.

### Motion

Artwork floats/breathes on approximately 6–7-second cycles; navigation uses about 280 ms fade/rise; care rows use a roughly 60 ms stagger; progress moves to its actual value; completion has a short celebration. Respect system reduced motion and the prototype's motion toggle. Reduced mode removes continuous float, stagger, and confetti while keeping immediate state feedback. Animation must not delay a save, intercept taps, distract from medication text, or block emergency access. Avoid flashing and excessive parallax.

### Review website

Retain interactive mobile preview, searchable/grouped 66-screen gallery, previous/next navigation, hash-addressable screens, reset demo, design-system view, motion toggle, and downloadable illustration assets. Sample data and simulated actions remain visibly identified. Keep all runtime JS/CSS/images local and deployable from `dist/`. `npm start` serves the studio; `npm run check` runs existing prototype checks. These tools are not a production native app or backend.

## 6. Data model and business rules

Recommended entities are technology-neutral; exact schemas depend on the selected backend.

| Entity | Essential fields and relationships |
|---|---|
| User | ID, auth-provider subject, verified email, display name, locale, region, time zone, status |
| UserPreferences | User ID, units/currency/time format, notification categories, quiet hours, motion override |
| Pet | ID, primary owner, name/species, optional profile/identity fields, photo asset, archived state, version |
| PetMembership | Pet/user IDs, role, health/document grants, responsibilities, state, granted-by, timestamps |
| Invitation | ID, intended verified recipient, explicit pet IDs/role, hashed token, expiry, status, inviter |
| RoutineTemplate | Pet, type/title, instructions, assignee, local time/time zone, recurrence/start/end, enabled, version |
| TaskOccurrence | Template/version, pet, scheduled instant, status, completion event reference, revision |
| CareEvent | ID, pet, actor, type, occurred-at UTC, original time zone, type-specific structured payload, task/dose occurrence, client operation ID, created/edited/voided timestamps |
| Vaccination | Pet, name, administered date, optional due date/vet/notes/document |
| Medication | Pet, name, exact instructions, vet, schedule, start/end, active state, version |
| DoseOccurrence | Medication/version, scheduled instant, administered event, status; distinct from medication definition |
| WeightMeasurement | Pet, canonical value, original value/unit, measured date, notes, actor |
| Document/Upload | Pet, metadata, private storage key, MIME/size/checksum, processing state, uploader, dates |
| VetContact | Pet or explicitly shared contact set, contact type/name, phone, address, optional hours |
| Appointment | Pet, reason/clinic/contact, scheduled instant/time zone, reminder lead time, notes/status |
| HandoffGrant | ID, hashed opaque token, pet IDs, approved snapshot/version/sections/document IDs, start/expiry, revoked-at, creator |
| Expense | Pet, title, exact amount/currency, category/date, notes, optional receipt document |
| Article/Bookmark | Versioned approved content/category/author/dates; user/article bookmark |
| Notification/Delivery | User, type, safe metadata/deep link, occurrence/dedup key, scheduled/sent/read state |
| ConsentRecord | User, purpose, choice, policy/version/region, timestamp, source |
| Subscription | Purchaser, platform/product, verified original transaction, status/expiry, entitlement, last verification |
| Export/DeletionJob | Requester, authorized scope, status, expiry/progress, lifecycle completion metadata |
| AuditEvent | Actor/action/resource, relevant change references, timestamp; no raw passwords, tokens, or record bodies |

Use globally unique IDs and immutable actor identity references. Server-generated timestamps establish receipt order; client event time represents when care occurred. Store exact dates as dates, not arbitrary UTC midnight; display timestamps in the chosen context with zone details where ambiguity matters. Keep health/record content separate from operational analytics.

Scheduled care occurrences and manually logged events are different objects. Link them explicitly. A unique constraint on the occurrence's active completion plus an idempotency key prevents duplicate completion. Users may legitimately log multiple meals/walks; never deduplicate unrelated events simply because their text/time looks similar. Undo/correction creates an auditable state change, not a destructive rewrite of shared history.

## 7. Backend and integration requirements

### Recommended architecture

Native-capable mobile client → authenticated API → transactional database and private object storage. A background queue handles reminder delivery, upload processing, exports, deletion, and entitlement reconciliation. Realtime subscriptions or incremental polling update shared care. An audited admin/support interface manages content and operational failures. Framework, hosting, database, and identity vendor remain undecided; a cross-platform approach may suit the project, but the repository does not contain a React Native/Flutter implementation.

### API capability map

Routes below are illustrative contracts, not implemented endpoints. Every request enforces the role/pet scope on the server. List APIs paginate, mutations validate inputs, and responses include resource versions or sync cursors where needed.

| Capability | Suggested operations |
|---|---|
| Account | Session/provider auth; `GET/PATCH /me`; recovery/verification through auth provider; export/deletion requests |
| Pets | `GET/POST /pets`; `GET/PATCH /pets/{id}`; archive/delete/ownership-transfer operations |
| Routines/tasks | Pet-scoped template CRUD; occurrences by range; complete/undo occurrence with version and operation ID |
| Care | Pet-scoped event create/list/detail/correct/void; type-specific payload validation |
| Health | Pet-scoped vaccination, medication, dose, weight, appointment, and contact CRUD |
| Documents | Begin upload → short-lived private upload URL → finalize/validate; authorized preview/download URL; delete |
| Sharing | Membership list/update/remove; invite/create/accept/revoke; handoff preview/create/revoke; public-token guide projection |
| Reminders/inbox | Reminder CRUD; device registration/revocation; inbox pagination/read/read-all |
| Extras | Currency-separated expense CRUD; approved article search/detail; bookmarks; insights by pet/range |
| Preferences | Persist account preferences and consent records independently |
| Monetization | Store receipt/transaction verification; restore/reconcile; signed server notification processing |
| Operations | Support tickets; async export status/download; deletion status; audited admin actions |

Use standard HTTP outcomes: 401 unauthenticated, 403 unauthorized, 404 unavailable/hidden resource as appropriate, 409 version/completion conflict, 413 oversize file, 422 invalid fields, 429 rate limit, and recoverable 5xx. Return stable error codes and field-level messages without disclosing private identifiers. Retryable mutations use idempotency keys; privileged operations use recent authentication and transactional checks.

### External integrations

- Identity provider for secure authentication and verified recovery.
- Transactional email for verification, invitations, and requested exports.
- APNs/FCM or equivalent native push transport; on-device reminder scheduler where applicable.
- Private object storage, safe PDF/image viewing, upload validation, and malicious-file processing.
- Native share sheet, photo/file picker, and dialer; request only necessary permissions.
- App Store/Google Play subscription billing and server verification if Plus ships.
- AdMob or another selected ad provider only if advertising ships, with regional consent handling and platform tracking controls where applicable.
- Error monitoring and privacy-limited analytics; support-ticket delivery.

No live integration, credential, payment account, or platform provisioning is included in the prototype. Pin implementation-time provider documentation and requirements before shipping integrations; this document does not assert a current legal or store-policy determination.

## 8. Offline operation and notifications

### Offline behavior

Maintain an encrypted or platform-protected cache for authorized profile, routine, recent history, medication instructions, emergency contacts, and expressly downloaded documents. Show cache timestamps. Allow offline drafts/care events and mark them pending; invitation acceptance, permission changes, link creation/revocation, account deletion, purchases, and uncached downloads require online confirmation.

On reconnect: revalidate session and access → fetch permission changes → apply queued authorized operations with idempotency keys → resolve conflicts → refresh projections → report remaining failures. Do not upload queued writes for a pet after access has been revoked. Preserve a recoverable explanation rather than silently losing them. Clear unauthorized cached resources at next validation and explain that a device already offline cannot receive instant remote revocation.

Use operation IDs, resource versions, retry backoff, and a sync cursor. A deleted/archived pet must not be recreated by a stale client. Conflicting document/profile edits require explicit resolution or a documented field-level strategy; concurrent task completion uses server uniqueness. Persist a walk draft separately from a completed event. Sign-out clears private cache and pending data only after an explicit warning/export path if unsynced drafts exist.

### Scheduling rules

Choose and display a routine time zone; recommended default is the owner's zone at creation. Travel does not silently move shared meal/medication schedules. Offer an explicit zone change for future occurrences. Store recurrence in local wall time plus an IANA zone and derive UTC occurrences. Proposed DST default: nonexistent local time moves to the next valid instant; repeated time fires once at the first occurrence. Display this behavior during schedule edits and test it.

For monthly day 29/30/31, recommended default is the last valid day in shorter months, visibly explained. Quiet hours apply to optional updates; do not shift a prescribed-care schedule implicitly. Avoid duplicate local and remote due alerts by assigning one designated due-reminder channel per occurrence/device and using stable identifiers; exact-once visible push delivery cannot be guaranteed by mobile transports. Inbox and server state remain the canonical status.

## 9. Security, privacy, and data lifecycle

- Enforce object-level authorization for every pet, care event, file, export, invite, and guide section. UI hiding is not authorization.
- Protect sessions in secure device storage; transmit over TLS; encrypt managed storage and backups; keep secrets out of clients and this repository.
- Store invitation/handoff tokens hashed, unpredictable, bounded, and revocable. Avoid exposing token-bearing URLs in telemetry/referrers; redact them from logs. Limit signed download URL lifetime and regenerate only after permission checks.
- Rate-limit auth, invitations, downloads, public guide access, and support. Verify webhook signatures and deduplicate delivery IDs.
- Validate and normalize text, dates, enums, phone strings, upload bytes, and sizes server-side. Render user text safely and never execute uploaded content.
- Keep health notes, documents, email addresses, and contact details out of ad payloads, analytics event properties, and crash breadcrumbs. Hide detailed health text from lock-screen notifications by default.
- Record consent purpose/version independently of care-circle access. Withdrawing optional consent takes effect in SDK initialization and subsequent collection.
- Document data controller/contact, providers, purposes, international processing where relevant, retention, exports, deletion, and public-link risks before launch. Obtain region-specific review when selecting the actual launch market.
- Proposed lifecycle: live authorized care records retained while the pet/account remains active; links expire at configured end; pending invites expire after seven days; export downloads expire after 24 hours; temporary abandoned uploads are cleaned within 24 hours. These are engineering defaults pending approval.
- Proposed deletion service target: immediately disable access and schedules, remove primary data/files within 30 days, expire backup copies within a documented period proposed at 90 days. Any lawful retained billing/security data must be minimal and disclosed. Validate these choices with actual infrastructure and applicable obligations.
- Test backup restoration regularly and reapply deletion/revocation records after restoration so deleted accounts or links do not become accessible again.

The consent/settings screens and draft policy dialogs are design directions. They are not final policies or proof of regulatory compliance. The app should not claim a specific healthcare certification or blanket “GDPR compliant” status without an implemented and reviewed basis.

## 10. Quality and nonfunctional requirements

The following are proposed measurable launch targets, to be confirmed against supported devices and infrastructure.

| Area | Target / requirement |
|---|---|
| Platforms | Native iOS/Android implementation; minimum OS versions selected during technical setup; review studio remains responsive web |
| Accessibility | 44 pt iOS / 48 dp Android touch targets; scalable text; screen-reader order/labels; visible web focus; text alternatives for charts; reduced motion |
| Contrast | Aim for WCAG AA text contrast (4.5:1 normal, 3:1 large) and discernible control/status boundaries; verify actual combinations |
| Layout | Safe areas, keyboard avoidance, long names, translated strings, narrow screens, landscape where supported, and large text without clipping essential actions |
| Startup | Useful cached Today view within 2 seconds on agreed representative midrange devices; avoid blocking on ads |
| Interaction | Immediate local feedback; animation around 60 fps where supported; navigation not gated by background requests |
| API | Proposed p95 ordinary authenticated reads/writes under 500 ms server processing at pilot load, excluding uploads/external providers |
| Reliability | Proposed 99.5% monthly API availability initially and at least 99.5% crash-free mobile sessions |
| Durability | A save is called synchronized only after durable server acknowledgement; upload finalization and mutation retries are idempotent |
| Recovery | Proposed database RPO ≤24 hours and RTO ≤4 hours; restore exercise before launch; tighten as adoption grows |
| Scale | Paginated histories, indexed pet/date queries, bounded guide responses, queued jobs; load-test an agreed pilot forecast and a burst above it |
| Localization | Locale-aware dates/numbers/units/currency; Unicode; no hardcoded sample dates; translate only supported launch languages |
| Connectivity | Useful denied-permission, slow-network, expired-session, offline, timeout, upload-failure, and retry states |
| Maintainability | Shared components/tokens, typed data contracts, migrations, environment separation, CI checks, reproducible builds, versioned API compatibility |

These targets are requirements to validate, not performance already measured for the web prototype.

## 11. Delivery phases and launch strategy

### Phase 0 — Product and technical setup

Select app name and market hypothesis; confirm P0 scope; interview owners; choose native stack and backend; define schema/permissions; finalize privacy and ownership flows; configure environments and CI; create production icons; map all demo actions to real service work. Deliver a clickable design plus an implementation backlog and approved open decisions.

### Phase 1 — Core pilot (P0)

Ship identity, pet management, editable routines/reminders, daily logging/timeline, core health records/document storage, emergency contacts, circle invitations, sitter handoff, secure sync/offline cache, preferences, support, export/deletion, and required operational controls. If this is too broad for a first pilot, split health/document breadth into a follow-up increment deliberately; do not remove shared-access security or represent incomplete services as working.

Keep first-pilot monetization simple. Ads/Plus, insights, expenses, weight charts, and the full editorial library can follow after repeat-use validation. Health records and essential care should remain accessible under the planned free model.

### Phase 2 — Complete organizer (P1)

Add real insights, weight trends, expenses/receipts, approved editorial content/bookmarks, wider localization, and optional consent-respecting ads/Plus with verified purchases. Complete additional production UI states and benchmark quality on the supported platforms.

### Phase 3 — Optional services (P2)

Sitter/provider discovery, reviews, clinic integrations, paid bookings, marketplace payouts, insurance, and telemedicine are independent expansions. They require supply recruitment, verification, disputes/refunds, provider permissions, marketplace economics, additional regulations/integrations, and a new screen inventory. They are not included in the existing care-guide handoff.

### Market hypothesis

Previous discussion recommended building from Pakistan and testing the digital care organizer with US users early; this is a recommendation, not a confirmed country launch. Recruit approximately 30–50 target-market pet owners for the initial usability/retention pilot, with local users helping rapid usability iteration. A Pakistan pilot alone cannot establish willingness to pay in the US. If pursuing local services, test in one city with reliable providers first. For Europe, choose a specific country and validate language/local support requirements before expansion.

No market-size or acquisition-cost forecast is embedded here. Revalidate payment onboarding, launch-country obligations, distribution, and subscription economics when the actual commercial model is chosen.

## 12. Analytics and success measures

Track minimal operational/product events without free-text care or document contents: onboarding step/completion; pet creation; routine creation; care event type saved/sync result; invite created/accepted; handoff created/viewed/revoked; export/deletion job status; notification permission; upload success/failure; subscription conversion/restore where enabled. Use random/pseudonymous identifiers and aggregate reporting with the user's optional choices respected.

| Measure | Definition / purpose |
|---|---|
| Activation | Registered household creates a pet and logs first care within 24 hours |
| Shared-care activation | A second verified member accepts and logs authorized care within seven days |
| Retention | Activated household has a meaningful care action in day-7/day-30 windows; report cohorts |
| Repeat utility | Care actions per active household/week and number of distinct logging days; avoid counting navigation alone |
| Handoff usefulness | Guides created, legitimately opened during validity, and owner/sitter feedback |
| Data trust | Duplicate completion rate, failed sync rate, unresolved conflicts, upload reliability |
| Monetization | Actual conversion, subscriber retention, refunds, and contribution after store/provider costs |
| Growth economics | Acquired activated households and cost per retained/paying household by channel |

Baseline targets should follow pilot observations; no fabricated retention or revenue goal is asserted. Expand acquisition only after repeat use and viable economics are observed. Ad revenue is not assumed to fund growth merely because ad slots exist in the prototype.

## 13. Validation and release checklist

### Required validation

1. Unit/domain tests: recurrence/DST, money and measurement conversion, role matrix, task/dose uniqueness, guide projection/expiry, and deletion/ownership rules.
2. API/integration tests: auth, recipient verification, horizontal-access denial, private upload lifecycle, reminder cancellation, transaction verification, and async job recovery.
3. Concurrent/offline tests: two completions; retries; stale versions; revoked access with queued writes; archive/delete during sync; sign-out with unsynced drafts; interrupted timer/upload.
4. Mobile end-to-end journeys: registration, first pet, meal, walk, medication confirmation, invitation acceptance, guide viewing/revocation, export, deletion, and restore purchase if monetization ships.
5. Visual/accessibility QA: every screen and new production state; keyboard; large text; screen readers; focus; contrast; motion off; long/translated content; real device safe areas.
6. Notification QA: denied permission, time-zone changes, DST/month boundaries, quiet hours, stale deep links, reminder edited/completed before send, multi-device deduplication.
7. Security/operations: authorization probes, token/log redaction, signed links/webhooks, storage isolation, load and restore exercise, deletion verification.

### Release gates

- P0 acceptance criteria satisfied and critical defects resolved.
- No demo account, fake clinic, sample payment, or simulated network success presented as live behavior.
- Privacy/terms/support links, retention/deletion flows, country choices, and data-sharing copy approved for the actual launch.
- Production secrets managed externally; private storage and API permissions verified independently of the client.
- Real permission prompts and store listing/privacy declarations reflect integrated SDK behavior.
- Billing/ads omitted from launch or fully verified; essential care and emergency flows remain accessible.
- Error monitoring, alerts, queue health, backups, recovery runbook, and rollback/deployment process working.
- Device QA and pilot feedback reviewed; exported reports accurate; original asset packaging complete.

Existing `npm run check` renders all 66 screens in a simulated DOM, checks routes/assets, and exercises selected sample interactions. It is useful for this prototype and does not satisfy native visual QA, real access control, service integration, or production release gates.

## 14. Prototype limitations and implementation gaps

| Existing demonstration | Work required for production |
|---|---|
| Static HTML/CSS/JS mobile frame and review studio | Native application, persistent data, production backend, distribution |
| Sample profiles, records, dates, clinics, and timeline | User-owned records; correct dates/pet filters; removal of fictional defaults |
| In-memory form interactions reset on reload | Persist every submitted field and multiple records per pet; avoid overwriting a single demo object |
| Registration/recovery/social dialogs | Real identity, verification, secure sessions, recovery, linked-provider handling |
| Selected photo/document filenames | Real upload, metadata, processing, per-pet storage, authorization, preview, failure recovery |
| Sample charts, calendar, insights | Queries and chart aggregation from actual data; ranges and chart accessibility |
| Invitation/role/remove-access previews | Verified invitations and server-enforced role/scope lifecycle |
| Example sharing URL and local revoke flag | Live token grant, exact section projection, start/expiry/revocation, protected downloads |
| Timer while page is active | Durable timer draft and correct background elapsed-time handling |
| Date validation and currency/weight samples | Full date/time zone behavior, recurrence, exact money/measurement representation |
| Offline/reconnect layout | Protected cache, outbox, conflict resolution, access revalidation, accurate sync state |
| Notification/read-all/settings preview | Real preferences persistence, push scheduling, inbox state, quiet hours |
| Plus purchase/restore dialogs and sample price | Store products, receipt verification, regional pricing, entitlements, cancellation/refund handling |
| Sample ad placement/consent UI | Integrated consent and ad SDK behavior, policy review, placement QA |
| TXT sample exports and deletion preview | Real account export/report generation, ownership-transfer/deletion lifecycle |
| Static FAQs/support/policy outlines | Approved help/content, delivered support tickets, final policies |

Some prototype forms only store a subset of entered fields, and some lists still include fixed sample rows after mutations. No requirement here asserts those production behaviors already work. The existing package is a complete design prototype; implementation completion is a separate milestone.

## 15. Decisions still needed

| Decision | Proposed starting point / dependency |
|---|---|
| Final name, domain, logo, icon | Undecided; retain neutral working name until selected |
| Primary country and segment | Test US digital-care demand; confirm founder's market choice and pet-owner segment |
| iOS/Android launch order and minimum OS | Select based on recruited users and available budget |
| Native/backend stack and hosting | Technology-neutral until engineering setup; no vendor commitment implied |
| Pilot breadth | P0 organizer; consider phased health/document depth if resources are limited |
| Owner/co-owner powers and health visibility | Adopt proposed matrix only after review; health scope explicit for non-owners |
| Sitter-guide authentication/snapshot changes | Opaque read-only expiring token, reviewed snapshot; optional PIN/verified recipient |
| Recurrence/travel rules | Fixed selected routine zone; explicit changes; proposed DST/month rules above |
| Storage quota and file limits | Proposed 10 MB per file; account quota/fair-use policy still needed |
| Business model and price | Essential care free; optional ads/Plus after validation; concept $2.99 not approved |
| Plus sharing and cancellation behavior | Recommended purchaser-account scope; finalize product/entitlement semantics |
| Translations, PKR, other currencies | English first; enable only implemented locales/currencies |
| Retention/deletion and backups | Review proposed 30-day primary / 90-day backup lifecycle against actual infrastructure |
| Admin/support workflow and response target | Define owner, access process, business hours, and escalation routes |
| Launch analytics/performance targets | Confirm proposed targets against pilot forecast/devices and privacy choices |
| Marketplace/service expansion | Separate roadmap and requirements; not part of current prototype scope |

## 16. Complete screen-by-screen specification

The following inventory is generated from `docs/screens.json`. All 66 screen IDs correspond to the current prototype. Each screen's production implementation must also satisfy the functional requirements, permissions, validation, and recovery behavior above. Additional dialogs/screens noted in this document are not counted as existing prototype screens.

### Getting started (10 screens)

| # | Screen ID | Screen | Purpose / required behavior | Requirements |
|---|---|---|---|---|
| 1 | `welcome` | Welcome | A memorable first impression, with a single clear action and a sign-in shortcut. | FR-02 |
| 2 | `intro-care` | Everyday care | Introduce the daily care timeline and fast logging. | FR-02 |
| 3 | `intro-family` | Shared care | Explain the care circle: everyone can see who did what. | FR-02 |
| 4 | `intro-health` | Health in one place | Introduce the health passport and sitter handoff. | FR-02 |
| 5 | `signup` | Create account | Email registration with terms acknowledgement and alternative sign-in options. | FR-01 |
| 6 | `signin` | Sign in | Existing-account entry with password recovery. | FR-01 |
| 7 | `forgot` | Reset password | Request a recovery link without exposing whether an account exists. | FR-01 |
| 8 | `reset-sent` | Check your inbox | A clear recovery confirmation and route back to sign-in. | FR-01 |
| 9 | `consent` | Ad preferences | Equal-choice consent controls with a granular preference screen. | FR-02 |
| 10 | `permissions` | Notifications | Explain reminder permissions before the native permission request. | FR-02 |

### Pet setup (4 screens)

| # | Screen ID | Screen | Purpose / required behavior | Requirements |
|---|---|---|---|---|
| 11 | `add-pet` | Add your pet | A welcoming first step: photo, name, and species. | FR-03 |
| 12 | `pet-details` | Pet details | Optional details support a useful profile without blocking setup. | FR-03 |
| 13 | `routine-setup` | Build a routine | Starter tasks with editable timing and no medical assumptions. | FR-04, FR-05 |
| 14 | `ready` | You’re all set | Completion feedback with the next action clearly visible. | FR-03 |

### Daily care (10 screens)

| # | Screen ID | Screen | Purpose / required behavior | Requirements |
|---|---|---|---|---|
| 15 | `home` | Today | The daily care overview shows progress, next tasks, and who completed care. | FR-05 |
| 16 | `schedule` | Care calendar | Browse the weekly routine and complete or inspect a task. | FR-04, FR-05 |
| 17 | `task` | Task details | Show timing, assignee, instructions, and completion status. | FR-04, FR-05 |
| 18 | `log` | Log care | A quick action hub with a visible pet selector. | FR-05–09 |
| 19 | `feeding` | Log a meal | Track food amount, timing, and notes. | FR-06 |
| 20 | `walk` | Track a walk | A simple timer with pause, resume, and a manual-entry alternative. | FR-07 |
| 21 | `walk-summary` | Walk summary | Review duration and notes before adding a walk to the shared timeline. | FR-07 |
| 22 | `wellbeing` | Wellbeing check-in | Record observations and mood without diagnostic claims. | FR-08 |
| 23 | `history` | Care history | A readable shared timeline identifies the pet and person for each entry. | FR-09 |
| 24 | `care-saved` | Care logged | Confirm success and return to the daily overview. | FR-05–09 |

### Pets & health (17 screens)

| # | Screen ID | Screen | Purpose / required behavior | Requirements |
|---|---|---|---|---|
| 25 | `pets` | My pets | Manage multiple pets from individual, visual profile cards. | FR-03 |
| 26 | `profile` | Pet profile | The central entry to health, routine, care circle, and emergency information. | FR-03 |
| 27 | `edit-pet` | Edit pet | Edit personal and identifying details with clear save feedback. | FR-03 |
| 28 | `passport` | Health passport | A portable summary of identification and health records with deliberate sharing. | FR-14 |
| 29 | `vaccines` | Vaccinations | Separate recorded doses from scheduled due dates. | FR-10 |
| 30 | `add-vaccine` | Add vaccination | Record vet-provided vaccine information and an optional follow-up. | FR-10 |
| 31 | `medications` | Medications | Keep dosage instructions and the last administered event together. | FR-11 |
| 32 | `add-medication` | Add medication | Use the prescribed dosage verbatim; reminders do not invent treatment. | FR-11 |
| 33 | `weight` | Weight trends | A labelled weight chart, unit control, and dated entries. | FR-12 |
| 34 | `add-weight` | Log weight | A focused measurement entry with pet and unit context. | FR-12 |
| 35 | `records` | Health documents | A searchable record vault with document types and dates. | FR-13 |
| 36 | `add-record` | Add document | A file picker, document metadata, and upload guidance. | FR-13 |
| 37 | `record` | Document details | Review a document and its associated pet before sharing or download. | FR-13 |
| 38 | `visits` | Vet visits | Upcoming appointments and visit history. | FR-14 |
| 39 | `add-visit` | Book a reminder | Record an existing appointment; it does not book with the clinic. | FR-14 |
| 40 | `vet` | Vet contacts | Keep the usual clinic and emergency contacts accessible. | FR-14 |
| 41 | `emergency` | Emergency card | Ad-free access to critical contacts and relevant pet information. | FR-15 |

### Shared care (6 screens)

| # | Screen ID | Screen | Purpose / required behavior | Requirements |
|---|---|---|---|---|
| 42 | `circle` | Care circle | Roles, membership, and recent shared actions. | FR-16 |
| 43 | `invite` | Invite a caregiver | Preview an invitation with a deliberate role and pet scope. | FR-16 |
| 44 | `member` | Caregiver details | Manage role, responsibilities, and access. | FR-16 |
| 45 | `handoff` | Sitter handoff | Set a time-bound handoff and review everything the sitter will receive. | FR-17 |
| 46 | `sitter` | Sitter care guide | A practical pet-specific guide with meals, routine, and contacts. | FR-17 |
| 47 | `handoff-ready` | Handoff ready | Show link expiry and the ability to stop sharing. | FR-17 |

### Explore & more (9 screens)

| # | Screen ID | Screen | Purpose / required behavior | Requirements |
|---|---|---|---|---|
| 48 | `insights` | Care insights | Summarize logged activity with transparent units and no health score. | FR-19 |
| 49 | `library` | Care library | Browse practical care content and clearly separated sponsored placements. | FR-21 |
| 50 | `article` | Care article | An editorial layout with a save action and related content. | FR-21 |
| 51 | `expenses` | Pet expenses | A simple dated spending log with totals and categories. | FR-20 |
| 52 | `add-expense` | Add expense | Capture category, currency, and amount with receipt support. | FR-20 |
| 53 | `reminders` | Reminders | Manage upcoming and recurring reminders. | FR-18 |
| 54 | `add-reminder` | New reminder | Create a pet-specific recurring reminder and assign a caregiver. | FR-18 |
| 55 | `notifications` | Activity inbox | Reminders, shared-care activity, and document updates. | FR-18 |
| 56 | `more` | More | A quiet home for settings, expenses, learning, and support. | FR-18–24 |

### Account & states (10 screens)

| # | Screen ID | Screen | Purpose / required behavior | Requirements |
|---|---|---|---|---|
| 57 | `settings` | Settings | Account, units, notifications, privacy, and data controls. | FR-01, FR-22 |
| 58 | `account` | Your account | Basic account details and deliberate account deletion. | FR-01, FR-22 |
| 59 | `notification-settings` | Notification settings | Separate necessary care reminders from optional updates. | FR-01, FR-22 |
| 60 | `privacy` | Privacy & ads | Manage ad choices and shared-data access; preferences can be changed later. | FR-23, FR-24 |
| 61 | `premium` | Pet care Plus | An optional ad-free plan that preserves all essential care features for free users. | FR-23, FR-24 |
| 62 | `help` | Help & support | Searchable practical help with a support entry point. | FR-24 |
| 63 | `empty` | No pets yet | A warm empty state with a single contextual action. | FR-03 |
| 64 | `offline` | You’re offline | Show what remains available and make sync status explicit. | Section 8, FR-15 |
| 65 | `error` | Upload interrupted | Recoverable failure with retry and preservation of form context. | FR-13 |
| 66 | `delete` | Delete account | A deliberate, typed confirmation for an irreversible account action. | FR-24 |


## Document maintenance

Update this document and the screen inventory together when scope changes. Record newly approved decisions with dates; keep proposals distinct from shipped behavior. Add native implementation tickets linking to the FR IDs and acceptance criteria, then mark delivery status only after verification. The original source/assets remain independently deployable from the repository.
