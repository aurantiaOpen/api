# arancia api — unofficial documentation

> [!NOTE]
> documentazione ricavata tramite reverse engineering dell'APK Android (v3.3).
> tutti gli endpoint sono di sola lettura tranne `device/new` e `Login`.
> non esiste documentazione ufficiale pubblica.
> alcuni endpoint possono essere WIP perché non testati completamente.

---

## Autenticazione & Device ID

### Come funziona

ogni richiesta richiede un `{longstring}` nell'URL — un device ID che identifica il "dispositivo" presso il server.

Il device ID è hardcoded nell'app stessa. Basta fare spoofing con un proxy che intercetta le richiesta.
Questo ID è già registrato nel DB del server e funziona per tutte le richieste pubbliche (contenuti gratuiti). Non è legato a nessun utente reale.

Per registrare un nuovo device ID, vedere [device/new](#registrazione-device).

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

Restituisce nome e versione API. Non richiede device ID.

**Risposta:**
```json
["AranciaLive", "1.1"]
```

---

### Registrazione device

```
POST https://www.arancialive.com/api/app/1/{longstring}/device/new
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

Il campo `DEVICEID` è un secondo identificatore nel formato `I` + 64 caratteri hex uppercase (SHA-256 di bytes casuali). È distinto dal `{longstring}` nell'URL.

**Risposta:**
```json
{
  "result": "OK-1",
  "error": null
}
```

> [!NOTE]
> Se si usa il device ID hardcoded, questo step non è necessario.

---

### Lista eventi live

```
GET https://www.arancialive.com/api/app/1/{longstring}/live/list
```

Restituisce gli eventi in diretta in questo momento. Ritorna `[]` se non c'è nulla in onda, `null` se il device ID non è registrato.

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
GET https://www.arancialive.com/api/app/1/{longstring}/live/next
```

Restituisce il prossimo evento programmato.

---

### Catalogo on demand

```
GET https://www.arancialive.com/api/app/1/{longstring}/ondemandSuddivisi
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
GET https://www.arancialive.com/api/app/1/{longstring}/ondemand/list/{NPAGINA}
```

Versione flat in pagine non raggruppate del catalogo on demand.
Esempio (https://www.arancialive.com/api/app/1/{longstring}/ondemand/list/1 per pagina 1)

---

### Dettagli video di un evento

```
GET https://www.arancialive.com/api/app/1/{longstring}/ondemand/video/{IDEVENTO}/1
```

Restituisce i video associati a un evento (utile per eventi multi-episodio). Il `VideoUrl` è un link `.m3u8` (HLS) riproducibile direttamente.

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
GET https://www.arancialive.com/api/app/1/{longstring}/tariffa/
```

Restituisce le tariffe disponibili per i contenuti a pagamento (`IDTARIFFA != 0`).
esempio (/tariffa/9)

---

### Utenti online - WIP

```
GET https://www.arancialive.com/api/app/1/{longstring}/GetUtentiOnline/{IDVIDEO} o {IDEVENTO}
```

Numero di utenti connessi in questo momento all'evento live.

---

### Incrementa visualizzazioni

```
GET https://www.arancialive.com/api/app/1/{longstring}/IncrementaVisualizzazione/{IDVIDEO}/1
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
GET https://www.arancialive.com/api/app/1/{longstring}/Login/{email}/{password}/
```

**Risposta:**
```json
{
  "IDACCOUNT": 1,
  "Email": "utente@esempio.it",
  "Password": "latuapassword", // wtf is thisss
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

Il campo `Hash` è necessario per gli endpoint che richiedono autenticazione (acquisti, verifica acquisti).

---

### Verifica acquisto

```
GET https://www.arancialive.com/api/app/1/{longstring}/VerificaAcquisto/
GET https://www.arancialive.com/api/app/1/{longstring}/VerificaAcquistoToken
```

Verifica se l'utente loggato ha acquistato un contenuto a pagamento.

---

### Esegui pagamento

```
GET https://www.arancialive.com/api/app/1/{longstring}/EseguiPagamento
```

Avvia il flusso di acquisto. Richiede login.

---

### Verifica autorizzazione PPV - WIP

```
GET https://www.arancialive.com/api/app/1/{longstring}/isPPVAutorizzato/
```

Verifica se l'utente è autorizzato a vedere un contenuto Pay-Per-View.

---

### Email da device

```
GET https://www.arancialive.com/api/app/1/{longstring}/getEmailDaDevice/{DEVICEID}/
```

Ritorna l'email associata a un device ID. Risponde `[]` se il device non è associato a nessun account.

---

### Cancella account

```
GET https://www.arancialive.com/api/app/1/{longstring}/CancellaAccount/
```

Elimina l'account utente. Richiede login.

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

--
# DISCLAIMER
see (disclaimer.md)[./DISCLAIMER.MD]
