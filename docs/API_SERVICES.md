# API Services Reference

[← Back to README](README.md) · [Architecture](ARCHITECTURE.md) · [Installation](INSTALLATION.md) · [Components](COMPONENTS.md) · [User Guide](USER_GUIDE.md)

> **Audience:** Developers integrating, testing, or extending the backend that this frontend consumes.

This document catalogues every HTTP endpoint and WebSocket event consumed by the frontend, grouped by the service class that calls them. All REST calls are made with **Axios**; base URLs are resolved at runtime from `localStorage` (see [ARCHITECTURE.md — Base URL Resolution](ARCHITECTURE.md#service-layer)).

> **Authentication model:** Every request carries the JWT as an `authorization` query parameter rather than an HTTP `Authorization` header. This is a backend legacy convention maintained for compatibility. Write operations additionally include a `win_token` query parameter (sourced from `ShareService`) to prevent concurrent-window overwrite conflicts.

---

## Table of Contents

1. [Authentication (AppService)](#authentication-appservice)
2. [Meteo Bulletins (MeteoService)](#meteo-bulletins-meteoservice)
3. [CFD Bulletins (CfdService)](#cfd-bulletins-cfdservice)
4. [Print — Meteo (PrintMeteoService)](#print--meteo-printmeteoservice)
5. [Print — CFD (PrintCfdService)](#print--cfd-printcfdservice)
6. [Authentication Headers](#authentication-headers)
7. [Error Handling Conventions](#error-handling-conventions)

---

## Authentication (AppService)

`src/services/AppService.js`

### Login

> Performed directly by `LoginView.vue` via `axios.post`, not through `AppService`.

| | |
|---|---|
| **Endpoint** | `POST LOGIN_URL/login` |
| **Body** | `FormData { username, password }` |
| **Response** | `{ token: string, user: object }` |
| **Side-effects** | Stores `auth_token`, `user`, `db_server`, `login_server`, `ws_server` in `localStorage` |

---

### Renew Token

| | |
|---|---|
| **Method** | `AppService.renewAuthorization(token)` |
| **Endpoint** | `GET SERVER_URL/get/new/token` |
| **Query params** | `authorization` (current JWT) |
| **Response** | `{ token: string }` |
| **Usage** | Triggered when the operator clicks the token-timer in the nav bar |

---

### Upload Profile Picture

| | |
|---|---|
| **Method** | `AppService.uploadProfilePic(formData, json_settings)` |
| **Endpoint** | `POST SERVER_URL/user/upload` |
| **Body** | `FormData` (multipart) |

---

## Meteo Bulletins (MeteoService)

`src/services/MeteoService.js`

All methods accept `token` as their first argument and send it as the `authorization` query parameter.

### Reference Data (read-only)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `getAreaMeteo(token)` | `GET /meteo/get_area_meteo` | Alert areas used in the meteo grid |
| `getFenomenoMeteo(token)` | `GET /meteo/get_fenomeno_meteo` | Atmospheric phenomena catalogue |
| `getFasciaAltimetricaFenomenoMeteo(token)` | `GET /meteo/get_fascia_altimetrica_fenomeno_meteo` | Altitude-band / phenomenon associations |
| `getAssAreeFenomeni(token)` | `GET /meteo/get_ass_aree_fenomeni` | Area–phenomenon associations |
| `getAssScaleFenomeni(token)` | `GET /meteo/get_ass_scale_fenomeni` | Scale–phenomenon associations |
| `getFrasiPredefinite(token)` | `GET /meteo/get_frasi_predefinite` | Predefined text phrases for notes |
| `getZoneDisagio(token)` | `GET /meteo/get_zona_disagio` | Physical discomfort zones |
| `getAssExtraRischio(token)` | `GET /meteo/get_ass_extra_rischio` | Extra risk associations |

---

### Bulletin CRUD

| Method | Endpoint | Description |
|--------|----------|-------------|
| `getBollettiniMeteo(token, search)` | `POST /meteo/get_bollettino` | List bulletins matching search criteria |
| `getBollettinoMeteo(token, id_bollettino)` | `GET /meteo/get_bollettino` | Fetch a single bulletin header |
| `getDatoBollettinoMeteo(token, id, id_periodo)` | `POST /meteo/get_dato_bollettino` | Fetch grid data for a bulletin/period |
| `getExtraDato(token, id_bollettino, id_rischio)` | `POST /meteo/get_dato_extra_rischio` | Fetch extra risk data |
| `getNoteFenomeno(token, id_bollettino, id_periodo)` | `POST /meteo/get_note_fenomeno` | Fetch phenomenon notes |
| `saveBollettinoMeteo(token, bollettino)` | `POST /meteo/set_bollettino` | Create or update a bulletin header |
| `saveDatoBollettino(token, dati_json)` | `POST /meteo/set_dato_bollettino` | Save a grid cell value |
| `saveDatoExtraRischio(token, dati_json)` | `POST /meteo/set_dato_extra_rischio` | Save extra risk value |
| `saveNoteFenomeno(token, dati_json)` | `POST /meteo/set_note_fenomeno` | Save phenomenon notes |
| `saveAvvisoBollettino(token, dati_json)` | `POST /meteo/set_avviso` | Set/clear the "avviso meteo" flag |
| `saveTendenzaBollettino(token, dati_json)` | `POST /meteo/set_tendenza` | Save tendency section |
| `saveDescrizioneMeteo(token, dati_json)` | `POST /meteo/set_descrizione_meteo` | Save bulletin description |
| `saveTendenzaDisagio(token, dati_json)` | `POST /meteo/set_tendenza_disagio` | Save physical discomfort tendency |

---

### Precipitation Data

| Method | Endpoint | Description |
|--------|----------|-------------|
| `getSogliePluvio(token)` | `GET /meteo/get_soglie_pluvio?last=1` | Latest pluviometric thresholds |
| `updateTabellePioggia(token, data_validita)` | `GET /meteo/import_tabelle_pioggia` | Trigger import of rain tables |

---

### Request/Response Format

- **GET** requests: all parameters passed as query string (`params` in Axios).
- **POST** requests: body is `FormData` with a single field `dati_json` containing JSON-serialised payload, **plus** `authorization` (and optionally `win_token`) as query parameters.
- Timestamps (`data_compilazione`, `data_validita`) are Unix seconds (divided by 1000 before sending).

---

## CFD Bulletins (CfdService)

`src/services/CfdService.js`

### Reference Data (read-only)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `getZoneAllerta(token)` | `GET /cfd/get_zone_allerta` | Alert zone definitions |
| `getLivelliAllerta(token)` | `GET /cfd/get_livelli_allerta` | Alert level catalogue |
| `getRischi(token)` | `GET /cfd/get_rischi` | Risk type catalogue |
| `getGeojsonZoneAllerta(token)` | `GET /cfd/get_geojson_zone_allerta` | GeoJSON FeatureCollection of alert zones |

---

### Alert Level Calculation

| Method | Endpoint | Description |
|--------|----------|-------------|
| `getCalcoloAllerta(token, id_boll_meteo, id_boll_cfd, mode, id_periodo, id_rischio, json_params)` | `POST /cfd/get_livello_allerta_calcolo` | Server-side calculation of alert levels; optional extra params via `FormData` |

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id_bollettino_meteo` | int | Associated meteo bulletin ID |
| `id_bollettino_cfd` | int | CFD bulletin ID |
| `mode` | int `0\|1` | Calculation mode |
| `id_periodo` | int\|null | Period index (today/tomorrow) |
| `id_rischio` | int\|null | Risk type filter |
| `extra_params` | bool | Set to `true` when `json_params` is provided |

---

### Bulletin CRUD

| Method | Endpoint | Description |
|--------|----------|-------------|
| `getBollettiniCFD(token, search)` | `POST /cfd/get_bollettino` | List CFD bulletins |
| `getBollettiniCFDAll(token, search)` | `POST /cfd/get_bollettino` | List all CFD bulletins |
| `getBollettinoCFD(token, id)` | `GET /cfd/get_bollettino` | Fetch a single CFD bulletin header |
| `getDatoBollettinoCFD(token, id, id_periodo)` | `POST /cfd/get_dato_bollettino` | Fetch alert-level data for a bulletin/period |
| `saveBollettinoCFD(token, bollettino)` | `POST /cfd/set_bollettino` | Create or update a CFD bulletin header |
| `saveDatoBollettino(token, dati_json)` | `POST /cfd/set_dato_bollettino` | Save a single alert-zone/risk/period value |
| `saveMultiDatoBollettino(token, dati_json)` | `POST /cfd/set_multi_dato_bollettino` | Bulk-save multiple alert values |
| `deleteDatoBollettino(token, dati_json)` | `POST /cfd/delete_dato_bollettino` | Delete alert-zone data |
| `updateZonaAllerta(token, dati_json)` | `POST /cfd/set_zona_allerta` | Update alert zone properties |

---

### Soil State & Section Levels

| Method | Endpoint | Description |
|--------|----------|-------------|
| `getStatoSuolo(token, search)` | `GET /cfd/get_stato_suolo` | List soil-state snapshots |
| `updateStatoSuolo(token, data_validita)` | `GET /cfd/import_stato_suolo` | Trigger import of soil state |
| `getLivelloSezioni(token, search)` | `GET /cfd/get_livello_sezioni` | List section-level snapshots |
| `updateLivelloSezioni(token, data_validita)` | `GET /cfd/import_livello_sezioni` | Trigger import of section levels |
| `getRunModellistica(token, data_validita, validazione)` | `GET /cfd/get_run_modellistica` | Get hydrological model run info |

---

## Print — Meteo (PrintMeteoService)

`src/services/PrintMeteoService.js`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `getAreaMeteo(token)` | `GET /meteo/get_area_meteo` | Areas for print layout |
| `getFenomenoMeteo(token)` | `GET /meteo/get_fenomeno_meteo` | Phenomena for print layout |
| `getFasciaAltimetricaFenomenoMeteo(token)` | `GET /meteo/get_fascia_altimetrica_fenomeno_meteo` | Altitude bands for print layout |
| `getAssAreeFenomeni(token)` | `GET /meteo/get_ass_aree_fenomeni` | Area–phenomenon associations |
| `getAssScaleFenomeni(token)` | `GET /meteo/get_ass_scale_fenomeni` | Scale–phenomenon associations |
| `getBollettiniMeteo(token, search)` | `POST /meteo/get_bollettino` | Bulletins for selector dropdown |
| `getBollettinoMeteo(token, id)` | `GET /meteo/get_bollettino` | Fetch bulletin for print |
| `getDatoBollettinoMeteo(token, id, id_periodo)` | `POST /meteo/get_dato_bollettino` | Grid data for print |
| `getNoteFenomeno(token, id, id_periodo)` | `POST /meteo/get_note_fenomeno` | Notes for print |

---

## Print — CFD (PrintCfdService)

`src/services/PrintCfdService.js`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `getMeteoAllDati(token, id_bollettino)` | `GET /cfd/get_all_dati` | Full CFD bulletin data package for PDF generation |

The response contains a base64-encoded PDF (or structured data), decoded using `b64toBlob` in `PrintViewC.vue`.

---

## Authentication Headers

All service methods pass the JWT token as the **`authorization` query parameter** rather than an HTTP `Authorization` header. This is a backend legacy convention maintained for compatibility:

```
GET /cfd/get_rischi?authorization=<JWT>
```

Write operations additionally include `win_token` (from `ShareService.getWinToken()`), which identifies the browser window and prevents simultaneous edits from different sessions from silently overwriting each other.

---

## Error Handling Conventions

| HTTP Status | Frontend behaviour |
|-------------|-------------------|
| `200` | Normal processing |
| `401` Unauthorized | `isUnauthorized()` called → toastr error → redirect to login after 500 ms |
| `4xx / 5xx` | Toastr error message shown; caught in `try/catch` blocks in mutating service methods |
| Network error | Axios throws; component-level `catch` shows toastr error |

Read-only `GET` methods return Axios promises without wrapping them in `try/catch` — errors bubble to the calling component.

Write methods (save, delete, import) are `async` and wrap their calls in `try/catch`, re-throwing to allow the component to display specific error messages.

---

[↑ Back to top](#api-services-reference) · [← Back to README](README.md)
