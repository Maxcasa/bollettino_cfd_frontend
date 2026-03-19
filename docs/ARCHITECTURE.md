# Architecture

[← Back to README](README.md) · [Installation](INSTALLATION.md) · [Components](COMPONENTS.md) · [API Services](API_SERVICES.md) · [User Guide](USER_GUIDE.md)

> **Audience:** Developers contributing to, maintaining, or integrating with this codebase.

This document describes the internal structure of the **Bollettino Multirischio** frontend: the overall system topology, application bootstrap sequence, routing strategy, component hierarchy, service layer design, state management approach, map infrastructure, real-time communication, authentication flow, print architecture, and styling conventions.

---

## Table of Contents

1. [High-Level Architecture](#high-level-architecture)
2. [Application Bootstrap](#application-bootstrap)
3. [Routing](#routing)
4. [Component Hierarchy](#component-hierarchy)
5. [Service Layer](#service-layer)
6. [State Management](#state-management)
7. [Map Architecture (LeafletService)](#map-architecture-leafletservice)
8. [Real-Time Communication](#real-time-communication)
9. [Authentication Flow](#authentication-flow)
10. [Print Architecture](#print-architecture)
11. [CSS & Styling Strategy](#css--styling-strategy)

---

## High-Level Architecture

The application is a Vue 3 SPA served as static files by nginx. All data is fetched from a separate Flask backend over HTTP (Axios) and WebSocket (Socket.IO). There is no server-side rendering.

```mermaid
graph TD
    subgraph Browser["🌐 Browser"]
        nginx["nginx\nStatic file server"]
        subgraph SPA["Vue 3 SPA"]
            App["App.vue\nShell · Navbar · Auth guard"]
            Views["Views\nLogin · Base · Meteo · CFD · Print"]
            Cards["CFD Risk Cards\nSimple · Idro · IdroGeo · Tendenza"]
            Maps["Map Components\nCfdMapComponent · MeteoMapComponent"]
            SvcLayer["Service Layer\nAppService · MeteoService · CfdService\nLeafletService · ShareService · PrintServices"]
        end
    end

    subgraph BackendNode["🖥️ Backend  (Flask)"]
        API["REST API\n/login · /meteo/* · /cfd/* · /user/*"]
        WSS["WebSocket\nSocket.IO"]
    end

    nginx -->|"serves index.html + assets"| SPA
    App --> Views
    Views --> Cards
    Cards --> Maps
    Views --> SvcLayer
    Cards --> SvcLayer
    SvcLayer -->|"HTTP · Axios"| API
    SvcLayer -->|"WebSocket"| WSS
```

---

## Application Bootstrap

`src/main.js` performs all global setup in order:

1. Creates the Vue application (`createApp(App)`).
2. Imports global CSS: UIkit, `dashboard.css`, `personal.css`.
3. Imports global JS: jQuery, UIkit JS + icons, jwt-decode, toastr.
4. Registers the `<font-awesome-icon>` component globally with the required icon subset.
5. Installs a global Vue error handler that logs component errors to the console.
6. Mounts the app on `<div id="app">` in `index.html`.

> **Design note:** The Vue Router instance defined in `src/router/index.js` is declared but intentionally **not registered** via `app.use(router)`. Navigation between main sections is managed in `App.vue` by toggling a reactive `content_page` integer, which keeps the entire application within a single URL. Vue Router is only used for the print views (`/stampameteo`, `/stampacfd`), which open in separate browser windows.

---

## Routing

Although `vue-router` is configured, only the print views are reachable via URL routing in the current implementation. Main navigation is driven by `content_page` in `App.vue`.

| Route | Component | Notes |
|-------|-----------|-------|
| `/` | `BaseView` | Landing / home |
| `/base` | `BaseView` | Alias |
| `/dashboard` | `StatsView` | Statistics (placeholder) |
| `/meteo` | `MeteoView` | Meteo bulletin editor |
| `/cfd` | `CfdView` | CFD bulletin editor |
| `/stampameteo` | `PrintViewM` | Meteo print — opens in new tab |
| `/stampacfd` | `PrintViewC` | CFD print — opens in new tab |
| `/*` | Redirect to `/` | 404 catch-all |

### `content_page` Switch Values (App.vue)

| Value | View shown |
|-------|-----------|
| `0` | `LoginView` |
| `1` | `BaseView` |
| `10` | `MeteoSelectView` (→ `MeteoView` after selecting a bulletin) |
| `11` | `MeteoView` |
| `20` | `CFDSelectView` (→ `CfdView` after selecting) |
| `21` | `CfdView` |

---

## Component Hierarchy

```mermaid
graph TD
    App["App.vue"]

    subgraph auth["Authentication"]
        LoginView["LoginView\ncontent_page = 0"]
    end

    subgraph landing["Landing"]
        BaseView["BaseView\ncontent_page = 1"]
    end

    subgraph meteo["Meteo Module"]
        MeteoSel["MeteoSelectView\ncontent_page = 10"]
        MeteoV["MeteoView\ncontent_page = 11"]
        MeteoMap["MeteoMapComponent\n(one per risk / period)"]
    end

    subgraph cfd["CFD Module"]
        CfdSel["CFDSelectView\ncontent_page = 20"]
        CfdV["CfdView\ncontent_page = 21"]
        RevModal["RevisioneModal\n(on AGGIORNA)"]
        Simple["SimpleCfdCard\ntipo_rischio = 0"]
        Idro["IdroCfdCard\ntipo_rischio = 1"]
        IdroGeo["IdroGeoCfdCard\ntipo_rischio = 2"]
        Tend["TendenzaCfdCard\ntipo_rischio = -1"]
        CfdMap["CfdMapComponent"]
        MeteoMap2["MeteoMapComponent"]
    end

    subgraph printmod["Print Module"]
        PrintM["PrintViewM\n/stampameteo"]
        PrintC["PrintViewC\n/stampacfd"]
    end

    App --> LoginView
    App --> BaseView
    App --> MeteoSel
    MeteoSel -->|selects bulletin| MeteoV
    MeteoV --> MeteoMap
    App --> CfdSel
    CfdSel -->|selects bulletins| CfdV
    CfdV --> RevModal
    CfdV -->|"dynamic :is"| Simple
    CfdV -->|"dynamic :is"| Idro
    CfdV -->|"dynamic :is"| IdroGeo
    CfdV -->|"dynamic :is"| Tend
    Simple --> CfdMap
    Simple --> MeteoMap2
    Idro --> CfdMap
    Idro --> MeteoMap2
    IdroGeo --> CfdMap
    IdroGeo --> MeteoMap2
    App --> PrintM
    App --> PrintC
```

### Dynamic Card Selection in CfdView

`CfdView` selects which card component to render based on the current `rischio.tipo_rischio` value:

| `tipo_rischio` | Component rendered |
|---------------|-------------------|
| `0` | `SimpleCfdCard` |
| `1` | `IdroCfdCard` |
| `2` | `IdroGeoCfdCard` |
| `-1` | `TendenzaCfdCard` |

---

## Service Layer

All services live in `src/services/`. They are plain JavaScript classes (not Vue composables) and are instantiated directly inside components with `new ServiceName()`.

### `service.js` — Base URL Resolution

```js
const SERVER_URL = localStorage.getItem("db_server");
const LOGIN_URL  = localStorage.getItem("login_server");
const WS_URL     = localStorage.getItem("ws_server");
```

The three keys are written to `localStorage` by `LoginView` after reading `public/config.json` at startup. All other services import `SERVER_URL` from this module.

### Service Responsibility Matrix

| Service | Instantiation | Responsibility |
|---------|--------------|---------------|
| `AppService` | Per-component `new` | Token renewal, profile-picture upload |
| `MeteoService` | Per-component `new` | All meteo bulletin CRUD |
| `CfdService` | Per-component `new` | All CFD bulletin CRUD, soil state, section levels |
| `PrintMeteoService` | Per-component `new` | Fetch meteo print data |
| `PrintCfdService` | Per-component `new` | Fetch all CFD data for print |
| `LeafletService` | **Singleton** (module-level `new`) | Map creation, GeoJSON layers, section markers, alert colouring |
| `ShareService` | **Singleton** (module-level `new`) | Cross-component: alert levels list, window token |

---

## State Management

The application uses a **lightweight local state** approach rather than a global store:

| Mechanism | Usage |
|-----------|-------|
| Vue `ref` / `reactive` | Per-component reactive state |
| `localStorage` | Persisted auth token (`auth_token`), user object (`user`), server URLs (`db_server`, `login_server`, `ws_server`) |
| `ShareService` singleton | Shared non-reactive state (alert levels, token reference) accessible from any service/component |
| Pinia | Listed as a dependency but not actively used in current code |

### localStorage Keys

| Key | Type | Set by | Read by |
|-----|------|--------|---------|
| `auth_token` | `string` | `LoginView` on login | All service classes |
| `user` | `JSON string` | `LoginView` on login | `MeteoMapComponent`, `CfdMapComponent` |
| `db_server` | `string` | `LoginView` on config load | `service.js` |
| `login_server` | `string` | `LoginView` on config load | `service.js` |
| `ws_server` | `string` | `LoginView` on config load | `service.js` |

---

## Map Architecture (LeafletService)

`LeafletService` is a singleton responsible for the entire lifecycle of all Leaflet maps in the application.

### Internal Data Structures

```
LeafletService {
  maps_CFD:          { [id_rischio]: L.Map }
  maps_METEO:        { [id_rischio]: L.Map }
  geoJsonLayer_CFD:  { [id_rischio]: L.GeoJSON }
  geoJsonLayer_METEO:{ [id_rischio]: L.GeoJSON }
  sezioniLayer_CFD:  { [id_rischio]: L.LayerGroup }
  sezioniLayer_METEO:{ [id_rischio]: L.LayerGroup }
  filter_CFD:        { [id_rischio]: Function }
  filter_METEO:      { [id_rischio]: Function }
  geoJsonData:       FeatureCollection | null   // alert zones
  sezioniData:       Array | null               // river sections
  meteo_allerta:     Array                       // meteo alert data
  cfd_allerta:       Array                       // CFD alert data
  updated:           { CFD: [], METEO: [] }
}
```

### Map Creation Flow

```mermaid
flowchart TD
    Mount["Component onMounted"]
    Create["LeafletService.createMap()\nid_rischio · tipo · domElement"]
    HasSezioni{"tipo ends with\n_SEZIONI?"}
    AddSezioni["Create L.LayerGroup\nAdd circle markers\n(river section overlays)"]
    AddGeoJSON["Add L.GeoJSON layer\n(alert zone polygons)"]
    StyleCb["Apply style callback\n(fill colour from alert level)"]
    TooltipCb["Apply onEachFeature\n(zone name tooltip)"]
    Ready["Map ready for user interaction"]
    WatchKey["Reactive watch(key_val)"]
    Update["LeafletService.updateMap()\nRepaint zone polygon fills"]
    Unmount["Component onUnmounted"]
    Destroy["Remove L.Map instance from registry"]

    Mount --> Create
    Create --> HasSezioni
    HasSezioni -->|Yes| AddSezioni
    HasSezioni -->|No| AddGeoJSON
    AddSezioni --> AddGeoJSON
    AddGeoJSON --> StyleCb
    StyleCb --> TooltipCb
    TooltipCb --> Ready
    WatchKey --> Update
    Unmount --> Destroy
```

### Map Types

| `tipo` string | CFD/METEO | Sezioni overlay |
|---------------|-----------|-----------------|
| `'CFD'` | CFD | No |
| `'CFD_SEZIONI'` | CFD | Yes |
| `'METEO'` | Meteo | No |
| `'METEO_SEZIONI'` | Meteo | Yes |

---

## Real-Time Communication

`Socket.IO` is used to receive push notifications from the backend when:

- another operator changes an alert level;
- a computation finishes on the server.

Components that require real-time updates connect to the WebSocket using `WS_URL` obtained from `service.js`, and call `LeafletService` update methods upon receiving messages.

---

## Authentication Flow

```mermaid
sequenceDiagram
    actor User
    participant LV as LoginView
    participant Cfg as public/config.json
    participant LS as localStorage
    participant API as Backend API
    participant App as App.vue

    User->>LV: Open application
    LV->>Cfg: GET /config.json
    Cfg-->>LV: LOGIN_URL · SERVER_URL · WS_URL
    LV->>LS: Store login_server, db_server, ws_server

    User->>LV: Enter username & password
    LV->>API: POST LOGIN_URL/login
    alt Credentials valid
        API-->>LV: { token, user }
        LV->>LS: Store auth_token, user
        LV->>App: logged = true
        App-->>User: Show main navigation
    else Credentials invalid
        API-->>LV: 401 Unauthorized
        LV-->>User: Display error message
    end

    Note over User,App: Token renewal (user-initiated)
    User->>App: Click token timer in navbar
    App->>API: GET /get/new/token?authorization=JWT
    API-->>App: { token }
    App->>LS: Update auth_token

    Note over User,App: Any subsequent 401 response
    API-->>LV: 401 Unauthorized
    LV-->>User: Toast — Credenziali scadute
    LV->>LV: Redirect to login after 500 ms
```

---

## Print Architecture

Print views (`PrintViewM`, `PrintViewC`) are opened in **new browser windows/tabs** via `window.open()`. They share the same origin and read `auth_token` from `localStorage`. `PrintViewM` supports both horizontal and vertical page orientations.

```mermaid
flowchart TD
    Operator["Operator clicks 'Stampa'\nin CfdView or MeteoView"]
    NewTab["window.open()\nNew browser tab (same origin)"]
    ReadToken["Read auth_token from localStorage"]
    FetchData["PrintMeteoService / PrintCfdService\nGET complete bulletin data"]
    HasPDF{"Backend returns\nBase64 PDF?"}
    Decode["b64toBlob()\nDecode to Blob"]
    SaveAs["file-saver saveAs()\nTrigger browser download"]
    Render["Render print-optimised layout\nin browser window"]

    Operator --> NewTab
    NewTab --> ReadToken
    ReadToken --> FetchData
    FetchData --> HasPDF
    HasPDF -->|Yes| Decode
    Decode --> SaveAs
    HasPDF -->|No| Render
```

---

## CSS & Styling Strategy

| Source | Purpose |
|--------|---------|
| `node_modules/uikit/dist/css/uikit.min.css` | Primary UI component styles |  
| `css/dashboard.css` | Layout overrides for the application shell |
| `css/personal.css` | Application-specific customisations and colour overrides |
| `css/scrolling_table.css` | Custom layout for the synchronised meteo scrolling grid |
| `leaflet/dist/leaflet.css` | Leaflet map base styles (imported via `LeafletService.js`) |
| `toastr/build/toastr.min.css` | Toast notification styles |

Responsive breakpoints follow UIkit naming conventions (`@m`, `@l`, `@xl`). No CSS preprocessor is used; all custom styles are plain CSS.

---

[↑ Back to top](#architecture) · [← Back to README](README.md)
