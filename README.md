# Man-Hours & Labor Cost System

A single-file PHP web application for tracking daily worker hours, shift-based costs, and payroll reporting at a construction/industrial site. Built for **Dynamic Technologies (Pvt) Ltd — Pallekelle Site, Sri Lanka**.

Part of a larger suite that includes a Contractor Attendance System, a DT Employee Attendance System, and a Document Manager — all sharing one MySQL database.

---

## Screenshots

> Add your screenshots to a `/screenshots` folder in the repo and update the paths below.

| | |
|---|---|
| **Login** | **Dashboard** |
| `![Login](screenshots/login.png)` | `![Dashboard](screenshots/login.png)` |
| **New Time Entry** | **Batch Entry** |
| `![Time Entry](screenshots/time-entry.png)` | `![Batch Entry](screenshots/batch-entry.png)` |
| **Browse Entries** | **Run Calculation** |
| `![Browse](screenshots/browse-entries.png)` | `![Calculate](screenshots/calculate.png)` |
| **Daily Report** | **Monthly Report** |
| `![Daily](screenshots/report-daily.png)` | `![Monthly](screenshots/report-monthly.png)` |
| **Cost Summary** | **By Employee** |
| `![Cost Summary](screenshots/cost-summary.png)` | `![Employee](screenshots/report-employee.png)` |
| **Shift Config** | **Hourly Rates** |
| `![Shifts](screenshots/shift-config.png)` | `![Rates](screenshots/rates.png)` |
| **Special Days** | **Manage Users** |
| `![Holidays](screenshots/special-days.png)` | `![Users](screenshots/manage-users.png)` |
| **Audit Log** | **My Profile** |
| `![Audit](screenshots/audit-log.png)` | `![Profile](screenshots/my-profile.png)` |

---

## Technology

| | |
|---|---|
| **Backend** | PHP 8.0+ with PDO (no framework) |
| **Database** | MySQL 5.7+ or MariaDB 10.3+ |
| **UI** | Bootstrap 5.3, Font Awesome 6.4 |
| **Charts** | Chart.js 4.4 |
| **Auth** | PHP sessions + `password_hash()` / `password_verify()` |
| **CDN** | All assets via `cdnjs.cloudflare.com` |

No Composer. No npm. No framework. Drop two files on any LAMP/XAMPP server and it runs.

---

## Files

```
mh_system.php      ← Entire application (3 400+ lines, single file)
mh_setup.php       ← One-click database installer (run once)
```

These sit alongside the rest of the suite:

```
index.php           Contractor Attendance
dt_attendance.php   DT Employee Attendance  
pdf_documents.php   Document Manager
footer.php          Shared footer
```

---

## Database

All tables use the `mh_` prefix inside the existing `contractor_attendance` database. The installer never touches any existing tables.

| Table | What it stores |
|-------|---------------|
| `mh_shift_config` | Shift times and pay multipliers (history preserved) |
| `mh_employee_rates` | Per-employee LKR/hour rates with date ranges |
| `mh_special_days` | Public holidays and special days calendar |
| `mh_time_entries` | Raw clock-in / clock-out records |
| `mh_daily_summary` | Computed hours and costs per employee per day |
| `mh_calculation_log` | Record of every calculation engine run |
| `mh_users` | Login accounts (admin / viewer roles) |
| `mh_audit_log` | Every user action with timestamp and IP |

The system reads `dt_employees` and `dt_attendance_records` from the DT Attendance module but never writes to them.

---

## Installation

**Requirements:** PHP 8.0+, MySQL 5.7+, a web server, the DT Attendance database already installed.

**Step 1 — Edit credentials** in `mh_system.php` lines 20–24:

```php
define('DB_HOST', 'localhost');
define('DB_NAME', 'contractor_attendance');
define('DB_USER', 'your_db_user');
define('DB_PASS', 'your_db_password');
define('DB_PORT', '3306');
```

