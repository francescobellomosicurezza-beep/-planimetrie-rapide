# Manuale rapido

## 1. Avvio

Avviare un server locale:

```bash
python3 -m http.server 8080
```

Aprire `http://localhost:8080`.

## 2. Nuovo progetto

Inserire il nome azienda/sopralluogo nella barra superiore. Dal menu aprire "Intestazione documento" e compilare azienda, indirizzo, revisione, redattore e qualifica.

## 3. Foto o scansione

Usare "Foto base" o "Carica foto base". L'immagine viene compressa nel browser e salvata nel JSON quando possibile. Regolare l'opacita dal menu.

## 4. Calibrazione

Dal menu scegliere "Calibra scala su foto", toccare due punti su una distanza nota e inserire la misura reale in metri. Finche il piano non e calibrato, l'app mostra "Scala indicativa".

## 5. Disegno struttura

Usare gli strumenti Muri, Muri REI/EI, Porte, Finestre, Vano scala, Ascensore e Aree/locali. Le aree possono rappresentare destinazioni d'uso come ufficio, magazzino, officina, laboratorio, locale tecnico o altro.

## 6. Selezione e modifica

La modalita iniziale e "Seleziona". Toccare un elemento per evidenziarlo, trascinarlo per spostarlo e aprire il pannello proprieta per correggere etichetta, misure, classe, note o tipologia. Dal pannello si puo duplicare o eliminare.

## 7. Emergenza

Selezionare il contesto Emergenza e usare "Elementi" per posizionare presidi, uscite, punto di raccolta, "Sei qui", primo soccorso e intercettazioni. Usare "Vie di esodo" per tracciare percorsi a segmenti.

## 8. Misurazioni

Selezionare il contesto Misurazioni. Inserire punti di misura; il codice progressivo usa il prefisso M. Dal pannello proprieta compilare tipo, valore, unita, altezza, durata, data, ora, strumento, operatore e note.

## 9. DVR

Selezionare il contesto DVR. Inserire macchine, attrezzature, impianti, postazioni, depositi, zone di rischio indicative, percorsi pedonali e percorsi mezzi. Le zone ATEX o di rischio sono rappresentazioni grafiche indicative da verificare a cura del professionista competente.

## 10. Salvataggio

L'autosalvataggio usa IndexedDB con fallback localStorage. Per archiviazione e scambio usare "Salva progetto (.json)" dal menu.

## 11. Esportazione

Usare "Esporta PDF" e scegliere:

- Planimetria di emergenza;
- Piano misurazioni;
- Layout DVR;
- Completo.

Il PNG esporta il piano attivo. I controlli pre-export sono avvisi professionali e non blocchi assoluti.

## 12. Recupero

Alla riapertura, l'app tenta di ripristinare l'ultimo autosalvataggio recente. Se lo spazio browser e insufficiente, salvare manualmente il file JSON.

## 13. Limiti professionali

L'app e una piattaforma locale di editing grafico. La planimetria prodotta puo essere usata professionalmente quando il redattore/committente ne assume contenuti, misure, simboli, classificazioni e conformita secondo incarico e normativa applicabile. La piattaforma non effettua validazioni tecniche o normative automatiche.
