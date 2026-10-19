# WorkPulse

**A workspace-based team and project management application built with React, TypeScript, Tailwind CSS, Vite, and Supabase.**

WorkPulse brings task and project workflows, daily reporting, team communication, notifications, and personal planning into one role-oriented application. It supports **Members, Managers, and Admins**, with role-specific interfaces and workspace-aware workflows.

> **Project status:** Core application workflows and the role-based route structure are implemented. Responsive refinement, physical-device checks, and production-readiness verification remain in progress. No live demo URL is listed yet.

## Table of Contents

- [Project Highlights](#project-highlights)
- [Features by Role](#features-by-role)
- [Technical Stack](#technical-stack)
- [Engineering Highlights](#engineering-highlights)
- [Data and Security](#data-and-security)
- [Run Locally](#run-locally)
- [Current Status and Next Steps](#current-status-and-next-steps)

## Project Highlights

- **Multi-role application:** Separate Admin, Manager, and Member experiences with role-aware navigation.
- **Workspace-oriented design:** Users operate within a selected workspace, with team membership and role relationships shaping available workflows.
- **End-to-end work management:** Assigned tasks, personal task lists, project workflows, daily reports, team reports, and next-day planning.
- **Built-in team communication:** Direct messages, Team Groups, and Workspace Groups, with read receipts and presence.
- **Account personalization:** Profile and avatar management, password updates, notifications, and persistent light/dark themes.
- **Database-backed workflows:** Supabase Auth, PostgreSQL, Row Level Security (RLS), Storage policies, database functions, and Realtime features.
- **Consistent route conventions:** Authenticated pages follow a `/role/page` pattern, such as `/admin/inbox`, `/manager/my-tasks`, and `/member/inbox`.

## Features by Role

### Member

- Submit and manage Daily Reports.
- View assigned tasks and related project workflows.
- Track time against tasks and review accumulated time in My Tasks.
- Plan the next workday using **Plan Your Tomorrow**.
- Use the Inbox for direct conversations, Team Group, and Workspace Group.
- Receive and manage notifications.
- Manage profile details, avatar, and password.
- Choose a light or dark theme in Settings.
- Leave a workspace, subject to workspace membership rules.

### Manager

- Submit Manager Daily Reports and review Team Reports.
- Work with team project and task workflows.
- Create a personal **Plan Your Tomorrow** plan.
- Manage personal items in **My Tasks**, separately from team tasks.
- Use direct conversations, Team Group, and Workspace Group in the Inbox.
- Receive notifications and manage profile/preferences.
- Leave a workspace only when not assigned as a team manager.

### Admin

- Access the Admin workspace interface and workspace details.
- Manage workspace-facing workflows and project/team areas available in the application.
- Update the workspace name; the workspace URL is read-only in Settings.
- Delete a workspace through a confirmation flow that requires typing the workspace name.
- Use direct conversations, the Admin/Managers Team Group, and Workspace Group.
- Manage profile details, avatar, password, and theme preferences.

## Technical Stack

| Technology | Use in WorkPulse |
|---|---|
| React | Component-based user interface |
| TypeScript | Typed components, props, state, and application logic |
| Vite | Development server and production build tooling |
| Tailwind CSS | Utility-first UI styling and responsive layout patterns |
| React Router | Client-side routes, nested role paths, and navigation |
| Supabase Auth | User authentication and password-related flows |
| Supabase PostgreSQL | Application data and relational workflows |
| Supabase Storage | Profile avatar uploads |
| Supabase Realtime | Live messaging and presence-related updates |

## Engineering Highlights

### Role-Based Routing

Authenticated pages use a consistent role-prefixed route convention, for example `/admin/dashboard`, `/admin/managers`, `/manager/dashboard`, `/manager/team-reports`, `/member/dashboard`, and `/member/tasks/:taskId`. Public authentication and workspace-entry routes remain separate. Internal navigation and redirects should use canonical paths, while role and workspace authorization continue to be enforced independently of the URL.

### Messaging and Inbox

The Inbox supports direct conversations plus shared Team Group and Workspace Group conversations. Implemented capabilities include message editing within the permitted time window, copying and deleting supported messages, read receipts, presence status, and loading/error handling when conversations are loaded or switched. Pin/Unpin was removed from the active message action menus in all three role-specific Inboxes.

The data model separates conversations, participants, messages, receipts, and user presence. Database functions support conversation creation and selected messaging actions. Presence handling was updated to scope activity to the selected workspace.

### Task and Time-Tracking Workflows

Project/team tasks are distinct from personal task lists. Member My Tasks displays accumulated task time using existing time-log data. Status updates persist to Supabase and now include visible error feedback for failed updates, guard against updating the same task concurrently, and clean up loading state on failure.

### Daily Reports and Planning

WorkPulse includes Member Daily Reports, Manager Daily Reports, and Manager Team Reports. Report editing preserves existing attachments unless a user explicitly removes them. Report-date handling was adjusted to use local calendar dates for date-only values. New report submissions also follow the manager-notification path.

Plan Your Tomorrow stores each user's plan by workspace, user, and plan date, retaining previous plans rather than replacing them. The active planning date changes at 4 PM: before 4 PM the active plan is for the current day; from 4 PM onward it is for the next day.

### Workspace and Data-Integrity Workflows

Workspace creation and invitation setup use a database-side RPC workflow intended to keep related creation steps atomic. Workspace deletion and leaving a workspace are separate actions: leaving removes the user's membership without deleting shared workspace data. Managers assigned to a team cannot leave until they are no longer assigned as that team's manager.

### Error Handling and UI Reliability

Recent QA-driven changes added or improved user-facing handling for failed task-status updates and Manager Inbox bootstrap queries, clarified selected creation-flow errors, and improved several data-persistence and navigation edge cases. These are code-level improvements; not every failure path has been deliberately induced in a runtime test.

## Data and Security

WorkPulse uses Supabase Auth and PostgreSQL for identity and application data. The database design includes workspace membership, role and team relationships, reports, projects, tasks, notifications, planning records, and messaging tables.

Security and data-handling measures in the project include:

- **Row Level Security (RLS)** policies for access to application records.
- Authenticated-role grants and row policies for task access, with authorization determined by the configured database rules.
- Storage policies for avatar access and writes to the user's own storage folder.
- Database functions for selected workflows, including workspace setup, conversation creation, and workspace leaving/deletion.
- Membership and role checks inside relevant database functions.
- Confirmation and workspace-name matching before the Admin workspace-delete action proceeds.

**Security note:** Review the actual deployed Supabase schema, grants, RLS policies, Storage rules, RPCs, and authentication redirect settings before production use. Role isolation and all negative-access cases have not been fully exercised against a production-like test environment. Never expose a Supabase `service_role` key in client code or a public repository; use only the public/anon key in the browser.

## Run Locally

1. Clone the repository.
2. Install dependencies using the package manager and lockfile included with the project.
3. Configure the Supabase project URL and public/anon client key used by the Supabase client initialization in `src/supabaseClient.ts`. For a personal deployment, prefer environment variables such as `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`, with matching client-code configuration and a non-secret `.env.example`; do not commit private secrets.
4. Confirm that the required database tables, RLS policies, Storage bucket/policies, and SQL functions have been applied to your Supabase project.
5. Start the development server with `npm run dev`.
6. Create a production build with `npm run build`.

The frontend depends on its Supabase schema and configuration; a fresh frontend checkout alone is not enough to reproduce all workflows.

## Current Status and Next Steps

The project has undergone iterative implementation and QA across Admin, Manager, and Member flows. The production build has passed during recent code-change checks, with a Vite bundle-size warning. The tracked code-level QA findings have been addressed or resolved, but this should not be interpreted as proof that the application has no remaining defects.

Initial responsive adjustments were made to shared headers, notification panels, profile-field wrapping, and the Member My Tasks table. The mobile experience has not yet been confirmed on a physical phone, so responsive work remains in progress.

Before describing WorkPulse as production-ready:

- Finish responsive layout refinements and validate the actual site on physical mobile devices at representative viewport sizes.
- Deploy to a stable URL and verify nested route refreshes and hosting fallback configuration.
- Exercise key end-to-end flows with role-separated test accounts, including workspace isolation and unauthorized access attempts.
- Test authentication/OAuth redirects, invitation flows, report attachments and notifications, Inbox failure handling, and task-update error paths.
- Review production Supabase policies, functions, Storage permissions, and grants.

The next planned milestone is deployment and focused acceptance testing, rather than another full static audit. Add the live demo URL and screenshots to this README when the deployed application has been checked.
