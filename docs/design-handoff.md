# Design handoff

## Naming and direction

App name undecided; “Pet care” is a neutral prototype label. Concept 02 uses electric violet `#7549EF`, lime `#DDF688`, dark ink `#262139`, and cloud `#F8F7FC`. Serif italic accents soften selected editorial headings. Original transparent pet illustrations support onboarding, daily care, health, and shared-care screens.

## Core UX

Five destinations: Today, Pets, Log, Circle, and More. Each care event should identify the pet, caregiver, timestamp, type, and sync status. The distinguishing flow is a shared care timeline and time-bound sitter guide with role-specific access.

The screen inventory is in `screens.json`. The interactive Design system view contains the full component, typography, spacing, accessibility, advertising, and implementation guidance.

## Motion

- Artwork: slow 6–7 second breathing/floating cycles; never applied to essential clinical text.
- Navigation: 280 ms fade and rise.
- Care lists: 60 ms stagger.
- Progress: animated to its current value.
- Completion: a short celebratory burst.
- Support both the in-studio motion toggle and the system reduced-motion preference.

## Native implementation

Use safe areas, keyboard-aware forms, suitable input keyboards, scalable typography, platform screen-reader semantics, and minimum touch targets of 44 pt iOS / 48 dp Android. Localize dates, units, currency, language, and time zones. Test keyboard, large text, contrast, screen readers, and reduced motion on actual devices.

Implement authentication, shared-data permissions, storage, upload progress, notifications, sync, invites, subscription purchase/restore, and data deletion. Prevent duplicate care completion across caregivers. Scope sharing to pets and roles; enforce expiry and revocation on the server.

Medication screens organize prescribed instructions; do not infer doses or diagnose. The health passport is a care summary rather than an official travel certificate.

## Monetization

Sample native-ad slots appear in browsing areas. Emergency, medication, consent, and care-logging flows remain ad-free. Use clear sponsored labels and later-editable ad preferences. Plus is an optional ad-free concept, with essential care available to free users. Native AdMob, consent management, ATT where applicable, and store billing require implementation.

## Validation already performed

All 66 screen templates render in a simulated DOM; local routes and asset references resolve. Sample meal, reminder, currency, sitter date, weight unit, and empty-routine flows pass. CSS structure and script syntax checks passed. These checks do not constitute visual browser QA or native testing.
