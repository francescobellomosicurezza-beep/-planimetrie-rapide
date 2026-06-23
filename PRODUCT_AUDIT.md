# Product audit V4.1

## Architettura trovata

L'app e un singolo `index.html` con CSS e JavaScript inline, piu `sw.js`. Non sono presenti framework, build, backend o repository Git nella cartella analizzata.

## Audit normativo e responsabilita

La formulazione "solleva la piattaforma da responsabilita penale" non e stata inserita in forma letterale perche sarebbe tecnicamente fragile: un disclaimer software non puo eliminare obblighi o responsabilita previsti da legge, incarico professionale o ruolo aziendale.

La formula adottata negli elaborati chiarisce invece tre punti corretti senza svalutare il documento:

- l'elaborato e redatto dall'utilizzatore con Planimetrie Rapide;
- la piattaforma e uno strumento locale di editing grafico;
- contenuti, misure, simboli, classificazioni e conformita dell'elaborato restano sotto responsabilita del redattore/committente.

Formula PDF adottata:

"Elaborato redatto dall'utilizzatore con Planimetrie Rapide, piattaforma locale di editing grafico. Contenuti, misure, simboli, classificazioni e conformita dell'elaborato restano sotto responsabilita del redattore/committente."

Questa impostazione protegge meglio l'erogatore della piattaforma senza rendere la planimetria una "demo": il documento puo essere professionale quando viene compilato, verificato e assunto dal redattore competente.

Riferimenti consultati:

- UNI ISO 23601:2024, che riguarda principi di progettazione delle planimetrie di emergenza da esporre e precisa che non copre disegni tecnico-professionali dettagliati per specialisti: https://store.uni.com/uni-iso-23601-2024
- D.Lgs. 81/2008 per il quadro DVR e valutazione dei rischi, da trattare come responsabilita e processo professionale esterno all'editor.
- DM 2 settembre 2021 per gestione emergenza nei luoghi di lavoro, da considerare riferimento di contesto senza trasformare l'app in generatore automatico di piano.

## Benchmark prodotto

I prodotti generalisti osservati nel mercato puntano su template, librerie simboli, import di basi grafiche, collaborazione cloud e integrazioni. Planimetrie Rapide deve differenziarsi non copiando un diagram editor generico, ma restando:

- locale/offline;
- rapido in sopralluogo;
- orientato a emergenza, misurazioni e DVR;
- senza cloud, account o telemetria;
- esplicito sui limiti professionali.

Miglioramento applicato: il catalogo simboli non e piu una riga piatta; e diviso in sezioni leggibili come Antincendio, Esodo e sicurezza, Primo soccorso, Impianti e intercettazioni, Misurazioni, DVR, Depositi, Percorsi DVR e Zone di rischio.

## Audit pittogrammi antincendio

Il catalogo antincendio e stato allineato alla serie ISO 7010 F001-F019 come disponibilita funzionale nell'editor:

- F001 Estintore;
- F002 Idrante/naspo/manichetta;
- F003 Scala antincendio;
- F004 Gruppo attrezzature antincendio;
- F005 Pulsante allarme incendio;
- F006 Telefono emergenza incendio;
- F007 Porta tagliafuoco;
- F008 Batteria estinguente fissa;
- F009 Estintore carrellato;
- F010 Applicatore schiumogeno;
- F011 Applicatore nebbia d'acqua;
- F012 Impianto fisso di spegnimento;
- F013 Bombola estinguente fissa;
- F014 Stazione rilascio remoto;
- F015 Monitor antincendio;
- F016 Coperta antincendio;
- F017 Ascensore antincendio;
- F018 Lampeggiante allarme incendio;
- F019 Manichetta scollegata.

I disegni sono pittogrammi schematici proprietari in stile coerente. L'app non dichiara certificazione automatica dei simboli; il codice ISO e riportato come riferimento di catalogazione quando pertinente.

## Modello dati

Il progetto contiene:

- `version`, `meta`, `gridSizeM`, `header`, `instructions`;
- `floors`, ognuno con `walls`, `reiWalls`, `windows`, `doors`, `stairs`, `elevators`, `areas`, `texts`, `markers`, `routes`, `photo`, `photoOpacity`, `northDeg`, `calibrated`.

Sono mantenuti alias sullo stato per il piano attivo, cosi le funzioni storiche continuano a leggere `state.walls`, `state.markers` ecc.

## Rendering

Il canvas usa coordinate mondo in metri e conversione `worldToScreen` / `screenToWorld`. Il rendering e immediato su canvas 2D: griglia, foto, aree, muri, REI, porte, finestre, vani scala, ascensori, vie di esodo, marker, testi, quote, nord e selezione.

## Hit testing

Sono presenti funzioni dedicate per marker, muri, REI, porte, finestre, scale, ascensori, aree, testi e vie. La modalita Seleziona usa un ordine dall'elemento piu specifico a quello piu esteso.

## Storico

Annulla/ripristina usa snapshot JSON serializzati. Le modifiche da disegno, pannello proprieta, duplicazione, cancellazione e trascinamento vengono salvate nello storico.

## Salvataggio

Autosave su IndexedDB con fallback localStorage. Il fallback puo perdere la foto per limiti di spazio, ma l'app mostra un avviso. Salvataggio e apertura JSON restano disponibili.

## Export

Export PNG e PDF multipiano restano client-side. PDF usa jsPDF da CDN/cache. Gli export sono: planimetria di emergenza, piano misurazioni, layout DVR e documento completo.

## Offline

Il service worker aggiorna cache e include `index.html`, manifest, icona e jsPDF CDN. L'offline completo dipende dal primo caricamento riuscito della libreria PDF.

## Limiti residui

- Non e un CAD: non sono implementate maniglie geometriche complete per ogni vertice.
- Le porte e finestre non fanno ancora snap automatico a un muro reale, ma sono modificabili e orientabili tramite coordinate.
- La foto di base non ha ancora crop/rotazione dedicata.
- L'import PDF/DWG/DXF non e implementato.
- I test console/PDF reali richiedono browser automation o collaudo manuale su browser.

## Miglioramenti consigliati successivi

- Libreria simboli ancora piu guidata con ricerca testuale e preferiti.
- Snap automatico porte/finestre al muro piu vicino.
- Maniglie vertice per aree e vie di esodo.
- Ritaglio/rotazione foto base.
- Export SVG opzionale solo se non compromette PDF/PNG.
