# OCTable

A feature-rich data table component built on top of [DataTables.net](https://datatables.net/) with Vue 3 integration. Supports server-side and client-side data, multi-sort, row reorder, responsive collapse, fixed columns, line-clamp detail modal, persisted state, row selection, card templates, and more.

Location: [components/OCTable.vue](OCTable.vue)

---

## Quick start

### Client-side data

```vue
<OCTable
  :data="list"
  :columns="columns"
  :options="{ paging: true, searching: true }"
/>

<script setup>
const list = ref([
  { Code: 'A001', Name: 'Alice', Age: 22 },
  { Code: 'A002', Name: 'Bob',   Age: 25 },
])
const columns = [
  { title: 'Code', data: 'Code' },
  { title: 'Name', data: 'Name' },
  { title: 'Age',  data: 'Age', alignRight: true },
]
</script>
```

### Server-side data (API)

```vue
<OCTable
  ref="tableRef"
  :columns="columns"
  :api="{
    url: 'api/student/list',
    method: 'POST',
    filter: { ClassCode: classCode }
  }"
  @update:data="onSendingPayload"
  @update:dataSrc="onSrcData"
/>
```

The request body posted by OCTable is shaped like:

```json
{
  "Pages": 1,
  "Records": 10,
  "Search": "",
  "OrderBy": "Code",
  "OrderDir": "asc",
  "...filter": "..."
}
```

The expected response is an array of rows. To get `total`, the first row may include `RecordCount`, `RecordsCount`, `RecordCounts`, or `TotalRecord` — otherwise `total = list.length`.

If the response is wrapped: pass `api.where` to unwrap (`data = response[where]`), or have the API return `{ dataSrc: [...] }`.

---

## Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `data` | `Array` | `[]` | Client-side rows. Ignored if `api` is set. |
| `api` | `String \| Object` | `undefined` | Either a URL string, or `{ url, method, filter, where }`. Triggers server-side mode. |
| `columns` | `Array<ColumnDef>` | `[]` | Column definitions. See [Column definition](#column-definition). |
| `keyTable` | `any` | — | Change this value to force re-render the table (e.g. tab switching). |
| `options` | `Object` | see [Options](#options) | DataTables.net options + component options (deep-merged). |
| `showTemplate` | `Boolean` | `false` | Switches to card/grid template rendering via `#template` slot. |
| `selectedPk` | `String` | — | Primary key field used for row selection (renders checkbox column). |
| `selectAllPk` | `String` | — | When the API response contains a list of IDs at `data[selectAllPk]` to support "Select all across pages". |
| `hideGroupBtnSelect` | `Boolean` | `false` | Hide the group selection toolbar buttons. |
| `waitFilter` | `Boolean` | — | (Reserved) hold loading until parent triggers `filter()`. |
| `isTotalDetail` | `Boolean` | `false` | Hide the footer and rely on `#totalDetail` slot instead. |
| `haveBorder` | `Boolean` | `false` | Show outer border on the table container. |
| `minWidth` | `Number` | `768` | Threshold below which scroll-height calculation is skipped (mobile). |
| `stateKey` | `String` | — | If set, table state (page, search, order, rows) is persisted to `useOCTableDataStore` under this key. |
| `autoload` | `Boolean` | `true` | If `false`, skip the initial API fetch — caller must invoke `reload()` / `filter()`. |
| `reRenderNotClearData` | `Boolean` | `false` | When `keyTable` changes, keep existing rows instead of clearing them. |
| `loading` (v-model) | `Boolean` | `false` | Two-way bound loading flag. |
| `modelValue` (v-model) | `Array` | `[]` | The selected rows / IDs (works with `selectedPk`). |

### Column definition

```ts
interface ColumnDef {
  title: string
  data: string                 // field name in the row object
  className?: string
  orderable?: boolean
  hidden?: boolean             // start hidden
  alignRight?: boolean         // right-align numeric/currency style
  isLineClamp?: boolean        // 2-line clamp + auto modal preview when overflowing
  render?: string | Function   // "#slotName" to use a named slot, or a DataTables render fn
  sortData?: Array<{ data: string, title: string }>  // multi-sort popover
}
```

Use `render: '#mySlot'` to delegate cell rendering to a `<template #mySlot="props">` slot. `props.cellData` is the row object, `props.rowData` alias also works.

### Options

Defaults (deep-merged with caller's `options`):

```js
{
  isScroll: true,        // calculate scroll height; false = let it grow naturally
  extraHeight: 0,        // tweak the auto-height calculation
  scrollY: 300,          // min scroll body height
  scrollX: false,
  info: true,
  destroy: true,
  retrieve: true,
  searching: true,
  paging: true,
  reload: true,          // show the reload button in header
  order: { idx: 0, dir: '' },  // default ordering
  responsive: { ... },
  lengthMenu: [10, 25, 50, 100],
  pageLength: 10,        // 1000 when in API mode
  showNoData: true,
  fixedColumns: undefined,     // pass-through to DataTables FixedColumns plugin
}
```

---

## Events

| Event | Payload | Fires when |
|---|---|---|
| `update:data` | `dataSender` | Right before an API request — mutate to inject custom filters. |
| `update:dataSrc` | `data.value` | After API response, before assigning to `resultData`. |
| `update:loading` | `Boolean` | v-model:loading. |
| `update:modelValue` | `Array` | v-model selection list. |
| `actionCheck` | `row \| Array` | Row checkbox toggled (or select-all). |
| `reloadByFixedData` | — | Reload requested while in client-side mode. |
| `onReload` | — | Reload button clicked. |
| `onOrder` | `order` | Column ordering changed. |
| `onSaveState` | `state` | Right before pushing state into `ocTableStore`. |
| `onChangePagination` | `page, record` | Pagination changed. |
| `onSelectRecord` | `record, page` | Page-size (records-per-page) changed. |
| `onSearching` | `search` | Search input changed (or cleared). |
| `onRowReorder` | `event, diff, edit` | Row drag-reorder finished. |

---

## Slots

### Cell slots
Any value used as `render: '#name'` in a column becomes a slot:

```vue
<template #Status="{ cellData }">
  <OCStatus :text="cellData.Status" />
</template>
```

Special built-in slot names:
- `#actions` — rendered as the trailing "Action" column when the parent provides `<template #actions>`.
- `#checkbox` — auto-generated when `selectedPk` is set (you usually don't override this).
- `#tdNull` — collapse-control column.

### Layout slots

| Slot | Purpose |
|---|---|
| `headerTable` | Block above the header toolbar (e.g., breadcrumbs, title). |
| `headerLeft` / `headerCenter` / `headerRight` | Inside the header toolbar. |
| `headerApplyFilter` | Filter chips area. |
| `middleTable` | Between header and table body. |
| `loading` | Custom loader instead of the default. |
| `template` | Card/grid view (requires `showTemplate=true`). Receives `[dataTemplate]`. |
| `totalDetail` | Footer-side detail block; receives `{ total }`. Setting this hides the standard footer. |
| `ShowDetailAfterTableOne` / `ShowDetailAfterTableTwo` | Extra blocks below the table (their height is subtracted from scroll height). |
| `footerLeft` / `footerCenter` | Inside the footer toolbar. |

---

## Exposed methods (via `ref`)

```vue
<OCTable ref="tableRef" ... />

// inside <script setup>
const tableRef = ref()

tableRef.value.dt()                          // raw DataTables instance
tableRef.value.reload(currentPage=true, isNotCalc=false)
tableRef.value.filter(isNotCalc=false)       // resets to page 1 and reloads
tableRef.value.responsive()                  // re-run responsive/height calc
tableRef.value.setListData(rows, isNotCalc)  // replace rows (client-side)
tableRef.value.addRowData(row, isNotCalc)    // prepend a row
tableRef.value.updateRowData(row, pk, isNotCalc)  // patch by primary key
tableRef.value.setTotal(list)                // recompute total from a list
tableRef.value.setPage(n)                    // jump to page n
```

---

## Patterns

### 1. Server-side with reactive filter

```vue
<OCTable
  ref="tableRef"
  :columns="columns"
  :api="{ url: 'api/order/list', method: 'POST', filter }"
/>

<script setup>
const tableRef = ref()
const filter = reactive({ FromDate: '', ToDate: '', Status: '' })

function onApply() {
  tableRef.value.filter()  // resets page to 1 and re-fetches
}
</script>
```

### 2. Inject extra payload right before request

```vue
<OCTable :api="api" @update:data="injectExtras" />

<script setup>
function injectExtras(payload) {
  payload.BranchCode = currentBranch.value
  payload.IncludeArchived = showArchived.value
}
</script>
```

### 3. Selectable rows

```vue
<OCTable
  v-model="selected"
  :data="rows"
  :columns="columns"
  selectedPk="Code"
/>

<script setup>
const selected = ref([])           // entire row objects
</script>
```

For server-side "select all across pages", have the API return an `IDs` array and:

```vue
<OCTable
  v-model="selected"
  :api="api"
  :columns="columns"
  selectedPk="Code"
  selectAllPk="IDs"
/>
```

### 4. Action column

```vue
<OCTable :data="rows" :columns="columns">
  <template #actions="{ cellData }">
    <UButton icon="i-ri-edit-line" @click="onEdit(cellData)" />
    <UButton icon="i-ri-delete-bin-line" color="red" @click="onDelete(cellData)" />
  </template>
</OCTable>
```

The Action column is auto-appended when the `actions` slot is present.

### 5. Multi-sort popover on one column

```js
const columns = [
  { title: 'Code', data: 'Code' },
  {
    title: 'Date',
    data: 'CreatedAt',
    sortData: [
      { data: 'CreatedAt', title: 'Created' },
      { data: 'UpdatedAt', title: 'Updated' },
    ],
  },
]
```

### 6. Line-clamp with auto detail modal

```js
{ title: 'Description', data: 'Description', isLineClamp: true }
```

When the cell content overflows two lines, clicking opens a modal with the full text.

### 7. Persisted state across navigation

```vue
<OCTable
  :api="api"
  :columns="columns"
  stateKey="orders.list"
/>
```

Page, search, order, and the last data set are stashed in `useOCTableDataStore`. Re-mounting the same `stateKey` rehydrates without an extra request.

### 8. Card / template mode

```vue
<OCTable :data="rows" :columns="columns" :showTemplate="true">
  <template #template="[rows]">
    <div class="grid grid-cols-3 gap-3">
      <Card v-for="r in rows" :key="r.Code" :item="r" />
    </div>
  </template>
</OCTable>
```

### 9. Drag-reorder rows

```vue
<OCTable
  :data="rows"
  :columns="columns"
  :options="{ rowReorder: true }"
  @onRowReorder="onReorder"
/>
```

---

## Tips

- **Server-side mode disables DataTables internal paging/search/order** — those events are routed through OCTable and forwarded to your API via the `dataSender` payload.
- **`stateKey` + `autoload: false`** is the pattern when you want full control over when the first request fires (e.g., wait for a filter to be valid).
- The default `pageLength` jumps to `1000` in API mode because the server already paginates — keep `Records` aligned via `lengthMenu`.
- For modals containing tables, set `:options="{ scrollY: 'whatever' }"` — height auto-calc is skipped inside `.oc-modal-container`.
- If a column's content is purely numeric/currency, set `alignRight: true` for proper alignment with order indicators.