# Rapporto di test - Planimetrie Rapide V4.1

Data: 18 giugno 2026

## Test automatici eseguiti

1. `pwd`: confermata cartella `/Users/francescobellomo/Desktop/planimetrie_rapide_v4`.
2. `ls -la`: identificati file iniziali.
3. `rg --files`: identificati tutti i file della cartella.
4. `git status --short`: cartella non Git.
5. Verifica sintattica iniziale di `sw.js` con `node --check`.
6. Verifica sintattica iniziale dello script inline di `index.html` con `new Function`.
7. Verifica sintattica intermedia dopo modifiche.
8. Parsing JSON di `demo_officina_v4_1.json`.
9. Avvio server locale: porta 8080 occupata, avviato server su 8081.
10. HTTP 200 su `/`, `/sw.js`, `/manifest.webmanifest`, `/icon.svg`.

## Test statici/funzionali coperti dal codice

- Migrazione JSON V3/V4 tramite `normalizeProjectData`.
- Migrazione marker `scala` verso `stairs`.
- Migrazione marker `ascensore` verso `elevators`.
- Round-trip serializzabile tramite `serializeProject`.
- Autosave IndexedDB e fallback localStorage preservati.
- Export PNG/PDF mantiene pipeline esistente e include nuovi elementi.

## Test non eseguiti automaticamente

Playwright e Puppeteer non sono installati nell'ambiente locale. Non sono quindi stati eseguiti automaticamente:

- controllo console browser reale;
- interazioni canvas con pointer/touch;
- esportazioni PDF reali con jsPDF nel browser;
- test responsive visuale;
- test offline reale via DevTools;
- verifica assenza richieste di rete dal pannello Network.

Questi controlli sono riportati in `COLLAUDO_MANUALE.md` e devono essere eseguiti in browser prima di pubblicare.

## Risultato

La versione passa i controlli sintattici e statici eseguiti, il server locale risponde correttamente e gli asset offline/PWA sono serviti. Restano necessari collaudo browser e verifica visiva degli export.