**Step 2 — Run the installer** by opening `mh_setup.php` in your browser. It creates all tables and seeds:
- Default shift configuration (Day 07:00–17:00, Evening 17:00–00:00, Night 17:00–07:00)
- Default multipliers: Sunday ×2.00, Holiday ×2.00, Night ×1.50, Overtime ×1.25
- 19 Sri Lanka public holidays for 2026

**Step 3 — Log in.** The `mh_users` table is auto-created on first page load with one default account:

| Username | Password | Role |
|----------|----------|------|
| `admin` | `admin123` | Admin |

> ⚠️ Change the default password immediately via *My Profile → Change My Password*.

---

## Modules

### Login

Every page requires authentication. Visitors who are not logged in are redirected to the login screen automatically.

```
Visit any page
  → Not logged in → redirect to ?action=login
  → Enter credentials → session created
  → Redirect to dashboard
```

The login form has a show/hide password toggle. Sessions are destroyed cleanly on logout.

---

### Dashboard  `?action=dashboard`

Overview of the current month at a glance.

- **4 KPI cards** — total man-hours, total adjusted cost, open entries pending confirmation, employees without a rate set
- **14-day trend chart** — bar chart (hours) overlaid with a line chart (cost), dual Y-axes
- **Recent entries table** — last 8 time entries across all employees with status badges
- **System status card** — active shift configuration name, last calculation run date/time

---

### Shift Configuration  `?action=shift_config`  *(Admin)*

Define when each shift starts and ends, and what each condition pays.

| Shift / Condition | Default hours | Default multiplier |
|---|---|---|
| Day | 07:00 – 17:00 | ×1.00 (base rate) |
| Evening | 17:00 – 00:00 | ×1.00 (base rate) |
| Night | 17:00 – 07:00 next day | ×1.50 |
| Sunday | Any shift | ×2.00 |
| Public Holiday | Any shift | ×2.00 |
| Overtime | Beyond normal hours | ×1.25 |

Every time you save a new configuration it is inserted as a new record — the previous config stays in history. Entries always reference the config that was active when they were created.

**Example:** A night-shift worker works 10 hours on a Sunday. The engine picks the Sunday multiplier (×2.00) because Sunday outranks Night in the priority chain.

---

### Special Days  `?action=special_days`  *(Admin)*

Manage the holiday calendar year by year.

**Day types:** `public_holiday`, `special_holiday`, `half_day`, `sunday_override`, `normal_working`

You can override the multiplier for any specific date (e.g. give a company anniversary day ×3.00 instead of the default holiday rate).

**Pre-seeded 2026 holidays (19 dates):**

| Date | Holiday |
|------|---------|
| 01 Jan | New Year Day |
| 14 Jan | Tamil Thai Pongal Day |
| 04 Feb | National Day |
| 03 Mar | Maha Sivarathri |
| 02 Apr | Good Friday |
| 13 Apr | Day before Sinhala & Tamil New Year |
| 14 Apr | Sinhala & Tamil New Year |
| 01 May | Labour Day |
| 13 May | Vesak Full Moon Poya |
| 14 May | Day following Vesak |
| 11 Jun | Poson Full Moon Poya |
| 11 Jul | Esala Full Moon Poya |
| 10 Aug | Nikini Full Moon Poya |
| 08 Sep | Binara Full Moon Poya |
| 07 Oct | Vap Full Moon Poya |
| 20 Oct | Deepavali |
| 06 Nov | Il Full Moon Poya |
| 06 Dec | Unduvap Full Moon Poya |
| 25 Dec | Christmas Day |

---

### Hourly Rates  `?action=rates`  *(Admin to edit, Viewer to browse)*

Set and track the LKR/hour rate for every DT Worker and DT Staff member.

- Browse by category (Workers / Staff)
- Click **Set Rate** on any employee to open the rate panel
- Enter the new rate and effective date — the previous rate is closed automatically the day before
- Full history table per employee shows every rate ever set

**Example:**

