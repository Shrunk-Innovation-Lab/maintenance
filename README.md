# Elizabeth Maintenance – Kanban board

A sign-in-protected kanban board for managing the maintenance requests in the **Tasks** worksheet of the
[Elizabeth Maintenance](https://docs.google.com/spreadsheets/d/1qYSSeJlbhCf9iI3rO-FCkHUKRjzaVBZHur8UgMq3Dc8/edit)
spreadsheet. It runs as a Google Apps Script web app, so there is no server to host and nothing to pay for.

| File | What it is |
| --- | --- |
| `Code.gs` | Backend: sign-in, the sheets, emails, search, dashboard figures, background jobs |
| `Index.html` | The board (served by the script) |
| `appsscript.json` | Project manifest (optional; also needed if you deploy with `clasp`) |
| `web/` | The other files for GitHub Pages: ribbon icons, `manifest.webmanifest` (home-screen app) and `sw.js` (opens offline) |

## What people can do

- **Board:** drag requests between **New → In Progress → On Hold → Done**. The card lifts and wiggles while you carry it,
  then glides into its new column, settles with a small wiggle and glows. Columns always fit the screen (nothing scrolls
  sideways), and each column lists the newest request first, with its date and time. Columns show 40 cards at a time, with
  a **Show more** button. Search and filters still cover every card.
  **On desktop, a New request can't leave New until someone is assigned.** While you drag it, the other columns show as
  blocked and the card says "Assign first". If you drop it there, it flies back. The status dropdown follows the same rule.
- **Finishing a job (Done):** the person the job is assigned to must add an **after photo** or a **final comment**. Moving
  their job to Done (swipe, drag, button or status) opens a **Finish job** tray: add one or more photos (phones open the camera
  or library), write what was done, or both, then **Mark as Done**. Nobody else can move a job to Done until its assignee has
  added a photo or comment since the job last moved; until then the card shows "Greg needs to add an after photo or a final
  comment first". Unassigned jobs can't be Done. The server enforces this too. Bulk Done skips jobs that aren't ready, and
  **Close old requests…** is the only thing that closes jobs without it.
- **Phones:** the header shows the Elizabeth City Centre logo and the app name. One status at a time, with tabs to switch
  between them. Cards are moved by swiping:

  | Card | Swipe right | Swipe left |
  | --- | --- | --- |
  | Unassigned (New, In Progress or On Hold) | **Assign to me** (stays put) | **Assign…**: a tray slides up to pick someone else |
  | New, assigned to you | **Start**: to In Progress | **Hold**: to On Hold |
  | In Progress, assigned to you | **Done**: opens the Finish job tray | **Hold**: to On Hold |
  | On Hold, assigned to you | **Resume**: to In Progress | **Done**: opens the Finish job tray |
  | Assigned to someone else | Won't move: "Only Greg can move this" | Same |

  A coloured strip shows what the swipe will do, and a tip explains it until the first swipe. Done cards have a **Reopen**
  button, shown only to the person the job is assigned to. On phones the status dropdown in the
  request window follows the same rule. Text boxes use 16px text, so iPhones don't zoom in when you tap one. Phones show a
  one-off tip on adding the board to the home screen.
- **Action bar (phones):** the open request window has its main buttons pinned to the bottom, so nobody scrolls for them:
  **Start / Hold / Done / Comment** for your own job (following the same rules as swiping), or **Assign to me / Assign… /
  Comment** when it's unassigned.
- **Offline:** if the signal drops, a bar says "You're offline. Changes will save when you're back." Moves and assignments made
  meanwhile are kept on the device and saved as soon as the connection returns ("Back online. Your changes are saved.").
  Don't close the page while offline, or those changes are lost.
- **First sign-in tips:** three quick tips the first time someone signs in: swiping (dragging on desktop), **My jobs** and the
  watch eye. **Show tips again** in the menu replays them.
- **Assigning from the card:** each card shows who has the job (initials and name, or "You"). Unassigned cards show a **+ person** button
  (on phones only on Done cards, since swiping assigns the others). Tap it (phone or
  desktop) for a short list, with you first, to hand the job to anyone without opening it. On assigned cards, tap the
  initials to reassign or unassign.
- **Undo:** every move, swipe, assignment and bulk change shows a toast with **Undo** for 5 seconds. The change is only saved
  after that, so undoing really cancels it and nobody is emailed. If the page is closed or the phone locked during those
  5 seconds, the change is saved straight away.
- **Live pop-ups:** the board checks for changes every 30 seconds and pops up a blue message with **View** when someone else
  assigns you a job, moves your job, comments on a job you're on or watching, or adds or flags an Urgent request.
- **@mentions:** type **@** in a comment to pick a person. They're emailed straight away (even on the daily summary) and start
  watching the request. Mentions are highlighted in the comment.
- **Watching:** the eye button at the top of each request window. Highlighted means you're emailed about that request; tap to
  stop or start.
- **Select several cards (desktop):** click **Select** (or Ctrl/⌘-click cards), pick cards, then use the bar to assign, set
  priority or move them all at once. Unassigned New cards are skipped when moving, and so are jobs not ready for Done. Each affected person gets one summary email.
- **Messages:** green with a tick when something worked, blue with an info icon for information (e.g. "Unassigned"), red
  with a warning icon when something couldn't be done and why. All icons are flat line icons (no emoji), so they look the
  same on every phone and computer.
- **Header:** search the board, a **Filters** panel (type, assignee, priority, overdue only), **Select** (desktop), a **My jobs**
  toggle, and a menu with Dashboard, Search all requests, Mark all as read, Email settings, Show tips again, **Users** (admins
  only) and Sign out. New requests come in
  through the form only; the board is for managing them.
- **Priority:** Urgent / High / Normal, set on the request. Urgent requests are always emailed straight away.
- **Overdue:** a card is flagged once it sits without an update for longer than its status allows (New 2 days, In Progress 7, On Hold 14).
- **Signed out after 30 minutes of inactivity:** clicks, taps, typing and scrolling count; the board's own refreshes don't.
  A message warns 2 minutes before, with **Stay signed in**. Anything waiting to save is saved first. The server applies the
  same limit (`SESSION_SECONDS` in `Code.gs`), so a phone left asleep also needs signing in again.
- **Opens on My jobs:** everyone starts on their own jobs at sign-in (tap **My jobs** to see the whole board).
  People with nothing assigned see the whole board.
- **Unread:** a blue dot marks cards someone else changed since you last opened them. This is per device; "Mark all as read" clears them.
  The **My jobs** count gets a red dot when one of your jobs has changed.
- **Job details (optional):** contractor, PO number and cost, folded away under "Contractor, PO & cost" on each request.
- **Comments and files:** comment threads; attach photos, PDF, Word, Excel, PowerPoint, text and CSV files (up to 10 MB each).
  Photos are shrunk to 2000 px before uploading. iPhone HEIC photos are converted to JPEG on the phone.
- **Photos:** thumbnails on cards, a full-screen viewer (swipe or arrow keys), and download for documents.
- **Someone else got there first:** if a card changed after you loaded it, your move, assignment or edit is refused with a note
  saying who changed what. The card then shows their update, so nobody's work is overwritten.
- **Search all requests:** finds any request, including finished ones that have left the board, by words, date range and status.
- **Dashboard:** open, overdue, urgent and unassigned counts; jobs completed, average days to complete and the cost of completed jobs
  (last 90 days); raised vs completed per week; open requests by status and type; the most-reported locations.

## What it changes in the spreadsheet

The **Tasks** sheet is never modified. The board keeps its data in its own tabs:

| Tab | Holds |
| --- | --- |
| **Board Status** | One row per request: status, assignee, watchers, last update, priority, contractor, PO number, cost. Joined to Tasks by the form's submission ID (column L). Requests created in the board by earlier versions also keep their details here. |
| **Board Users** | Who can sign in, their role, what they're emailed about, and access requests waiting for approval |
| **Board Activity** | Every comment, move, assignment, detail change and attachment |
| **Board Attachments** | One row per attached file. The files themselves are in a private Drive folder, **Elizabeth Maintenance Attachments**. |
| **Board Thumbnails** | Small previews of photos, for cards |

A Tasks row with no ID in column L gets an ID beginning `H-`, made from its date, name, type, location and description.
If you later edit those cells in Tasks, that row will appear as a new request.

**Times are Adelaide time** (`TIME_ZONE` in `Code.gs`): the form's column A is read as Adelaide time, the board records its
own times in Adelaide time, and every device shows Adelaide time. Column A is read as a real date where the cell holds one,
so the sheet's date format doesn't matter.

**Too many open requests?** **Maintenance Board → Close old requests…** closes every open request with no update for a number
of days you choose. It shows how many first. They move to Done, are logged in Board Activity, stay in Search all requests,
and don't fill the Done column or skew the dashboard.

**Health check:** in the spreadsheet, **Maintenance Board → Health check: newest requests** shows how many requests the board
sees, how many are in each status, and the 10 newest with the date it read. Use it if a request seems to be missing.

> **Existing history.** The first time setup runs, rows older than **60 days** are marked `Done` and newer ones `New`,
> so the board doesn't open with ~7,900 cards. No emails are sent for these. Change `SETUP_OPEN_DAYS` **before** the first setup
> to use a different cut-off.

## Setup

1. Open the spreadsheet → **Extensions → Apps Script**.
2. Replace the contents of `Code.gs` with this folder's `Code.gs`.
3. Click **+ → HTML**, name it `Index` (exactly), and paste in `Index.html`.
4. Choose **setup** in the function dropdown and click **Run**. Approve the permissions
   (spreadsheet, email, Google Drive, external requests, triggers). Setup:
   - creates the board's tabs, with you as the first user and an **Admin**, and adds any new columns to tabs from an earlier
     version
   - gives every existing request a status (see above)
   - installs two triggers: every 5 minutes (new form submissions and photo thumbnails) and every morning at 7am (daily summaries)
5. **Deploy → New deployment → Web app**, with *Execute as*: **Me** and *Who has access*: **Anyone**.
   The board does its own sign-in, so people don't need Google accounts. The **/exec** link is the board.
6. Put the page on GitHub Pages (below). Email buttons link there (`APP_URL` in `Code.gs`), and the /exec link keeps working
   for anyone who already uses it.

### GitHub Pages (where people open the board)

The page people use is on GitHub Pages: https://shrunk-innovation-lab.github.io/maintenance/ (repository
[Shrunk-Innovation-Lab/maintenance](https://github.com/Shrunk-Innovation-Lab/maintenance)). Apps Script still holds everything
else: sign-in, the sheets, emails and the background jobs. The page sends its requests to the /exec link (`doPost` in
`Code.gs`), which only answers the board's own actions, and each action still checks the sign-in.

Why: phones can add it to the home screen as a proper full-screen app with the ribbon icon (Apps Script pages can't give
iPhones an icon), there's no Google banner, the page opens instantly, and it still opens with no signal.

The repository holds six files at its top level:

| File | From this folder |
| --- | --- |
| `index.html` | `Index.html`, the same file as in Apps Script (with `API_URL` set) |
| `sw.js`, `manifest.webmanifest`, `apple-touch-icon.png`, `icon-192.png`, `icon-512.png` | `web/` |

1. In `Index.html`, set `API_URL` (near `ON_APPS_SCRIPT`) to the /exec link from **Deploy → Manage deployments**. Use the same
   file in Apps Script and on GitHub.
2. Upload the six files to the repository's top level.
3. **Settings → Pages**: Source **Deploy from a branch**, branch **main**, folder **/ (root)**, Save. After a minute or two the
   board is at the link above.
4. Deploy a new version in Apps Script (it adds the `doPost` entry point the page talks to).
5. Send staff the GitHub link to open on their phones and add to the home screen.

The repository is public (GitHub Pages is free for public repositories). That's fine: it holds only the page, not data,
passwords or emails. Anyone can read the page's code and see the /exec link, but they still need an approved sign-in.

### Updating to a new version

Paste in the new `Code.gs` and `Index.html`, run **setup** once (it's safe to re-run: it adds new columns, tabs and triggers
and approves new permissions), then **Deploy → Manage deployments → ✏️ → Version: New version → Deploy**. The link doesn't change.
If `Index.html` changed, also upload it to the GitHub repository as `index.html`. Phones pick up the new page the next time
they open it with signal. If `sw.js` changes, raise its `CACHE` name (e.g. `board-v2`).

Each setup run also puts back in **New** any request from the last 60 days that the first setup closed as Done without
anyone touching it. Cards a person has moved are never changed.

> If you ran the very first version (which added columns to Tasks), delete the `Status`, `Assigned To`, `Watchers`,
> `Last Updated` and `Updated By` columns from Tasks. In **Script properties**, delete `SETUP_DONE`, then run setup.

## Updates for whoever reported the problem

The reporter gets a short email when their request is **started**, **put on hold** and **done**. There's no board link in
it, since they can't sign in. To switch this on for form submissions, **add an email question to your Paperform** headed
*Email* (or *Email address*, *Your email* or *Contact email*). The board finds that column by its heading; nothing else
needs changing. The wording of these emails is in
`REQUESTER_UPDATES` in `Code.gs`.

## People and emails

### Getting access (approved by an admin)

Nobody gets in until an Admin approves them. Admins are the emails listed in `ADMIN_EMAILS` in `Code.gs`, plus anyone with
**Admin** in the Role column of **Board Users**.

1. A new person opens the board link and taps **Request access**, giving their name, email and a short reason.
   They're added to **Board Users** with Active = `Pending`.
2. Admins get an email with **Approve** and **Decline** buttons. Both open the board, where the admin confirms while signed
   in. The admin's menu also shows **Users · 1 access request**, and a pop-up appears when a new request arrives.
3. **Approve:** the person is emailed a **Set my password** link. It opens the board with their email filled in; they get a
   6-digit code and choose a password. **Decline:** they get a polite email and stay out.

A pending person who tries to sign in is told their request is waiting for approval. Requests are limited to 20 an hour.

### Managing people (admins only)

**Menu → Users** lists access requests first, then everyone. Admins can approve or decline requests, deactivate or reactivate
people, make someone an Admin, and change their emails (Instant / Daily summary, and which new requests they hear about),
all without opening the sheet. Admins can't deactivate themselves or remove their own Admin role.

You can still add someone by typing a row into **Board Users** and sending them the link. They then use **First time, or forgot
your password?** to set a password. Only a salted hash is stored. Set **Active** to `N` to remove someone.

| Column | Meaning |
| --- | --- |
| Notify New | `ALL`, blank, or maintenance types separated by commas (e.g. `Plumbing, Aircon`) |
| Notify All Updates | `Y` to hear about every change on every request |
| Emails (Instant / Daily) | Each person can change this under **Email settings** in the board |
| Role (Admin / Staff) | Admins see **Users** and approve access. Anyone in `ADMIN_EMAILS` is always an admin. |
| Access Request | The reason a person gave when asking for access, and when they asked |

| Event | Who is emailed |
| --- | --- |
| New request | Users whose **Notify New** matches its type |
| Access request | Active admins |
| Moved, assigned, commented on, file attached, marked Urgent | The assignee, everyone watching the request, and **Notify All Updates** users |

People on **Daily** get one summary each morning: their open jobs, new requests for their types, and the last day's updates on
requests they follow. **Urgent** requests always email straight away. Whoever made a change isn't emailed about it.
Every email carries the Elizabeth City Centre logo at the top, sent as an inline image (taken from `Index.html`) so it
shows in Gmail and Outlook, and the copyright line at the bottom.
Gmail allows about 100 recipients a day on free accounts and 1,500 on Google Workspace. If that runs out, the change is still saved.

## Settings (top of `Code.gs`)

| Setting | Default | |
| --- | --- | --- |
| `STATUSES` | New, In Progress, On Hold, Done | Board columns (colours in `STATUS_COLORS`, phone buttons in `QUICK`, both in `Index.html`) |
| `PRIORITIES` | Urgent, High, Normal | Highest first |
| `OVERDUE_DAYS` | New 2, In Progress 7, On Hold 14 | Days without an update before a card is flagged |
| `DONE_VISIBLE_DAYS` | 30 | How long Done cards stay on the board |
| `APP_URL` | The GitHub Pages link | Where email buttons send people (Script property `APP_URL` overrides) |
| `ICON_URL` | The ribbon on GitHub Pages | Browser-tab icon for the /exec link |
| `ADMIN_EMAILS` | The Admin's email | Always admins, whatever the Role column says |
| `ACCESS_REQUESTS_PER_HOUR` | 20 | Limit on new access requests |
| `SESSION_SECONDS` | 30 minutes | Sign-out after inactivity |
| `DIGEST_HOUR` | 7 | Daily summary time (the spreadsheet's time zone). Re-run setup after changing it. |
| `STATS_DAYS` | 90 | Dashboard look-back |
| `MAX_UPLOAD_BYTES` | 10 MB | Largest attachment |
| `THUMBS_PER_RUN` | 8 | Photo thumbnails made every 5 minutes |
| `SETUP_OPEN_DAYS` | 60 | First setup only |
| `COL` | A, B, C, D, E, G, H, L | Which Tasks columns the board reads |

## How some of it works

- **Speed:** the card list is built once and kept in the script cache for everyone. It's rebuilt when the board changes
  something or the Tasks sheet grows, so typical loads skip reading the sheets. The 30-second refresh sends only "nothing
  changed" unless something did. Edits typed straight into the Board tabs show up within 10 minutes.
  The browser draws 40 cards per column and loads thumbnails only for cards on screen.
- **Thumbnails:** Apps Script can't resize images, so the 5-minute job copies new form photos into a working folder in Drive.
  Drive makes a thumbnail, and the next run saves it as a small JPEG and bins the copy. Thumbnails for existing open requests
  build up over the first few hours. Attached photos get their thumbnail straight away from the uploader's browser.
- **Several people at once:** every change is saved one at a time behind a lock, and each change carries the card's version, so
  edits never overwrite each other unnoticed. Boards refresh every 30 seconds.
- **Security:** every server function checks the sign-in first. Five wrong passwords lock an email out for 15 minutes.
  Photos and files can only be fetched if they belong to a request. Only admins can see or change users through the board,
  and new people need an admin's approval. But anyone with **edit access to the spreadsheet** can add users or make themselves
  Admin in **Board Users**, so keep edit access to the Admin.
- **Home screen:** added from the GitHub Pages link, the board opens full-screen with the ribbon icon on iPhone and Android.
  Added from the /exec link instead, the icon and full-screen depend on the phone, because that page is Google's.
