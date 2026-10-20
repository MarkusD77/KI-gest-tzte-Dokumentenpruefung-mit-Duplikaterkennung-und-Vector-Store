# KI-gestützte Dokumentenprüfung mit Duplikaterkennung und Vector Store

KI-gestützte Dokumentenprüfung mit Duplikaterkennung und Vector Store

1. Kurzbeschreibung
Dieser n8n-Workflow verarbeitet hochgeladene Geschäftsdokumente (z. B. Angebote) und extrahiert zentrale Informationen wie Titel, Dokumenttyp, Datum, Organisation und Kurzbeschreibung. Vor dem Aufruf eines externen Sprachmodells prüft er anhand eines SHA-256-Hashes (digitaler Fingerabdruck), ob das Dokument bereits bekannt ist. Neue Dokumente werden strukturiert in einer n8n Data Table abgelegt und über lokale Ollama-Embeddings in einen Simple Vector Store übernommen. Der Nutzer erhält eine Abschlussmeldung.
Gelöstes Problem: Manuelle Dokumentenerfassung kostet Zeit; doppelte Uploads führen zu Mehrarbeit, doppelten Datensätzen und unnötigen KI-Kosten.

2. Ziel und Nutzen
•	Zeit: Weniger manuelle Erfassung von Dokumentdaten.
•	Kosten: Bekannte Dokumente werden vor dem Groq-Aufruf ausgesondert; lokale Embeddings verursachen keine Groq-Tokenkosten.
•	Qualität: Einheitliche Datenfelder und ein definierter Status für fehlende Pflichtinformationen.
•	Auffindbarkeit: Inhalte können semantisch durchsucht werden.
•	Nachvollziehbarkeit: Dokument-Hash, Dateiname, Prüfzeitpunkt und Status ermöglichen die Zuordnung von Ergebnissen.
Die Einsparung ist erst nach einem kontrollierten Vergleich von Bearbeitungszeit und Tokenverbrauch quantifizierbar.

3. Beschreibung der Erweiterung
Gegenüber einer direkten KI-Analyse aller Uploads wurden folgende Bausteine ergänzt bzw. angepasst:
1.	Crypto (SHA-256) (SHA-256 Hash berechnen): Berechnet einen stabilen Hash des Dokumentinhalts als Schlüssel zur Duplikaterkennung.
2.	Check Duplicate (Duplikat in Data Table prüfen): Liest vorhandene Dokument-Hashes aus Dokument-Prüfung.
3.	Duplicate Summary (JavaScript) (Duplikatprüfung zusammenfassen): Vergleicht Uploads mit gespeicherten Hashes und erkennt auch doppelte Dateien innerhalb eines Uploads.
4.	IF node (Neue Dokumente vorhanden?) und Form (Hinweis: Nur Duplikate): Bei ausschließlich bekannten Dokumenten wird die KI-Verarbeitung übersprungen und ein Hinweis angezeigt.
5.	Neue Dokumente weitergeben: Übergibt nur neue Dokumente einzeln an die Basic LLM Chain.
6.	Insert row (Prüfergebnisse speichern): Speichert strukturierte Ergebnisse in der n8n Data Table statt in Notion.
7.	Simple Vector Store mit Ollama Embeddings (Dokument im Vector Store speichern): Indiziert Dokumentinhalte für semantische Weiterverarbeitung.
8.	Final result und Form Ending (Erfolgsmeldung anzeigen / Manuelle Prüfung erforderlich): Erstellen eine Rückmeldung nach der Verarbeitung.

4. Ablauf des Workflows
Der Workflow ist in sieben farblich gekennzeichnete Bereiche gegliedert:
01 – Eingabe:
Ablauf = Dokumente hochladen  Uploads einzeln aufteilen
Zweck = Mehrer hochgeladene Dateien getrennt bearbeiten
02 – Erkennung:
Ablauf = Dateityp erkennen  PDF-/DOCX-/Bild-Pfad  Dokumenttext vorbereiten
Zweck = Text aus unterschiedlichen Formaten extrahieren und vereinheitlichen
03 – Duplikate:
Ablauf = SHA-256 Hash berechnen → Duplikat in Data Table prüfen → Duplikatprüfung zusammenfassen → Neue Dokumente vorhanden?
Zweck = Bereits bekannte Dokumente vor kostenpflichtigen KI-Aufrufen ausfiltern
04 – KI:
Ablauf = Neue Dokumente weitergeben → Dokumentdaten mit KI extrahieren, unterstützt durch Groq und JSON-Parser
Zweck = Titel, Typ, Datum, Organisation und Zusammenfassung ermitteln
05 – Speichern:
Ablauf = KI-Ergebnis aufbereiten → Prüfergebnis speichern
Zweck = Metadaten, Hash und Prüfstatus in der n8n Data Table sichern
06 – Vektoren:
Ablauf = Vektordokument vorbereiten → Dokument im Vector Store speichern, mit Ollama und Data Loader
Zweck = Semantische Indizierung der Dokumente
07 – Ergebnis und Rückmeldung:
Ablauf = Verarbeitungsergebnis erstellen → Prüfung erfolgreich? → Abschlussformular
Zweck = Erfolg oder erforderliche manuelle Prüfung anzeigen
Sonderfall – Duplikatprüfung: Sind alle hochgeladenen Dokumente bereits im System vorhanden, wird der Hinweis „Nur Duplikate“ ausgegeben und die Groq-Analyse übersprungen. Werden sowohl neue als auch bereits vorhandene Dokumente hochgeladen, werden ausschließlich die neuen Dokumente an die KI zur weiteren Analyse weitergeleitet.

