# `plugins/` — Usage reference

Two Nuxt plugins ship with this module. Both register Nuxt-app-wide helpers using `nuxtApp.provide(name, value)`, which makes them available as:

- `$name` on the template via `useNuxtApp()`
- `{ $name }` via destructuring `useNuxtApp()`
- `nuxtApp.$name` in plugins / middleware

---

## `plugin.js`

Wires up reactive screen tracking + connectivity status, and exposes a handful of util shortcuts to templates.

### What it does

1. **Resize tracking** — calls `useScreenStore().setSize(width, height)` on load and on every `window.resize`. The HMR `import.meta.hot.dispose` callback unwires the listener cleanly in dev.
2. **Online / offline tracking** — wires `window` `online`/`offline` listeners into a local `ref(true)`; exposes the current value through `$isOnline()`.
3. **Helper proxies** — exposes the most commonly used `utils/` functions on `nuxtApp` so templates can call them with the `$` prefix without importing.

### Provided helpers

| Provided as | Underlying util |
|---|---|
| `$isOnline()` | local `ref` → `boolean` |
| `$isEmpty` | [`isEmpty`](../utils/README.md#empty-checks) |
| `$isNotEmpty` | [`isNotEmpty`](../utils/README.md#empty-checks) |
| `$getDeviceId` | [`getDeviceId`](../utils/README.md#auth--user) |
| `$ocdate` | [`ocdate`](../utils/README.md#dates) |
| `$tBy` | [`tBy`](../utils/README.md#strings--numbers) |
| `$queryParams` | [`queryParams`](../utils/README.md#http--url) |
| `$copyWith` | [`copyWith`](../utils/README.md#objects--arrays) |
| `$colorByBackground` | [`colorByBackground`](../utils/README.md#colors) |

> Auto-imported `utils/` functions are still callable without the `$` prefix in `<script>`. The `$` proxies are mainly there for **template-only** expressions and for code paths where auto-imports aren't resolved (raw `<script>` blocks in vanilla HTML embeds, third-party widget mounts, etc.).

### Usage — in components

```vue
<template>
  <div :class="$isOnline() ? 'text-green-500' : 'text-red-500'">
    {{ $isOnline() ? 'Online' : 'Offline' }}
  </div>

  <span v-if="$isNotEmpty(row.PhoneNumber)">
    {{ row.PhoneNumber }}
  </span>

  <span :style="{ color: $colorByBackground(theme.bg) }">
    {{ $tBy({ en: 'Hello', km: 'សួស្ដី' }) }}
  </span>

  <time>{{ $ocdate(row.CreatedAt).format('yyyy-MM-dd HH:mm') }}</time>
</template>

<script setup>
const { $isOnline, $getDeviceId } = useNuxtApp()

// react when connectivity flips
watchEffect(() => {
  if (!$isOnline()) toast.add({ color: 'yellow', title: 'You are offline' })
})

console.log('Device:', $getDeviceId())
</script>
```

### Usage — reading the screen size

The plugin populates `useScreenStore` — read it where you need responsive logic:

```vue
<script setup>
const screen = useScreenStore()

// reactive
watch(() => screen.isMobile, (mobile) => {
  console.log('Mobile?', mobile)
})

// derived flags
if (screen.isLaptopL) showSidebar.value = true
</script>
```

### Usage — in another plugin / middleware

```js
export default defineNuxtPlugin((nuxtApp) => {
  if (!nuxtApp.$isOnline()) {
    nuxtApp.hook('app:mounted', () => {
      console.warn('Booted while offline')
    })
  }
})
```

### HMR note

The plugin registers its `resize` listener through `import.meta.hot.dispose(...)` so dev rebuilds don't pile up duplicate listeners. **Don't add another `window.addEventListener('resize', …)` for the same purpose elsewhere** — read from `useScreenStore` instead.

---

## `assets.js`

Resolves a path inside the module's public folder, prefixed with `/_kedtec/`. Use this for any asset shipped by `KEDTEC_MODULE` (icons, flags, illustrations).

### Provided helper

| Provided as | Signature |
|---|---|
| `$public(path)` | `(path: string) => string` |

### Usage

```vue
<template>
  <img :src="$public('img/logo.svg')" alt="Logo" />
  <img :src="$public('/img/currencyFlags/USD.svg')" alt="USD" />
</template>

<script setup>
const { $public } = useNuxtApp()

const flagUrl = computed(() => $public(`img/currencyFlags/${currencyCode.value}.svg`))
</script>
```

The function strips a leading `/` if you pass one, then returns `/_kedtec/<path>`. Equivalent to writing the literal path by hand, but keeps the module's public-asset prefix in one place so it can be re-mapped later.

### Where this prefix comes from

The module registers its `public/` directory under the URL prefix `/_kedtec/` at build time. Anything you drop in `KEDTEC_MODULE/src/runtime/public/` is served from `/_kedtec/<filename>` — that's what `$public` builds.

### Related util

[`getCurrencyFlag(code)`](../utils/README.md#app-config--helpers) is a thin wrapper around this same convention for currency flags:

```js
getCurrencyFlag('USD')   // "/_kedtec/img/currencyFlags/USD.svg"
$public('img/currencyFlags/USD.svg')  // same
```

---

## Quick reference

```js
// Connectivity
$isOnline()                          // boolean

// Empty checks
$isEmpty(value)                      // boolean
$isNotEmpty(value)                   // boolean

// Device & locale
$getDeviceId()                       // uuid v4 (persisted cookie)
$tBy({ en, km })                     // localized string

// Dates
$ocdate(d).format('yyyy-MM-dd')      // OCDate wrapper

// URLs
$queryParams('filter', { a: 1 })     // base64-encoded query string

// Objects
$copyWith(row)                       // shallow clone (preserves Date)

// Colors
$colorByBackground('#222')           // '#fff'

// Assets
$public('img/logo.svg')              // '/_kedtec/img/logo.svg'
```

---

## When to add a new plugin vs. a util

Reach for a **plugin** when you need to:
- run code at app boot (`onMounted`-style side effects),
- attach `window`/`document` listeners with HMR-safe cleanup,
- expose something on `nuxtApp` that templates need with the `$` prefix.

Reach for a **util** in [`../utils/`](../utils/) when you just need a pure function — Nuxt auto-imports it everywhere already.