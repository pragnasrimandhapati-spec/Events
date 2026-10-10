# Prism — Campus event management

A colorful, responsive event management app for students and admins. Built with Bootstrap 5, vanilla JavaScript modules, Firebase Authentication, and Cloud Firestore.

## Included

- Student account creation and login; admin login only (no admin signup).
- Student dashboard with events, event registration, and personal registration status.
- Admin dashboard with student management, event management, and registration analytics.
- Email/password validation, duplicate email handling, and case-insensitive unique roll numbers enforced through a Firestore transaction.
- One registration per student per event, enforced with a deterministic registration document ID and a Firestore transaction.
- Firestore rules in `firestore.rules`.

## Firebase setup

1. In the Firebase Console for project `freshers26`, enable **Authentication → Sign-in method → Email/Password**.
2. Create a **Cloud Firestore** database.
3. Open Firestore **Rules**, paste in `firestore.rules`, and publish. Do not use open/test-mode rules for a deployed app.
4. Create the admin account yourself in **Authentication → Users → Add user**. There is no admin signup form in the app.
5. Copy the admin user's Authentication UID. In Firestore, create `users/{ADMIN_UID}` with these fields:

   | Field | Type | Value |
   |---|---|---|
   | `uid` | string | same as document ID / Auth UID |
   | `name` | string | administrator's display name |
   | `email` | string | administrator's email |
   | `role` | string | `admin` |
   | `createdAt` | timestamp | current timestamp |

   Admin profile documents are managed manually. The admin role is trusted by Firestore rules and must only be assigned to trusted accounts.

6. Serve this folder from a local/static web server. For example, from this directory run `python -m http.server 8000`, then open `http://localhost:8000`. ES modules do not work reliably when opening `index.html` directly as a `file://` URL.
7. For production, add your deployed hostname to **Authentication → Settings → Authorized domains** and configure Firebase App Check. The Firebase web config is public client configuration; Firestore rules and Auth are what protect data.

## Firestore collections

- `users/{uid}`: student profile or manually provisioned admin profile.
- `rollNumbers/{lowercase-roll}`: unique roll number reservation mapped to a student UID.
- `events/{eventId}`: event details and a transactionally maintained `registeredCount`.
- `registrations/{uid_eventId}`: one registration for each student/event pair.

## Notes

- Deleting a student from the admin view removes the Firestore profile and roll-number reservation. Firebase Auth user deletion is intentionally not attempted from a browser admin session; disable/delete that Auth account in Firebase Console as well.
- Event deletion leaves registration records in place for historical analytics.
- Analytics are based on Firestore registration records. For very large deployments, use server-side aggregation and Cloud Functions rather than reading every registration into the admin browser.
- This is a static client app. The included Firestore rules are essential; keep admin profile creation limited to the Firebase Console or another trusted server-side process.
