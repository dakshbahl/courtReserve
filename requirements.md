# 🎾 RallyPass — Requirements Document  

## 1. App Overview  

RallyPass is a simple court reservation app for Montgomery Township tennis and pickleball courts.  

The app lets users reserve courts online instead of waiting in a physical line during busy times.  
Users can create an account, choose a sport, pick a court, select an available time slot, and confirm a reservation.  

The app is designed for local public courts that get crowded in the summer.  
It helps players know when courts are available and helps reduce arguments, confusion, and long wait times.

The first version is built for one location with:

- 8 tennis courts
- 6 pickleball courts

The app is **iOS-only for MVP**, built with **Swift and SwiftUI**.

A future **Android** release is planned, but is out of scope for MVP. All booking rules and data live in **Supabase** so a later Android client can use the same backend without rewriting business logic.

---

## 1.1 Platform & Tech Stack

### MVP (build now)

| Layer | Choice | Why |
|--------|--------|-----|
| Mobile | **Swift + SwiftUI** (iOS) | Best native iOS experience, strong Apple tooling, matches your learning path and App Store goals. |
| Backend | **Supabase** (Auth, Postgres, RLS, Edge Functions later) | Same API for any future client (iOS, Android, web). |
| Entry point | `CourtReserveApp.swift` | `@main` app file; single Xcode target, no extra modules for MVP. |

### Post-MVP (do not block launch)

| Layer | Choice | When |
|--------|--------|------|
| Analytics | **TelemetryDeck** (optional) | After core booking flows work; privacy-friendly event tracking only. |
| Monetization | **RevenueCat** (placeholder only) | When township SaaS or paid tiers are defined; **no paywall in MVP**. |

### Future Android (not MVP)

SwiftUI does **not** run on Android. When you ship on Google Play, pick one path and document it in a follow-up requirements update:

1. **Native Android app (Kotlin + Jetpack Compose)** — Recommended if you want the best experience on each platform. Reuse Supabase; reimplement UI and call the same tables/APIs.
2. **Cross-platform rewrite (Flutter or React Native)** — One UI codebase for iOS + Android, but means rebuilding the iOS app you already shipped (or maintaining two stacks during migration).
3. **Kotlin Multiplatform (KMP)** — Share business logic in Kotlin; still separate SwiftUI and Compose UIs.

**Recommendation for RallyPass:** Build the **iOS SwiftUI MVP now**. Keep **all rules in Supabase** (court overlap checks, user overlap limits, RLS). That makes a later **Android (Kotlin)** app straightforward without locking you into a cross-platform framework on day one.

---

## 2. Main Goals  

1. Let users reserve tennis and pickleball courts from their phone.  
2. Let users choose their own court and time slot.  
3. Prevent one person from having two active reservations at overlapping times (tennis and pickleball on the same day are fine if times do not overlap).  
4. Prevent double-booking the same court at the same time.  
5. Show upcoming, past, and cancelled reservations.  
6. Let users cancel reservations anytime.  
7. Give admins tools to manage courts, reservations, blocks, and hours.  
8. Make the app clean, simple, and easy for local residents to use.  
9. Build the app as a pitchable SaaS product for Montgomery Township.  
10. Keep the MVP free for players; do not integrate paid features or paywalls at launch.  
11. Keep backend and booking rules in Supabase so a future Android app can share the same data and policies.

---

## 3. User Stories  

- **US-001**: As a user, I want to create an account so I can reserve courts fairly.  
- **US-002**: As a user, I want to log in with my email and password so my reservations are saved to my account.  
- **US-003**: As a user, I want to choose between tennis and pickleball so I can reserve the right type of court.  
- **US-004**: As a user, I want to pick a specific court so I know exactly where I am playing.  
- **US-005**: As a user, I want to see available time slots so I do not accidentally book an unavailable time.  
- **US-006**: As a user, I want to book 30, 60, or 90 minutes depending on how long I want to play.  
- **US-007**: As a user, I want the app to stop people from double-booking a court.  
- **US-008**: As a user, I want the app to stop me from booking two reservations at overlapping times (but allow tennis and pickleball on the same day if the times do not conflict).  
- **US-009**: As a user, I want to cancel my reservation if I cannot make it.  
- **US-010**: As a user, I want to see my upcoming and past reservations in one place.  
- **US-011**: As a user, I want to receive a confirmation email after booking.  
- **US-012**: As an admin, I want to view all reservations so I can manage court usage.  
- **US-013**: As an admin, I want to block courts for maintenance or events.  
- **US-014**: As an admin, I want to change court hours if the township schedule changes.  
- **US-015**: As an admin, I want to cancel reservations when needed.  

