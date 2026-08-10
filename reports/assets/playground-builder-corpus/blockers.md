# Browser and DevTools failure log

## 2026-08-10 — plugin installer nonce rejection

### Reproduction

1. In a persistent WordPress Playground site, open **Plugins → Add Plugin**.
2. Search **Kadence Blocks — Page Builder Toolkit for Gutenberg Editor**.
3. Click **Install Kadence Blocks — Page Builder Toolkit for Gutenberg Editor 3.7.9 now**.

### UI result

`Installation failed: -1`

### Chrome DevTools network record

```
POST https://playground.wordpress.net/scope:confident-classic-river/wp-admin/admin-ajax.php
Status: 403
Request body:
slug=kadence-blocks&action=install-plugin&_ajax_nonce=82ac5a7e05&_fs_nonce=&username=&password=&connection_type=&public_key=&private_key=
Response body:
-1
```

Chrome console also reported: `Failed to load resource: the server responded with a status of 403 ()`.

### Resolution used

The inner WordPress frame had retained an old `window._wpUpdatesSettings.ajax_nonce` value. Reloading the inner WordPress plugin screen changed it from `82ac5a7e05` to `5ed4b68c1b`. Repeating the same browser UI flow then completed the install and exposed the **Activate Kadence Blocks** button. The plugin was activated through that button.

## DevTools file-upload restriction

### Reproduction

1. Open **Plugins → Add Plugin → Upload Plugin**.
2. Call Chrome DevTools `upload_file` for the zip selected through the browser download UI.

### Tool result

```
Error: Access denied: path /Users/cloudnik/Downloads/kadence-blocks.3.7.9.zip (canonical: /Users/cloudnik/Downloads/kadence-blocks.3.7.9.zip) is not within any of the configured workspace roots.
```

The same result occurred after copying the zip to both writable workspace roots. This prevented using the normal Upload Plugin UI as a fallback.

## Brizy activation

### Reproduction

1. In a fresh Playground site, search **Brizy – Page Builder v2.8.21** in Add Plugins.
2. Install it and click **Activate Brizy – Page Builder**.

### UI result

The browser moved to `/wp-admin/admin.php?page=getting-started` and displayed the standard WordPress critical-error screen: `There has been a critical error on this website.` The administrative screen and PHP bootstrap remained unavailable for the site afterwards.

### Log follow-up

After the later desktop restart, the Playground MCP returned this exact connection state when asked to enumerate the site for its logs:

```
{"connectedTabs":0,"sites":[],"message":"No browser connected. Open the Playground website at https://playground.wordpress.net/?mcp-port=56390 to connect."}
```

Consequently, no server-side Brizy stack trace or SQL error could be recovered from that stopped Playground session. The visible browser result above is retained rather than guessing at a database cause. The replacement Beaver Builder run was created in Docker; its WordPress and MySQL logs contain no plugin-activation errors.

## Spectra attempts

### Reproduction

In Add Plugins, install each of the two search results below:

- `Spectra Legacy – Gutenberg Blocks 2.20.1`
- `Spectra Blocks – AI Website Builder for the Block Editor 1.0.3`

### UI result

Each install control changed to `Update failed.` The initial attempt used the same stale installer nonce pattern described above. In a fresh site, repeating the install after the inner WordPress frame loaded a current nonce completed the install; **Spectra Legacy – Gutenberg Blocks 2.20.1** is now active and its exported artifacts are included in the report.
