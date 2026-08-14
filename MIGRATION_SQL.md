# OrbitBills — Local SQLite Migration

## What changed

- **IndexedDB removed** as the primary store.
- All business data now lives in **SQLite** (`techserenia_pos.db`).
- Public API is **unchanged**: `tsLocalApi()`, `tsLogin()`, `tsWhoami()`, etc.
- HTML panels (`billing.html`, `admin-dashboard.html`, …) need **no logic rewrites**.

## How storage works

| Environment | Engine |
|-------------|--------|
| Capacitor Android / iOS | `@capacitor-community/sqlite` (native file) |
| Browser / static site | `sql.js` (WASM SQLite in memory + `localStorage` persistence) |

Each former IndexedDB object store is a SQLite table:

```sql
CREATE TABLE <store> (
  id TEXT PRIMARY KEY,   -- string form of numeric id, or email/key
  json TEXT NOT NULL     -- full document JSON
);
```

`settings` uses `id` = setting key.  
`users` uses `id` = email (lowercase).

## Install (APK)

```bash
npm install
npx cap sync android
cd android && ./gradlew assembleDebug
```

## One-time data migration from IndexedDB

On first run after update, `Db.js` tries to:

1. Open SQLite
2. If SQLite is empty **and** IndexedDB `techserenia_pos` has data → copy all stores into SQLite
3. Mark migration done in settings (`sql_migrated_from_idb = 1`)

Users keep products, invoices, stock, etc.

## Backup / restore

Same as before:

- Admin → Backup → ZIP/JSON export
- Restore accepts previous OrbitBills backup format

Additionally you can copy the native DB file off-device if needed (advanced).

## Files in this package

| File | Role |
|------|------|
| `Db.js` | SQLite-backed data layer (drop-in replacement) |
| `www/Db.js` | Same file for Capacitor `webDir` |
| `package.json` | Adds SQLite + sql.js deps |
| `SQL_README.md` | This guide |
| `capacitor.config.ts` | Unchanged behaviour; still reads `app-config.json` |

Replace existing `Db.js` and `www/Db.js`, merge `package.json` dependencies, then `npm install` + rebuild APK.

## Roll-back

Keep a copy of the old IndexedDB `Db.js`.  
If needed, swap files back and rebuild — IndexedDB data is not deleted by this migration.
