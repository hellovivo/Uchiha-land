# Uchiha Land — Futuristic Anime Vault

Multi-anime video library with user IDs, streaming, original-file downloads, a hidden admin menu, a hard limit of two admins, and editable interface settings. Episodes are grouped by anime name.

## Run locally
1. Install Node.js 20+.
2. Copy `.env.example` to `.env` and change both secrets.
3. Run `npm install`, then `npm start`.
4. Open `http://localhost:3000`.

## Admin
Open the top-left menu → Admin access. Enter the private `ADMIN_SETUP_CODE`, then create an admin ID and a 12+ character password. Repeat for the second admin. The server blocks creation after two admin accounts.

## Hosting notes
This is a Node.js app, not a static GitHub Pages site. Deploy to a Node.js host with persistent disk and set environment variables in the host dashboard. Back up `data/` and `uploads/`. The app preserves original upload quality; it cannot magically create true 32K video. Upload only content you have permission to distribute.
