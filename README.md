# ElixForms · AutocompletePlugin POC (GitHub Pages)

Repository statica per verificare che **ElixForms** riesca a chiamare, tramite il
proxy Anthesi, un endpoint JSON pubblicato su **GitHub Pages**.

## Contenuto

- `index.html`: pagina di controllo con verifica HTTP/JSON e filtro **locale**.
- `sedi.json`: 6 sedi **sintetiche**, nel formato `[{ "label": "...", "value": "..." }]`.
- `.nojekyll`: evita la trasformazione Jekyll dei file.

Nessun framework, nessun backend, nessun token nella repository.

## Pubblicazione

1. Su GitHub crea una repository **pubblica**: per esempio `elixforms-autocomplete-test`.
2. Carica **tutti i file** di questa cartella nella **radice** della repository
   (non caricare soltanto lo ZIP e non racchiuderli in una sottocartella).
3. Apri **Settings → Pages → Build and deployment**.
4. **Source: Deploy from a branch**; **Branch: main**; cartella **/(root)**; **Save**.
5. Apri `https://USERNAME.github.io/elixforms-autocomplete-test/`.
6. Verifica il JSON direttamente in
   `https://USERNAME.github.io/elixforms-autocomplete-test/sedi.json`.

Sostituisci `USERNAME` con il tuo username/organizzazione GitHub, e il nome
del repository con quello scelto.

**Alternativa più semplice per il routing Anthesi**: se la repository si chiama
esattamente `USERNAME.github.io` (e non hai già un sito utente), la pagina è
`https://USERNAME.github.io/` e il JSON è
`https://USERNAME.github.io/sedi.json` (senza il segmento aggiuntivo del repository).

## Prova diretta JSON

```text
GET https://USERNAME.github.io/elixforms-autocomplete-test/sedi.json?term=cardio
```

La risorsa dovrà rispondere con HTTP `200` e l'intero array, anche con
`?term=cardio`: **GitHub Pages NON esegue codice server-side** e non filtra per
`term`. È solo un *test di trasporto*, formato e routing.

## Configurazione ElixForms da provare

- Campo: `STRING`.
- Attributo: `ELIX_DISPLAY_ATTR - AutocompletePlugin`.
- Abilita: **Sì**.
- Configurazione specifica:

```json
{
  "minLength": 2,
  "delay": 400,
  "maxShowItems": 10
}
```

- Metodo HTTP: **GET**.
- Proxy interno: **No**.
- `x-api-key`: utilizzare **solo** la chiave rilasciata da Anthesi per
  autorizzare il loro proxy, se richiesta/abilitata. NON inserirla nella repo.
- `term`: il widget lo aggiunge automaticamente, come verificato nella
  prova di rete del POC.
- *Target URI*: va costruito con le regole di encoding del **remote proxy Anthesi**
  (host con `.` → `-`; percorso remoto secondo le sue regole).
  **Non** usare il solo nome del repository come se fosse l'hostname.

### Esempio solo per un sito Pages alla radice

Se l'URL pubblico verificato è:

```text
https://USERNAME.github.io/sedi.json
```

il *candidato* target URI (da verificare con la versione di proxy attiva) è:

```text
/eF/services/api/remote/USERNAME-github-io/sedi.json
```

Con un progetto Pages sotto `/NOME-REPO/sedi.json`, il path richiede una
codifica coerente con quella prevista dal proxy Anthesi; verificare la URI
effettivamente generata e la risposta in **Network** prima di procedere.

In `F12 → Network → Fetch/XHR`, digitare almeno due caratteri e verificare
l'eventuale chiamata e il suo status HTTP.

## Interpretazione degli esiti

- HTTP `200` + array JSON corretto: servizio raggiungibile attraverso il
  percorso usato.
- HTTP `404`: distinguere URL pubblico inesistente da mapping del proxy errato.
- HTTP `401/403`: autenticazione, autorizzazioni o API Anthesi non abilitate.
- Nessuna richiesta: controllare inizializzazione widget/attributo, console JS.
- HTTP `200` senza suggerimenti: controllare payload, configurazione e filtri.

## Sicurezza e limiti

- Questo progetto contiene esclusivamente dati dimostrativi.
- Le pagine GitHub Pages pubblicate sono raggiungibili via Internet.
- Non pubblicare estrazioni GRU, credenziali o token senza autorizzazione.
- Per la ricerca reale `GET /sedi?term=...` e la validazione del valore selezionato
  occorre un backend (es. un futuro servizio Go AOSP) capace di filtrare e
  verificare i dati, con opportuna autenticazione e autorizzazione.

Riferimenti:
- https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
- https://docs.elixforms.it/elixDocs/extended_attributes.jsp