5.	Tools und KI-Modelle
n8n:
Zweck = Orchestrierung, Formulare, Verzweigungen
Auswahlgrund = Visuelle und wartbare Automatisierung
JavaScript Code Node:
Zweck = Normalisierung, Vergleich, Prüfung
Auswahlgrund = Regelbasierte Verarbeitung ohne KI-Tokens
Crypto SHA-256:
Zweck = Inhaltsfingerabdruck
Auswahlgrund = Schneller Duplikatvergleich; keine Verschlüsselung
n8n Data Table:
Zweck = Speicherung von Hashes und Prüfergebnissen
Auswahlgrund = Einfache interne Datenhaltung ohne Notion
Groq Chat Model:
Zweck = Sprachmodell-Inferenz zur Metadatenextraktion
Auswahlgrund = API-basierte Verarbeitung; nur für neue Dokumente
Structured Output Parser:
Zweck = Schemaorientierte Ausgabe
Auswahlgrund = Einheitliche Datenfelder
Ollama Embeddings:
Zweck = Lokale Textvektoren
Auswahlgrund = Reduziert externe Datenübertragung und Groq-Tokenverbrauch
Simple Vector Store:
Zweck = Semantische Indizierung
Auswahlgrund = Einfacher Prototyp

6.	Daten
Verarbeitet: hochgeladene PDF-/DOCX-/Bilddateien, extrahierte Volltexte, Dateinamen, SHA-256-Hashes, Dokumenttyp, Datum, Organisation, Titel, Kurzbeschreibung, Prüfstatus, Zeitstempel, Fehlerhinweise und Embeddings.
Schutzbedarf: Angebote können Preise, Namen, geschäftliche Kontaktdaten, Adressen, Projektinformationen und vertrauliche Vertragsdetails enthalten. Es können personenbezogene Daten nach DSGVO sowie Geschäftsgeheimnisse betroffen sein.
Datenminimierung: Nur erforderliche Felder speichern; Volltext und Embeddings nur bei freigegebenem Zweck und mit begrenzter Aufbewahrung vorhalten. Keine echten Kundendokumente in öffentliche Test-Repositories aufnehmen. Dokumentinhalte nicht unnötig in n8n-Ausführungslogs protokollieren

7.	Integration im Unternehmen
Technische Voraussetzungen: n8n-Instanz mit Zugriff auf die Data Table; konfigurierter Groq-API-Zugang; erreichbarer Ollama-Server und installiertes Embedding-Modell; geeigneter Dokumentextraktionspfad; HTTPS, Backups, Monitoring und kontrollierter Netzwerkzugriff.
Organisatorische Voraussetzungen und Rollen (vor Produktivstart verbindlich festlegen) – Rollen mit Verantwortungen unten aufgeführt:
Fachbereich / Dokumenteneigentümer = Anforderungen, Datenqualität, fachliche Freigabe
IT / n8n-Administration = Betrieb, Updates, Zugriffsschutz, Backup, Wiederherstellung
Informationssicherheit = Sicherheitsprüfung, Geheimnis- und Zugriffsschutz
Datenschutzbeauftragte/r = DSGVO-Prüfung, Rechtsgrundlage, Auftragsverarbeitung
KI-Verantwortliche/r = Modellfreigabe, Monitoring, Änderungen und Qualitätstests

8.	Governance und Compliance
DSGVO: Rechtsgrundlage und Zwecke dokumentieren; Datenminimierung, Löschfristen, Betroffenenrechte und technische/organisatorische Maßnahmen umsetzen. Bei Übermittlung an Groq sind Auftragsverarbeitung, Speicherort, Drittlandtransfer und gegebenenfalls Transfermechanismen zu prüfen. Eine Datenschutz-Folgenabschätzung ist risikobasiert zu prüfen.
Informationssicherheit: API-Schlüssel in n8n Credentials/Secret-Management, niemals in Git; TLS, Rollenrechte, Backup, Patchmanagement, Protokollierung ohne unnötige Inhaltsdaten.
EU AI Act: Anwendungsfall und Rollen bewerten. Eine reine Dokumentenmetadatenextraktion ist nicht automatisch ein Hochrisiko-KI-System. Bei einer späteren Verwendung für sensible Entscheidungen ist die Einstufung neu zu prüfen. KI-Kompetenz, Transparenz und menschliche Aufsicht angemessen sicherstellen.
Urheberrecht und Geschäftsgeheimnisse: Nur Dokumente mit zulässigen Nutzungsrechten verarbeiten; externe Modellübertragung vertraglich und organisatorisch absichern.
Mitbestimmung: Bei Beschäftigtendaten oder Leistungs-/Verhaltenskontrolle Betriebsrat und zuständige Stellen frühzeitig einbeziehen.
Interne Richtlinien: Freigaben für Cloud-KI, Klassifizierung vertraulicher Dokumente, Löschung und Auditierung beachten.

