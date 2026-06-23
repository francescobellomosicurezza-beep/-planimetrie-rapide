# Collaudo operativo Fase 1 - Planimetrie Rapide

Data collaudo: 2026-06-23

Workspace: `/Users/francescobellomo/Desktop/planimetrie_rapide_v4`

URL usato per il collaudo online: `http://localhost:8765/`

Browser reale usato: Google Chrome headless `Chrome/149.0.7827.156`, controllato via Chrome DevTools Protocol.

Cartella temporanea download: `/tmp/planimetrie_phase1_downloads`

## Verifiche iniziali

| Verifica | Esito | Evidenza |
| --- | --- | --- |
| `pwd` | PASS | `/Users/francescobellomo/Desktop/planimetrie_rapide_v4` |
| `git status` | NON VERIFICABILE | La cartella non e una repository Git: `fatal: not a git repository (or any of the parent directories): .git` |
| `git log --oneline -5` | NON VERIFICABILE | La cartella non e una repository Git: `fatal: not a git repository (or any of the parent directories): .git` |
| Presenza `index.html` | PASS | File presente, 187919 byte |
| Presenza `sw.js` | PASS | File presente, 1002 byte |
| Presenza `vendor/jspdf.umd.min.js` | PASS | File presente, circa 410 KB |
| Avvio server HTTP locale | PASS | Server avviato su porta 8765; porta 8080 bloccata dal sandbox, 8081 occupata |

## Test richiesti

| # | Test | Esito | Evidenza osservata | Errore | Correzione effettuata |
| --- | --- | --- | --- | --- | --- |
| 1 | Apertura app senza errori JavaScript bloccanti | PASS | `document.readyState=complete`; nessuna `Runtime.exceptionThrown` raccolta in Chrome | Nessuno | Nessuna |
| 2 | Caricamento locale di jsPDF | PASS | `window.jspdf.jsPDF` presente; script caricato da `http://localhost:8765/vendor/jspdf.umd.min.js` | Nessuno | Nessuna |
| 3 | Assenza richieste verso `unpkg.com` | PASS | Richieste osservate solo verso `/`, `/vendor/jspdf.umd.min.js`, `/manifest.webmanifest`, `/icon.svg`, `/sw.js`; nessuna verso `unpkg.com` | Nessuno | Nessuna |
| 4 | Creazione planimetria minima | PASS | Progetto salvato con `walls=4`, `doors=1`, `routes=1`, marker `estintore`, `raccolta`, `sei_qui`, `uscita` | Nessuno | Nessuna |
| 5 | Esportazione PDF emergenza | PASS | Scaricato `Collaudo_Istruzioni_rev1_20260623_pee.pdf`, 2 pagine | Nessuno | Nessuna |
| 6a | PDF contiene `Planimetrie Rapide` | PASS | Testo PDF estratto contiene `Planimetrie Rapide` | Nessuno | Nessuna |
| 6b | PDF non contiene `Sicuro360` / `Sicuro 360` | PASS | Testo PDF estratto non contiene `Sicuro360` ne `Sicuro 360` | Nessuno | Nessuna |
| 6c | PDF contiene numero 112 | PASS | Testo PDF estratto contiene `112` nella pagina istruzioni | Nessuno | Nessuna |
| 6d | PDF contiene la planimetria | PASS | PDF contiene XObject immagine nelle pagine esportate; `xobject_pages=2` | Nessuno | Nessuna |
| 6e | PDF contiene la legenda | PASS | Testo PDF estratto contiene `Legenda` | Nessuno | Nessuna |
| 6f | PDF contiene istruzioni quando abilitate | PASS | Testo PDF estratto contiene `Istruzioni di comportamento`; PDF ha 2 pagine | Nessuno | Nessuna |
| 7 | Disattivazione istruzioni e nuovo export | PASS | Caricato progetto con `includeInstructions=false`; scaricato `Collaudo_No_Istruzioni_rev1_20260623_pee.pdf`; nessuna pagina/testo istruzioni, 1 pagina | Nessuno | Nessuna |
| 8 | Export PNG | PASS | Scaricato `Collaudo_No_Istruzioni_Piano_Terra_pee.png`, 37192 byte | Nessuno | Nessuna |
| 9 | Salvataggio progetto JSON | PASS | Scaricato `Collaudo_Istruzioni_rev1_20260623.json`; JSON contiene `floors` e gli oggetti creati | Nessuno | Nessuna |
| 10 | Riapertura progetto JSON | PASS | Riapertura via input file simulato; `empty-state` resta hidden dopo caricamento | Nessuno | Nessuna |
| 11 | Ricaricamento e recupero autosave | PASS | Dopo reload browser, `empty-state hidden=True`; lavoro recuperato da autosave | Nessuno | Nessuna |
| 12a | Offline: server interrotto | PASS | Dopo `Ctrl-C`, richiesta a `http://localhost:8765/` restituisce `ConnectionRefusedError [Errno 61]` | Nessuno | Nessuna |
| 12b | Offline: apertura app | PASS | Con server spento, Chrome ricarica `http://localhost:8765/`; `ready=complete`, titolo `Planimetrie Rapide`, `navigator.serviceWorker.controller=true` | Nessuno | Nessuna |
| 12c | Offline: jsPDF locale disponibile | PASS | Con server spento, `window.jspdf.jsPDF=true`; script risulta `http://localhost:8765/vendor/jspdf.umd.min.js` servito da cache/service worker | Nessuno | Nessuna |
| 12d | Offline: esportazione PDF | PASS | Con server spento, scaricato `Offline_Collaudo_rev1_20260623_pee.pdf`, 279631 byte; contiene `Planimetrie Rapide`, non contiene `Sicuro360` / `Sicuro 360` | Nessuno | Nessuna |
| 13 | Console browser e service worker | PASS | Nessuna eccezione JavaScript app; nessun messaggio console app; service worker supportato, controllante, 1 registrazione | Nessuno applicativo. Chrome headless ha emesso log nativi non applicativi su display/updater/GCM. | Nessuna |
| 14 | Apertura `demo_officina_v4_1.json` | PASS | Demo aperto via input file simulato; `emptyHidden=True`; piano attivo `Piano Terra` | Nessuno | Nessuna |

