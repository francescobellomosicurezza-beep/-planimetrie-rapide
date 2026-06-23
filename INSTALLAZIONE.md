# Installazione locale

Planimetrie Rapide V4.1 e una web app statica. Non richiede build, backend, account o database remoto.

## Avvio consigliato

```bash
cd ~/Desktop/planimetrie_rapide_v4
python3 -m http.server 8080
```

Aprire:

```text
http://localhost:8080
```

Se la porta 8080 e gia occupata:

```bash
python3 -m http.server 8081
```

## Cache e aggiornamenti

Il service worker usa la cache `planimetrie-rapide-v4-1-20260618`. Dopo una modifica, se il browser mostra ancora una versione precedente:

1. aprire DevTools;
2. Application > Service Workers;
3. premere Unregister;
4. Application > Storage > Clear site data;
5. ricaricare con `Cmd/Ctrl + Shift + R`.

In alternativa usare una finestra anonima per un controllo rapido.

## Offline

L'app funziona offline dopo un primo caricamento completo via `http://localhost` o hosting statico. Aprendo direttamente `index.html` come file locale, il service worker non viene attivato.

La generazione PDF richiede che jsPDF sia gia disponibile nel browser o nella cache del service worker.
