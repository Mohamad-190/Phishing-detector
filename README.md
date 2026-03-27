# Phishing Detector

Ein Machine-Learning-basierter Phishing-Detektor, der dein Gmail-Postfach überwacht und verdächtige E-Mails anhand von Textanalyse und E-Mail-Header-Authentifizierung identifiziert.

## Überblick

Dieses Projekt kombiniert einen trainierten ML-Klassifikator mit E-Mail-Header-Authentifizierungschecks (SPF, DKIM, DMARC), um Phishing-Versuche zuverlässig zu erkennen. Das System analysiert eingehende E-Mails automatisch und warnt bei verdächtigen Nachrichten.

## Features

- **ML-basierte Erkennung:** TF-IDF Vektorisierung mit Logistic Regression Classifier
- **SPF-Check:** Überprüft ob der sendende Server autorisiert ist, Mails im Namen der Absender-Domain zu versenden
- **DKIM-Check:** Verifiziert die kryptographische Signatur der E-Mail um Manipulation zu erkennen
- **DMARC-Check:** Prüft ob die From-Domain mit den SPF/DKIM-Ergebnissen übereinstimmt (Spoofing-Erkennung)
- **Score-System:** Header-Ergebnisse beeinflussen die Phishing-Wahrscheinlichkeit gewichtet
- **Gmail Integration:** Direkte Anbindung über Google Gmail API

## Technologie-Stack

| Komponente | Technologie |
|---|---|
| Sprache | Python 3 |
| ML Framework | scikit-learn |
| Text Processing | TF-IDF Vectorizer (n-grams) |
| Klassifikator | Logistic Regression |
| E-Mail API | Google Gmail API |
| HTML Parsing | BeautifulSoup4 |
| SPF-Prüfung | pyspf + dnspython |
| DKIM-Prüfung | dkimpy |
| DMARC-Prüfung | dnspython (eigene Implementierung) |

## Installation

### 1. Repository klonen

```bash
git clone https://github.com/Mohamad-190/Phising_detector.git
cd Phising_detector
```

### 2. Abhängigkeiten installieren

```bash
pip install pandas scikit-learn joblib beautifulsoup4 google-auth google-auth-oauthlib google-api-python-client dnspython pyspf dkimpy
```

### 3. Gmail API einrichten

1. Gehe zur [Google Cloud Console](https://console.cloud.google.com/)
2. Erstelle ein neues Projekt
3. Aktiviere die Gmail API
4. Erstelle OAuth 2.0 Credentials (Desktop App)
5. Lade `credentials.json` herunter und platziere sie im Projektordner

### 4. Daten vorbereiten

Stelle sicher, dass folgende Datei im `data/` Ordner vorhanden ist:

- `phishing_emails.csv` – E-Mail-Datensatz mit Spalten `subject`, `body`, `label`

## Verwendung

### Modell trainieren

```bash
python train.py
```

Ausgabe:

```
Lade Daten...
→ X E-Mails geladen
→ Phishing: Y
→ Legitim: Z
Training abgeschlossen!
Genauigkeit: XX.XX%
Modell gespeichert!
```

Das trainierte Modell wird unter `./models/phishing_model.pkl` gespeichert.

### Phishing-Detektor starten

```bash
python main.py
```

Bei erstmaligem Start öffnet sich ein Browser-Fenster zur Gmail-Authentifizierung. Danach prüft das System deine letzten E-Mails und gibt eine Bewertung für jede Mail aus.

## Funktionsweise

### Erkennungs-Pipeline

```
E-Mail eingehend
       ↓
┌──────────────────────────┐
│  Text extrahieren        │  (Betreff + Absender + Body)
└──────────────────────────┘
       ↓
┌──────────────────────────┐
│  ML-Modell Prediction    │  (TF-IDF → Logistic Regression)
└──────────────────────────┘
       ↓
┌──────────────────────────┐
│  Header-Authentifizierung│
│  • SPF-Check             │  (+30% bei fail)
│  • DKIM-Check            │  (+25% bei fail)
│  • DMARC-Check           │  (+35% bei fail)
└──────────────────────────┘
       ↓
┌──────────────────────────┐
│  Finale Bewertung        │  (≥50% → Phishing-Warnung)
└──────────────────────────┘
```

### Score-Anpassungen

| Check | Ergebnis | Anpassung |
|---|---|---|
| SPF | fail | +30% |
| SPF | softfail | +15% |
| SPF | none/neutral | +10% |
| SPF | pass | ±0% |
| DKIM | fail | +25% |
| DKIM | error | +10% |
| DKIM | pass | ±0% |
| DMARC | fail | +35% |
| DMARC | pass | ±0% |
| Alle drei | pass | -15% |

Negative Signale erhöhen die Phishing-Wahrscheinlichkeit. Ein einzelner Pass gilt nicht als Freispruch – nur wenn alle drei Checks bestehen, wird die Wahrscheinlichkeit leicht gesenkt.

## Projektstruktur

```
Phishing_detector/
├── main.py                  # Hauptanwendung (Gmail-Monitoring)
├── train.py                 # Modell-Training
├── headerchecks/
│   ├── __init__.py          # Package-Importe
│   ├── spf_check.py         # SPF-Authentifizierung
│   ├── dkim_check.py        # DKIM-Signaturprüfung
│   └── dmarc_check.py       # DMARC-Alignment-Check
├── data/
│   └── phishing_emails.csv  # Trainingsdaten
├── models/
│   └── phishing_model.pkl   # Trainiertes Modell
├── credentials.json         # Google OAuth (nicht committen!)
├── token.json               # Auth Token (nicht committen!)
└── README.md
```

## Sicherheitshinweise

> ⚠️ **Wichtig:** `credentials.json` und `token.json` müssen in der `.gitignore` stehen, um sensible Daten nicht zu veröffentlichen.
