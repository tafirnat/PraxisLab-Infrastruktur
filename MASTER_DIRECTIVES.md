# Master-Direktiven & Sicherheitsrichtlinien (PraxisLab-Infrastruktur)


Dieses Dokument definiert die **Sicherheitsregeln, Arbeitsweisen und Richtlinien** für den Entwickler und alle KI-Assistenten (wie Gemini, Cursor usw.), die an diesem Repository arbeiten.

---

## 🇩🇪 1. Kontext & Zielsetzung (Kontext & Ziel)

*   **Entwickler-Kontext:** Der Entwickler lebt in Deutschland und strebt eine Beschäftigung (Praktikum oder Festanstellung) auf dem **deutschen IT-Markt** an.
*   **Sprachanforderung & Niveau (B2):** Alle Dokumentationen, Kommentare im Code, Fehlermeldungen und Architekturberichte müssen **ausschließlich in professionellem, natürlichem und grammatikalisch korrektem Deutsch auf B2-Niveau (Fachdeutsch)** verfasst werden. Die Sprache soll präzise, klar und fachlich fundiert sein, ohne jedoch übertrieben akademisch oder künstlich gehoben (z. B. C2-Ebene) zu wirken. Dies stellt sicher, dass die Dokumentation authentisch zum B2-Zertifikat des Entwicklers passt und bei Bewerbungen einen stimmigen Gesamteindruck hinterlässt.
*   **Interaktionsmodus:** Der Entwickler stellt Anfragen, Erklärungen und Fehlerprotokolle primär auf **Türkisch** bereit. Die KI übersetzt und formuliert diese Inhalte eigenständig in das oben definierte B2-Fachdeutsch für alle GitHub-Dokumente und Repositories.

---

## 🛡️ 2. Sicherheitsrichtlinien (Sicherheitsregeln)

### A. Schutz vertraulicher Daten (Keine fest kodierten Secrets)
*   Es dürfen **keine echten API-Keys, Passwörter, Token, Datenbank-Verbindungszeichenfolgen (Connection Strings) oder personenbezogenen Daten (PII)** im Code oder in der Dokumentation verwendet werden.
*   Stattdessen müssen Umgebungsvariablen (z. B. `process.env.VARIABLE_NAME` oder `os.getenv("VARIABLE_NAME")`) oder sichere Platzhalter (z. B. `IHR_API_KEY`, `PASSWORT_PLATZHALTER`) genutzt werden.
*   Entwicklungslösungen müssen den OWASP Top 10 Sicherheitsstandards entsprechen.

### B. Erkennung von Zugangsdaten (Credentials Hunter)
*   Sollten im vom Benutzer bereitgestellten Code, in Logs oder Konfigurationen echte Passwörter, private Schlüssel (Private Keys) oder sensible Daten entdeckt werden, **muss die KI den Benutzer sofort warnen und diese Daten in der Antwort maskieren** (z. B. `************`).

### C. Dateiisolation (File Isolation)
*   Auf Dateien wie `.env`, `.git`, sensible `config.json` oder Zertifikatsdateien (`.pem`, `.crt`, `.key`) darf nur zugegriffen oder analysiert werden, wenn der Benutzer dies explizit verlangt.
*   Lokale sensible Dateien dürfen unter keinen Umständen in das Repository committet werden.

### D. Netzwerk- und IP-Maskierung
*   Interne RFC1918-IP-Adressen (z. B. `10.0.0.0/8`, `192.168.0.0/16`) können in der lokalen Dokumentation verwendet werden. Öffentliche WAN-IPs, ISP-Daten oder spezifische externe Domainnamen **müssen jedoch streng maskiert oder durch Dummy-Werte ersetzt werden**.

---

## 🤝 3. Richtlinien für die KI-Zusammenarbeit (AI-Collaboration)

### 1. Planungsphase (Planning Mode)
*   Vor der Durchführung komplexer Konfigurationen oder Codeänderungen muss die KI einen **Implementierungsplan** auf Deutsch erstellen.
*   Der Plan muss die Sicherheitsrisiken und Validierungsschritte enthalten.

### 2. Protokollierung von Fehlern (Fehlermanagement)
*   Fehler und misslungene Versuche dürfen nicht verschwiegen werden. Sie sind ein wichtiger Teil des Lernprozesses und müssen wie folgt dokumentiert werden:
    1. *Was war der Fehler und warum ist er aufgetreten?*
    2. *Welche Lösungswege wurden ausprobiert?*
    3. *Wie wurde das Problem endgültig gelöst und wie kann es zukünftig vermieden werden?*

### 3. Keine automatischen Git-Operationen
*   Die KI darf **keine Git-Befehle** (wie `git init`, `git add`, `git commit`, `git push`) ohne die explizite Aufforderung und Bestätigung des Benutzers ausführen. Der Benutzer verwaltet seine Git-Historie selbst.