```
Employee:  W. Bandara
Past rate: LKR 450.00 / hr   (2025-01-01 → 2026-01-31)
New rate:  LKR 500.00 / hr   (2026-02-01 → present)
```

If an employee has no rate set, the dashboard shows a warning and the calculation engine skips them.

---

### New Time Entry  `?action=time_entry`  *(Admin)*

Single-employee clock-in / clock-out form.

| Field | Notes |
|-------|-------|
| Date | Selecting a Sunday or holiday auto-shows a badge warning |
| Category | DT Workers or DT Staff |
| Employee | Dropdown shows the employee's current hourly rate |
| Shift Type | Day / Evening / Night / Overtime |
| Time In / Out | 24-hour time pickers |
| Break Minutes | Deducted from billable time before calculation |
| Crosses Midnight | Tick for Night shifts that span to the next calendar day |
| Night Shift Flag | Marks entry for the night premium multiplier |
| Remark | Free-text note (e.g. "Overtime approved by PM") |

Submitting creates a new entry with status `open`. Clicking Edit on an existing entry re-opens this form pre-filled.

---

### Batch Time Entry  `?action=batch_entry`  *(Admin)*

Enter the same shift for an entire team in one screen.

1. Choose **Date**, **Category**, and **Shift Type**, click Load
2. The full employee list appears — one row per person
3. Each row shows attendance status from `dt_attendance_records` (✓ Present / AB Absent), whether an entry already exists (Existing badge), and individual input fields
4. **Fill All from First Row** — copies the top row's times to all selected employees
5. **Tick All Present** — selects the full list in one click
6. Click Save All Selected

**Example — 38 workers, day shift:**

```
Date: 2026-05-15   Category: Workers   Shift: Day
→ Set first row: 07:00 in, 17:30 out, 60 min break
→ Click "Fill All from First Row"
→ Click "Tick All Present"
→ Save All Selected
→ 38 entries created in one action
```

---

### Browse Entries  `?action=time_browse`  *(Admin to act, Viewer to view)*

Filter and manage all time entries by date, month, category, or status.

**Entry status lifecycle:**

```
  open  ──────────►  confirmed  ──────►  locked
  ↑                      ↑                  ↑
Created                Admin ✓          Admin 🔒
(editable)         (ready for calc)  (payroll done,
                                      cannot edit)
```

Only `confirmed` entries with a Time Out are processed by the calculation engine.

---

### Run Calculation  `?action=calculate`  *(Admin)*

Reads all confirmed entries for a date range and writes computed costs to `mh_daily_summary`.

**Parameters:** Date From · Date To · Category (All / Workers / Staff)

**What the engine does per employee per day:**

1. Read all confirmed entries that have a Time Out
2. Subtract break minutes → billable minutes
3. Split billable minutes into day / evening / night buckets using the active shift config
4. Look up the employee's active hourly rate on that date
5. Determine the multiplier — **highest priority wins:**

```
  Day-specific override  (custom rate set in Special Days)
      ↓
  Public Holiday         (×2.00 default)
      ↓
  Sunday                 (×2.00 default)
      ↓
  Night shift flag       (×1.50 default)
      ↓
  Overtime threshold     (×1.25 default)
      ↓
  Base rate              (×1.00)
```

6. `base_cost = total_hours × hourly_rate`
7. `adjusted_cost = base_cost × multiplier`
8. Write to `mh_daily_summary` (UPSERT — safe to re-run)
9. Log the run in `mh_calculation_log`

---

### Daily Report  `?action=report_daily`  *(All users)*

All employees for one date, grouped Workers / Staff.

Columns: Employee · Day hrs · Evening hrs · Night hrs · Total hrs · Rate · Base cost · Multiplier · Adjusted cost · Notes

Category subtotals and grand total footer. Sunday and holiday rows are highlighted. Prev / Next day navigation. Print-ready (sidebar hidden).

---

### Monthly Report  `?action=report_monthly`  *(All users)*

