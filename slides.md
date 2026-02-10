# Abwehrmechanismen gegen Symlink-Attacken
## auf RHEL-Systemen mit Dateiupload
### Beispiel: LAMP-Setup

---

## Kontext: LAMP unter RHEL

**Stack**
- RHEL 8 / 9
- Apache httpd
- PHP (mod_php / php-fpm)
- MySQL / MariaDB

**Typisch**
- Webbasierte Dateiuploads
- Schreibrechte für Apache/PHP
- Serverseitige Weiterverarbeitung

---

## Was ist eine Symlink-Attacke?

- Symbolischer Link zeigt auf andere Datei
- Anwendung folgt dem Link unbemerkt
- Schreibzugriff landet im falschen Ziel

**Folgen**
- Überschreiben sensibler Dateien
- Privilege Escalation
- Denial of Service

---
## Typisches LAMP-Upload-Beispiel (unsicher)

```php
move_uploaded_file(
  $_FILES['file']['tmp_name'],
  "/var/www/html/uploads/" . $_FILES['file']['name']
);

Probleme

    Keine Symlink-Prüfung

    Upload im Webroot

    Apache folgt Symlinks

Was braucht eine Angreifer*in?

Minimal

    Zugriff auf Upload-Funktion

    Möglichkeit, Symlinks zu erzeugen

    Fehlende serverseitige Prüfung

Optional

    Wissen über Pfade & Rechte

    Race-Condition-Fenster

➡️ Kein Root-Zugriff notwendig
Beispiel-Angriff

    Symlink erzeugen

ln -s /etc/passwd evil.jpg

    Upload von evil.jpg

    PHP schreibt Datei

    /etc/passwd wird überschrieben

Warum betrifft das RHEL?

    Linux folgt Symlinks per Design

    RHEL ist nicht unsicher

    Ursache liegt in:

        fehlerhafter App-Logik

        fehlendem Hardening

        Legacy-Code

Abwehrprinzip

Defense in Depth

    Anwendung

    Webserver

    Filesystem

    Betriebssystem (SELinux)

➡️ Einzelmaßnahmen reichen nicht
Abwehr 1: Upload richtig ablegen

Best Practice

    Nicht im Webroot:

/var/www/html/uploads

    Stattdessen:

/var/lib/app/uploads

Vorteil

    Kein direkter Webzugriff

    Klare Trennung

Abwehr 2: PHP-Validierung

Maßnahmen

    Keine Symlinks erlauben

    Nur reguläre Dateien

    Zufällige Dateinamen

Beispiele

    is_link()

    lstat()

    realpath() + Prefix-Check

Abwehr 3: Apache Hardening

<Directory "/var/lib/app/uploads">
  Options -FollowSymLinks -ExecCGI
  AllowOverride None
</Directory>

Schützt vor

    Symlink-Folgen

    Code-Ausführung

    .htaccess-Missbrauch

Abwehr 4: Filesystem-Optionen

Dediziertes Filesystem

noexec,nodev,nosuid

Effekt

    Keine Code-Ausführung

    Schaden begrenzt

Abwehr 5: SELinux (RHEL-Stärke)

    Apache läuft in httpd_t

    Upload-Verzeichnis:

httpd_sys_rw_content_t

Wirkung

    Apache darf nicht in /etc schreiben

    Symlink-Ziel außerhalb → Zugriff blockiert

Angriff mit vs. ohne SELinux
Szenario	Ergebnis
Kein SELinux	Datei überschrieben
SELinux enforcing	Zugriff verweigert
App-Bug	Schaden begrenzt
Aufwand vs. Nutzen
Maßnahme	Aufwand	Nutzen
Upload außerhalb Webroot	gering	hoch
PHP-Prüfungen	mittel	sehr hoch
Apache Hardening	gering	hoch
FS-Mount-Optionen	gering–mittel	hoch
SELinux enforcing	mittel	sehr hoch
Herausforderungen

    Legacy-PHP-Code

    Fehlendes SELinux-Know-how

    Unterschätztes Risiko

    Race-Conditions schwer testbar

Fazit

    Dateiuploads sind High Risk

    Symlink-Attacken sind:

        einfach

        effektiv

        oft übersehen

    RHEL bietet starke Bordmittel

    Kombination aus App + OS + SELinux ist entscheidend

Takeaway

    Wenn ein Upload schreibbar ist,
    ist er auch angreifbar.

Sicherheit beginnt beim Upload.