---

## 4. Features  

- **F-001 — User Account Creation**  
  - What it does: Lets users create an account with full name, email, phone number, and password.  
  - When: User opens the app for the first time or taps sign up.  
  - What happens if it fails: Show a clear message like “Could not create account. Please try again.”  
  - Uses Supabase Auth.

- **F-002 — Login System**  
  - What it does: Lets users log in using email and password.  
  - When: Returning users open the app.  
  - What happens if it fails: Show a message like “Invalid email or password.”  
  - Uses Supabase Auth.

- **F-003 — User Profile**  
  - What it does: Saves user details after sign up.  
  - Stores:
    - full name
    - email
    - phone number
    - role
  - Roles:
    - user
    - admin

- **F-004 — Sport Selection**  
  - What it does: Lets users choose between tennis and pickleball.  
  - Tennis shows 8 courts.  
  - Pickleball shows 6 courts.  

- **F-005 — Court Selection**  
  - What it does: Lets users choose their own individual court.  
  - Tennis courts:
    - Tennis Court 1
    - Tennis Court 2
    - Tennis Court 3
    - Tennis Court 4
    - Tennis Court 5
    - Tennis Court 6
    - Tennis Court 7
    - Tennis Court 8
  - Pickleball courts:
    - Pickleball Court 1
    - Pickleball Court 2
    - Pickleball Court 3
    - Pickleball Court 4
    - Pickleball Court 5
    - Pickleball Court 6

- **F-006 — Date Selection**  
  - What it does: Lets users choose a reservation date.  
  - Users can book from today up to 7 days in advance.  
  - Dates more than 7 days ahead should be disabled.  

- **F-007 — Time Slot Selection**  
  - What it does: Shows available start times every 30 minutes.  
  - Court hours are 6:00 AM to 9:00 PM.  
  - The app should not allow a reservation to go past 9:00 PM.  
  - Unavailable times should be greyed out or disabled.

- **F-008 — Duration Selection**  
  - What it does: Lets users choose how long they want to play.  
  - Options:
    - 30 minutes
    - 60 minutes
    - 90 minutes
  - Maximum reservation length is 90 minutes.

- **F-009 — Singles/Doubles Selection**  
  - What it does: Lets users choose whether they are playing singles or doubles.  
  - Options:
    - Singles
    - Doubles

- **F-010 — Player Count Selection**  
  - What it does: Lets users enter or select how many players are coming.  
  - This helps show how the court is being used.

- **F-011 — Booking Confirmation**  
  - What it does: Shows a summary before the user confirms.  
  - Summary includes:
    - sport
    - court name
    - date
    - start time
    - end time
    - duration
    - singles/doubles
    - player count
  - User taps “Confirm Reservation” to book.

- **F-012 — Reservation Storage**  
  - What it does: Saves confirmed reservations to Supabase.  
  - Each reservation stores:
    - user ID
    - court ID
    - sport
    - reservation date
    - start time
    - end time
    - duration
    - play type
    - player count
    - status
    - created date

- **F-013 — Double Booking Prevention**  
  - What it does: Prevents the same court from being booked by two people at overlapping times.  
  - A court is unavailable if another active reservation overlaps the requested time.  
  - Overlap rule:
    - existingStart < requestedEnd && existingEnd > requestedStart

- **F-014 — User Booking Overlap Limit**  
  - What it does: Stops one user from being double-booked against themselves.  
  - A user cannot have two **active, not-yet-ended** reservations whose times overlap — regardless of sport.  
  - Tennis and pickleball on the same calendar day are allowed when the time windows do not overlap (e.g. tennis 2:00–3:00 PM and pickleball 4:00–5:00 PM).  
  - If a reservation for that day has **already ended**, it does not block another booking the same day (including the same sport).  
  - Cancelled reservations do not block new bookings.  
  - Uses the same overlap rule as court double-booking: `existingStart < requestedEnd && existingEnd > requestedStart`

