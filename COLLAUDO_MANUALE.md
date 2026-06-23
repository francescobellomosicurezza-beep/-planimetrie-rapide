# Collaudo manuale V4.1

Usare `demo_officina_v4_1.json` per una prova rappresentativa.

## Avvio

1. Avviare `python3 -m http.server 8080` o porta libera.
2. Aprire l'app nel browser.
3. Verificare assenza di errori console.
4. Controllare che sia visibile V4.1.

## Funzioni principali

1. Creare un muro, selezionarlo, spostarlo, modificarne coordinate e cancellarlo.
2. Provare annulla/ripristina.
3. Creare porta e finestra, selezionarle e modificarle.
4. Creare un vano scala, collegarlo a un altro piano, modificarlo e cancellarlo.
5. Creare un ascensore, cambiare tipologia, ridimensionarlo e cancellarlo.
6. Creare un'area, assegnare destinazione d'uso e note.
7. Creare una nota/testo, ruotarla e modificarla.
8. Creare un presidio, duplicarlo e modificarlo.
9. Creare una via di esodo e modificarne proprieta.
10. Creare punti M1, M2 e M3, duplicare un punto e verificare che il codice non si sovrapponga involontariamente.
11. Creare macchina, zona di rischio e percorso mezzi.
12. Creare due piani, duplicare piano e verificare che scale/ascensori restino gestibili.

## Salvataggio

1. Verificare indicatore Salvataggio/Salvato.
2. Salvare JSON.
3. Ricaricare pagina e verificare recupero autosalvataggio.
4. Aprire il JSON salvato.
5. Aprire `demo_officina_v4_1.json`.

## Export

1. Esportare Planimetria di emergenza in A3.
2. Esportare Piano misurazioni.
3. Esportare Layout DVR.
4. Esportare Documento completo multipiano.
5. Esportare PNG del piano attivo.
6. Controllare legenda, intestazione, disclaimer, scala indicativa/calibrata, tabelle e pagine.

## Responsive

Controllare almeno:

- desktop 1440x900;
- laptop 1280x800;
- tablet;
- smartphone 390x844.

Su smartphone verificare toolbar scorrevole, pannelli dal basso, pulsanti grandi e chiusura pannelli.

## Offline/cache

1. Caricare l'app online via server locale.
2. Verificare registrazione service worker.
3. Disconnettere rete o usare DevTools Offline.
4. Ricaricare.
5. Se appare versione vecchia: unregister service worker, clear site data, hard reload.
