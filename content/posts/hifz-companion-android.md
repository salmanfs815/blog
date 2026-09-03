---
title: "Building Hifz Companion: An Offline-First Revision Scheduler for Android"
date: 2026-08-04
tags:
  - projects
  - android
  - kotlin
---

Hifz Companion is an Android app I built to make Qur'an memorization revision more manageable. It is available as an [open-source project](https://github.com/salmanfs815/HifzCompanion), with signed APK releases available from GitHub.

The problem it solves is simple to describe, but more involved to model well: once someone has memorized a growing set of pages, remembering *what* to revise each day becomes a planning problem. Some pages are newly memorized and need frequent repetition. Some are well-established and should still be revisited before they fade. Others have become weak and need immediate, repeated attention. A flat checklist does not express those differences very well.

This post covers the product idea, the scheduling model, and the technical decisions behind the app.

## From a personal workflow to a product

Hifz is the Arabic term commonly used for memorizing the Qur'an. Revision is a long-term part of that practice: the goal is not just to memorize a page once, but to retain it over time.

The first idea was not "build a spaced-repetition app." It was to answer a more concrete daily question: *given the pages I have memorized, what should I revise today?* Existing trackers can record progress, but I wanted a tool that could distinguish between newly memorized material, pages that need ongoing rotation, and pages that have been forgotten or become unstable.

That led to three queues:

- **New** pages move through a short stabilization schedule.
- **Long-term** pages are rotated on a configurable interval.
- **Recovery** pages receive priority after a Weak or Failed review.

During a review session, a user rates a page as Strong, Good, Weak, or Failed. That result updates the page's state and determines when it will appear again. A strong long-term review can extend the next interval, while a weak or failed result moves the page into a recovery path. Recovery requires consecutive Strong reviews before a page returns to normal rotation.

The result is intentionally not a black-box algorithm. The schedules, daily capacity, maximum overload, review order, and minimum daily long-term workload are all configurable. The app should help people maintain a realistic routine, rather than insist that one schedule fits everyone.

## The scheduler is the core of the app

The most important design decision was to make the scheduler a pure Kotlin module with no Android dependency. Given a page's current `Progress`, a review result, the date, and a `SchedulerConfig`, it returns the next state.

Conceptually, it looks like this:

```kotlin
fun scheduleNext(
    progress: Progress,
    result: ReviewResult,
    today: LocalDate,
    config: SchedulerConfig
): Progress
```

This gives the scheduling logic a small, deterministic boundary. It does not know about Compose screens, a database, or background workers. It only applies policy.

There are two separate problems here. The first is a state transition: after reviewing one page, when should it next be due and which queue does it belong in? The second is daily selection: given all pages that are due, which ones fit into today's workload?

For daily selection, recovery work is ordered first, then new material, then long-term work. The scheduler respects a maximum daily capacity, but it also calculates a long-term target from the number of memorized pages and the desired rotation period. This keeps an expanding set of memorized pages from silently falling behind just because the daily list happens to be light.

Separating transition logic from daily selection made the rules easier to reason about and test. It also helped avoid a common mistake in scheduling products: treating "due" as the same thing as "should be shown right now." A page may be due, but a user still needs a bounded, ordered list that is possible to complete.

## Correcting a review is harder than it looks

One of the more interesting edge cases was allowing a user to correct a rating in the activity log. Changing a rating is not merely updating a label on an old record. That old rating may have sent the page into recovery, changed its strength, or altered every subsequent due date.

The solution is to retain review history and replay it when a rating is corrected. The app rebuilds the page's progress state from the first recorded review, applies each event in order using the same scheduler, and then persists the reconstructed state. This preserves an important invariant: current progress is derived from its review sequence, rather than becoming disconnected from it after an edit.

That approach costs a little more work than overwriting a field, but it is easier to trust. It also means the behaviour remains consistent as the scheduling rules evolve.

## Why local-first matters

The app is useful in contexts where a connection is unreliable or simply not desired. More importantly, revision history is personal information. I did not want an account requirement or a remote service to be the normal path through the product.