- **F-015 — Cancel Reservation**  
  - What it does: Lets users cancel their own reservation anytime.  
  - Cancelled reservations should update status to cancelled.  
  - The court time becomes available again after cancellation.

- **F-016 — My Reservations List**  
  - What it does: Shows the user their reservations.  
  - Sections:
    - Upcoming
    - Past
    - Cancelled
  - Each reservation card shows:
    - sport
    - court
    - date
    - start time
    - end time
    - duration
    - singles/doubles
    - player count
    - status

- **F-017 — Confirmation Email**  
  - What it does: Sends or triggers a confirmation email after booking.  
  - For MVP, this can be a clean placeholder service that later connects to Supabase Edge Functions, Resend, or SendGrid.  
  - Email should include:
    - sport
    - court
    - date
    - time
    - duration
    - cancellation reminder

- **F-018 — Admin Dashboard**  
  - What it does: Gives admins tools to manage the court system.  
  - Only users with role “admin” can access it.  
  - Normal users should not see the Admin tab.

- **F-019 — Admin Reservation Management**  
  - What it does: Lets admins view, filter, cancel, or delete reservations.  
  - Filters:
    - date
    - sport
    - court
    - user
    - status

- **F-020 — Court Blocking**  
  - What it does: Lets admins block a court for maintenance, events, or township programs.  
  - Blocked courts should not be available for users to reserve.  

- **F-021 — Court Management**  
  - What it does: Lets admins view courts and activate/deactivate courts.  
  - Inactive courts should not be bookable.

- **F-022 — App Settings Management**  
  - What it does: Lets admins manage app settings.  
  - Settings:
    - opening time
    - closing time
    - max booking minutes
    - booking window days

- **F-023 — TelemetryDeck Analytics (post-MVP / optional)**  
  - What it does: Tracks important app events without collecting sensitive data.  
  - **Not required for MVP launch.** Add after auth and booking flows are stable.  
  - Events:
    - app_opened
    - signup_started
    - signup_completed
    - login_completed
    - logout_completed
    - booking_started
    - sport_selected
    - date_selected
    - court_selected
    - time_selected
    - booking_confirmed
    - booking_failed
    - reservation_cancelled
    - admin_dashboard_opened
    - admin_reservation_cancelled
    - court_block_created

- **F-024 — RevenueCat Placeholder (post-MVP / optional)**  
  - What it does: Adds a clean RevenueCat service layer for future monetization.  
  - **Do not integrate for MVP** unless scaffolding an empty no-op service stub.  
  - MVP should not charge users.  
  - Do not block booking behind a paywall.  
  - Future monetization could be township SaaS, admin subscription, premium analytics, or multi-location support.

- **F-025 — Debug Logs and Comments**  
  - What it does: Adds safe, readable debugging during development.  
  - Use `#if DEBUG` (or equivalent) so verbose logs are not shipped to production.  
  - Add comments only for non-obvious business logic (overlap rules, user overlap limits, admin checks)—not for self-explanatory UI code.  
  - Debug logs should be added for:
    - app launch
    - auth state changes
    - booking attempts
    - booking success
    - booking failure
    - reservation cancellation
    - Supabase errors
    - RevenueCat configuration
    - TelemetryDeck events
  - Never log passwords, auth tokens, or private keys.

---

## 5. Screens  

- **S-000 — Welcome Screen**  
  - Shows:
    - "Welcome to RallyPass" title.
    - Short explanation of the app.
    - Text explaining that users can reserve tennis and pickleball courts online.
    - "Get Started" button.
  - Navigation:
    - Tap "Get Started" → goes to **S-001 Auth Screen** if not logged in.
    - If already logged in, goes to **S-002 Home Screen**.

- **S-001 — Auth Screen**  
  - Shows:
    - Login form.
    - Sign up form.
    - Forgot password placeholder.
  - Sign up fields:
    - full name
    - email
    - phone number
    - password
  - Login fields:
    - email
    - password
  - Navigation:
    - Successful login → goes to **S-002 Home Screen**.
    - Successful sign up → creates profile and goes to **S-002 Home Screen**.

