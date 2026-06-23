# Planimetrie Rapide V4.1

Editor statico locale per consulenti, RSPP, tecnici della prevenzione e professionisti antincendio che devono preparare rapidamente elaborati schematici durante o dopo un sopralluogo.

Il prodotto serve a creare:

- planimetrie di emergenza allegabili al PEE;
- piani dei punti di misurazione;
- layout di supporto al DVR;
- elaborati schematici completi con piu categorie di elementi.

Non genera automaticamente un Piano di emergenza ed evacuazione completo, non e un CAD certificato, non sostituisce rilievi metrici professionali, valutazione dei rischi, progettazione antincendio o verifica normativa e non dichiara conformita automatica.

## Novita V4.1

- Interfaccia riorganizzata per contesto: Completo, Emergenza, Misurazioni, DVR.
- Strumenti separati per Struttura, elementi di elaborato, vie di esodo, vista e cancellazione.
- Modalita Seleziona con evidenziazione, trascinamento, pannello proprieta, duplicazione ed eliminazione.
- Vano scala come elemento strutturale dedicato, migrato dai vecchi marker `scala`.
- Ascensore come elemento strutturale dedicato, con tipologia e piani serviti, migrato dai vecchi marker `ascensore`.
- Testi e annotazioni libere.
- Catalogo simboli esteso per emergenza, misurazioni e DVR.
- Campi misura estesi: codice, tipo, valore, unita, altezza, durata, data, ora, strumento, operatore e note.
- Controlli pre-export differenziati per planimetria di emergenza, misurazioni, DVR e completo.
- Indicazione professionale "Calibrato" / "Scala indicativa"; il PDF non stampa una scala certa se il piano non e calibrato.
- Manifest PWA e icona locale, senza backend e senza telemetria.
- Service worker aggiornato con cache V4.1.
- Progetto demo separato: `demo_officina_v4_1.json`.

## Avvio locale

```bash
cd ~/Desktop/planimetrie_rapide_v4
python3 -m http.server 8080
```

Se la porta 8080 e occupata:

```bash
python3 -m http.server 8081
```

Aprire poi `http://localhost:8080` oppure `http://localhost:8081`.

## Dati e privacy

L'app e composta da HTML, CSS, JavaScript e asset statici. Non usa backend, login, database remoto, analytics o telemetria. I dati restano sul dispositivo tramite:

- file JSON esportati volontariamente dall'utilizzatore;
- autosalvataggio IndexedDB;
- fallback localStorage senza foto quando IndexedDB non e disponibile o lo spazio e insufficiente.

La libreria PDF jsPDF resta caricata da CDN e viene messa in cache dal service worker dopo il primo caricamento online.

## Responsabilita dell'elaborato

Formula usata negli elaborati: "Elaborato redatto dall'utilizzatore con Planimetrie Rapide, piattaforma locale di editing grafico. Contenuti, misure, simboli, classificazioni e conformita dell'elaborato restano sotto responsabilita del redattore/committente."

L'elaborato puo essere usato professionalmente quando viene redatto e assunto dal redattore competente. La piattaforma resta invece uno strumento di composizione grafica e non effettua validazioni tecniche, metriche, antincendio o normative automatiche.

## File principali

- `index.html`: applicazione completa.
- `sw.js`: service worker offline.
- `manifest.webmanifest` e `icon.svg`: installabilita PWA.
- `demo_officina_v4_1.json`: progetto demo per collaudo.
- `MANUALE_RAPIDO.md`: guida operativa.
- `COLLAUDO_MANUALE.md`: checklist manuale.
- `PRODUCT_AUDIT.md`: audit tecnico/prodotto.
- `TEST_REPORT.md`: test eseguiti e limiti.
