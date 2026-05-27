# `composables/` — Usage reference

Composables are Nuxt-auto-imported — call them by name from anywhere (components, plugins, other composables) without an explicit `import`.

Quick index:

- [HTTP & API](#http--api): `useHttp`, `useOptions`
- [Authentication](#authentication): `useAuth`, `useUserData`, `useRefreshToken`, `useAuthorizeIframe`
- [Validation (Zod)](#validation-zod): `useCheckEmail`, `useCheckPhone`, `OCCheckExistCode`, `checkExistCode` (legacy)
- [Menu & reports](#menu--reports): `useMenuData`, `useOCReport`
- [UI / DOM](#ui--dom): `useOCToast`, `useOnScroll`, `useScannerInput`, `useScriptLoader`, `usePopper`
- [File upload](#file-upload): `useFileUpload`
- [Formatting & misc](#formatting--misc): `useCurrencyFormat`, `useRemoveDuplicate`, `useTitle`, `localePath`

---

## HTTP & API

### `useHttp(url, options?)` → `Promise<{ data, error }>`

The canonical HTTP wrapper. Auto-refreshes tokens, applies app headers (auth, language, device id, database cookie), respects offline state, and optionally toasts success/error.

**Options** (`HttpOptions`):

| Field | Notes |
|---|---|
| `method` | `'get' \| 'post' \| 'put' \| 'delete' \| …` (default `'get'`) |
| `data` | Body (POST/PUT/DELETE) or query (GET). |
| `headers` | Merged on top of the defaults. |
| `signal` | `AbortController.signal` |
| `isGlobal` | Use basic-auth key instead of user access token (skip refresh). |
| `isWeb` | Resolve URL against `webUrl` instead of `apiUrl`. |
| `alertSuccess` | `true` for default toast, or `{ title, description, type, color }`. |
| `alertError` | Same shape — only the error toast. |

**Return**: `{ data, error }` (both refs). `data.value` is the parsed JSON; `error.value` is `null` on success and the API error body on failure.

```js
const { data, error } = await useHttp('api/order/list', {
  method: 'POST',
  data: { Status: 1 },
})
if (error.value) return
list.value = data.value

// abort
const ctrl = new AbortController()
const req  = useHttp('api/big', { signal: ctrl.signal })
ctrl.abort()

// global (no user token)
await useHttp('api/public/health', { isGlobal: true })

// toast on success/error automatically
await useHttp('api/order/save', { method: 'POST', data: row, alertSuccess: true, alertError: true })
```

> Offline: throws a synthetic `{ Message: 'No internet connection', StatusCode: 1001 }` error so the caller can handle it like any other API error.

### `useOptions(url, options?)` → `{ url, options } | undefined`

Lower-level helper that `useHttp` uses internally. Builds the resolved URL plus the headers object (auth, content-type, lang, device id, database). Returns `undefined` if not `isGlobal` and there's no user data — that's the cue to bail out before issuing a request.

```js
const opts = useOptions('api/upload', { isWeb: true, method: 'POST' })
if (!opts) return
axios(opts.url, { ...opts.options, data: form })
```

---

## Authentication

### `useAuth(args?)` → `Promise<{ data, error }>`

Logs the user in (or refreshes via `refresh_token`) by POSTing to `/ocsauth/oauth`. Persists the result to `useUserData()` and broadcasts the auth cookies to the authorize iframe.

**Args**: `{ rememberMe?: boolean }` (defaults to `true`).

On `invalid_grant` it clears the user cookie and navigates to `/login`.

```js
const { data, error } = await useAuth({ rememberMe: true })
if (error.value) toast.add({ color: 'red', title: error.value.Message })
```

### `useUserData(value?)` → `Ref<UserData | null>`

Getter/setter for the encrypted user-data cookie (`oc_user_data`). Pass a value to write; call with no args to read.

```js
const user = useUserData().value
console.log(user.access_token)

// Manually persist a payload
useUserData({ access_token, refresh_token, expires_in, expires, refresh_lifetime, token_type })
```

Writing also sets the supporting cookies `oc_access_token`, `oc_refresh_token`, `oc_expire`, `oc_lang`.

### `useRefreshToken(force?)` → `Promise<void>`

Idempotent token-refresh primitive. Called by `useHttp` before every authenticated request — usually you don't call it directly. Pass `force = true` to refresh even if the token isn't near expiry yet.

```js
await useRefreshToken()       // refresh if expiring within 5 min
await useRefreshToken(true)   // force refresh now
```

Uses a module-level `isRefreshing` guard so concurrent callers wait instead of stampeding.

### `useAuthorizeIframe(data, retry?)` → `Promise<Ref<HTMLIFrameElement>>`

Sends a message to the hidden `#SETATHORIZECOOKIE` iframe used to sync auth cookies across subdomains. Retries up to 1000 times (300 ms apart) if the iframe hasn't mounted yet.

```js
useAuthorizeIframe({
  action: 'set_authorize_cookie',
  data: {
    cookie: {
      oc_lang: 'en',
      oc_database: 'KEDTEC',
      oc_device_id: getDeviceId(),
      oc_access_token: token,
      oc_expire: addAccessTokenExpired(expires),
    },
    sameSite: 'lax',
  },
})
```

---

## Validation (Zod)

### `useCheckEmail(required = true)` → `ZodEffects<string>`

Returns a Zod schema that whitespace-strips the input and validates it as an email.

```js
const schema = z.object({
  Email: useCheckEmail(),           // required
  AltEmail: useCheckEmail(false),   // optional
})
```

### `useCheckPhone(required = true)` → `ZodEffects<string>`

Phone schema — strips whitespace, validates against `/^(0\d{6,14}|\+\d{7,15})$/`, length 9–15.

```js
const schema = z.object({ Phone: useCheckPhone() })
```

### `OCCheckExistCode({ api, pk, messageRequire, messageExist, allowNull, allowPass })` → `ZodEffects<string>`

Async unique-code validator. Calls `api` with `{ [pk]: code, ...api.data }` and treats a truthy response as "code exists" → validation fails.

| Option | Notes |
|---|---|
| `api` | `{ url, method, data }` — endpoint that returns truthy when the code exists. |
| `pk` | Field name to put `code` into (default `'Code'`). |
| `messageRequire` | Error when blank. |
| `messageExist` | Error when the API reports it exists. |
| `allowNull` | Pass when the value is empty (use for optional codes). |
| `allowPass` | Hard-skip the check (use for update mode). |

```js
const schema = z.object({
  Code: OCCheckExistCode({
    api: { url: 'api/branch/checkCode', method: 'POST', data: { Status: 1 } },
    pk: 'Code',
    allowPass: isUpdate,             // skip when editing
  }),
})
```

### `checkExistCode(api, isCreate?, pk?, message?)` — legacy

Older variant of the same idea. Prefer `OCCheckExistCode` for new code — it has explicit options and better error handling.

```js
const schema = z.object({
  Code: checkExistCode({ url: 'api/branch/checkCode', method: 'POST' }, isCreate),
})
```

---

## Menu & reports

### `useMenuData()` → `Promise<MenuItem[]>`

Resolves the user's permitted menu. Loads:
- `assets/json/menu_data.json` (built-in modules)
- `/customer/customMenu.json` (per-customer overrides)
- Adds dev/teacher entries based on access token / `isDevelopment`

Then fetches `api.permissions` and recursively keeps only menu items the user has permission for.

```js
const menu = await useMenuData()
const sidebar = useMenuStore()
sidebar.setMenu(menu)
```

Items that pass return with an `actions` field listing per-action permissions for that page.

### `useOCReport(types)` → `OCReport`

Loads the report list for the given type(s) and returns an instance you can use to print/preview reports through `useMyOCReportStore`.

```js
const reports = useOCReport('Order')

// Open with type filter — chooses 1-report direct print or shows the picker modal
await reports.open({
  url: 'api/report/order/data',
  data: { OrderId: 123 },
  reportType: 'Order',
  title: 'Order Report',
})

// Or directly by id / code
await reports.open({ reportId: 42,   url, data })
await reports.open({ reportCode: 'INV', url, data })

// Or pass a fully-resolved report descriptor
await reports.open({ report: chosenReport, url, data })
```

Optional callbacks: `beforePrintOrPreview(payload)`, `callBack(res)`, `onLoading(bool)`, `onLastPrint(meta)`.

---

## UI / DOM

### `useOCToast()` → `{ toasts, add, update, remove, clear }`

Queueing toast manager backed by `useState('toasts')`. Caps at 20 visible toasts; honors per-toast `duration` for auto-dismiss.

```js
const t = useOCToast()
const id = t.add({ title: 'Saved', variant: 'success', duration: 3000 })
t.update(id, { description: 'Now finalized' })
t.remove(id)
t.clear()
```

`variant` is consumer-defined — pair with your toast component for styling (e.g. `'success' | 'danger' | 'warning'`).

### `useOnScroll(elRef, selector?)` → `{ isDragging }`

Adds click-drag scroll behavior to a container. Ignores form controls (input/textarea/select/contentEditable).

```vue
<template>
  <div ref="scroller" class="horizontal-list">…</div>
</template>

<script setup>
const scroller = ref()
const { isDragging } = useOnScroll(scroller)            // drag the wrapper itself
useOnScroll(scroller, '.scroll-inner')                  // or a child by selector
</script>
```

Rebinds automatically when the ref or selector changes; cleans up on unmount.

### `useScannerInput(onScan, timeout = 30, minLength = 3)`

Listens to `keydown` globally and calls `onScan(buffer)` when an Enter key fires after a fast burst of characters (typical of barcode scanners). Rejects non-ASCII input with a toast asking the user to switch keyboard language.

```js
useScannerInput((code) => {
  console.log('Scanned:', code)
  lookupProduct(code)
}, 30, 5)
```

> Auto-unbinds on `onBeforeUnmount`. Buffer also resets after `timeout * 3` ms of inactivity.

### `useScriptLoader(scriptUrl, styleUrl?)` → `{ init, removeScript, removeStyle }`

Loads a `<script>` (and optionally a `<link rel="stylesheet">`) once per URL, then runs a callback (via `$(document).ready` if jQuery is present).

```js
const loader = useScriptLoader(str.cdn_signalR)
loader.init(() => {
  // window.$ is loaded
  const conn = $.hubConnection(url)
  conn.start()
})

// cleanup
loader.removeScript(str.cdn_signalR)
```

Used internally by `ocSignalR` to pull in the legacy jQuery SignalR client.

### `usePopper(options, virtualReference?)` → `[reference, popper, instance]`

Vue-style Popper.js wrapper. Returns three refs you bind to the trigger, the floating element, and (optionally) the Popper instance for runtime calls.

```vue
<template>
  <button ref="reference">Open</button>
  <Teleport to="body">
    <div ref="popper" v-if="open" class="popover">…</div>
  </Teleport>
</template>

<script setup>
const [reference, popper, instance] = usePopper({ placement: 'bottom-start', offsetDistance: 6 })
</script>
```

The shared `PopperOptions` type is exported from `./popper.ts`:

```ts
interface PopperOptions {
  locked?: boolean
  overflowPadding?: number
  offsetDistance?: number
  offsetSkid?: number
  placement?: Placement       // '@popperjs/core' Placement
  strategy?: PositioningStrategy
  gpuAcceleration?: boolean
  adaptive?: boolean
  scroll?: boolean
  resize?: boolean
  arrow?: boolean
}
```

---

## File upload

### `useFileUpload(url, filters, onProgress)` → `{ data, error }`

Wraps [`splitFileUpload`](../utils/README.md#new-splitfileuploadupload-filetarget-url-local-data-signal-complete-error-progress-) to upload one or many files with per-file + aggregate progress.

**Args:**
- `url` — upload endpoint (resolved via `getUrl`).
- `filters` — `{ files: File | File[], data?: Record<string, any>, signal? }`.
- `onProgress({ progressAverage, progress })` — called whenever any chunk completes.

```js
const { data, error } = await useFileUpload(
  'api/file/upload',
  {
    files: pickedFiles,
    data: { OwnerType: 'Student', OwnerId: studentId },
    signal: abortCtrl.signal,
  },
  ({ progressAverage, progress }) => {
    percent.value = progressAverage
  }
)

if (error.value) toast.add({ color: 'red', title: error.value })
```

Each file in `data.value` carries its own `progress` (0–100) and `loading` flag while in-flight.

---

## Formatting & misc

### `useCurrencyFormat()` → `(amount, options?) => string`

Locale-aware currency formatter (uses `Intl.NumberFormat`) with optional accounting parentheses for negatives.

| Option | Notes |
|---|---|
| `currency` | ISO code, default `'USD'`. |
| `locale` | Override (defaults to `navigator.language`). |
| `round` | Decimal precision. `<= 0` rounds to integer (no `.00`). |
| `useSymbol` | Show currency symbol (default `true`). |
| `symbol` | Force a custom symbol (replaces the auto one). |
| `accounting` | Wrap negatives in `( … )` instead of `-`. |

```js
const fmt = useCurrencyFormat()

fmt(1234.5)                                          // "$1,234.50"
fmt(1234.5, { currency: 'KHR', round: 0 })            // "៛1,235"
fmt(-89.5)                                           // "($89.50)" (accounting)
fmt(-89.5, { accounting: false })                    // "-$89.50"
fmt(1234.5, { useSymbol: false })                    // "1,234.50"
fmt(1234.5, { symbol: '៛' })                          // "៛1,234.50"
```

### `useRemoveDuplicate()` → `(list, keys) => list`

Returns a deduper. `keys` can be a single field name or an array of fields used to build a composite key.

```js
const dedupe = useRemoveDuplicate()

dedupe(items, 'Code')              // unique by Code
dedupe(items, ['ClassId', 'Year']) // composite unique
```

Null-safe — missing fields become `''` in the key.

### `useTitle(name, appName?)` → `string`

Builds an HTML title like `"<page> | <App>"`. Falls back to `useRuntimeConfig().public._appName` when no app name is passed.

```vue
<Head><Title>{{ useTitle('Orders') }}</Title></Head>
<!-- → "Orders | KEDTEC" -->
```

### `localePath(path)` → `string`

Prepends the active locale prefix when the language isn't English. Reads the `i18n_redirected` cookie.

```js
localePath('/orders')   // "/orders" when en, "/km/orders" when km
```

> This is a thin alternative when you don't want to inject `useLocalePath()` from Nuxt i18n directly.

---

## Conventions

- **All composables are auto-imported** — Nuxt registers them by their export name. The default-export modules (`checkExistCode.js`, `localePath.js`) auto-import using the filename.
- **Async composables return `{ data, error }`** as Vue refs so callers can `await` and react with `watch`/`computed`. Pattern matches Nuxt's built-in `useFetch`.
- **Validators return Zod schemas** so they slot directly into your form's overall schema via `z.object({ … })`.
- **Token refresh is automatic** in `useHttp`. Only call `useRefreshToken()` directly when issuing a non-`useHttp` request that needs a fresh token (e.g. a raw `axios` call from a third-party widget).
- **Offline state is short-circuited at the HTTP layer.** No need to wrap every call in an `if (online)` check — handle the synthetic 1001 error in your catch instead.