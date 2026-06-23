# CHECKPOINT 2C - Legenda automatica professionale e paginabile

Data: 23/06/2026  
Workspace: `/Users/francescobellomo/Desktop/planimetrie_rapide_v4`  
Branch: `main`

## Sintesi

Checkpoint 2C completato con intervento limitato all'export PDF. La legenda ora viene raccolta come lista di elementi esportati, ordinata per categorie, deduplicata e spostata su pagina dedicata quando non rispetta le soglie di leggibilita' della pagina principale.

Non sono stati modificati modello dati globale, canvas engine, schema JSON, IndexedDB, localStorage, serializzazione o `symId`.

## Logica implementata

La legenda viene costruita in tre passaggi:

1. raccolta degli item da marker gia' filtrati dal renderer PDF e da elementi strutturali realmente presenti nel piano;
2. deduplicazione e ordinamento stabile;
3. misura del layout per decidere se disegnare in pagina principale o in pagine `Legenda` dedicate.

Il cartiglio del checkpoint 2B resta invariato e continua a essere disegnato nella fascia inferiore della pagina planimetrica.

## Funzioni aggiunte o modificate

File: `index.html`

Funzioni modificate:

- `buildPDF`
- `estimateLegendH`
- `drawLegendPDF`

Funzioni aggiunte:

- `normalizeLegendLabel`
- `legendCategoryRank`
- `legendCategoryForSym`
- `legendItemSort`
- `addLegendItem`
- `collectLegendItems`
- `mainLegendLimit`
- `measureLegendLayout`
- `drawLegendIcon`
- `legendText`
- `drawLegendPages`

File: `sw.js`

- Aggiornato solo il nome cache da checkpoint 2B a checkpoint 2C.

## Criteri di filtraggio

La legenda usa:

- marker effettivamente renderizzati in `renderFloorToCanvas`, quindi gia' filtrati da `layerVisible`;
- vie di esodo solo se `routesVisible()` e `floor.routes.length > 0`;
- muri REI/EI solo se `reiVisible()` e `floor.reiWalls.length > 0`;
- porte, finestre, scale, ascensori e testi solo se presenti nel piano esportato;
- aree/locali solo per export non PEE;
- elementi DVR e misure solo se pertinenti al tipo export tramite layer gia' esistenti.

Se non ci sono item legenda, non viene creato spazio grande inutilizzato e non viene generata una pagina vuota.

## Criteri di deduplicazione

Deduplicazione applicata:

- marker per `symId`;
- strutturali per chiave semantica interna, ad esempio `struct:doors`, `struct:windows`, `route:esodo`;
- fallback per descrizione normalizzata.

Non sono stati uniti elementi distinti come:

- `uscita` e `uscita_emerg`;
- `estintore` e `estintore_carrellato`;
- `primo_socc` e `pacchetto_med`;
- elementi DVR e punti di misura.

## Ordine categorie

Ordine stabile implementato:

1. esodo ed evacuazione;
2. antincendio;
3. primo soccorso;
4. impianti e intercettazioni;
5. elementi strutturali;
6. DVR e rischi;
7. punti di misura;
8. altri elementi.

All'interno delle categorie l'ordine segue il catalogo `SYMBOLS` o un ordine tecnico fisso per gli elementi strutturali.

## Soglie usate

Pagina principale:

- font legenda: 8.2 pt;
- icona: 5 mm;
- riga: 7 mm;
- larghezza minima colonna: 55 mm;
- massimo 2 colonne;
- massimo 28% dell'altezza utile;
- planimetria non sotto il 60% dell'area utile stimata.

Limiti numerici:

- A4 verticale: massimo 6 voci;
- A4 orizzontale: massimo 8 voci;
- A3 verticale: massimo 8 voci;
- A3 orizzontale: massimo 10 voci.

Pagina dedicata:

- A4: massimo 2 colonne;
- A3: massimo 3 colonne;
- font: 8.5 pt;
- icona: 6 mm;
- pagine ulteriori se la capacita' della pagina non basta.

## Comportamento pagina principale

La legenda resta nella pagina planimetrica solo se:

- gli item sono entro soglia;
- la colonna resta almeno 55 mm;
- l'altezza legenda resta sotto il 28% dell'area utile;
- l'immagine planimetrica non viene compressa sotto la soglia interna del 60%.

Nei casi brevi testati la legenda resta nella pagina principale.

## Comportamento pagina aggiuntiva

Quando gli item superano le soglie, viene aggiunta una o piu' pagine `Legenda` dopo le pagine planimetriche e prima di tabelle misure/istruzioni.

La pagina legenda riporta:

- titolo `Legenda`;
- azienda;
- piano;
- revisione;
- footer con numerazione pagina;
- item in colonne, senza taglio.

## File modificati

- `index.html`
- `sw.js`
- `CHECKPOINT_2C_REPORT.md`

## PDF campione generati

Cartella: `/tmp/planimetrie_2c_downloads`

- `/tmp/planimetrie_2c_downloads/CP2C_PEE_A4_breve_rev3_20260623_pee.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_PEE_A4_lunga_rev3_20260623_pee.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_PEE_A3_breve_rev3_20260623_pee.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_PEE_A3_verticale_rev3_20260623_pee.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_Completo_complesso_rev3_20260623_completo.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_DVR_rev3_20260623_dvr.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_Misure_rev3_20260623_misure.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_Multipiano_rev3_20260623_pee.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_Legenda_vuota_rev3_20260623_pee.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_Duplicati_rev3_20260623_pee.pdf`
- `/tmp/planimetrie_2c_downloads/CP2C_Principali_rev3_20260623_pee.pdf`
- `/tmp/planimetrie_2c_downloads/Officina_Demo_S_r_l_rev1_20260623_pee.pdf`

