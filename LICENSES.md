# Project Dependencies Licenses

This document lists all the licenses of the project dependencies used in **bollettino_cfd_frontend**.
**Authors**: 
- Massimo Casarin (massimo.casarin@arpa.veneto.it)
- Alessandro Martinato (alessandro.martinato@arpa.veneto.it)

Generated: March 2, 2026

---

## Project Dependencies

### Production Dependencies

| Package | Version | License |
|---------|---------|---------|
| @fortawesome/fontawesome-svg-core | 6.7.1 | MIT |
| @fortawesome/free-brands-svg-icons | 6.7.1 | CC-BY-4.0 AND MIT |
| @fortawesome/free-regular-svg-icons | 6.7.1 | CC-BY-4.0 AND MIT |
| @fortawesome/free-solid-svg-icons | 6.7.1 | CC-BY-4.0 AND MIT |
| @fortawesome/vue-fontawesome | 3.0.8 | MIT |
| @tinymce/tinymce-vue | 4.0.7 | Apache-2.0 |
| @vuepic/vue-datepicker | 10.0.0 | MIT |
| axios | 1.7.9 | MIT |
| chart.js | 2.9.4 | MIT |
| file-saver | 2.0.5 | MIT |
| jquery | 3.6.0 | MIT |
| jwt-decode | 3.1.2 | MIT |
| pinia | 2.3.0 | MIT |
| socket.io-client | 4.8.1 | MIT |
| tinymce | 5.10.9 | LGPL-2.1 |
| toastr | 2.1.4 | MIT |
| uikit | 3.16.15 | MIT |
| vue | 3.5.13 | MIT |
| vue-router | 4.5.0 | MIT |
| vue-scrolling-table | 2.0.0 | MIT |

### Development Dependencies

| Package | Version | License |
|---------|---------|---------|
| @rushstack/eslint-patch | 1.10.4 | MIT |
| @tsconfig/node18 | 2.0.1 | MIT |
| @types/node | 18.19.68 | MIT |
| @vitejs/plugin-vue | 4.6.2 | MIT |
| @vitejs/plugin-vue-jsx | 3.1.0 | MIT |
| @vue-leaflet/vue-leaflet | 0.10.1 | MIT |
| @vue/eslint-config-prettier | 7.1.0 | MIT |
| @vue/eslint-config-typescript | 11.0.3 | MIT |
| @vue/tsconfig | 0.4.0 | MIT |
| eslint | 8.57.1 | MIT |
| eslint-plugin-vue | 9.32.0 | MIT |
| leaflet | 1.9.4 | BSD-2-Clause |
| npm-run-all | 4.1.5 | MIT |
| prettier | 2.8.8 | MIT |
| typescript | 5.0.4 | Apache-2.0 |
| vite | 4.5.5 | MIT |
| vue-tsc | 1.8.27 | MIT |

---

## License Summary

### MIT License
The majority of packages (34 out of 37) are licensed under the MIT License.

### Dual Licenses
- **@fortawesome/free-brands-svg-icons**: CC-BY-4.0 AND MIT
- **@fortawesome/free-regular-svg-icons**: CC-BY-4.0 AND MIT
- **@fortawesome/free-solid-svg-icons**: CC-BY-4.0 AND MIT

### BSD-2-Clause License
- **leaflet**: BSD-2-Clause

### Apache-2.0 License
- **@tinymce/tinymce-vue**: Apache-2.0
- **typescript**: Apache-2.0

### LGPL-2.1 License
- **tinymce**: LGPL-2.1

---

## Brief License Descriptions

### MIT License
A permissive free software license originating from the Massachusetts Institute of Technology. The license permits commercial use, distribution, modification, and private use. It requires only that the license and copyright notice be included with the code.

**Packages:** @fortawesome/fontawesome-svg-core, @fortawesome/vue-fontawesome, @vuepic/vue-datepicker, axios, chart.js, file-saver, jquery, jwt-decode, pinia, socket.io-client, toastr, uikit, vue, vue-router, vue-scrolling-table, @rushstack/eslint-patch, @tsconfig/node18, @types/node, @vitejs/plugin-vue, @vitejs/plugin-vue-jsx, @vue-leaflet/vue-leaflet, @vue/eslint-config-prettier, @vue/eslint-config-typescript, @vue/tsconfig, eslint, eslint-plugin-vue, npm-run-all, prettier, vite, vue-tsc

### CC-BY-4.0 (Creative Commons Attribution 4.0)
Allows free use and sharing of works with attribution to the original creator.

**Packages:** FontAwesome Free Icons (Brands, Regular, Solid)

### BSD-2-Clause License
A permissive open source license similar to MIT but slightly simpler. Allows commercial use, distribution, modification, and private use, requiring the license and copyright notice to be included.

**Packages:** leaflet

### Apache-2.0 License
A permissive open source license that allows commercial and private use. It includes explicit grants of patent rights and requires documenting significant changes.

**Packages:** @tinymce/tinymce-vue, typescript

### LGPL-2.1 (GNU Lesser General Public License)
A less restrictive version of the GPL that allows linking with commercial software. If distributed as a library, modifications to the LGPL-licensed code must be released under LGPL terms, but the application itself can have a different license.

**Packages:** tinymce

---

## Important Notes

1. **LGPL-2.1 Consideration**: The `tinymce` package is licensed under LGPL-2.1. This means if you're distributing your application, ensure you comply with LGPL terms (e.g., providing means for users to replace the LGPL-licensed library if needed).

2. **FontAwesome Icons**: The free Font Awesome icons require attribution. Ensure proper attribution is included in your project.

3. **Transitive Dependencies**: This list includes only direct dependencies. Indirect (transitive) dependencies also exist but are not listed here. To see all transitive dependencies, run:
   ```bash
   npm list
   ```

4. **License Verification**: Always verify these licenses with the official package repositories and LICENSE files to ensure accuracy.

5. **Commercial Use**: Most of these packages permit commercial use. However, always review the specific terms of any license before commercial deployment.

---

## How to Update This File

When adding new dependencies, run:
```bash
npm list --depth=0
```

Then update this document with the new package's license information, which can be found in the package's `package.json` file or LICENSE file.

