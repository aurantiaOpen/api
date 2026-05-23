# arancia api — unofficial documentation

> [!NOTE]
> documentazione ricavata tramite reverse engineering dell'APK Android (v3.3) e del codice sorgente C# (`ApiRestClient.cs`).
> tutti gli endpoint sono di sola lettura tranne `device/new`, `EseguiPagamento`, `GeneraToken`, `AssociaEmail` e `CancellaAccount`.
> non esiste documentazione ufficiale pubblica.

---

## Autenticazione & Device ID / ApiKey

### Come funziona

ogni richiesta richiede una `{APIKEY}` nell'URL, una chiave hardcodata nell'app stessa che identifica il "canale" presso il server.

questo valore è già registrato nel DB del server e funziona per tutte le richieste pubbliche (contenuti gratuiti).

Per registrare un nuovo device ID, vedere [device/new](#registrazione-device).

> [!NOTE]
> Il campo `{IDCANALE}` è sempre `1`.

---

## Header comuni

```
Host: www.arancialive.com
Accept: */*
Accept-Language: it-IT,it;q=0.9
Connection: keep-alive
Accept-Encoding: gzip, deflate, br
User-Agent: AranciaLiveApp/19 CFNetwork/3826.600.41 Darwin/24.6.0
```

> [!WARNING]
> Il server fa redirect da `http://arancialive.com` a `https://www.arancialive.com`.
> Usa direttamente HTTPS con `www` per evitare il redirect.

---

## Endpoint

### Info app

```
GET https://www.arancialive.com/api/app
```

Restituisce nome e versione API. Non richiede ApiKey.

**Risposta:**
```json
["AranciaLive", "1.1"]
```

---

### Registrazione device

```
POST https://www.arancialive.com/api/app/1/{APIKEY}/device/new
```

Registra un nuovo device ID sul server. Va chiamato prima di qualsiasi altro endpoint se si usa un device ID nuovo.

**Body:**
```json
{
  "IDCANALE": 1,
  "DEVICEID": "I<SHA256_IN_UPPERCASE>",
  "TipoDispositivo": 0,
  "Stato": 0,
  "Versione": "3.3 (19)",
  "NotificationUrl": ""
}
```

Il campo `DEVICEID` è un secondo identificatore nel formato `I` + 64 caratteri hex uppercase (SHA-256 di bytes casuali). È distinto dalla `{APIKEY}` nell'URL.

**Risposta:**
```json
{
  "result": "OK-1",
  "error": null
}
```

> [!NOTE]
> Se si usa l'ApiKey hardcodata, questo step non è necessario.

---

### Lista eventi live

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/live/list
```

Restituisce gli eventi in diretta in questo momento. Ritorna `[]` se non c'è nulla in onda, `null` se l'ApiKey non è registrata.

**Risposta:**
```json
[
  {
    "liveinfo": {
      "IDEVENTO": 150,
      "IDCANALE": 1,
      "IDCATEGORIA": 2,
      "DataEvento": "2026-05-15T08:00:00",
      "IDTARIFFA": 0,
      "Stato": 1,
      "DurataMinuti": 180,
      "LimiteMassimoUtenti": 0,
      "ApiKey": "ooCAMP8IfrA0HczwfLITG1YAXwyyMcBB",
      "Nome": "Festa dei Ceri 2026",
      "Descrizione": "...",
      "CopertinaUrl": "/media/eventi/2026/..."
    }
  }
]
```

`Stato: 1` = evento attivo/in corso. `IDTARIFFA: 0` = gratuito.

---

### Prossimo evento live

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/live/next
```

Restituisce il prossimo evento programmato.

---

### Canale live

```
GET https://www.arancialive.com/admin/getUrlstreaming.ashx?https=si&debug=no&psw={PSW}
```

Restituisce l'm3u8 del canale live multi definizione.

> [!NOTE]
> La password è hardcodata nell'app. Sostituire `si`/`no` rispettivamente ai parametri `https` e `debug`.

---

### Server streaming evento live

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/server/{IDEVENTO}/{DEVICEID}/{ts}/{hs}/{opt}
```

Restituisce le info di streaming per un evento live specifico.

| Parametro | Descrizione |
|-----------|-------------|
| `IDEVENTO` | ID dell'evento |
| `DEVICEID` | ID del dispositivo |
| `ts` | Timestamp |
| `hs` | Hash di sicurezza |
| `opt` | Opzione (default: `A`) |

**Risposta:** `StreamingResponse` (struttura non ancora documentata completamente).

---

### Moltiplicatore

```
GET https://www.arancialive.com/admin/getmol.ashx
```

Restituisce un valore float (moltiplicatore). Il valore è sempre almeno `1.0`.

---

### Catalogo on demand

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/ondemandSuddivisi
```

Restituisce tutti gli eventi on demand raggruppati per categoria.

**Risposta:**
```json
[
  {
    "nomeCategoria": "In evidenza",
    "ListaEventiOndemand": [
      {
        "liveinfo": {
          "IDEVENTO": 138,
          "IDCANALE": 1,
          "IDCATEGORIA": 1,
          "DataEvento": "2025-05-15T08:00:00",
          "IDTARIFFA": 0,
          "Stato": 0,
          "DurataMinuti": 600,
          "ApiKey": "xgtNzIH4ImMHDv6LzCOe9aC4zZSob9ua",
          "Nome": "Festa dei Ceri 2025",
          "Descrizione": "...",
          "CopertinaUrl": "/media/eventi/2025/..."
        },
        "IDEVENTO": 138,
        "nVisualizzazioni": 29763,
        "nOrdineEvidenza": 130338,
        "LingueDisponibili": ""
      }
    ]
  }
]
```

---

### Lista on demand (flat), divisa in pagine

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/ondemand/list/{NPAGINA}
```

Versione flat in pagine non raggruppate del catalogo on demand.
Esempio: `/ondemand/list/1` per la pagina 1.

#### Filtro per categoria

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/ondemand/list/{IDCATEGORIA}/{NPAGINA}
```

Stessa struttura, filtra per categoria.

---

### Dettagli video di un evento

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/ondemand/video/{IDEVENTO}/{TIPOVIDEO}
```

Restituisce i video associati a un evento (utile per eventi multi-episodio). Il `VideoUrl` è un link `.m3u8` (HLS) riproducibile direttamente.

| Parametro | Descrizione |
|-----------|-------------|
| `IDEVENTO` | ID dell'evento |
| `TIPOVIDEO` | Tipo di video (es. `1`) |

**Risposta:**
```json
[
  {
    "IDVIDEO": 366,
    "IDEVENTO": 138,
    "TipoVideo": 1,
    "OrderNum": 100,
    "Stato": 1,
    "Durata": 120,
    "CopertinaUrl": "/ondemand2/2025/ceri2025-00-messa.jpg",
    "VideoUrl": "https://c10.arancialive.com/ondemand/messa2025/index.m3u8",
    "Data": "2025-05-15T08:00:00",
    "Nome": "Messa",
    "Descrizione": "",
    "VideoUrlADV": ""
  }
]
```

I `CopertinaUrl` relativi vanno prefissati con `https://www.arancialive.com`.

---

### Tariffa

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/tariffa/{IDTARIFFA}
```

Restituisce le tariffe disponibili per i contenuti a pagamento (`IDTARIFFA != 0`).
Esempio: `/tariffa/9`

---

### Utenti online

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/GetUtentiOnline/{IDEVENTO}
```

Restituisce il numero di utenti connessi in questo momento all'evento live.

---

### Registra utente online

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/UtenteOnlineInsert/{IDEVENTO}/{DEVICEID}
```

Segnala al server che un dispositivo sta guardando un evento live.

**Risposta:**
```json
{
  "result": "OK",
  "error": null
}
```

---

### Incrementa visualizzazioni

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/IncrementaVisualizzazione/{IDEVENTO}
```

Segnala al server che un video è stato visualizzato (per aggiornare il contatore `nVisualizzazioni`).

**Risposta:**
```json
{
  "result": "OK",
  "error": null
}
```

---

### Login

> [!CAUTION]
> **Endpoint estremamente insicuro.** Le credenziali viaggiano in chiaro nell'URL e quindi visibili nei log del server o nei proxy. Non usarlo su reti non fidate.

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/Login/{email}/{password}/
```

**Risposta:**
```json
{
  "IDACCOUNT": 1,
  "Email": "utente@esempio.it",
  "Password": "latuapassword", // perché no... XD
  "IDDEVICE": "",
  "IDDEVICE2": "",
  "IDDEVICE3": "",
  "NomeD1": "",
  "NomeD2": "",
  "NomeD3": "",
  "NomeCognome": "Nome Cognome",
  "DataRegistrazione": "2020-01-01T00:00:00",
  "Stato": 1,
  "Hash": "7JdS2wK9pL5mN1t"
}
```

Il campo `Hash` è necessario per gli endpoint che richiedono autenticazione (acquisti, verifica acquisti, PPV, cancellazione account).
Il campo `IDACCOUNT` è `ida` negli endpoint autenticati.

---

### Genera token web

```
POST https://www.arancialive.com/api/app/1/{APIKEY}/GeneraToken/{DEVICEID}
```

Genera un codice/token per associare il dispositivo a una sessione web.

**Body:** stringa vuota `""`

**Risposta:** stringa con il token generato.

---

### Associa email a device

```
POST https://www.arancialive.com/api/app/1/{APIKEY}/AssociaEmail/{DEVICEID}/{EMAIL}/
```

Associa un indirizzo email a un device ID.

**Body:** stringa vuota `""`

---

### Email da device

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/getEmailDaDevice/{DEVICEID}/
```

Ritorna l'email (account) associata a un device ID. Risponde `[]` se il device non è associato a nessun account.

---

### Verifica acquisto

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/VerificaAcquisto/{IDEVENTO}/{IDDEVICE}
GET https://www.arancialive.com/api/app/1/{APIKEY}/VerificaAcquistoToken/{IDEVENTO}/{IDDEVICE}
```

Verifica se l'utente loggato ha acquistato un contenuto a pagamento.

---

### Verifica autorizzazione PPV

```
GET https://www.arancialive.com/api/app/1/{APIKEY}/isPPVAutorizzato/{IDACCOUNT}/{HASHACCOUNT}/{IDEVENTO}/
```

Verifica se l'utente è autorizzato a vedere un contenuto Pay-Per-View.

| Parametro | Descrizione |
|-----------|-------------|
| `IDACCOUNT` | Campo `IDACCOUNT` dalla risposta Login |
| `HASHACCOUNT` | Campo `Hash` dalla risposta Login |
| `IDEVENTO` | ID dell'evento |

---

### Esegui pagamento

```
POST https://www.arancialive.com/api/app/1/{APIKEY}/EseguiPagamento
```

Avvia il flusso di acquisto. Richiede login. Body: `UtenteAutorizzatoRequest` (struttura non ancora documentata completamente).

---

### Cancella account

```
POST https://www.arancialive.com/api/app/1/{APIKEY}/CancellaAccount/{IDACCOUNT}/{HASHACCOUNT}
```

Elimina l'account utente. Richiede le credenziali ottenute dal Login.

**Body:** stringa vuota `""`

---

## URL media

Le copertine degli eventi hanno URL relativi nel campo `CopertinaUrl`:
```
https://www.arancialive.com{CopertinaUrl}
```

I video on demand sono stream HLS con URL assoluti nel campo `VideoUrl`:
```
https://c10.arancialive.com/ondemand/.../index.m3u8
```

Riproducibili con qualsiasi player HLS (hls.js, Video.js, VLC, ecc.).

---

## Note sui campi

| Campo | Descrizione |
|-------|-------------|
| `IDTARIFFA: 0` | Contenuto gratuito |
| `IDTARIFFA: 1+` | Contenuto a pagamento |
| `Stato: 0` | Evento terminato / non in onda |
| `Stato: 1` | Evento attivo / in diretta |
| `TimeToStart` | Secondi alla messa in onda (negativo = già iniziato) |
| `TimeToEnd` | Secondi alla fine (negativo = già finito) |
| `DurataMinuti` | Durata prevista in minuti |

---

# DISCLAIMER
see [disclaimer.md](./DISCLAIMER.md)
