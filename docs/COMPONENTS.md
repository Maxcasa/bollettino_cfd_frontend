# Components Reference

[← Back to README](README.md) · [Architecture](ARCHITECTURE.md) · [API Services](API_SERVICES.md) · [Installation](INSTALLATION.md) · [User Guide](USER_GUIDE.md)

> **Audience:** Frontend developers maintaining, extending, or reviewing the Vue component layer.

Detailed documentation for every Vue component and utility in the project. Each entry describes props, emitted events, internal reactive state, lifecycle behaviour, and notable design decisions.

---

## Table of Contents

1. [App.vue — Root Shell](#appvue--root-shell)
2. [Views](#views)
   - [LoginView.vue](#loginviewvue)
   - [BaseView.vue](#baseviewvue)
   - [MeteoSelectView.vue](#meteoselectviewvue)
   - [MeteoView.vue](#meteoviewvue)
   - [CFDSelectView.vue](#cfdselectviewvue)
   - [CfdView.vue](#cfdviewvue)
   - [PrintViewM.vue](#printviewmvue)
   - [PrintViewC.vue](#printviewcvue)
   - [StatsView.vue](#statsviewvue)
   - [TabellaTemporali.vue](#tabellatemporalivue)
3. [CFD Card Sub-Components](#cfd-card-sub-components)
   - [SimpleCfdCard.vue](#simplecfdcardvue)
   - [IdroCfdCard.vue](#idrocfdcardvue)
   - [IdroGeoCfdCard.vue](#idrogeocfdcardvue)
   - [TendenzaCfdCard.vue](#tendenzacfdcardvue)
4. [Map Components](#map-components)
   - [CfdMapComponent.vue](#cfdmapcomponentvue)
   - [MeteoMapComponent.vue](#meteomapcomponentvue)
   - [TablePioggie.vue](#tablepioggevue)
5. [Shared Components](#shared-components)
   - [RevisioneModal.vue](#revisionemodalvue)
6. [Utilities](#utilities)
   - [CommonFunction.js](#commonfunctionjs)

---

## App.vue — Root Shell

**Path:** `src/App.vue`

The top-level component. Implements the application shell: header navigation bar, collapsible sidebar (*aside*), and the main content area that switches between views.

### Reactive state (key refs)

| Ref / variable | Type | Description |
|---------------|------|-------------|
| `logged` | `boolean` | Whether the user is authenticated |
| `content_page` | `number` | Controls which view is rendered (see routing table in ARCHITECTURE.md) |
| `isAsideVisible` | `boolean` | Whether the sidebar is expanded |

### Navigation buttons

| Button | Sets `content_page` to |
|--------|----------------------|
| Base | `1` |
| Meteo | `10` |
| CFD | `20` |

### Key methods

| Method | Description |
|--------|-------------|
| `showAside()` | Toggles sidebar visibility |
| `renewAuthorization()` | Calls `AppService.renewAuthorization` to refresh JWT |
| `openInNewWindow(url)` | Opens a reference table/document in a new browser tab |

---

## Views

### LoginView.vue

**Path:** `src/views/LoginView.vue`  
**Route:** Shown when `content_page = 0` (pre-auth)

Handles authentication. On mount it fetches `public/config.json` to obtain server URLs and stores them in `localStorage`.

#### Key refs

| Ref | Description |
|-----|-------------|
| `username` | Bound to username input |
| `password` | Bound to password input |
| `loading` | Disables the login button during the API call |
| `errorMessage` | Displayed on credential failure |
| `isDevelopment` | Shows a "TEST VERSION" banner in dev environments |

#### Login flow

1. Fetches `public/config.json`.
2. Stores `LOGIN_URL`, `SERVER_URL`, `WS_URL` in `localStorage`.
3. `POST LOGIN_URL/login` with `FormData { username, password }`.
4. On success: stores token and user, emits navigation signal to `App.vue`.
5. On 401: shows error message.

---

### BaseView.vue

**Path:** `src/views/BaseView.vue`  
**Route:** `/base`, `/` — shown at `content_page = 1`

Simple landing page displaying the ARPAV logo. No reactive logic.

---

### MeteoSelectView.vue

**Path:** `src/views/MeteoSelectView.vue`  
**Route:** Shown at `content_page = 10`

Bulletin selector for the Meteo module.

#### Features

- Date/time inputs for `data_validita` (validity datetime) with format validation (`dd/mm/yyyy` and `hh:mm`).
- Tabular list of existing meteorological bulletins colour-coded by date proximity.
- Click a row → emits or sets `content_page = 11` to open `MeteoView` with the selected bulletin.
- "New bulletin" action creates a bulletin for the chosen validity datetime.

#### Table columns

| Column | Description |
|--------|-------------|
| Numero | Sequential bulletin number |
| Data validità | Validity date (formatted) |
| Data compilazione | Compilation date |
| Tipo | "AVVISO METEO" or "BOLLETTINO METEO" |
| Stato | Validation state label from `CommonFunctions.label_bollettino` |

---

### MeteoView.vue

**Path:** `src/views/MeteoView.vue`  
**Route:** `/meteo`, shown at `content_page = 11`

Main meteo bulletin editor.

#### Props / injected state

Receives the selected `id_bollettino_meteo` from the parent.

#### Tabs

| Tab | Content |
|-----|---------|
| OGGI | Today's meteorological grid |
| DOMANI | Tomorrow's meteorological grid |
| DISAGIO FISICO | Physical discomfort zones |
| NOTE e TENDENZA | Free-text notes and trend section |

#### Scrolling grid

The meteo grid is rendered as a split layout:
- **Left sidebar** (`table-sidebar`): fixed column with phenomenon/altitude-band labels.
- **Right content** (`table-content-section`): horizontally scrollable columns, one per alert area.

Scrolling is synchronised between the two panels via the `handleScroll` event handler.

#### Action buttons

| Button | Available when | Action |
|--------|---------------|--------|
| SALVA | `validazione ≤ 1` | Save as draft (`validazione = 2`) |
| MODIFICA | `validazione = 2` | Re-open for editing (`validazione = 1`) |
| PRONTO per Pubblicare | `validazione < 4` | Mark ready for validation |
| AGGIORNA | `validazione = 4 or 6` | Open `RevisioneModal` to start a revision |
| VALIDA | `validazione = 5` | Re-validate after revision |
| COPIA da Boll. di IERI | `validazione ≤ 1`, period = TODAY | Copy data from yesterday's bulletin |
| COPIA da OGGI | `validazione ≤ 1`, period = TOMORROW | Copy today's data to tomorrow |

---

### CFDSelectView.vue

**Path:** `src/views/CFDSelectView.vue`  
**Route:** Shown at `content_page = 20`

Similar to `MeteoSelectView` but for CFD bulletins. Shows two parallel tables:

| Left table | Right table |
|-----------|-------------|
| Bollettini di Vigilanza Meteo (reference) | Bollettini di Allerta (CFD) |

Operator selects one meteo bulletin and one CFD bulletin to link them, then opens the editor.

---

### CfdView.vue

**Path:** `src/views/CfdView.vue`  
**Route:** `/cfd`, shown at `content_page = 21`

Main CFD bulletin editor.

#### Tabs (Risk types)

Rendered dynamically for each `rischio.tipo_rischio ≤ 0` from the risks catalogue. Each tab displays a different risk sub-component.

#### Dynamic card rendering

The component calls `getComponent(rischio)` to determine which card component to render:

```js
// pseudo-code
switch (rischio.tipo_rischio) {
  case  0: return SimpleCfdCard
  case  1: return IdroCfdCard
  case  2: return IdroGeoCfdCard
  case -1: return TendenzaCfdCard
}
```

#### Action buttons

| Button | Available when | Action |
|--------|---------------|--------|
| SALVA | `validazione ≤ 1` | Save draft |
| MODIFICA | `validazione > 1` | Re-open for editing |
| VALIDA | `validazione < 4` | Validate bulletin |
| AGGIORNA | `validazione = 4 or 6` | Open revision modal |
| COPIA OGGI SU DOMANI | `validazione ≤ 1 or = 5` | Copy today's alert levels to tomorrow |

#### Events received from cards

| Event | Data | Action |
|-------|------|--------|
| `cambia_attivo` | `{ id_rischio, attivo }` | Toggles a risk type active/inactive |

---

### PrintViewM.vue

**Path:** `src/views/PrintViewM.vue`  
**Route:** `/stampameteo`

Opened in a new browser window. Allows printing of a selected meteo bulletin.

- Dropdown to select bulletin from `getBollettiniMeteo`.
- **Stampa Orizzontale**: horizontal page orientation.
- **Stampa Verticale**: vertical page orientation.
- Base64 PDF data decoded and downloaded via `b64toBlob` + `file-saver`.

---

### PrintViewC.vue

**Path:** `src/views/PrintViewC.vue`  
**Route:** `/stampacfd`

Opened in a new browser window. Allows printing of a selected CFD bulletin.

- Dropdown to select from `getBollettiniCFD`.
- **Stampa** button triggers `PrintCfdService.getMeteoAllDati` and initiates PDF download.
- Bulletin state shown as one of six status strings.

---

### StatsView.vue

**Path:** `src/views/StatsView.vue`  
**Route:** `/dashboard`

Placeholder statistics dashboard with mock KPI cards (Users, Social Media, Traffic hours, Week Search). Intended for future implementation with real backend data.

---

### TabellaTemporali.vue

**Path:** `src/views/TabellaTemporali.vue`

Displays the thunderstorm reference table, likely embedded as an iframe or rendered inline. Used for operator reference during bulletin compilation.

---

## CFD Card Sub-Components

These components are rendered inside `CfdView` based on the active risk type. They all receive the same core props.

### Common Props

| Prop | Type | Description |
|------|------|-------------|
| `rischi` | `Array` | Filtered list of risks for this card |
| `bollettino_cfd` | `Object` | Current CFD bulletin (including `validazione` state) |
| `stati_suolo` | `Array` | Available soil-state snapshots |
| `livelli_sezioni` | `Array` | Available section-level snapshots |
| `note_rischio` | `String/Object` | Risk notes for current period |
| `meteoKey` | `Object` | Keyed meteo alert data (used to re-render maps) |
| `cfdKey` | `Object` | Keyed CFD alert data |
| `change_style` | `Object` | Style overrides for map zones |
| `socket_msg` | `Any` | Real-time socket message |

### Common Emits

| Event | Payload | When emitted |
|-------|---------|-------------|
| `cambia_attivo` | `{ id_rischio, attivo }` | When operator toggles a risk on/off |

---

### SimpleCfdCard.vue

**Path:** `src/views/CfdCards/SimpleCfdCard.vue`  
**Used for:** `tipo_rischio = 0` (general meteorological risks)

Renders a two-tab card (Oggi / Domani) with side-by-side maps:
- Left: `MeteoMapComponent` (colour from meteo bulletin)
- Right: `CfdMapComponent` (colour from CFD bulletin)
- Text area for risk notes
- Optional "Attivo" checkbox (if `descrizione_agg` is set)
- COPIA button to copy alerts from today to tomorrow

**Special handling for `id_rischio = 15` (Valanghe):** Shows an "AGGIORNA" button to trigger avalanche data update.

---

### IdroCfdCard.vue

**Path:** `src/views/CfdCards/IdroCfdCard.vue`  
**Used for:** `tipo_rischio = 1` (hydrological risks)

Extends `SimpleCfdCard` with:
- **Stato Suolo** selector (`id_rischio = 7`): dropdown to select soil-state snapshot + "CARICA NUOVO" button.
- **Livello Sezioni** selector (`id_rischio = 17`): dropdown to select section-level snapshot + "CARICA NUOVO" button.
- **Run Modellistica** display (`id_rischio = 18`): shows the date of the last hydrological model run.

---

### IdroGeoCfdCard.vue

**Path:** `src/views/CfdCards/IdroGeoCfdCard.vue`  
**Used for:** `tipo_rischio = 2` (hydro-geological risks)

Combines hydrological and geological risk display. Layout and functionality similar to `IdroCfdCard` with additional geological risk-specific UI elements.

---

### TendenzaCfdCard.vue

**Path:** `src/views/CfdCards/TendenzaCfdCard.vue`  
**Used for:** `tipo_rischio = -1` (tendenza / trend)

Renders the multi-day tendency section of the CFD bulletin. No maps shown; uses a rich-text or structured input for free-text trend descriptions.

---

## Map Components

### CfdMapComponent.vue

**Path:** `src/views/MapComponents/CfdMapComponent.vue`

Leaflet map showing CFD alert zones coloured by alert level.

#### Props

| Prop | Type | Description |
|------|------|-------------|
| `rischio` | `Object` | Current risk (`{ id_rischio, descrizione_rischio }`) |
| `bollettino_cfd` | `Object` | Bulletin (for `validazione` state check) |
| `periodo` | `number` | `1` = today, `2` = tomorrow |
| `key_val` | `Any` | Reactive key to force map refresh |
| `change_style` | `Object` | Style overrides from parent |
| `disabled` | `boolean` | When `true`, zone clicks are ignored |

#### Lifecycle

- **`onMounted`**: Calls `LeafletService.createMap(id_rischio, 'CFD', ...)` with a style function and an `onEachFeature` callback that registers click handlers.
- **`watch(key_val)`**: When the key changes, calls `LeafletService.updateMap(...)` to repaint zone colours.
- **`onUnmounted`**: Removes the Leaflet map to prevent memory leaks.

#### Centre coordinates

Default centre: `[45.62, 11.93]` (Veneto region, Italy), zoom level `8`.

---

### MeteoMapComponent.vue

**Path:** `src/views/MapComponents/MeteoMapComponent.vue`

Leaflet map showing meteorological alert zones coloured by meteo intensity level.

#### Props

| Prop | Type | Description |
|------|------|-------------|
| `rischio` | `Object` | Current risk |
| `periodo` | `number` | Period index |
| `key_val` | `Any` | Reactive key for refresh |

Read-only (no click interactions for data entry).

---

### TablePioggie.vue

**Path:** `src/views/MapComponents/TablePioggie.vue`

Inline table component showing precipitation data (tabelle piogge) for the active bulletin period. Fetches and displays threshold comparison data.

---

## Shared Components

### RevisioneModal.vue

**Path:** `src/components/RevisioneModal.vue`

A modal dialog for selecting a new validity date/time when revising a bulletin (state `4 → 5 → 6`).

#### Props

| Prop | Type | Description |
|------|------|-------------|
| `bollettino` | `Object` | The bulletin being revised (reads `data_validita`) |

#### Emits

| Event | Payload | Description |
|-------|---------|-------------|
| `cancel` | — | Modal dismissed without action |
| `revisiona` | `number` (epoch ms) | New validity timestamp confirmed |

#### Behaviour

On mount, pre-fills date and time inputs with the current `bollettino.data_validita`. On confirm, constructs a new `Date` from the inputs and emits the epoch millisecond value.

---

## Utilities

### CommonFunction.js

**Path:** `src/utils/CommonFunction.js`

Pure-function helper module. Import with:

```js
import * as CommonFunctions from '/src/utils/CommonFunction.js';
```

#### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `data2Str` | `(date: Date) → string` | Formats a `Date` as `YYYY-MM-DD HH:mm:ss` |
| `stessa_data_ora` | `(ts1, ts2) → boolean` | Compares two timestamps for same date+time (minute precision) |
| `label_bollettino` | `(bollettino) → string` | Returns human-readable state label + CFD flag |
| `format_data` | `(timestamp) → string` | Formats a timestamp as `DD/MM/YYYY` for display |
| `format_ora` | `(timestamp) → string` | Formats a timestamp as `HH:mm` for display |

#### `label_bollettino` Output Values

| `validazione` | Output |
|--------------|--------|
| `0` or `1` or `5` | `IN CORSO DI COMPILAZIONE` |
| `2` | `IN BOZZA` |
| `4` or `6` | `VALIDATO` |
| other | `ERRORE` |
| + `per_allerta = true` | appends ` - PER ALLERTA CFD` |

---

[↑ Back to top](#components-reference) · [← Back to README](README.md)
