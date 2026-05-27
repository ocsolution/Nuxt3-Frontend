# OCFormCreateModal

A drawer-based create/update form. It wraps [`OCFormCreate`](OCFormCreate.vue) inside an `OCDrawer` and ships its own header (auto title) and footer (Cancel / Reset / Submit buttons). It detects update vs. create mode automatically based on `data.Id`.

Location: [components/form/OCFormCreateModal.vue](OCFormCreateModal.vue)

---

## Quick start

```vue
<template>
  <OCBtn label="New Room" @click="onCreate" />

  <OCFormCreateModal
    v-model:show="openDrawer"
    v-model:data="dataModel"
    :title="textHeader"
    :inputs="drawInput"
    :form="{
      showReset: true,
      api: {
        create: 'api/room/create',
        update: { url: 'api/room/update', method: 'PUT' },
      },
      confirmUpdate: $t('confirm_update_room'),
    }"
    @onSuccess="reloadTable"
  />
</template>

<script setup>
import { z } from 'zod'

const openDrawer = ref(false)
const textHeader = ref('')
const dataModel  = ref()

const drawInput = [
  { type: 'text', name: 'Code', label: 'Code', required: true },
  { type: 'text', name: 'Name', label: 'Name', required: true,
    schema: z.string().min(2, 'At least 2 chars') },
  { type: 'textarea', name: 'Description', label: 'Description' },
]

function onCreate() {
  textHeader.value = 'Create Room'
  dataModel.value  = undefined           // no Id => create mode
  openDrawer.value = true
}

function onEdit(row) {
  textHeader.value = 'Edit Room'
  dataModel.value  = { ...row }          // contains Id => update mode
  openDrawer.value = true
}

function reloadTable() {
  // refresh your table here
}
</script>
```

---

## How create vs update is decided

```js
isUpdate = isClone ? false : isNotEmpty(dataUpdate.value?.Id)
```

- `data` has an `Id` → **update** mode → hits `form.api.update`, submit button label = `update`.
- `data` has no `Id` → **create** mode → hits `form.api.create`, submit button label = `save`.
- `isClone="true"` → **clone** mode → hits `form.api.create`, submit button label = `clone`.

---

## Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `show` (v-model) | `Boolean` | — | Drawer open state. |
| `data` (v-model) | `Object` | — | Form data. Include `Id` to switch to update mode. |
| `title` | `String` | — | Header text. Falls back to `update_form` / `create_form` i18n keys. |
| `inputs` | `Array` | — | Field definitions for `OCFormCreate` (flat inputs). |
| `tabs` | `Object` | — | Tabbed input layout (alternative to `inputs`). See [`OCFormCreate`](OCFormCreate.vue). |
| `form` | `Object` | — | Form configuration (see [Form config](#form-config)). |
| `isClone` | `Boolean` | `false` | Force clone mode (creates a new record using `data` as initial values). |
| `disabledUpdate` | `Boolean` | `false` | Disable Cancel and Submit (e.g., during external validation). |

### Form config

The `form` prop is forwarded to `OCFormCreate.form`. The modal injects `showSubmit: false`, `showReset: false`, `showCancel: false` so the footer buttons are driven by the modal itself — your `form.showSubmit` / `form.showReset` / `form.showCancel` toggles still control **visibility of the modal footer buttons** via `conditionShowBtn` (`undefined` / `null` / `""` ⇒ shown, `false` ⇒ hidden).

| Field | Type | Description |
|---|---|---|
| `api.create` | `String \| Object` | Endpoint when creating. Object form: `{ url, method }`. |
| `api.update` | `String \| Object` | Endpoint when updating. |
| `width` | `Number` | Drawer width in px (max-width applied). Default `320`. |
| `showSubmit` | `Boolean` | Hide the submit button if `false`. |
| `showReset` | `Boolean` | Hide the reset button if `false`. |
| `showCancel` | `Boolean` | Hide the cancel button if `false`. |
| `showErrorMassage` | `Boolean` | Toast on server error. |
| `showSuccessMessage` | `Boolean` | Toast on success. |
| `confirmUpdate` | `String` | If set, shows a confirm dialog with this message before update. |

---

## Events

| Event | Payload | When |
|---|---|---|
| `onSubmitted` | `formData` | After validation passes, before API request (proxied from `OCFormCreate`). |
| `onCancel` | — | Cancel button clicked or drawer closed. |
| `onReset` | — | Reset button clicked. |
| `onLoading` | `Boolean` | Submit request in-flight. |
| `onError` | `error` | API request errored. |
| `onSuccess` | `response` | API request succeeded — the modal also auto-closes. |

---

## Exposed methods (via `ref`)

```vue
<OCFormCreateModal ref="modalRef" ... />

modalRef.value.jumpTo(index)             // switch tab (when using tabs)
modalRef.value.disabledTab(i, disabled)  // disable/enable a tab
modalRef.value.refInput()                // get refs to individual inputs (for focus/programmatic ops)
```

---

## Patterns

### 1. Toggle between create and update

```vue
<OCFormCreateModal
  v-model:show="open"
  v-model:data="model"
  :inputs="inputs"
  :form="form"
  @onSuccess="reload"
/>

<script setup>
const open  = ref(false)
const model = ref()

function openCreate() {
  model.value = undefined
  open.value  = true
}

function openEdit(row) {
  model.value = { ...row }        // Id present → update mode
  open.value  = true
}
</script>
```

### 2. Clone existing record

```vue
<OCFormCreateModal
  v-model:show="open"
  v-model:data="model"
  :inputs="inputs"
  :form="form"
  isClone
/>
```

`data` keeps its values but `Id` is ignored — submit hits `api.create` and the button reads "Clone".

### 3. Tabbed form

```vue
<OCFormCreateModal
  v-model:show="open"
  v-model:data="model"
  :form="form"
  :inputs="[]"
  :tabs="{
    defaultIndex: 0,
    items: [
      { label: 'General', inputs: [
        { type: 'text', name: 'Code', label: 'Code', required: true },
        { type: 'text', name: 'Name', label: 'Name', required: true },
      ]},
      { label: 'Pricing', inputs: [
        { type: 'number', name: 'Price', label: 'Price' },
      ]},
    ],
  }"
/>

<script setup>
const modalRef = ref()
// jump to a tab from outside:
modalRef.value.jumpTo(1)
</script>
```

### 4. Confirm before update

```js
const form = {
  api: { create: 'api/room/create', update: 'api/room/update' },
  confirmUpdate: t('are_you_sure_update_room'),
}
```

A confirm dialog is shown before the update request fires (handled inside `OCFormCreate`).

### 5. Disable submit during external check

```vue
<OCFormCreateModal
  v-model:show="open"
  v-model:data="model"
  :inputs="inputs"
  :form="form"
  :disabledUpdate="codeAlreadyExists"
/>
```

### 6. Custom width

```js
const form = {
  width: 720,                              // → max-w-[720px]
  api: { create: '...', update: '...' },
}
```

### 7. Hide a footer button

```js
const form = {
  showReset: false,                        // hide the Reset button
  api: { ... },
}
```

> The footer buttons in this modal use `conditionShowBtn`: an explicit `false` hides the button. Leaving the flag undefined keeps it visible.

---

## Tips

- Keep your `data` model **without `Id`** for create mode. Spreading a row (`{ ...row }`) is the easiest way to enter update mode.
- The drawer is `prevent-close`, so backdrop clicks won't accidentally drop unsaved edits — use the Cancel button (or set `show = false`).
- On success the modal closes automatically via `successForm → cancelForm` — listen to `@onSuccess` to refresh the parent table.
- For complex validation rules, attach `schema` (Zod) directly to individual inputs — same as plain `OCFormCreate`.