9.	Risiken und Gegenmaßnahmen
Groq-Rate-Limits oder hohe Tokenkosten = Duplikatprüfung vor Groq; kurze Prompts; Output begrenzen; Retry mit Backoff und Limits
Halluzinierte oder fehlende Metadaten = Schema-/Plausibilitätsprüfung, REVIEW-Status, menschliche Freigabe
Ungültige/leere JSON-Antwort = Parserfehler behandeln, Ergebnis nicht als OK speichern, kontrollierte Wiederholung
Falsche Duplikaterkennung = Einheitliche Hash-Basis, Test mit identischen/geänderten Dokumenten; Hash-Kollisionen sind sehr unwahrscheinlich, aber nicht unmöglich
Parallel hochgeladene gleiche Dokumente = Eindeutigkeit von documentHash serverseitig absichern, falls technisch verfügbar; Race Conditions prüfen
Fehlende Beständigkeit des Simple Vector Store = Für Produktion persistenten Store (z. B. Qdrant) einsetzen und Wiederherstellung testen
Unterschied zwischen Data Table und Vector Store = Status für Indizierung führen, Fehlerbehandlung und Wiederanlauf implementieren
Datenschutzverletzung durch externe APIs = Vorabfreigabe, Datenklassifizierung, AVV-/Transferprüfung, ggf. lokale Modelle
Prompt Injection in Dokumenten = Dokumentinhalt als untrusted behandeln; keine Anweisungen aus Dokumenten ausführen
Unberechtigter Zugriff = RBAC, HTTPS, Secret-Management, eingeschränkte Logs
Doppelte Formularabschlüsse / falscher IF-Zweig = Endbedingungen und gemischte Uploads testen; pro Upload genau eine Nutzerantwort sicherstellen

10.	Menschliche Kontrolle
Die KI-Ausgabe ist ein Vorschlag zur Datenerfassung, keine rechtsverbindliche Dokumentenfreigabe. Der Fachbereich prüft und bestätigt mindestens: fehlende Pflichtfelder, widersprüchliche Angaben, fehlerhafte Datums-/Organisationswerte, unklare Dokumenttypen und wirtschaftlich oder rechtlich relevante Inhalte. Angebote, Vertragsbedingungen, Preise und Zahlungsfreigaben dürfen nicht allein aufgrund der KI-Ausgabe genehmigt werden. REVIEW-Fälle sind an eine verantwortliche Person zu übergeben. Jede manuelle Korrektur sollte mit Zeitpunkt und Verantwortlichem nachvollziehbar sein.

11.	Test und Erfolgsmessung
Test 1 – Auswahl von Dateien, die schon vorher hochgeladen wurden (siehe Screenshot im Worddokument).
Erwartetes Ergebnis = Workflow stoppt nach der Duplikatprüfung: Nur Duplikate gefunden (siehe Screenshot im Worddokument).
 
Test 2 – Auswahl von Dateien, die schon vorher hochgeladen wurden plus einer Datei, die das System noch nicht kennt (siehe Screenshot im Worddokument).
 .
Erwartetes Ergebnis = Workflow läuft komplett durch mit finaler Form, dass Dokumente verarbeitet wurden (siehe Screenshot im Worddokument).

12.	Installation und Nutzung
1.	Docker Desktop: Zur Ausführung der benötigten Container.
2.	n8n (Self-hosted): Zur Erstellung und Ausführung des Workflows.
3.	Ollama: Zur lokalen Erzeugung von Embeddings mit dem Modell qwen3-embedding:0.6b.
4.	Groq-API: Für die KI-gestützte Dokumentenanalyse (API-Schlüssel erforderlich).
5.	Git: Zur Verwaltung und Abgabe des Workflows auf GitHub.
Einrichtung: Docker und n8n starten, Ollama installieren und das Embedding-Modell herunterladen. Anschließend den Workflow als JSON in n8n importieren, die Groq-Zugangsdaten hinterlegen und die benötigte Data Table Dokument-Prüfung einrichten.
Nutzung: Formular öffnen, unterstützte Dokumente hochladen und absenden. Bei ausschließlich bekannten Dateien erscheint eine Duplikatmeldung. Neue Dokumente werden analysiert und indiziert; auffällige Ergebnisse müssen fachlich geprüft werden. Der Workflow soll pro Upload genau eine zusammenfassende Rückmeldung erzeugen.
