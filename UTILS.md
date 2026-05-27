# `utils/` — Usage reference

Every file in this folder default-exports a function and is **auto-imported by Nuxt** — call them directly anywhere in components, composables, pages, or other utils. No `import` line needed unless explicitly noted.

Quick index:

- [Empty checks](#empty-checks): `isEmpty`, `isNotEmpty`
- [Strings & numbers](#strings--numbers): `ocNumber`, `ocToFixed`, `round`, `currencyNumber`, `formatPhoneNumber`, `toSnakeCase`, `threeDot`, `tBy`
- [Dates](#dates): `ocdate`, `ocStrTimeToDate`, `dateToAge`, `formatDateDiff`, `addAccessTokenExpired`
- [Objects & arrays](#objects--arrays): `copyWith`, `mergeObjDeep`, `deepRemoveProperty`
- [Encoding / crypto](#encoding--crypto): `encryptAES`, `decryptAES`, `convertBase64`, `decodeBase64`, `extractJwtToken`, `extractDataFromAccessToken`
- [Colors](#colors): `convertColor`, `colorByBackground`, `getLuminance`, `generateColor`, `generateRootColor`, `colorPicker`
- [HTTP & URL](#http--url): `getUrl`, `getServerUrl`, `getData`, `queryParams`
- [App config & helpers](#app-config--helpers): `helper`, `ocConfig`, `bizHelper`, `getHelper`, `checkGlobal`, `getCurrencyFlag`
- [Auth / user](#auth--user): `getUserLogin`, `saveUserData`, `getDeviceId`, `createToken`
- [Page lifecycle](#page-lifecycle): `mounted`, `onTabChange`, `pagePath`, `oauthHandler`, `previewReport`
- [File upload & cropping](#file-upload--cropping): `splitFileUpload`, `ocProfileCropper`, `googleDrive`
- [Realtime](#realtime): `ocSignalR`
- [Form helpers](#form-helpers): `validateSelect`
- [Build-time (Nuxt module)](#build-time-nuxt-module): `getComponentPaths`, `getLayersComponentPaths`, `getPages`, `ocExtendPages`, `copyJsonFile`, `createTsConfigFile`

---

## Empty checks

### `isEmpty(value)` → `boolean`

True for `null`/`undefined`, falsy booleans, blank strings (after trim), empty arrays, empty objects, invalid dates, `NaN`.

```js
isEmpty('')          // true
isEmpty('  ')        // true
isEmpty([])          // true
isEmpty({})          // true
isEmpty(0)           // true (because falsy)
isEmpty(new Date('x')) // true
```

### `isNotEmpty(value)` → `boolean`

Logical inverse of `isEmpty`. Use whichever reads better at the call site.

```js
if (isNotEmpty(user.email)) sendWelcomeEmail(user.email)
```

---

## Strings & numbers

### `ocNumber({ number, round, isRound, isRoundUp, isString })`

The canonical number formatter — respects `useDatabaseStore().selected.BusinessDecimal` and supports rounding up/down with arbitrary precision (including **negative** precision for rounding to nearest 10/100/…).

```js
ocNumber({ number: 1234.567 })                   // 1234.57 (default 2 decimals)
ocNumber({ number: 1234.567, round: 4 })         // 1234.567
ocNumber({ number: 1234.567, round: 0 })         // 1235
ocNumber({ number: 1234.567, round: -2 })        // 1200    (round to hundreds)
ocNumber({ number: 1234.567, round: -2, isRoundUp: true })  // 1300
ocNumber({ number: 1234.5, isString: true })     // "1,234.5"
ocNumber({ number: 1.234567, isRound: true })    // up to 6 decimals
```

### `ocToFixed(value, round = 2)` → `number`

Evaluates a stringified arithmetic expression and rounds. Returns `NaN` if the expression is invalid.

```js
ocToFixed('10 + 2.5')         // 12.5
ocToFixed(1.23456, 3)         // 1.235
ocToFixed('not a number')     // NaN
```

> Internally uses `Function(...)` to evaluate — only feed it trusted input.

### `round(amount, round, isString = false, roundUp = false)` → `number | string`

Plain decimal rounder. Negative `round` rounds to tens/hundreds/etc.

```js
round(1234.5678, 2)               // 1234.57
round(1234.5678, 2, true)         // "1234.57"
round(1234, -2)                   // 1200
round(1234, -2, false, true)      // 1300
```

### `currencyNumber(number, round = 2)` → `string`

Picks a sensible decimal precision: shows ≥2 decimals; expands up to `round` decimals if the source has more.

```js
currencyNumber(12)         // "12.00"
currencyNumber(12.5)       // "12.50"
currencyNumber(12.34567)   // "12.35"  (capped at 2)
currencyNumber(12.34567, 4)// "12.3457"
```

### `formatPhoneNumber(phoneNumber)` → `string`

Pretty-prints local & international numbers based on length (9, 10, 12, or 13 digits).

```js
formatPhoneNumber('012345678')      // "012 345 678"
formatPhoneNumber('0123456789')     // "012 345 6789"
formatPhoneNumber('855123456789')   // "8551 23 456 789"
```

### `toSnakeCase(str)` → `string`

```js
toSnakeCase('UserFirstName')   // "user_first_name"
toSnakeCase('APIKey')          // "api_key"
```

### `threeDot(value, type)` → `string | OCDate`

Returns `".."` when the value is empty; otherwise echoes it (passes through `ocdate(value)` if `type === 'date'`). Useful for table cells.

```js
threeDot(student.PhoneNumber)              // "012 345 678" or ".."
threeDot(student.CreatedDate, 'date')      // ocdate instance, or ".."
```

### `tBy({ km, en })` → `string`

Picks the right localized string based on the current language cookie. Falls back to the other language if the chosen one is empty.

```js
tBy({ km: 'អានុមត្តិ', en: 'Hello' })
```

---

## Dates

### `ocdate(date?, endDate?)` → `OCDate`

A `date-fns` wrapper that reads default format/divider from `helper().config`. Supports a range (when `endDate` is passed `.format()` returns `"start <divider> end"`).

```js
ocdate().format('yyyy-MM-dd')          // today, formatted
ocdate(row.CreatedAt).format()         // uses config.dateFormat

// arithmetic
ocdate().add(7).days                   // Date 7 days from now
ocdate().sub(1).months                 // Date 1 month ago

// differences
ocdate('2026-01-01').diff(new Date()).days
ocdate('2026-01-01').diff(new Date()).businessDays

// breakdown
const g = ocdate().get()  // { year, month, day, hours, minutes, week, ... }

// range
ocdate(from, to).format('dd/MM/yyyy')  // "01/01/2026 - 31/01/2026"

// month bounds
ocdate().startOfMonth('yyyy-MM-dd')
ocdate().endOfMonth('yyyy-MM-dd')
```

Methods returned by an `OCDate` instance:

| Method | Returns |
|---|---|
| `.format(str?, options?)` | Formatted string. |
| `.toString(...)` | Alias for `.format(...)`. |
| `.get(options?)` | Object with every part (seconds…iSOWeeksInYear). |
| `.add(amount)` | `{ days, months, years, hours, ... }` — each a new `Date`. |
| `.sub(amount)` | Same shape as `.add`. |
| `.diff(to, options?)` | Difference in every unit. |
| `.startOfMonth(format?)` / `.endOfMonth(format?)` | `Date` or formatted string. |

### `ocStrTimeToDate('HH:mm[:ss]')` → `Date`

Combines today's date with a clock string.

```js
ocStrTimeToDate('14:30')      // today @ 14:30:00
ocStrTimeToDate('09:00:45')   // today @ 09:00:45
```

### `dateToAge(date)` → `number | ''`

Whole-year age between `date` and now.

```js
dateToAge('2000-06-15')   // 25 (today is past 2025-06-15)
dateToAge(null)           // ''
```

### `formatDateDiff(str, { date, time })` → `string`

Renders a `'y,m,d,h,m,s'` tuple as a human label, hiding zero parts and localising.

```js
formatDateDiff('2,5,28,0,0,0')                          // "2 years 5 months 28 days"
formatDateDiff('0,0,3,4,12,0', { date: true, time: true }) // "3 days 4 hours 12 minutes"
```

### `addAccessTokenExpired(expired)` → `ISO string`

Extends a date by **+1 day** in UTC. Used to pad token expiry when syncing cookies across subdomains.

```js
addAccessTokenExpired('2026-05-26T10:00:00Z')  // "2026-05-27T10:00:00.000Z"
```

---

## Objects & arrays

### `copyWith(value)` → cloned value

Shallow-by-default clone that preserves `Date` instances (recursive only for arrays).

```js
const next = copyWith(row)
next.Name = 'edited'   // row.Name untouched
```

### `mergeObjDeep(target, source)` → `target` (mutated)

Recursive deep-merge. Objects are merged key-wise; arrays are merged **by index** (object items get deep-merged element-by-element).

```js
mergeObjDeep(
  { ui: { color: 'red', size: 'm' } },
  { ui: { color: 'blue' } }
)
// { ui: { color: 'blue', size: 'm' } }
```

### `deepRemoveProperty(data, keysToRemove)` → cleaned data

Recursively strips one or more keys from any nested object/array. Items with `{ isDefault: true }` are dropped entirely. Numbers are coerced to strings.

```js
deepRemoveProperty(payload, 'isInternal')
deepRemoveProperty(payload, ['createdAt', 'updatedAt'])
```

---

## Encoding / crypto

### `encryptAES(text, secretKey?, cutSpace?)` → `string`

AES-CBC, PKCS7. Key is padded/trimmed to 32 chars; IV is fixed 16 `*`s. Default key falls back to `str.ocsappconfig168`. `cutSpace=true` strips all whitespace; `false` only strips newlines.

```js
encryptAES('hello')                  // "base64 ciphertext"
encryptAES('hello', 'my-secret')
encryptAES('multi\nline text', undefined, true)  // strips spaces+newlines
```

### `decryptAES(cipherBase64, secretKey?)` → `string`

Inverse of `encryptAES`. Returns `""` on any decode failure.

```js
decryptAES(cipher)
decryptAES(cipher, 'my-secret')
```

### `convertBase64(value, isDecode = false)` → `string`

Base64 round-tripper. Stringifies objects automatically; on decode, handles both ASCII and Unicode payloads.

```js
convertBase64({ a: 1 })            // encoded
convertBase64(encoded, true)       // decoded back to "{\"a\":1}"
```

### `decodeBase64(base64Url)` → `string`

Decodes URL-safe Base64 (handles `-`/`_` and missing padding). Used by `extractJwtToken`.

### `extractJwtToken(token)` → `{ header, payload, signature }`

Parses a 3-part JWT into a typed object. Throws if the token isn't well-formed.

### `extractDataFromAccessToken(token?)` → JWT payload

Convenience — if no token is given, reads `useUserData().value.access_token` and returns the payload object.

```js
const claims = extractDataFromAccessToken()
console.log(claims.uid, claims['o:r'])
```

---

## Colors

### `convertColor(color)` → string

Converts between `#hex` / `rgb()` / `rgba()` / `{ r, g, b, a }` representations. Throws on unparseable input.

```js
convertColor('#fff')               // "rgb(255, 255, 255)"
convertColor('#00000080')          // "rgba(0, 0, 0, 0.50)"
convertColor('rgb(255, 0, 0)')     // "rgb(255, 0, 0)"
convertColor({ r: 0, g: 0, b: 0, a: 0.5 })  // "#00000080"
```

### `colorByBackground(hex)` → `"#000" | "#fff"`

Picks readable text color (black/white) for a given background hex.

```js
colorByBackground('#ffffff')  // "#000"
colorByBackground('#222')     // "#fff"
```

### `getLuminance(color?)` → `number` (0–1)

WCAG relative luminance. Defaults to the computed `--primary` background color if no arg is passed (browser only).

```js
getLuminance('rgb(0, 0, 0)')   // 0
getLuminance('rgb(255,255,255)') // 1
```

### `generateColor(name)` → `"rgb(r, g, b)"`

Deterministic color from the first char of a name — handy for avatar backgrounds.

```js
generateColor('Alice')   // always the same color for "Alice"
```

### `generateRootColor(name, red, green, blue)` → CSS string

Generates a `:root { … }` block with 11 shades (-50 to -950) around an RGB seed. Drop into a `<style>` to expose `--{name}-50` … `--{name}-950` to Tailwind/components.

```js
const css = generateRootColor('brand', 33, 150, 243)
// :root {
//   --brand-50: 57 174 ... ;
//   ...
// }
```

### `colorPicker()` → `@easylogic/colorpicker`

Returns the `ColorPicker` constructor. Use to mount a picker programmatically.

```js
const ColorPicker = colorPicker()
const picker = new ColorPicker({ container: el, /* … */ })
```

---

## HTTP & URL

### `getUrl(path, isWeb?)` → `string`

Resolves a relative path against the API base (or web base if `isWeb`). Honors `tmp_server` / `tmp_web_server` cookies for ad-hoc overrides.

```js
getUrl('api/order/list')           // "<apiUrl>/api/order/list"
getUrl('/reports/print', true)     // "<webUrl>/reports/print"
getUrl('https://other.com/x')      // returned unchanged
```

### `getServerUrl(url)` → `string`

Same as `getUrl` but only resolves against the API base (no web override, no cookie).

### `getData({ url, method = 'post', filter = {}, signal }, fn)` → `Promise<void>`

Thin wrapper around `useHttp` that invokes `fn(data)` on success and toasts an error otherwise.

```js
await getData({
  url: 'api/order/list',
  method: 'POST',
  filter: { Status: 1 },
}, (rows) => { list.value = rows })
```

### `queryParams(key, value)` — multi-mode helper

| Call | Effect |
|---|---|
| `queryParams()` | Decode every param in the current URL — returns an object of decoded JSON values. |
| `queryParams('keep')` | Return the current query string (no edits). |
| `queryParams('foo', 'delete')` | Drop `foo`. Returns the new query string. |
| `queryParams('filter', obj)` | Encode `obj` as URL-safe base64 under `filter`. Returns the new query string. |

```js
const url = queryParams('filter', { branch: 1, from: '2026-01-01' })
router.push({ path: route.path + url })

const decoded = queryParams()  // { filter: { branch: 1, from: '...' } }
```

---

## App config & helpers

### `helper()` → `{ config, str, api, userData, ... }`

The global helper. Decrypts `OCAPPCONFIGURATION`, merges with `OC_CONFIGURATION` and `OC_HELPER`, attaches `userData`.

```js
const { config, str, api } = helper()
useHttp(api.order.list)
```

### `ocConfig()` → `{ config, str }`

Lightweight variant — only exposes config fields needed at boot (no userData / OC_HELPER).

### `bizHelper()` → `helper() + { currency, baseCurrency, baseSymbol, exchangeRate }`

Same as `helper()` plus the loaded currency list and today's exchange rate. **Top-level await** at import time — first import triggers `api/currency/list` + `api/exchange_rate/get`.

```js
const { baseSymbol, exchangeRate, currency } = bizHelper()
```

### `getHelper(layers)` → `Promise<{ api, str }>`

Build/composition-time loader that walks the given layer folders and deep-merges each `helpers/apis.js` and `helpers/str.js`. Used by the Nuxt module.

### `checkGlobal(obj)` → `{ icon, hasEvent }`

Decides whether to show the "global" badge and react to clicks based on the current database cookie and the record's `IsGlobal` flag.

```js
const { icon, hasEvent } = checkGlobal(row)
```

### `getCurrencyFlag(code)` → `path`

```js
getCurrencyFlag('usd')  // "/_kedtec/img/currencyFlags/USD.svg"
```

---

## Auth / user

### `getUserLogin()` → `{ UserId, UserType, Username, … }`

Decodes the current access token and maps the claim shorthands (`o:t`, `o:r`, `o:f`, …) to friendly keys.

```js
const me = getUserLogin()
console.log(me.UserId, me.UserType, me.UserFirstName)
```

### `saveUserData(data, dbCode, locale)`

Persists the OAuth response into `useUserData()` **and** broadcasts the relevant cookies (`oc_access_token`, `oc_database`, `oc_lang`, `oc_device_id`, `oc_expire`) to the authorize iframe.

```js
await saveUserData(oauthRes, 'KEDTEC', 'en')
```

### `getDeviceId()` → `string` (uuid v4)

Reads/creates a persistent device-id cookie (`expires` in 1 year).

### `createToken(token, locale, UserId, deviceId)` → `Promise<void>`

Registers a Firebase Messaging token with the backend (no-op if the token hasn't changed).

```js
await createToken(fbToken, locale.value, me.UserId, getDeviceId())
```

---

## Page lifecycle

### `mounted(handler)`

Same as `onMounted` with a 10 ms delay — gives Vue a tick to finish rendering before measurement code runs.

```js
mounted(() => {
  const rect = el.value.getBoundingClientRect()
})
```

### `onTabChange()`

Wires a one-time `visibilitychange` listener that logs tab focus changes. Wrap your own handler inline if you need behavior, or extend this file.

### `pagePath(page = '', index = false)` → `string`

Returns the **parent** route path (drops the last segment). If `index` is `true`, keeps the current path. Append a page name to navigate to a sibling.

```js
// On /school/student/detail
pagePath()              // "/school/student/"
pagePath('list')        // "/school/student/list"
pagePath('', true)      // "/school/student/detail/"
```

### `oauthHandler({ url, handleMessage, title })`

Opens a centered 590×800 popup at `url`, listens for `message` events, and cleans the listener up when the popup closes.

```js
oauthHandler({
  url: '/auth/google',
  title: 'Sign in with Google',
  handleMessage: (e) => {
    if (e.data?.access_token) finishLogin(e.data)
  },
})
```

### `previewReport(messageEvent)`

`postMessage` handler — when `event.data.action === 'preview report'`, base64-encodes the payload and opens `/Report/preview?data=…&fullPath=…` in a new tab.

```js
window.addEventListener('message', previewReport)
```

---

## File upload & cropping

### `new splitFileUpload().upload({ fileTarget, url, local, data, signal, complete, error, progress })`

Chunked upload at 1.5 MB per part; the server reassembles via `<file>.part_<n>.<total>` naming. Calls `progress(percent)` continuously, `complete(response)` per part, and `error(msg)` on any failure (aborts the loop).

```js
const uploader = new splitFileUpload()
await uploader.upload({
  fileTarget: file,
  url: 'api/file/upload',
  data: { OwnerId: studentId },
  signal: abortController.signal,
  complete: (r) => console.log('part done', r),
  progress: (p) => (percent.value = p),
  error: (msg) => toast.add({ color: 'red', title: msg }),
})
```

### `ocProfileCropper(file, { width, height, emit, uploadProfile })`

Opens a profile cropper modal (3:4 ratio, circle preview, 0.9 quality). Calls `emit('getFile', data)` when the user finishes, and flips `uploadProfile.value = true` on close.

```js
ocProfileCropper(file, { width: 480, height: 640, emit, uploadProfile })
```

### `useGoogleDrive()` → `{ getFileInfo, uploadFile, downloadFile, deleteFile }`

Auto-refreshing Google Drive client backed by `/api/google_drive/refresh_token`. Token cached in `localStorage` under `google-drive-access-token`.

```js
const drive = useGoogleDrive()
const info  = await drive.getFileInfo(fileId)
const id    = await drive.uploadFile(file, [parentFolderId])
const url   = await drive.downloadFile(fileId, bearerToken)
await drive.deleteFile(fileId)
```

> **Note:** This util uses a **named** export. Add `import { useGoogleDrive } from '~/utils/googleDrive'` if it isn't auto-resolved in your context.

---

## Realtime

### `ocSignalR({ hubName, onConnecting, onConnected, onDisconnected, onReconnecting, onError, isLocal })` → `OCSignalR`

Wraps SignalR over the legacy jQuery transport (loaded via `useScriptLoader(str.cdn_signalR)`). Auto-reconnects every 20 s when offline. `await instance.ready` before calling `connect()`.

```js
const hub = ocSignalR({
  hubName: 'notificationHub',
  onConnected: () => console.log('connected'),
  onError: (e) => console.warn(e),
})

await hub.ready

hub.connect(() => {
  hub.on('newOrder', (order) => orders.value.unshift(order))
})

// later
hub.invoke('Ping', { userId })
hub.disconnect()
```

| Member | Purpose |
|---|---|
| `await hub.ready` | Resolves once the SignalR client is initialized. |
| `hub.connect(cb?)` | Start the connection. `cb` runs every time the state becomes "connected". |
| `hub.on(fn, cb)` | Subscribe to a server-side hub method. |
| `hub.invoke(fn, data)` | Invoke a server-side hub method. |
| `hub.disconnect()` | Stop, clean up listeners, remove the loaded script. |
| `hub.isConnected` | Boolean shortcut. |

---

## Form helpers

### `validateSelect(parent, isClear)` — Select2 validator (jQuery-dependent)

Used with `.oc-selector-892753` Select2 elements.

```js
validateSelect()              // → true if every required select has a value
validateSelect('#myForm')     // scope check
validateSelect('#myForm', true)        // clear error states
validateSelect('#myForm', 'data')      // clear values AND error states
```

---

## Build-time (Nuxt module)

These run during Nuxt build/start, not at runtime — they use `fs`/`path` and walk the project tree.

### `getComponentPaths()` → `Array<{ path, extensions }>`

Returns the components config for `nuxt.config.ts`: `~/components` + every subfolder of it.

### `getLayersComponentPaths(layerPaths = [])` → `Array<{ path, extensions }>`

Like `getComponentPaths` but for sibling layer folders (e.g., `['NUXT_BASE_LAYER', 'NUXT_MODULE']`).

```ts
// nuxt.config.ts
components: getLayersComponentPaths(['NUXT_BASE_LAYER', 'KEDTEC_MODULE'])
```

### `getPages(pageDirNames)` → `string[]`

Walks one or more `pages/` directories and returns the static route paths (skips `[dynamic]` segments). Use to pre-render or seed `nitro.prerender.routes`.

### `ocExtendPages(runtimeDir)` → `Array<{ name, path, file, meta }>`

Same idea, but built for `extendPages()`: returns Vue Router records mirrored under `/km/*` for the Khmer locale, with `definePageMeta` extracted via regex.

### `copyJsonFile({ inputPaths, outputPath, deep = true })`

Merges N JSON files into one. Used to combine layer-level locale files at build time.

```js
copyJsonFile({
  inputPaths: ['module/assets/locale/en.json', 'app/assets/locale/en.json'],
  outputPath: 'app/locale/en.json',
})
```

### `createTsConfigFile(layers)`

Generates a `tsconfig.json` inside each sibling layer that extends the host project's `.nuxt/tsconfig.json`. Lets layers type-check against the host's resolved aliases.

---

## Conventions

- **All functions are default exports** named after the file — `isEmpty.js` → `isEmpty()`. Nuxt auto-import surfaces them globally.
- **Async utils throw via toast** rather than rejecting (`getData`, `bizHelper`'s child fetchers). Wrap with `try/catch` only when you need custom error UI.
- **Date utilities are i18n-aware** through `helper().config.dateFormat` and `dateDivider`.
- **Build-time utils** (`getComponentPaths`, `getPages`, `ocExtendPages`, `copyJsonFile`, `createTsConfigFile`) require `fs`/`path` and must run in a Node context (Nuxt module / `nuxt.config.ts`), not in the browser.