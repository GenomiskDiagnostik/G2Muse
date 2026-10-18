# G2Muse — Simulator-screenshot til Even Hub-godkendelse

Even Realities kræver, at screenshots til app-godkendelse er taget med
**simulatorens egen screenshot-funktion** — ikke mockups eller
telefon-screenshots. Denne vejledning går ud fra en helt ren maskine.

## Trin 0: Installer forudsætninger

Du skal bruge **Node.js** (med npm) og **Python 3**.

**macOS:**
- Installer Homebrew hvis du ikke har det: https://brew.sh
- `brew install node python`

**Windows:**
- Node.js: hent LTS-installeren fra https://nodejs.org (næste-næste-færdig)
- Python: hent fra https://www.python.org — **vigtigt:** sæt flueben ved
  *"Add python.exe to PATH"* under installationen

**Ubuntu/Debian Linux:**
```
sudo apt update && sudo apt install -y nodejs npm python3
```

**Tjek at det virker** (nyt terminalvindue):
```
node --version
npm --version
python3 --version
```
Alle tre skal svare med et versionsnummer.

## Trin 1: Installer simulatoren

```
npm install -g @evenrealities/evenhub-simulator
```

Tjek: `evenhub-simulator --version` skal svare (f.eks. `0.9.3`).

## Trin 2: Pak appens filer ud og servér dem

1. Find **source-ZIP'en** til den version du vil indsende
   (f.eks. `g2muse-2.0.1-source.zip`) og pak den ud et sted,
   f.eks. mappen `g2muse-src`.
2. Åbn et terminalvindue, gå ind i mappen med `dist`-undermappen,
   og start en lokal webserver:
```
cd <sti til g2muse-src>
python3 -m http.server 5173
```
Lad dette vindue stå åbent — serveren skal køre mens simulatoren
bruges. (Windows: `py -m http.server 5173` hvis `python3` ikke findes.)

## Trin 3: Start simulatoren med engelsk UI

Åbn et **nyt** terminalvindue:
```
LANG=en_US.UTF-8 evenhub-simulator http://localhost:5173 --automation-port 9898
```
- `LANG=en_US.UTF-8` sørger for engelsk UI i appen
  (Windows: kør `set LANG=en_US.UTF-8` først i samme vindue,
  eller spring over hvis dit Windows i forvejen er engelsk).
- `--automation-port 9898` åbner screenshot-API'et.
- Vent ca. 5 sekunder til appen er startet i simulator-vinduet.

## Trin 4: Tag screenshottet

Åbn et **tredje** terminalvindue:
```
curl http://127.0.0.1:9898/api/screenshot/glasses -o g2muse-screenshot.png
```
Windows uden curl — brug PowerShell i stedet:
```
Invoke-WebRequest -Uri http://127.0.0.1:9898/api/screenshot/glasses -OutFile g2muse-screenshot.png
```

**Alternativ (uden terminal):** klik på screenshot-knappen i
simulator-vinduet. Billedet gemmes som PNG i den mappe, du startede
simulatoren fra.

## Hvad du bør se

- **Uden konfigureret relay** viser appen førstegangs-skærmen med
  setup-vejledning på engelsk. Det er helt fint — godkendelsen tjekker
  netop, at første start forklarer hvad man skal gøre (aldrig sort skærm).
- **Vil du vise besked-visningen med eksempel-indhold** (i stedet for
  den tomme førstegangs-skærm), så brug **demo-builden**:
  `g2muse-demo-simulator.zip`. Den er identisk med den rigtige app, men
  springer relay-opsætningen over og viser tre eksempel-beskeder på
  engelsk. Pak den ud og servér `dist-demo`-mappen i trin 2 i stedet
  for den almindelige `dist`-mappe. Demo-builden er ikke versioneret
  og må ikke indsendes — den er kun til screenshots.
- Billedet er **576×288 PNG** (simulatorens framebuffer, rå pixels).

## Ryd op

Luk de tre terminalvinduer med `Ctrl+C` når du er færdig.

## Fejlfinding

| Problem | Løsning |
|---|---|
| `evenhub-simulator: command not found` | Luk terminalen og åbn en ny efter `npm install -g` (PATH opdateres først da) |
| Simulatoren starter ikke / mangler biblioteker (Linux) | Kør `sudo apt install -y libwebkit2gtk-4.1-0` |
| `python3: command not found` (Windows) | Brug `py -m http.server 5173` i stedet |
| Sort skærm i simulatoren | Tjek at webserveren fra trin 2 stadig kører, og at URL'en er `http://localhost:5173` |
| Appen vises på dansk | Sørg for `LANG=en_US.UTF-8` foran simulator-kommandoen (trin 3) |
