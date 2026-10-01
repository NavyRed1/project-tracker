# SyncPlanner

A project planner with a Java REST API and a database behind it. Plan work as a list, a kanban board, a Gantt timeline, or a calendar. Check team workload, and see delivery health at a glance.

The front end is plain HTML, CSS and JavaScript with no build step. The backend is Spring Boot with JPA. If the backend isn't running, the app falls back to browser storage, so the front end always works on its own.

## What's inside

**Views**
- **Task list:** sortable table with search and a priority filter, plus edit and delete.
- **Board:** kanban columns (Planned, In Progress, Delayed, Completed). Drag a card to change its status.
- **Timeline:** Gantt chart with weekend shading, a "today" line, and bars that fill as checklist items are completed.
- **Calendar:** month grid, plus an hour-by-hour day view.
- **Workload:** hours per person measured against a 40-hour week. The meter turns red when someone is over.
- **Insights:** completion rate, status mix, hours by person, and a list of work that is overdue or due within 7 days.

**Everything else**
- Command palette: press `Ctrl` or `Cmd` + `K` to jump to a view, create a task, export, or edit any task by name.
- Daily briefing at the top of the page (what's due this week, what's overdue).
- World clock with a seconds ring and one-click timezone chips: WIB, SGT, UTC, EST, CET.
- Light and dark theme switch. The new theme spreads outward from the switch where the browser supports it.
- Checklists on every task, with progress shown on cards and the timeline.
- History of completed projects and an audit trail of changes.
- Export to CSV or a JSON backup.
- Layered "sticky note" buttons with a soft glow, in a green, lime and avocado palette.

## Project structure

```
index.html                  App markup and all JavaScript
styles.css                  All styling (design tokens, themes, components)
syncplanner-backend.zip     Spring Boot API
  pom.xml
  src/main/resources/application.properties
  src/main/java/com/syncplanner/
    SyncPlannerApplication.java   Entry point, CORS, sample data
    ApiController.java            REST endpoints
    Task.java, Subtask.java       Task data model
    HistoryEntry.java             Completed-project log
    AuditEntry.java               Audit trail
    Repositories.java             Spring Data repositories
```

`index.html` loads `styles.css` from the same folder, so keep the two files together.

## Quick start

### Front end only

Open `index.html` in a browser. Data is stored in your browser's local storage, and the subtitle under the logo reads **Local storage**.

### With the Java backend

Requirements: JDK 17 or newer and Maven.

```bash
unzip syncplanner-backend.zip
cd backend
mvn spring-boot:run
```

The API starts on `http://localhost:8080` and creates a database file in `./data` on first run. It loads sample tasks the first time, with dates relative to today.

Then open `index.html`. When the API is reachable, the subtitle under the logo reads **Java API connected** and all tasks, history and audit entries are saved in the database. If the API stops responding, the app switches back to local storage and shows a short notice.

To point the page at a different server, set this before the main script in `index.html`:

```html
<script>window.SYNC_API = 'https://your-host';</script>
```

## API

Base URL: `http://localhost:8080/api`

| Method | Path | Purpose |
|---|---|---|
| GET | `/tasks` | List all tasks |
| POST | `/tasks` | Create a task (the server assigns the ID) |
| PUT | `/tasks/{id}` | Update a task |
| DELETE | `/tasks/{id}` | Delete a task |
| GET | `/history` | Completed projects, newest first |
| GET | `/audit` | Audit trail, newest first |
| POST | `/audit` | Add an audit entry (`{ "action": "...", "details": "..." }`) |

Creating, updating and deleting tasks are written to the audit trail automatically. Moving a task to Completed adds a history entry.

Example task:

```json
{
  "id": "TSK-101",
  "name": "Brand identity and color palette redesign",
  "assignee": "Alex Morgan",
  "priority": "Urgent",
  "status": "In Progress",
  "start": "2026-10-01",
  "end": "2026-10-10",
  "hours": 16,
  "startHour": 9,
  "color": "#3fa34d",
  "subtasks": [
    { "text": "Define primary tokens", "done": true },
    { "text": "Create gradient assets", "done": false }
  ]
}
```

Priorities are `Low`, `Medium`, `High` and `Urgent`. Statuses are `Planned`, `In Progress`, `Delayed` and `Completed`.

## Database

By default the backend uses an embedded H2 database stored in `./data/syncplanner.mv.db`, so data survives restarts. Tables: `tasks`, `subtasks`, `history` and `audit_log`.

Browse it at `http://localhost:8080/h2-console` with JDBC URL `jdbc:h2:file:./data/syncplanner`, user `sa`, and an empty password.

**Switching to PostgreSQL**

1. In `pom.xml`, uncomment the PostgreSQL dependency.
2. In `application.properties`, replace the H2 datasource lines:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/syncplanner
spring.datasource.username=postgres
spring.datasource.password=change-me
spring.jpa.hibernate.ddl-auto=update
```

## Browser support

Works in current Chrome, Edge, Firefox and Safari. The circular theme-change animation uses the View Transitions API (Chrome, Edge, Safari). Other browsers, and anyone with reduced motion turned on, get an instant theme change instead.

## Notes before you deploy

- **CORS is open.** The sample configuration allows any origin on `/api/**` so the page can be opened from disk. Restrict `allowedOriginPatterns` in `SyncPlannerApplication.java` to your own domain in production.
- **There is no authentication yet.** Anyone who can reach the API can read and change tasks.
- **The H2 console is enabled** for convenience. Turn it off (`spring.h2.console.enabled=false`) for anything public.
- **Local storage keys** are `syncplanner_v2_tasks`, `syncplanner_v2_history`, `syncplanner_v2_audit` and `syncplanner_theme`. Clear them to reset the demo data.

## Roadmap ideas

- Sign-in and per-team workspaces
- Docker Compose setup (API, database and static front end)
- Automated tests for the API and the scheduling views
- Task comments, attachments and notifications
- Calendar (.ics) export from the server

## License

Add the license you want to distribute under before sharing or selling this project.
