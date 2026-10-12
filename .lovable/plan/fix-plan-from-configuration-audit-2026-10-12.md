# Fix plan from configuration audit

The overall setup is sound: every save checks sign-in and admin rights and validates its input, the calendar is the only source for dates, and archives load in pages. Three issues are worth fixing, in this order.

## 1. One owner for positions, departments and schedule types (high)
These lists can currently be saved from both System Configuration and People Configuration. Whichever screen saves last overwrites the other.
- Remove those fields from the System Configuration tenant-policy form and its save function. People Configuration becomes the only place to edit them.
- Update the stale helper comment that still points to System Configuration.
- Check: People save still works; System save no longer touches these lists; the dropdowns in Staff and User Management still load.

## 2. Stop silent overwrites on Academic saves (high)
Academic calendar and academic rules are saved by reading the whole settings record, changing one part, and writing it all back. Two admins saving around the same time can erase each other's work. People rules already guard against this with a version number.
- Add a version number to the calendar and the academic rules (older saved data defaults to 0).
- Before saving, reject the save with a "settings changed, reload" message if the version changed. Increment it on every save. Same approach as People rules.
- Write only the part of the settings being changed, so a calendar save cannot erase a rules save.
- Check: new tests for a stale-version rejection and for both saves keeping each other's data.

## 3. Smaller cleanups (medium/low)
- Move the default brand colours into one shared constant used by both the fallback logic and the form.
- Report which part failed when the configuration screen can't load, instead of a single combined error.
- Add tests for calendar lifecycle and coverage rules, finance-config rules and archive state logic.

## Not changing
- The two permission levels (Super Admin for configuration, admin for daily finance work) are deliberate.
- No dead code or leftover copies were found.

## Technical details
- Files: `src/lib/config.functions.ts` (saveTenantSettings, saveAcademicCalendar, saveAcademicSettings, getConfiguration), `SystemConfiguration.tsx`, `useTenantOptions.ts`, `academic-calendar.ts`, `academic-rules.ts`, `config.ts`, new tests under `src/test/`.
- Versions are stored inside the existing settings JSON, so no database change is needed.
