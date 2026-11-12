# School Library Manager

A browser-based school library tracker built in a single `index.html` file with HTML, CSS, and JavaScript. It manages books, students, and lending transactions, saves data in browser localStorage, and supports manual Supabase cloud sync.

## Features

- **Dashboard:** total, borrowed, and available book counts, recent activity, student search, and quick lending actions.
- **Catalog:** add, edit, and archive books; store author, ISBN, and tags; search, filter by status, and sort records.
- **Students:** import admission IDs, names, and classes from CSV.
- **Lending:** record borrow and return transactions using autocomplete, manual lookup, or camera scanning. Returns must match the current borrower.
- **History:** archive transactions with a reason and view their revision history. Books also store revision records.
- **Reports:** book status chart, monthly transaction counts, and a searchable catalog report.
- **Settings:** configure Supabase, push/pull records, import/export CSV, add archive reasons, and clear local library records.
- **Sign-in:** Google sign-in through Firebase Authentication.

## Run locally

No package installation or build step is required. From the repository directory, serve the files with a static server, for example:

```sh
python3 -m http.server 8000
```

Open [localhost:8000](http://localhost:8000). The app loads its libraries from CDNs, so an internet connection is required to load those dependencies and use Google sign-in. Camera scanning requires browser permission and a secure context such as HTTPS or localhost; manual entry is also available.

For deployment, serve the repository as a static site over HTTPS and configure the hosting domain in Firebase Authentication.

## Firebase configuration

`index.html` contains a `firebaseConfig` object and initializes Firebase's Google auth provider. To use your own Firebase project:

1. Replace `firebaseConfig` with your project's web app configuration.
2. Enable Google as a sign-in provider in Firebase Authentication.
3. Configure authorized domains for local development and deployment.
4. Open the app and select **Sign in with Google**.

The app shows its main interface to any user accepted by Firebase sign-in. Although the login screen says access is for authorized school staff, the code contains no staff email/domain allowlist or role check.

## Optional Supabase sync

See [SUPABASE_SCHEMA.md](SUPABASE_SCHEMA.md) for the inferred table schema, setup SQL, field descriptions, relationships, and access-control requirements.

1. Create the `books`, `students`, and `transactions` tables using that document.
2. Implement the identity integration and database policies required for your intended users. The supplied SQL enables row level security without granting client access.
3. In **Settings → Supabase Connection**, enter the project URL and public anon key, then select **Connect**.
4. Use **Push to Supabase** to upsert local records or **Pull from Supabase** to replace local library arrays with remote records.

Firebase sign-in is separate from Supabase authentication in the current code; no Firebase token is sent to Supabase. Never enter a service-role key in the browser app.

Sync is manual. Saved connection details are used to reconnect after sign-in, but reconnecting does not automatically push or pull. Push sends books, then students, then transactions as individual upserts keyed by `id`; it can partially complete on failure. Pull has no pagination or explicit ordering. The Online/Offline badge reports Supabase connection state.

## CSV formats

### Students

The dashboard's **Import CSV** accepts an admission/ID column, a name column, and an optional class column:

```csv
admission,name,class
00123,Asha Rao,6A
00124,Ravi Shah,7B
```

Existing students with the same ID are updated. The student step in **Import All CSV** instead requires headers named `id` and `name`, with optional `class`.

### Books

Book import requires `title`. Supported optional fields are `id`, `author`, `isbn`, `status`, `tags`, and `borrowedBy`. Missing IDs are generated; matching IDs update existing books. Use `available`, `borrowed`, or `archived` for status and semicolons between tags:

```csv
id,title,author,isbn,status,tags,borrowedBy
bk_001,Example Book,Example Author,9780000000000,available,fiction;class6,
```

In the book editor, enter tags with leading hashes, such as `#fiction #class6`. CSV tag values omit the hashes.

### Export and import limitations

- **Export All CSV** downloads separate books, students, and transactions files.
- **Import All CSV** imports books and students only; transaction CSV import is not implemented.
- CSV parsing uses simple comma and newline splitting, and export does not escape values. Avoid commas and line breaks inside field values; quoted CSV fields are not supported.
- CSV exports omit revision arrays and some timestamps. Book import generates new timestamps and an import revision, so CSV is not a complete history backup.

## Storage and behavior notes

- Books, students, transactions, archive reasons, and Supabase connection settings persist in localStorage for the current browser origin. They are not separated by signed-in account, and signing out does not erase them.
- **Clear Local Data** empties local books, students, and transactions after confirmation. It retains archive reasons and connection settings and does not delete Supabase rows.
- Archiving a transaction can automatically make its book available when the current borrower matches that transaction's student.
- Revision history is written by the client; it is not a tamper-proof database audit log.
- There is no due-date, overdue, fine, or reservation workflow in the current app.

## Dependencies and files

All runtime dependencies are loaded directly in `index.html`:

| Library | Version in app | Purpose |
| --- | --- | --- |
| Firebase compat SDK | 10.7.1 | Google authentication |
| Supabase JS | Major version 2 | Database sync |
| html5-qrcode | 2.3.8 | Camera scanning |
| Chart.js | 4.4.0 | Reports |

- [index.html](index.html): application UI, styles, and logic.
- [SUPABASE_SCHEMA.md](SUPABASE_SCHEMA.md): database setup and sync documentation.
- [LICENSE](LICENSE): GNU Affero General Public License, version 3.

The repository has no automated test suite or build configuration. To check changes manually, sign in, import students, add a book, borrow and return it, inspect reports, and reload to confirm local persistence. Test cloud sync separately against a configured Supabase project.
