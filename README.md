# A lightweight, single-file file explorer for web directories.  
## It reads the Apache AutoIndex page (directory listing) and displays files/folders in a clean UI – without any server code.

# Features
- Folder navigation (breadcrumbs + “Up one level”)
- Search/filter in the current directory
- Preview of images, video, audio, and text files
- Copy direct URL with one click
- Works in a single HTML document (no build, no dependencies)

# Requirements
The server must provide directory listing (Apache: Options +Indexes).  
The folder you want to list must not contain an index.html (otherwise AutoIndex won’t appear).  
File permissions: folders 755, files 644 (typical web hosting default).  
Tested against standard Apache AutoIndex (minor variations supported; see “Troubleshooting” if the theme is unusual).

# Installation
Place explorer.html in the folder you want to browse, e.g.:
    foldername/explorer.html

Make sure the folder does not contain an index.html.

## Open in browser:
https://url.com/explorer.html

## Alternative location
If you must keep an index.html in the root directory:

Create a subfolder, e.g. /_explorer/

## Place explorer.html there:
    /bank/_explorer/explorer.html
### Open:
    https://url.com/_explorer/explorer.html

# (Optional) CORS – useful if other tools need to fetch files
    <IfModule mod_headers.c>
    Header set Access-Control-Allow-Origin "*"
    Header set Access-Control-Allow-Methods "GET, HEAD, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type, Range"
    Header always set X-Content-Type-Options "nosniff"
    </IfModule>

# Security: do not execute scripts in this folder
    <FilesMatch "\.(php|phtml|phps|cgi|pl|py|sh|rb)$">
    Require all denied
    </FilesMatch>

    # (Optional) simple caching for static files
    <IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType text/css "access plus 7 days"
    ExpiresByType application/javascript "access plus 7 days"
    ExpiresByType image/png "access plus 30 days"
    ExpiresByType image/jpeg "access plus 30 days"
    ExpiresByType image/webp "access plus 30 days"
    ExpiresByType image/svg+xml "access plus 30 days"
    ExpiresDefault "access plus 1 day"
    </IfModule>

Enable HTTPS redirect only when the certificate for the subdomain has been issued, otherwise you might “lock yourself out.”

# Usage:
- Click on a folder to open it.
- Click on a file to open it in a new tab; supported formats also have built-in preview:
- Images: png, jpg, jpeg, webp, gif, svg
- Video: mp4, webm, mov
- Audio: mp3, wav, ogg, m4a, flac
- Text: txt, md, json, js, css, html, csv, log
- The search field filters the rows in the active folder.
- The copy button copies the direct URL of the file.

# Deep linking
The URL hash is used for subfolders. You can link directly:
    https://url.com/explorer.html#subfolder/deeper/

# Customization
- Icons per file type: adjust the function iconFor() in the code.
- Filtering/sorting: extend applyFilter() or the sorting logic before rendering.
- Preview: extend preview() if you want to support more formats.

# Troubleshooting
**Symptom:** Empty list / message “Failed to read directory …”  
Check:
1: Go to the folder directly in your browser:
    https://url.com
– Do you see a simple directory list?
No → AutoIndex is not active. Check .htaccess and make sure no parent .htaccess has Options -Indexes or Require all denied.  
Yes → AutoIndex is active; then usually:
Incorrect placement of explorer.html (not located in the folder you are testing).

A heavily customized AutoIndex template that the parser doesn’t recognize. Then you can:
Update parseAutoIndex() with selectors that match your theme.  
Or use the PHP variant (below).

## Symptom: 403/401 on some files
File permissions and/or hotlink/IP restrictions. Disable hotlink protection or whitelist your domain.

## Symptom: Doesn’t open via https://
Wait for the SSL certificate for the subdomain, or test temporarily via http:// without forced redirect.

# PHP variant (if AutoIndex is disabled)
If you cannot use Options +Indexes, there is an alternative single-file PHP lister that reads the filesystem directly (and doesn’t depend on AutoIndex HTML). Let me know, and an index.php version with the same UI/behavior will follow.

# Known limitations
Extremely large folders (thousands of files) can make parsing slow in the browser.

Depends on the server providing a “standard” AutoIndex page. Strongly customized themes may require small adjustments in parseAutoIndex().

# Example URLs:
Explorer: https://url.com/explorer.html  
Directory list (server): https://url.com/

# Information that must be included in the accompanying file named .htaccess:
## Place in the folder you want to list (here: /foldername/.htaccess):

    Options +Indexes
    Require all granted

    <IfModule mod_rewrite.c>
    RewriteEngine On
    # (optional) set base if something behaves oddly on web hosting
    # RewriteBase /bank/

    # Only if there’s no actual file or folder with the same name
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d

    # /bank/explorer  ->  /bank/explorer.html
    RewriteRule ^explorer/?$ explorer.html [L,NC]

    # (keep from before if needed)
    RewriteRule ^update/?$   update.html           [L,NC]
    RewriteRule ^refresh/?$  manifest.php?write=1  [L,NC]
    </IfModule>