- **S-002 — Home Screen**  
  - Shows:
    - App name: RallyPass.
    - Welcome message.
    - Tennis card.
    - Pickleball card.
    - Today’s availability summary.
    - "Book a Court" button.
  - Navigation:
    - Tap Tennis card → goes to **S-003 Book Court Screen** with Tennis selected.
    - Tap Pickleball card → goes to **S-003 Book Court Screen** with Pickleball selected.
    - Tap "Book a Court" → goes to **S-003 Book Court Screen**.
  - Design:
    - White background.
    - Blue accent color.
    - Rounded cards.
    - Simple sports app look.

- **S-003 — Book Court Screen**  
  - Shows the full booking flow:
    1. Select sport.
    2. Select date.
    3. Select court.
    4. Select start time.
    5. Select duration.
    6. Select singles or doubles.
    7. Select player count.
    8. View booking summary.
    9. Confirm reservation.
  - Interactions:
    - Unavailable times are disabled or greyed out.
    - Available times are clickable.
    - Confirm button creates reservation.
    - If booking fails, show a clear error.
  - Navigation:
    - Successful booking → goes to **S-004 Booking Confirmation Screen**.
    - Back button → returns to Home.

- **S-004 — Booking Confirmation Screen**  
  - Shows:
    - Success message.
    - Sport.
    - Court name.
    - Date.
    - Start time.
    - End time.
    - Duration.
    - Singles/doubles.
    - Player count.
    - Message saying confirmation email was triggered.
  - Navigation:
    - "View My Reservations" → goes to **S-005 My Reservations Screen**.
    - "Back Home" → goes to **S-002 Home Screen**.

- **S-005 — My Reservations Screen**  
  - Shows:
    - Upcoming reservations.
    - Past reservations.
    - Cancelled reservations if useful.
  - Each reservation row/card shows:
    - Court name.
    - Sport.
    - Date.
    - Start time.
    - End time.
    - Duration.
    - Play type.
    - Player count.
    - Status.
  - Interactions:
    - Upcoming reservations show "Cancel Reservation" button.
    - Tapping cancel asks for confirmation.
    - After cancellation, reservation status updates to cancelled.
  - Empty state:
    - "No reservations yet — book a court to get started!"

- **S-006 — Profile Screen**  
  - Shows:
    - full name
    - email
    - phone number
    - role
    - logout button
  - Navigation:
    - Logout → returns to **S-001 Auth Screen**.

- **S-007 — Admin Dashboard Screen**  
  - Only visible to admin users.
  - Shows:
    - all reservations
    - filters
    - court management
    - court block tools
    - app settings
    - basic usage summary
  - Admin tools:
    - View reservations by date.
    - Filter by sport.
    - Filter by court.
    - Filter by user.
    - Filter by status.
    - Cancel reservations.
    - Block courts.
    - Activate/deactivate courts.
    - Change opening and closing hours.
  - Navigation:
    - Only appears as an Admin tab if profile role is admin.

---

## 6. Data  

- **D-001:** User profile with:
  - user ID
  - full name
  - email
  - phone number
  - role
  - created date

- **D-002:** Courts with:
  - court ID
  - court name
  - sport
  - court number
  - active/inactive status
  - created date

- **D-003:** Reservations with:
  - reservation ID
  - user ID
  - court ID
  - sport
  - reservation date
  - start time
  - end time
  - duration in minutes
  - play type
  - player count
  - status
  - created date

- **D-004:** Court blocks with:
  - block ID
  - court ID
  - start time
  - end time
  - reason
  - created by
  - created date

- **D-005:** App settings with:
  - opening time
  - closing time
  - max booking minutes
  - booking window days
  - updated date

- **D-006:** Analytics events:
  - app opened
  - signup completed
  - login completed
  - booking started
  - booking confirmed
  - booking failed
  - reservation cancelled
  - admin actions

All reservation, court, user, and admin data is saved in **Supabase**.

---

## 7. Extra Details  