## Test eseguiti

| Test | Esito | Evidenza |
| --- | --- | --- |
| `pwd` | PASS | `/Users/francescobellomo/Desktop/planimetrie_rapide_v4` |
| `git status` iniziale | PASS | working tree pulito |
| `git log --oneline -5` | PASS | ultimo commit iniziale `ccb8ff5 Checkpoint 2B cartiglio PDF professionale` |
| Lettura file richiesti | PASS | letti audit, report 2A/2B, piano fase 2, Product Audit, `index.html`, `sw.js` |
| Sintassi `index.html` | PASS | script inline estratto e validato con `node --check` |
| Sintassi `sw.js` | PASS | `node --check sw.js` |
| Apertura app browser | PASS | `ready=complete`, titolo `Planimetrie Rapide`, jsPDF disponibile |
| Apertura demo V4.1 | PASS | `demo_officina_v4_1.json` caricato e poi esportato offline |
| PEE con un solo simbolo | PASS | `CP2C_PEE_A4_breve...pdf`, 1 pagina, legenda in pagina principale |
| PEE con simboli duplicati | PASS | `CP2C_Duplicati...pdf`, `Estintore`, `Uscita di emergenza`, `Voi siete qui` presenti una sola volta in legenda |
| PEE con simboli principali | PASS | `CP2C_Principali...pdf`, pagina legenda aggiuntiva con label professionali |
| PEE con piu' di 10 voci | PASS | `CP2C_PEE_A4_lunga...pdf`, legenda su pagina dedicata |
| Legenda vuota | PASS | `CP2C_Legenda_vuota...pdf`, nessun titolo legenda e nessuna pagina legenda vuota |
| A4 verticale | PASS | `CP2C_PEE_A4_breve...pdf`, 210 x 297 mm |
| A4 orizzontale | PASS | `CP2C_PEE_A4_lunga...pdf`, 297 x 210 mm |
| A3 verticale | PASS | `CP2C_PEE_A3_verticale...pdf`, 297 x 420 mm |
| A3 orizzontale | PASS | `CP2C_PEE_A3_breve...pdf`, 420 x 297 mm |
| Cartiglio checkpoint 2B | PASS | PDF estratti contengono `Ragione sociale`, `Indirizzo sede`, `Formato`, `Scala` |
| Assenza sovrapposizioni | PASS | layout riserva spazio legenda/cartiglio; PDF generati senza contenuti fuori pagina rilevati testualmente |
| Pagina legenda aggiuntiva | PASS | PDF lunghi e complessi hanno pagina `Legenda` dedicata |
| Legenda multipagina | PASS parziale | logica implementata con paginazione; i casi testati non hanno superato una pagina legenda per piano |
| Export completo | PASS | `CP2C_Completo_complesso...pdf`, 3 pagine |
| Export DVR | PASS | `CP2C_DVR...pdf`, 2 pagine |
| Export misurazioni | PASS | `CP2C_Misure...pdf`, 3 pagine con tabella misure |
| Export multipiano | PASS | `CP2C_Multipiano...pdf`, 4 pagine |
| Salvataggio JSON | PASS | salvato `/tmp/planimetrie_2c_downloads/CP2C_PEE_A4_lunga_rev3_20260623.json` |
| Riapertura JSON | PASS | JSON salvato riaperto correttamente |
| Apertura vecchio JSON V4.1 | PASS | demo aperta correttamente |
| Autosave | PASS | reload con progetto ripristinato |
| Offline dopo SW | PASS | server spento, app caricata da SW, jsPDF disponibile, PDF esportato |
| Nessuna richiesta remota app | PASS | `rg` non trova URL remoti in `index.html`, `sw.js`, `manifest.webmanifest` |
| Nessun riferimento Sicuro360 | PASS | assente nei file cercati e nei PDF estratti |
| Nessuna modifica schema JSON | PASS | diff non tocca `state`, serializzazione, normalizzazione, IndexedDB o localStorage |

## Problemi grafici residui

- La verifica visiva e' stata eseguita tramite PDF generati ed estrazione testo/dimensioni pagina; non e' stata eseguita una rasterizzazione pixel-perfect pagina per pagina.
- I log nativi di Chrome headless includono errori macOS/Chrome non applicativi su display/updater/GCM; non risultano errori JavaScript applicativi bloccanti.
- La paginazione multipagina della legenda e' implementata, ma non e' stata forzata da un caso reale con centinaia di voci perche' il catalogo corrente non genera abbastanza categorie distinte in un singolo piano.

## Retrocompatibilita'

Confermato:

- progetti V4.1 apribili;
- demo esistente apribile;
- salvataggio e riapertura JSON invariati;
- autosave invariato;
- export PEE, completo, DVR e misurazioni funzionanti;
- cartiglio 2B conservato;
- funzionamento offline conservato;
- nessun nuovo campo persistente;
- nessuna modifica a schema JSON, canvas engine, IndexedDB o localStorage;
- nessun `symId` modificato.

## Esito complessivo

PASS.

Checkpoint 2C completato. Non e' stato iniziato il checkpoint 2D e non sono stati eseguiti commit, push o pubblicazioni.
