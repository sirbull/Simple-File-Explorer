# Simple-File-Explorer
A simple file explorer for use in the browser. Simply add the HTML and inside your folder and run explorer.html. Search, see content and more!

---

# En lettvekts, én-fil filutforsker for webmapper. 
# Den leser Apache AutoIndex-siden (mappelisting) og viser filer/mapper i et pent UI – uten serverkode.

# Funksjoner
- Navigasjon i mapper (brødsmuler + «Opp ett nivå»)
- Søk/filter i aktuell mappe
- Forhåndsvisning av bilder, video, lyd og tekstfiler
- Kopiér direkte-URL med ett klikk
- Fungerer i ett enkelt HTML-dokument (ingen bygg, ingen avhengigheter)

# Forutsetninger
Serveren må levere mappelisting (Apache: Options +Indexes).
I mappen du vil liste må det ikke ligge en index.html (da vises ikke autoindex).
Filrettigheter: mapper 755, filer 644 (typisk webhotell-standard).
Testet mot standard Apache AutoIndex (små variasjoner støttes; se «Feilsøking» om temaet er uvanlig).

# Installasjon
Legg explorer.html i mappen du vil bla i, f.eks.:
    foldername/explorer.html

Sørg for at mappen ikke har en index.html.

## Åpne i nettleser:
https://url.com/explorer.html

## Alternativ plassering
Hvis du må beholde en index.html i rotmappen:

Opprett en undermappe, f.eks. /_explorer/

## Legg explorer.html der:
    /folder_name/_explorer/explorer.html
### Åpne:
    https://url.com/_explorer/explorer.html

# (Valgfritt) CORS, nyttig om andre verktøy skal hente filer
    <IfModule mod_headers.c>
    Header set Access-Control-Allow-Origin "*"
    Header set Access-Control-Allow-Methods "GET, HEAD, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type, Range"
    Header always set X-Content-Type-Options "nosniff"
    </IfModule>

# Sikkerhet: ikke kjør skript i denne mappen
    <FilesMatch "\.(php|phtml|phps|cgi|pl|py|sh|rb)$">
    Require all denied
    </FilesMatch>

    # (Valgfritt) enkel caching av statiske filer
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

Aktiver HTTPS-redirect først når sertifikatet for subdomenet er utstedt, ellers kan du «låse deg ute».

# Bruk:
- Klikk på mappe for å åpne den.
- Klikk på fil for å åpne i ny fane; støttede formater får også innebygd forhåndsvisning:
- Bilder: png, jpg, jpeg, webp, gif, svg
- Video: mp4, webm, mov
- Lyd: mp3, wav, ogg, m4a, flac
- Tekst: txt, md, json, js, css, html, csv, log
- Søkefeltet filtrerer radene i den aktive mappen.
- Copy-knapp kopierer direkte-URL til filen.

# Dyp lenking
URL-hash brukes for undermapper. Du kan lenke direkte:
    https://url.com/explorer.html#undermappe/enda/

# Tilpasning
- Ikoner per filtype: juster funksjonen iconFor() i koden.
- Filtrering/sortering: utvid applyFilter() eller sorteringslogikken før rendering.
- Forhåndsvisning: utvid preview() hvis du vil støtte flere formater.

# Feilsøking
Symptom: Tom liste / varsel “Klarte ikke å lese mappen …”
Sjekk:
1: Gå til mappen direkte i nettleseren:
    https://url.com
– ser du en enkel mappeliste?
Nei → AutoIndex er ikke aktiv. Sjekk .htaccess og at ingen overordnet .htaccess har Options -Indexes eller Require all denied.
Ja → AutoIndex er på; da er det som regel:
Feil plassering av explorer.html (ligger ikke i mappen du tester).

En svært tilpasset AutoIndex-mal som parseren ikke gjenkjenner. Da kan du:
Oppdatere parseAutoIndex() med selektorer som passer ditt tema.
Eller bruke PHP-varianten (under).

## Symptom: 403/401 på noen filer
Filrettigheter og/eller hotlink-/IP-begrensninger. Slå av hotlink-beskyttelse eller whitelist domenet.

## Symptom: Åpner ikke på https://
Vent på SSL-sertifikat for subdomenet, eller test midlertidig via http:// uten tvungen redirect.
PHP-variant (hvis AutoIndex er av)
Dersom du ikke kan bruke Options +Indexes, finnes en alternativ én-fil PHP-lister som leser filsystemet direkte (og ikke er avhengig av AutoIndex-HTML). Gi beskjed, så følger en index.php-versjon med samme UI/oppførsel.

# Kjente begrensninger
Ekstremt store mapper (tusenvis av filer) kan gjøre parsing treg i nettleser.

Avhengig av at serveren leverer en «vanlig» AutoIndex-side. Sterkt tilpassede tema kan kreve små justeringer i parseAutoIndex().

# Eksempel-URLer:
Utforsker: https://url.com/explorer.html
Mappeliste (server): https://url.com/

# Informasjon som må med i filen som ligger ved som heter .htaccess:
## Legg i mappen du vil liste (her: /foldername/.htaccess):

    Options +Indexes
    Require all granted

    <IfModule mod_rewrite.c>
    RewriteEngine On
    # (valgfritt) sett base hvis noe oppfører seg rart på webhotell
    # RewriteBase /bank/

    # Kun hvis det ikke finnes en faktisk fil eller mappe med samme navn
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d

    # /bank/explorer  ->  /bank/explorer.html
    RewriteRule ^explorer/?$ explorer.html [L,NC]

    # (om du vil beholde fra i sted)
    RewriteRule ^update/?$   update.html           [L,NC]
    RewriteRule ^refresh/?$  manifest.php?write=1  [L,NC]
    </IfModule>
