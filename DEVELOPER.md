# Developer Documentation — Nuxt 3 Project

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Project Setup](#project-setup)
3. [Project Structure](#project-structure)
4. [Naming Conventions](#naming-conventions)
5. [Routing](#routing)
6. [State Management (Pinia)](#state-management-pinia)
7. [Layouts](#layouts)
8. [Middlewares](#middlewares)
9. [Calling APIs](#calling-apis)
10. [Sharing Data Across Pages](#sharing-data-across-pages)
11. [Sharing Data Across Components](#sharing-data-across-components)
12. [Development Guidelines](#development-guidelines)
13. [Troubleshooting](#troubleshooting)
14. [Additional Resources](#additional-resources)

---

## Prerequisites

### Node.js
- Version **20.x** or newer (active LTS recommended)

### Recommended VS Code Extensions
| Extension | Purpose |
|---|---|
| Vue (Official) | Vue 3 language support |
| Nuxtr | Nuxt-specific tooling |
| Tailwind Snippets | Tailwind CSS autocomplete |
| ESLint | Code linting |
| GitHub Copilot | AI code completion |
| Auto Rename Tag | Sync HTML/Vue tag edits |
| Code Spell Checker | Catch typos |

### Other Requirements
- **Git** for version control
- Familiarity with HTML, CSS, JavaScript, Vue.js, and Node.js

---

## Project Setup

```bash
# 1. Install dependencies
npm install

# 2. Start development server
npm run dev

# 3. Build for production
npm run generate

# 4. Build for HCS environment
npm run genHcs

# 5. Deploy
bash deploy.sh
```

> **Windows users:** Use WSL for significantly faster HMR performance.

---

## Project Structure

```
project-root/
├── assets/
│   ├── lang/
│   │   ├── km.json          # Khmer translations
│   │   └── en.json          # English translations
│   └── json/
│       ├── studentType.json
│       └── item.json
│
├── components/
│   └── mainModule/
│       └── subModule/
│           └── HeaderComponent.vue   # PascalCase
│
├── composables/
│   └── mainComposable/
│       └── subComposable/
│           └── useAuth.ts            # camelCase with "use" prefix
│
├── pages/
│   └── main_page/
│       └── sub_page.vue              # snake_case
│
├── stores/
│   └── mainStore/
│       └── useCount.ts               # camelCase
│
├── helpers/
│   ├── apis.js                       # API endpoint definitions
│   └── str.js                        # String utilities
│
├── public/
│   └── _kedtec/
│       └── img/
│           └── file.png
│
├── utils/
└── nuxt.config.ts
```

---

## Naming Conventions

| Type | Convention | Example |
|---|---|---|
| Pages | `snake_case` | `home_page.vue`, `user_profile_page.vue` |
| Components | `PascalCase` | `HeaderComponent.vue`, `UserCardComponent.vue` |
| Composables | `camelCase` + `use` prefix | `useAuth.ts`, `useFetchData.ts` |
| Stores | `camelCase` | `userStore.ts`, `cartStore.ts` |
| Utilities | `camelCase` | `formatDate.js`, `parseJson.js` |
| Other files | `camelCase` | `configSettings.ts`, `apiClient.ts` |
| Folders | `camelCase` (no `Page` suffix) | `productDetail/`, not `productDetailPage/` |
| Variables | `camelCase` | `userName`, `isLoading` |

---

## Routing

Nuxt 3 uses **file-based routing** — every file in `pages/` becomes a route automatically.

| File | Route |
|---|---|
| `pages/home_page.vue` | `/home_page` |
| `pages/product_detail/[id].vue` | `/product_detail/:id` |

---

## State Management (Pinia)

### Rules
- File name = function name (e.g., `useCount.ts` contains `useCountStore`)
- Do **not** put a single store file inside a dedicated subfolder
- Always add HMR support at the bottom

### File Structure
```
stores/
└── useCount.ts
```

### Example Store
```typescript
// stores/useCount.ts
export const useCountStore = defineStore('CountStore', {
  state: () => ({
    data: 0
  }),

  getters: {
    getData: (state) => state.data
  },

  actions: {
    setData() {
      this.data++
    }
  }
})

// Required for Hot Module Replacement
if (import.meta.hot) {
  import.meta.hot.accept(acceptHMRUpdate(useCountStore, import.meta.hot))
}
```

### Naming Summary
| Part | Pattern | Example |
|---|---|---|
| File name | `useXxx.ts` | `useCount.ts` |
| Function name | `useXxxStore` | `useCountStore` |
| Store key | `XxxStore` | `CountStore` |

---

## Layouts

Two built-in layouts are available:

| Layout | Description |
|---|---|
| `default` | No navigation bar |
| `navbar` | Includes the navigation menu |

### Usage
```javascript
// In your page file
definePageMeta({
  layout: 'navbar'
})
```

---

## Middlewares

Apply route guards using `definePageMeta`. Multiple middlewares run in order.

```javascript
definePageMeta({
  middleware: ['auth', 'permission']
})
```

---

## Calling APIs

Use the `useHttp` composable for all API requests.

### Options Reference

| Option | Type | Description |
|---|---|---|
| `method` | `'GET'` \| `'POST'` | HTTP method (case-insensitive) |
| `data` | `object` | Request payload |
| `headers` | `object` | Custom request headers |
| `isGlobal` | `boolean` | `true` = public endpoint (no auth); `false` = requires user auth |
| `isWeb` | `boolean` | `true` = request targets a web URL |
| `alertError` | `boolean` | Show error alert on failure |
| `alertSuccess` | `boolean` | Show success alert on completion |
| `signal` | `AbortSignal` | Cancel the request via AbortController |

### Basic Usage
```javascript
const { data, error } = await useHttp(apis.studyClass, {
  method: 'POST',
  data: filter,
  isGlobal: false,
  alertError: true
})
```

### Cancellable Request
```javascript
// 1. Declare the controller
let controller

// 2. Make the request with a signal
controller = new AbortController()

const { data, error } = await useHttp(apis.studyClass, {
  method: 'POST',
  data: filter,
  signal: controller.signal
})

// 3. Cancel if needed (e.g., on component unmount or new search)
controller.abort()
```

---

## Sharing Data Across Pages

### Option 1 — Dynamic Routes
For data tied to a specific resource (e.g., viewing a record by ID):

```
pages/user/[id].vue  →  /user/123
```

### Option 2 — Query Params
For passing small, lightweight data (IDs, filters) between pages. **Not suitable for large payloads.**

```javascript
// Encode and navigate with data
const queryString = queryParams('data', { id: 1, type: 'student' })

// On the destination page, read it back
const queryData = queryParams()
console.log(queryData.data) // { id: 1, type: 'student' }
```

---

## Sharing Data Across Components

Use **Pinia stores** to share state between components that don't have a direct parent–child relationship. See the [State Management](#state-management-pinia) section for setup instructions.

---

## Development Guidelines

- **Linting:** Use ESLint and Prettier. Config lives in `.eslintrc` or `package.json`.
- **Code style:** Use `<script setup lang="ts">` with Composition API.
- **Modularity:** Break large logic into composables; keep components focused and small.
- **Comments:** Add brief comments for non-obvious logic, especially in composables and stores.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Slow HMR on Windows | Switch to WSL, or optimize Vite config in `nuxt.config.ts` |
| Dependency / install errors | Delete `node_modules` and `package-lock.json`, then run `npm install` |

---

## Additional Resources

- [Nuxt 3 Documentation](https://nuxt.com)
- [NuxtUI v2 Documentation](https://ui2.nuxt.com)
- [Nuxt Modules](https://modules.nuxtjs.org)
- [Vue.js Documentation](https://vuejs.org)
- [Flowbite Vue Documentation](https://flowbite-vue.com)
- [Nuxt Community Help](https://nuxt.com/docs/community/getting-help)