- **MVP platform:** iOS only, built with **Swift** and **SwiftUI**.
- Use UIKit only if absolutely necessary.
- **Future platform:** Android via a separate client (recommended: Kotlin + Jetpack Compose) sharing the same Supabase project—see **§1.1 Platform & Tech Stack**.
- Use Supabase for authentication, database, Row Level Security, and server-side booking rules where possible.
- **RevenueCat:** future monetization only; no paywall, no required SDK work for MVP.
- **TelemetryDeck:** optional post-MVP analytics; not required to ship MVP.
- Users must be logged in to book courts.
- Normal users should not pay in the MVP.
- The app should be pitched as a township SaaS product.
- The MVP should not include:
  - waitlist
  - GPS check-in
  - QR check-in
  - booking bans
  - no-show penalties
  - user booking fees
  - push notifications
- Confirmation email should be structured as a placeholder service for now.
- App design:
  - clean modern sports app
  - white background
  - blue accent color
  - rounded cards
  - simple layout
  - readable text
  - easy for older residents to use

Required environment values (MVP):

1. Supabase URL  
2. Supabase anon key  
3. Current environment: `dev` | `test` | `prod`

Optional (post-MVP only):

4. RevenueCat API key  
5. TelemetryDeck app ID  

Do not overwrite the real `.env` file without asking first.

### Domain rules (enforce in Supabase + app)

These rules must stay consistent across iOS today and any future Android client:

- **Court double booking:** Reject overlapping reservations on the same court (`existingStart < requestedEnd && existingEnd > requestedStart`).
- **User overlap limit:** Reject a new booking if the user already has an active, not-yet-ended reservation on that day whose time overlaps the requested slot — regardless of sport. Past-ended and cancelled reservations do not block. Tennis + pickleball on the same day are allowed when times do not overlap.
- **Booking window:** Up to 7 days in advance; court hours 6:00 AM–9:00 PM; max duration 90 minutes; slots every 30 minutes.
- **Auth required:** Users must be logged in to book.
- **Admin access:** Only `role = admin` sees admin tools; enforce with Supabase RLS, not UI alone.

---

## 8. Build Steps  

- **B-000:** Create **CourtReserveApp.swift** as the main app entry point.  
- **B-001:** Create core app structure with folders for Core, Models, Services, ViewModels, Views, Database, UnitTests, and UITests.  
- **B-002:** Add environment configuration for dev, test, and prod.  
- **B-003:** Add AppLogger for debug logs and safe readable debugging.  
- **B-004:** Add AppConstants for court hours, booking duration limits, colors, and app name.  
- **B-005:** Connect Supabase service using environment values.  
- **B-006:** Build Supabase AuthService with sign up, login, logout, and auth state handling.  
- **B-007:** Build **S-000 Welcome Screen** with app introduction and Get Started button.  
- **B-008:** Build **S-001 Auth Screen** with login, sign up, and forgot password placeholder.  
- **B-009:** Create user profile model and save profile after sign up.  
- **B-010:** Create Court model and Supabase courts table.  
- **B-011:** Add seed data for 8 tennis courts and 6 pickleball courts.  
- **B-012:** Build **S-002 Home Screen** with Tennis card, Pickleball card, availability summary, and Book a Court button.  
- **B-013:** Build **S-003 Book Court Screen** with sport, date, court, time, duration, play type, and player count selection.  
- **B-014:** Add time slot generation from 6:00 AM to 9:00 PM in 30-minute intervals.  
- **B-015:** Add booking duration validation for 30, 60, and 90 minutes.  
- **B-016:** Add reservation overlap logic to prevent double booking.  
- **B-017:** Add user booking overlap limit logic (no overlapping active/future reservations per user; same-day multi-sport allowed when times do not conflict).  
- **B-018:** Add court block checking so blocked courts cannot be reserved.  
- **B-019:** Create reservation in Supabase after successful validation.  
- **B-020:** Build **S-004 Booking Confirmation Screen**.  
- **B-021:** Add EmailConfirmationService placeholder and trigger it after successful booking.  
- **B-022:** Build **S-005 My Reservations Screen** with upcoming, past, and cancelled reservations.  
- **B-023:** Add cancel reservation flow and update reservation status to cancelled.  
- **B-024:** Build **S-006 Profile Screen** with user info and logout button.  
- **B-025:** Add role-based admin detection from profile role.  
- **B-026:** Build **S-007 Admin Dashboard Screen**.  
- **B-027:** Add admin reservation filters by date, sport, court, user, and status.  
- **B-028:** Add admin cancel/delete reservation functionality.  
- **B-029:** Add admin court blocking functionality.  
- **B-030:** Add admin court activation/deactivation functionality.  
- **B-031:** Add admin app settings management for opening time, closing time, max booking minutes, and booking window days.  
- **B-032:** Add Supabase Row Level Security policies (users, reservations, admin actions).  
- **B-033:** Add unit tests for time slot generation, court reservation overlap, booking validation, and user booking overlap limit.  
- **B-034:** Add UI tests for auth flow, booking flow, reservation cancellation, and admin dashboard access.  
- **B-035:** Polish UI with white background, blue accents, rounded cards, loading states, empty states, and error states.  
- **B-036 (optional, post-MVP):** Add TelemetryDeck analytics service and track core events from **F-023**.  
- **B-037 (optional, post-MVP):** Add RevenueCat service placeholder with no paywall from **F-024**.  