Room is the source of truth for pages, review events, and scheduling state. Preferences such as theme and scheduler settings live in DataStore. The UI observes state through Kotlin `Flow`, so screens update naturally as local data changes.

The app works fully offline. Google Drive is optional and is only used for backup and restore. Backups are written as a versioned JSON snapshot to the user's private Drive `appDataFolder`, rather than a visible Drive folder. The app requests only the Drive AppData scope, and scheduled backups are handled by WorkManager after the user has granted access.

There is a useful reliability benefit to this design as well: an OAuth or network failure cannot prevent onboarding, viewing the daily queue, or recording a review. The cloud feature is an enhancement to a complete local experience, not a dependency for it.

## Architecture and module boundaries

The project uses a modular clean-architecture layout:

```text
Compose UI → domain use cases → repository contracts
                                  ↓
                        Room / DataStore implementations

Domain use cases → pure Kotlin scheduler
```

The `app` module contains Compose screens, navigation, onboarding, and application wiring. `core-domain` contains models, repository contracts, and use cases. `core-data` owns Room, DataStore, migrations, and repository implementations. The scheduling policy has its own `scheduler` module.

Features with distinct responsibilities are separate modules as well: `backup`, `notifications`, `analytics`, `settings`, and `core-ui`. Hilt connects the concrete implementations at the application boundary.

This is more structure than a small app strictly requires, but it paid off in a few ways. It kept Android-specific implementation details out of the business rules, made the scheduler fast to test on the JVM, and made the Google Drive integration optional from the domain's perspective. The domain layer depends on a backup contract; the Drive implementation remains behind that contract.

## Technology choices

The app is written in Kotlin using Coroutines and Flow. Jetpack Compose and Material 3 were a good fit for a state-driven Android UI, while adaptive Compose layouts allow the app to work well on larger screens without maintaining a separate tablet UI.

For persistence, I chose Room because the data has real relationships: page progress, review events, scheduler state, and migrations all benefit from a relational local database. DataStore is a better fit for smaller preference-style values such as appearance and capacity settings.

Other important pieces of the stack are:

- **Hilt** for dependency injection and clear composition at the app boundary.
- **WorkManager** for resilient notification scheduling and periodic backup work that respects Android background-execution constraints.
- **Google Identity Services and the Drive API** for the optional private backup flow.
- **Kotlinx Serialization** for portable, versioned JSON snapshots.
- **Android Keystore and Tink** to protect locally stored OAuth material.
- **JUnit, coroutine tests, and Compose UI tests** for verification across the scheduler, domain workflows, backup flow, and UI.

I chose familiar Android platform libraries over introducing a backend or a custom synchronization service. For this problem, reducing operational complexity was more valuable than adding infrastructure.

## Testing the rules, not only the screens

The most valuable tests are not visual tests. They exercise the policy: queue transitions, capacity and overload behaviour, custom intervals, recovery exit rules, inactive pages, review ordering, and due-date selection.

There are also end-to-end JVM tests for the offline journey—onboarding, daily queue creation, review, recovery, and history—and for backups. The backup test uses in-memory Drive storage and separate source and destination repositories. This models a restore on a fresh installation without requiring live Google credentials in CI.

Instrumentation tests cover the Compose onboarding and the offline path. OAuth consent and real notification delivery remain in a manual release checklist because they depend on external accounts, Play Services, and OS scheduling behaviour.

## What I learned

The project reinforced that an app's hardest part is often not its UI. The interesting work here was deciding which business rules had to be explicit, preserving those rules when historical data changes, and making the product dependable without a server.

It also made me more deliberate about boundaries. A pure scheduling engine is easier to test than one buried in a ViewModel. A local-first core is easier to use and reason about than an app that assumes the network is always present. And an optional backup feature is safer when its permissions and failure modes are narrow and visible.

Hifz Companion is still evolving, but it is already a useful example of how a focused personal workflow can become a real product: one with clear constraints, meaningful state, and technical decisions shaped by the people who will rely on it.

The source code, releases, architecture notes, and testing guide are available on [GitHub](https://github.com/salmanfs815/HifzCompanion).
