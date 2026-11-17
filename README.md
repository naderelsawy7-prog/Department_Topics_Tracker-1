# EPROM Team Dashboard — GitHub Pages package

This folder is the static GitHub Pages frontend. It is intentionally configured as an **online-only team client**:

- `index.html` contains the dashboard UI and Excel import/export code.
- `online-config.js` is the only frontend configuration file that needs editing.
- `index.html` contains no project, task, or member records; it loads them from the protected Apps Script API.
- `.nojekyll` keeps GitHub Pages from applying Jekyll processing.

The Apps Script backend is in `../apps-script/`. GitHub Pages alone cannot securely store or share dashboard edits.

## Quick setup

1. Finish the Apps Script setup in `../apps-script/README.md` and copy the deployed web app URL ending in `/exec`.
2. Edit `online-config.js`:

   ```js
   window.EPROM_ONLINE_CONFIG = {
     apiUrl: 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec',
     fileId: '1w-Kvknl-WBwW2hN-f4X-qBiGjLHW2FVN',
     fileName: 'Department_Topics_Tracker.xlsx',
     autoLoad: true,
     autoSave: true,
     requireOnline: true,
     transport: 'jsonp-form'
   };
   ```

   The API URL and Drive file ID are identifiers, not secrets. Do not put service-account keys, passwords, or private tokens in this file.

3. Upload the contents of this folder to a GitHub repository. Enable **Settings → Pages** and publish from the branch/folder containing `index.html`.
4. Open the Pages URL while signed in to an allowlisted Google Workspace account. The app displays a team-access screen until the shared state loads successfully.

## Team-only access

The GitHub Pages shell is not a security boundary. The Apps Script API is the security boundary:

- Keep the Drive workbook private or inside the team Shared drive.
- Deploy the Apps Script web app for the organization/team, not as a public anonymous service.
- Set `ALLOWED_DOMAIN` and/or `ALLOWED_EMAILS` in `../apps-script/Config.gs`.
- Do not use `Anyone, even anonymous` for a team dashboard unless your organization has an additional identity gateway; anonymous Apps Script requests do not reliably expose the caller email.

Standard GitHub Pages may still expose the empty dashboard shell to someone who finds the URL. It does not contain the workbook data. If the page itself must be invisible to non-team users, use GitHub Enterprise private Pages or place the Pages site behind the organization's identity-aware proxy.

## Online workflow

- The first load reads the full shared dashboard state from the hidden `Dashboard State` sheet in the Drive workbook's Google Sheets mirror.
- Edits are saved automatically when `autoSave` is `true` and can also be forced with **Save online**.
- **Projects in motion** and **All tasks** are linked: project owner/status/progress/target-date changes propagate to linked task rows, and task edits recalculate the linked project card. Moving a task refreshes both project summaries.
- Each save includes the revision that was loaded. If another team member saved first, the API rejects the stale save instead of silently overwriting it. Load the latest state, review it, and save again.
- The backend rewrites the visible `Topics` sheet and exports the mirror back to the original `.xlsx` file, preserving the existing Drive workbook file ID.
- **Import Excel** remains available. Import a local `.xlsx` or `.csv`; after the import, the dashboard's online auto-save writes the imported state to the shared workbook.
- **Export Excel** remains available and downloads a current `.xlsx` copy. JSON backup, JSON import, CSV export, and portable HTML export remain available.

The old **Connect Excel** control is a local Chromium File System Access workflow. It is retained for offline use but is not the multi-user online sync mechanism.