One row per employee for the selected month, grouped by category.

Columns: Employee · Days worked · Day hrs · Evening hrs · Night hrs · Total hrs · Rate · Base cost · Adjusted cost · Sun days · Holiday days

Click any employee name to jump to their individual report for that month.

---

### By Employee Report  `?action=report_employee`  *(All users)*

Employee selector on the left. Choose any employee and month to see:

- 4 KPI cards: total hours, total cost, days worked, average hours/day
- Day-by-day breakdown with all cost components
- Sunday and holiday rows highlighted in amber
- Special day descriptions shown inline (e.g. "Vesak Full Moon Poya")

---

### Cost Summary  `?action=cost_summary`  *(All users, print-ready)*

Formal monthly report for management review.

- Month picker and category filter
- 6 KPI cards
- Daily bar + line chart (hours vs cost) — hidden on print
- Separate Workers and Staff cost tables with subtotals
- Grand total block
- Print renders a clean document with company header and footer

---

### Calculation Log  `?action=calc_log`  *(All users)*

Full history of every engine run with cumulative KPI cards and a detailed table: run time · period · category · rows processed/updated/inserted · hours · cost · status.

---

### Manage Users  `?action=manage_users`  *(Admin only)*

| | Admin | Viewer |
|---|---|---|
| View all reports | ✅ | ✅ |
| Browse entries | ✅ | ✅ read-only |
| Create / edit / delete entries | ✅ | ❌ |
| Run calculation | ✅ | ❌ |
| Edit rates, shifts, holidays | ✅ | ❌ |
| Manage users & audit log | ✅ | ❌ |
| Change own password | ✅ | ✅ |

Actions available: create user · change role · activate/deactivate · delete · change any user's password (🔑 button). Cannot delete your own account.

Viewer accounts see a blue notice bar on every page. All write actions are blocked server-side — not just hidden in the UI.

---

### Audit Log  `?action=audit_log`  *(Admin only)*

Every action stored in `mh_audit_log`:

| Action | When |
|--------|------|
| `login` / `logout` | Authentication |
| `time_entry_create` / `time_entry_update` | Entry saved or edited |
| `entry_confirm` / `entry_lock` / `entry_delete` | Status changes |
| `batch_entry_save` | Batch save |
| `rate_save` | Hourly rate set |
| `shift_config_save` | New shift config saved |
| `special_day_save` | Holiday added |
| `calculate` | Engine run |
| `user_create` / `user_update` / `user_delete` | User management |
| `password_change` | Any password change |

Each record: timestamp · username · action type · target table · record ID · description · IP address · session ID.

Filter by user, action type, or date. A 30-day activity summary shows as colour-coded pill badges at the top.

---

### My Profile  `?action=my_profile`  *(All users)*

View role badge, last login, login count, and member-since date. Change own password (current password required).

---

## Typical Daily Workflow

```
Morning
  1. Batch Entry → date = today, Workers, Day shift
     → Fill All from First Row, Tick All Present, Save

  2. Browse Entries → filter by today
     → Confirm each entry ✓

End of day
  3. Run Calculation → date range = today, All
     → Run

  4. Daily Report → navigate to today
     → Review totals, print if needed
```

---

## Notes

- CDN assets require an internet connection. To run offline, download Bootstrap, Font Awesome, and Chart.js and update the tags in `mh_system.php`.
- Re-running `mh_setup.php` is always safe — every statement uses `IF NOT EXISTS`.
- The `mh_daily_summary` table uses `ON DUPLICATE KEY UPDATE`, so re-running the calculation on the same range updates rows in place rather than duplicating them.
- Passwords are stored using PHP `password_hash()` (bcrypt). All queries use PDO prepared statements — no raw SQL concatenation anywhere.

---

*Dynamic Technologies (Pvt) Ltd · Pallekelle Site · Sri Lanka*  
*Stack: PHP · MySQL · Bootstrap 5 · Chart.js · Font Awesome*