## Note operative

- La disattivazione istruzioni e stata verificata caricando un JSON con `includeInstructions=false`, per testare esattamente la logica introdotta in Fase 1. Non e stata aggiunta UI.
- Nel primo tentativo diagnostico Chrome ha riutilizzato lo stesso filename PDF, rendendo non rilevabile un secondo download per nome. Il collaudo e stato ripetuto con nomi progetto distinti; nessun bug applicativo e stato rilevato.
- Il test offline e stato ripetuto impostando esplicitamente la cartella download CDP dopo la riconnessione al browser. L'app era gia caricata offline e mostrava `PDF multipiano esportato`; il retry ha prodotto il file PDF offline atteso.

## File modificati durante il collaudo

- `PHASE_1_VALIDATION.md`

Nessun file applicativo e stato modificato durante il collaudo. Nessuna correzione codice e stata necessaria.

## Risultato complessivo

PASS.

La Fase 1 risulta operativamente valida in browser reale per:

- app senza errori JS bloccanti;
- jsPDF locale;
- assenza CDN `unpkg.com`;
- export PDF emergenza;
- branding `Planimetrie Rapide`;
- assenza branding Sicuro360 nel PDF;
- numero 112 nel template istruzioni;
- rispetto di `includeInstructions=false`;
- export PNG;
- salvataggio/riapertura JSON;
- autosave;
- service worker;
- funzionamento offline dopo primo caricamento;
- apertura demo V4.1.

## Regressioni trovate

Nessuna regressione applicativa rilevata.

## Problemi residui

- La cartella non e una repository Git, quindi non e stato possibile verificare branch, status o log commit.
- L'app non espone una UI per attivare/disattivare `includeInstructions`; la verifica e stata fatta via JSON, senza aggiungere funzionalita.
- I log nativi di Chrome headless includono messaggi macOS/Chrome non applicativi; non risultano errori JavaScript dell'app.

## Chiusura Fase 1

La Fase 1 puo essere considerata chiusa dal punto di vista del collaudo operativo richiesto.

Non e stata avviata la Fase 2.
