<div align="center">

# Bollettino Multirischio — Frontend Documentation

**Documentation-only publication package for the Vue 3 application used to author, validate, and print multi-risk weather and civil-protection bulletins.**

[![Vue](https://img.shields.io/badge/Vue-3.x-42b883?logo=vue.js&logoColor=white)](https://vuejs.org/)
[![Vite](https://img.shields.io/badge/Vite-4.x-646cff?logo=vite&logoColor=white)](https://vitejs.dev/)
[![Node](https://img.shields.io/badge/Node-%E2%89%A518-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![Leaflet](https://img.shields.io/badge/Leaflet-1.9-199900?logo=leaflet&logoColor=white)](https://leafletjs.com/)
[![Axios](https://img.shields.io/badge/Axios-1.x-5a29e4?logo=axios&logoColor=white)](https://axios-http.com/)
[![Socket.IO](https://img.shields.io/badge/Socket.IO-4.x-010101?logo=socket.io&logoColor=white)](https://socket.io/)
[![License](https://img.shields.io/badge/license-see%20LICENSES.md-blue)](#license)

</div>

---

## Overview

This repository contains the publication-ready documentation for the **Bollettino Multirischio** frontend.

The underlying application is a Vue 3 single-page frontend developed for regional environmental and civil-protection operations. It supports:

- Authoring and validating **meteorological bulletins**.
- Authoring and validating **CFD multi-risk alert bulletins**.
- Visualising alert zones and section data on **interactive Leaflet maps**.
- Exporting bulletins to **print-ready PDF flows**.
- Coordinating operators through **real-time WebSocket updates**.

The reference deployment is for **ARPAV** — *Agenzia Regionale per la Prevenzione e Protezione Ambientale del Veneto*.

---

## Documentation Index

| Document | Description |
|----------|-------------|
| [docs/README.md](docs/README.md) | Full project overview, feature summary, stack, workflow, and lifecycle diagrams |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | System topology, component hierarchy, service layer, auth flow, map flow, and print flow |
| [docs/API_SERVICES.md](docs/API_SERVICES.md) | Backend REST endpoints, auth model, and service-by-service API reference |
| [docs/COMPONENTS.md](docs/COMPONENTS.md) | Responsibilities and interactions for views, cards, modals, map components, and utilities |
| [docs/INSTALLATION.md](docs/INSTALLATION.md) | Local setup, production build, nginx deployment, and troubleshooting guidance |
| [docs/USER_GUIDE.md](docs/USER_GUIDE.md) | End-user workflow for login, bulletin editing, validation, revision, and printing |

---

## Recommended Reading Order

1. Start with [docs/README.md](docs/README.md) for the functional overview.
2. Continue with [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the technical design.
3. Use [docs/API_SERVICES.md](docs/API_SERVICES.md) and [docs/COMPONENTS.md](docs/COMPONENTS.md) as developer references.
4. Use [docs/INSTALLATION.md](docs/INSTALLATION.md) for deployment.
5. Use [docs/USER_GUIDE.md](docs/USER_GUIDE.md) for operator onboarding.

---

## Repository Contents

```text
.
├── README.md
├── LICENSES.md
└── docs/
    ├── README.md
    ├── ARCHITECTURE.md
    ├── API_SERVICES.md
    ├── COMPONENTS.md
    ├── INSTALLATION.md
    └── USER_GUIDE.md
```

---

## Publication Notes

- Keep a copy of the original [docs/README.md](docs/README.md) inside the `docs/` folder.
- Use this file as the repository root `README.md` so GitHub shows a polished landing page.
- The supporting documents inside `docs/` already link back to `docs/README.md`, so no additional link fixes are required if both files are included.

---

## License

See [LICENSES.md](LICENSES.md) for the third-party dependency licenses documented for this project.

---

<div align="center">
<sub>Built for ARPAV — Agenzia Regionale per la Prevenzione e Protezione Ambientale del Veneto</sub>
</div>