---

## 9. Cursor Coding Rules  

Before making any code changes, Cursor must:

1. Read this requirements doc and `cursorRules.md`.  
2. Read the existing project structure.  
3. Identify relevant existing files.  
4. Check if similar code already exists.  
5. State confidence level; if below 95%, ask follow-up questions before coding.  
6. If confidence is 95% or higher, make the smallest clean change needed.  
7. Use `#if DEBUG` for verbose logs; avoid shipping debug noise to production.  
8. Avoid duplicate logic; keep files under 200–300 lines where possible.  
9. Do not create fake data for dev or prod.  
10. Do not overwrite `.env`.  
11. Do not create extra modules or packages.  
12. Prefer Supabase/RLS for booking rules so a future Android app stays in sync.  

Project rules file: add `cursorRules.md` to **Cursor → Settings → Rules → Project Rules** (or `.cursor/rules/`) so it applies every session—not only when @-mentioned.

---

## 10. Suggested File Organization  

CourtReserve/
  CourtReserveApp.swift

  Core/
    AppEnvironment.swift
    AppLogger.swift
    AppConstants.swift

  Models/
    UserProfile.swift
    Court.swift
    Reservation.swift
    CourtBlock.swift
    AppSettings.swift

  Services/
    SupabaseService.swift
    AuthService.swift
    CourtService.swift
    ReservationService.swift
    AdminService.swift
    RevenueCatService.swift
    AnalyticsService.swift
    EmailConfirmationService.swift

  ViewModels/
    AuthViewModel.swift
    HomeViewModel.swift
    BookingViewModel.swift
    ReservationsViewModel.swift
    ProfileViewModel.swift
    AdminViewModel.swift

  Views/
    Welcome/
      WelcomeView.swift

    Auth/
      AuthView.swift
      LoginView.swift
      SignUpView.swift

    Home/
      HomeView.swift

    Booking/
      BookCourtView.swift
      SportSelectorView.swift
      DateSelectorView.swift
      CourtSelectorView.swift
      TimeSlotSelectorView.swift
      DurationSelectorView.swift
      BookingSummaryView.swift
      BookingConfirmationView.swift

    Reservations/
      MyReservationsView.swift
      ReservationCardView.swift

    Profile/
      ProfileView.swift

    Admin/
      AdminDashboardView.swift
      AdminReservationListView.swift
      CourtBlockFormView.swift
      CourtManagementView.swift
      AppSettingsView.swift

    Components/
      PrimaryButton.swift
      SecondaryButton.swift
      CourtCardView.swift
      LoadingView.swift
      ErrorView.swift
      EmptyStateView.swift
      SectionHeaderView.swift

  Database/
    supabase_schema.sql
    supabase_seed.sql
    supabase_rls_policies.sql

  UnitTests/
    BookingValidationTests.swift
    TimeSlotGenerationTests.swift
    ReservationOverlapTests.swift
    UserBookingOverlapTests.swift

  UITests/
    AuthFlowUITests.swift
    BookingFlowUITests.swift
    ReservationCancelUITests.swift
    AdminAccessUITests.swift

---

### ✅ Done  

This document is ready to give to Cursor.  

It describes RallyPass’s behavior, features, screens, data, platform strategy, build order, and coding rules for MVP and future Android expansion.