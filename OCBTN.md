# `btn/` — Button components

All button-style components live in [src/runtime/components/btn/](.). This single doc covers every component in the folder: prop signatures, slots, events, and a tested usage snippet for each.

## Component map

| Component | Purpose |
|---|---|
| [OCBtn](#ocbtn) | Primary action button. Type-aware icons + colors, permission gating, loading state. |
| [OCBtns](#ocbtns) | Render a list of buttons; auto-collapses to an icon row + dropdown on small screens. |
| [OCBtnList](#ocbtnlist) | Single row item used inside dropdowns and menus. |
| [OCBtnListGrid](#ocbtnlistgrid) | Toggle between list/grid view (icon + tooltip). |
| [OCBtnDropdown](#ocbtndropdown) | Popover with a list of action items. |
| [OCBtnSwitchLang](#ocbtnswitchlang) | Language switcher icon + popover. |
| [OCTblBtns](#octblbtns) | Per-row action buttons for tables; overflow moves to a "more" dropdown. |
| [BtnDropdown](#btndropdown) | Headless-UI dropdown menu primitive. |
| [BtnIcon](#btnicon) | Plain icon-only round button. |
| [BtnBack](#btnback) | Page header back button with optional sub-label / breadcrumb. |
| [BtnCollapse](#btncollapse) | Sidebar collapse/expand toggle image. |
| [BtnHeaderAction](#btnheaderaction) | Composite app header right-side actions (theme, lang, notifications, module switcher). |
| [BtnSwitchMode](#btnswitchmode) | Light/dark mode toggle. |
| [BtnSwitchModule](#btnswitchmodule) | Switch between Main system ↔ Teacher module. |
| [BtnSwicthOnFooterNavbav](#btnswicthonfooternavbav) | Footer nav layout slot host with `left` / `center` / `right` / `IconCollapsed`. |
| [BtnShowUploadProcess](#btnshowuploadprocess) | Upload progress popover button. |
| [GroupImage](#groupimage) | Avatar group with hover-popover details. |

---

## Common conventions

### Type-driven defaults

`OCBtn`, `OCBtns`, `OCBtnList`, and `OCTblBtns` all share the same **type → icon + color** lookup. Setting `type="edit"` picks `ri-pencil-line` and green automatically. You only pass `icon`/`color` when you need to override.

### Canonical color palette

> Use **only** the types in this table. The same key drives the icon, the resting fill, and the hover state — keeping every button visually consistent across the app.

| Type | Color | CSS token / hex | Default icon | Typical action |
|---|---|---|---|---|
| `disabled` | gray (state) | `--color-w-b-4` bg, `--color-w-b-3` text | — | Auto-applied when `disabled` or `loading` is true. |
| `change` | blue | `--ocs-c-blue` | `ri-exchange-2-line` | Swap / replace / switch. |
| `return` | nenoBlue | `--ocs-c-nenoBlue` | `ri-arrow-go-back-line` | Return / go-back action. |
| `add` | nenoBlue | `--ocs-c-nenoBlue` | `ri-add-circle-line` | Add a related item. |
| `create` | nenoBlue | `--ocs-c-nenoBlue` | `ri-add-circle-line` | Create a new record. |
| `sub-payment` | nenoBlue | `--ocs-c-nenoBlue` | `ri-coins-fill` | Sub / partial payment. |
| `view` | nenoBlue | `--ocs-c-nenoBlue` | `ri-eye-line` | View detail. |
| `edit` | green | `--ocs-c-green` | `ri-pencil-line` | Edit an existing record. |
| `post` | green | `--ocs-c-green` | `ri-check-line` | Post / commit. |
| `approve` | green | `--ocs-c-green` | `ri-check-line` | Approve. |
| `helper` | orange | `--ocs-c-orange` | `ri-customer-service-2-fill` | Help / support. |
| `print` | orange | `--ocs-c-orange` | `ri-printer-line` | Print. |
| `chat` | orange | `--ocs-c-orange` | `ri-chat-3-line` | Open chat. |
| `void` | red | `--ocs-c-red` | `ri-forbid-2-line` | Void a record. |
| `delete` | red | `--ocs-c-red` | `ri-delete-bin-3-line` | Delete. |
| `run` | orangeRed | `--ocs-c-orangeRed` | `ri-play-line` | Run / execute. |
| `setting` | purple | `--ocs-c-purple` | `ri-settings-4-line` | Settings. |
| `copy` | purple | `--ocs-c-purple` | `ri-file-copy-line` | Copy. |
| `clone` | purple | `--ocs-c-purple` | `ri-file-copy-2-line` | Clone a record. |
| `release` | capture | `--ocs-c-capture` | `ri-corner-up-right-line` | Release / hand-off. |
| `more` | gray | `--color-w-b-3` | `ri-more-2-fill` | Overflow menu trigger. |
| `hold` | gray | `--color-w-b-3` | `ri-hand` | Put on hold. |
| `employee` | sage | `#659277` | — | Employee-related action. |
| `folder` | sage | `#659277` | `ri-folder-2-line` | Folder / grouping. |
| `change-en` | sky | `#4395f1` | — | Switch to English. |
| `formula` | olive | `#9cba54` | — | Edit formula. |
| `resume` | iOS blue | `#007aff` | — | Resume / continue. |
| `split` | pink | `#db34a1` | `ri-file-reduce-line` | Split. |
| `merge` | indigo | `#3455db` | `ri-merge-cells-horizontal` | Merge. |
| `message` | leaf green | `#4caf50` | `ri-message-3-line` | Send message. |
| `next` | cyan | `#4EC3F7` | `ri-arrow-right-s-line` | Next step. |

All entries support both `variant="solid"` (filled background) and `variant="outline"` (border + hover-fill in the same color). The `disabled` state overrides whichever color you pick.

### Permission gating

These buttons read permissions from `useMenuStore().getActionPermission(permission)`:

- `permission`: the action key (e.g., `"order.create"`); the menu store returns `0` (deny) or `1` (allow).
- `showIfAllowed`: `true` (default) hides the button when denied; clicks on a denied button show a toast.
- Passing `permission` as `0` or `1` directly bypasses the store and forces deny/allow.

### Variants and sizes

`variant`: `solid` (default) | `outline`
`size`: `''` (36px) | `s` (33px) | `xs` (30px)

---

## OCBtn

The primary button. File: [OCBtn.vue](OCBtn.vue).

### Props

| Prop | Type | Notes |
|---|---|---|
| `type` | `String` | Drives default icon, color, and (when `label` is unset) capitalized fallback label. |
| `label` | `String` | Button text. Defaults to capitalized `type`. |
| `icon` | `String` | Override the icon class. |
| `color` | `String` | Override color. **Must be one of the keys** from the [canonical palette](#canonical-color-palette) (e.g., `edit`, `delete`, `setting`). |
| `variant` | `'solid' \| 'outline'` | Default `solid`. |
| `size` | `'' \| 's' \| 'xs'` | |
| `loading` | `Boolean` | Replaces icon with spinner; disables clicks. |
| `disabled` | `Boolean` | |
| `noIcon` | `Boolean` | Hide icon column. |
| `isIconRight` | `Boolean` | Render icon on the right. |
| `isLabelBold` | `Boolean` | Use bold font. |
| `typeButton` | `String` | DOM `type` attribute (`button` default, `submit`, …). |
| `permission` | `String \| 0 \| 1` | See [Permission gating](#permission-gating). |
| `showIfAllowed` | `Boolean` | Default `true`. |

### Events

- `click` — emitted only when the user has permission. Denied clicks emit a yellow permission toast.

### Slots

- `#${type}icon` (e.g. `#editicon`, `#deleteicon`) — replace the icon for a given type.

### Usage

```vue
<OCBtn type="edit" @click="onSave" :loading="saving" />
<OCBtn type="delete" variant="outline" size="s" @click="onDelete" />
<OCBtn label="Generate" icon="ri-bar-chart-box-ai-line" color="setting" @click="onRun" />
<OCBtn type="create" permission="order.create" @click="open = true" />
```

---

## OCBtns

Renders an array of buttons. On small screens it collapses to a `BtnDropdown`; on `noDropdown` it falls back to an icon-only row using the same icon map. File: [OCBtns.vue](OCBtns.vue).

### Props

| Prop | Type | Notes |
|---|---|---|
| `data` | `Array<ButtonDef>` | List of buttons (same shape as `OCBtn` props plus `class`, `isRotate`, `fill`). |
| `responsive` | `'isMobile' \| 'isTablet' \| 'isDesktop' \| 'isLaptopL' \| 'isCustom'` | Which `useScreenStore` flag drives the collapse. Default: collapse when `screen.isMobile`. |
| `noDropdown` | `Boolean` | Force icon-only row (no overflow dropdown). |

### Events

- `click(type, item)` — fired on selection (button or dropdown row).

### Slots

- `#${item.type}icon` — custom icon for a given action.

### Usage

```vue
<OCBtns
  :data="[
    { type: 'create', label: t('create') },
    { type: 'print',  label: t('print') },
    { type: 'delete', label: t('delete'), disabled: !hasSelection },
  ]"
  @click="onAction"
/>

<script setup>
function onAction(type, item) {
  if (type === 'create') openCreate()
  if (type === 'print')  doPrint()
  if (type === 'delete') doDelete()
}
</script>
```

---

## OCBtnList

One row used inside dropdowns/menus. File: [OCBtnList.vue](OCBtnList.vue).

### Props

`type`, `icon`, `color`, `label`, `disabled`, `permission`, `showIfAllowed` — same semantics as `OCBtn`.

### Events

- `click`

### Slots

- `#${type}icon`

### Usage

```vue
<OCBtnList type="edit"   @click="onEdit"   />
<OCBtnList type="delete" @click="onDelete" :disabled="locked" />
```

---

## OCBtnListGrid

Toggles between list and grid layout (icon swaps and a tooltip describes the next state). File: [OCBtnListGrid.vue](OCBtnListGrid.vue).

### Models / Events

- `v-model` — `Boolean` (`true` = grid mode).
- `@onClick` — fired after the toggle.

### Usage

```vue
<OCBtnListGrid v-model="isGrid" @onClick="rerender" />
```

---

## OCBtnDropdown

A popover (`OCPopover`) that renders an item list with icons. File: [OCBtnDropdown.vue](OCBtnDropdown.vue).

### Props

| Prop | Notes |
|---|---|
| `items` | `[{ type, icon, label, color, disabled, permission, showIfAllowed }]` — `icon: 'add'`/`'completed'`/`'incompleted'` are mapped internally. |
| `use` | `'nuxtui'` (default) or whatever popover backend `OCPopover` supports. |
| `placement` | Popper placement; default `left-start`. |

### Events

- `click(type)` — when an enabled item is selected.

### Slots

- `#trigger` — custom trigger element.

### Usage

```vue
<OCBtnDropdown
  :items="[
    { type: 'edit',   icon: 'edit',   label: 'Edit'   },
    { type: 'delete', icon: 'delete', label: 'Delete' },
  ]"
  @click="onPick"
>
  <template #trigger>
    <BtnIcon icon="ri-more-2-fill" />
  </template>
</OCBtnDropdown>
```

---

## OCBtnSwitchLang

Language switcher icon + popover (reads `useI18n()` locales). File: [OCBtnSwitchLang.vue](OCBtnSwitchLang.vue).

### Props

- `layoutType` — passed through for theming context.
- `color` — background of the popover.

### Usage

```vue
<OCBtnSwitchLang layout-type="main" />
```

---

## OCTblBtns

Per-row action buttons for tables. Up to N primary actions render as icon buttons, the rest collapse to a "more" `BtnDropdown`. File: [OCTblBtns.vue](OCTblBtns.vue).

### Props

| Prop | Notes |
|---|---|
| `items` | `[{ icon, label, color, disabled, loading, fill, isRotate, class, placement, permission, showIfAllowed, isMore }]`. Items with `isMore: true` go into the overflow dropdown. |
| `closeInside` | Close popover on item click. |
| `showDropDown` | Force showing the "more" dropdown even with one action. |

### Events

- `actionClick(item)` — fired on selection.

### Usage

```vue
<OCTblBtns
  :items="[
    { icon: 'edit',   label: t('edit'),    permission: 'order.update' },
    { icon: 'delete', label: t('delete'),  permission: 'order.delete' },
    { icon: 'print',  label: t('print'),   isMore: true },
    { icon: 'copy',   label: t('copy'),    isMore: true },
  ]"
  @action-click="onAction"
/>
```

Use this inside an `OCTable` action slot:

```vue
<template #actions="{ cellData }">
  <OCTblBtns :items="rowActions(cellData)" @action-click="(a) => doAction(a, cellData)" />
</template>
```

---

## BtnDropdown

Low-level dropdown menu (Headless UI). Used internally by `OCBtns` and `OCTblBtns`. File: [BtnDropdown.vue](BtnDropdown.vue).

### Props

| Prop | Type | Default |
|---|---|---|
| `items` | `Array` | `[]` |
| `noBorder` | `Boolean` | `false` |
| `popper` | `Object` | `{}` — `{ placement, offsetDistance, offsetSkid }` |
| `ui` | `Object` | `{}` — `{ container, width, size, round, transition }` |
| `useInTable` | `Boolean` | `true` |
| `disabled` | `Boolean` | `false` |

### Models / Events

- `v-model` — open state (`Boolean`).
- `@selected(item)` — selected item.

### Slots

- default — custom trigger button content.
- `#items="{ items, onSelect }"` — fully custom item rendering.

### Usage

```vue
<BtnDropdown
  :items="actions"
  :ui="{ container: 'z-50', width: 'w-fit', size: 's', round: 's' }"
  :popper="{ placement: 'left-start', offsetDistance: 10 }"
  @selected="onPick"
>
  <i class="ri-more-2-fill" />
</BtnDropdown>
```

---

## BtnIcon

Round icon-only button (wraps `UButton`). File: [BtnIcon.vue](BtnIcon.vue).

### Props

- `icon` — RemixIcon class (e.g. `ri-pencil-line`).

### Slots

- `#path` — replace the icon body with custom markup (e.g. an `<img>`).

### Usage

```vue
<BtnIcon icon="ri-pencil-line" @click="edit" />
<BtnIcon icon="hidden">
  <template #path><img :src="flag" /></template>
</BtnIcon>
```

---

## BtnBack

Page header back button + optional sub-label / breadcrumb block. File: [BtnBack.vue](BtnBack.vue).

### Props

```ts
data: {
  label: string,             // HTML allowed
  subLabel?: string,
  hasBack?: boolean,
  toolTip?: string,
  breadcrumb?: { items, onClick }
}
```

### Usage

```vue
<BtnBack :data="{
  hasBack: true,
  label: $t('order_detail'),
  subLabel: orderCode,
  breadcrumb: {
    items: [{ label: $t('orders'), to: '/orders' }, { label: orderCode }],
    onClick: (to) => router.push(to),
  },
}" />
```

---

## BtnCollapse

Sidebar collapse/expand icon (light or dark asset). File: [BtnCollapse.vue](BtnCollapse.vue).

### Props

- `toSmall` — presence (any value) shows the "collapse to small" image; absent shows "expand to large".

### Usage

```vue
<BtnCollapse v-if="sidebarOpen"  toSmall />
<BtnCollapse v-else />
```

---

## BtnHeaderAction

Top-bar right side: notifications, module switcher, theme toggle, lang switcher, logout, plus an optional dynamic component slot from `useLayoutStore()`. File: [BtnHeaderAction.vue](BtnHeaderAction.vue).

### Props

- `layoutType` — passed down to children (`BtnSwitchMode`, `OCBtnSwitchLang`, `ModalLogOut`).

### Slots

- default — logout modal body.
- `#qr` — optional QR section between module switcher and theme toggle.

### Usage

```vue
<BtnHeaderAction layout-type="main">
  <template #qr><AppQRPopover /></template>
  <template #default>Are you sure you want to log out?</template>
</BtnHeaderAction>
```

---

## BtnSwitchMode

Theme toggle. Reads `useColorMode()` and `useModeStore()`. File: [BtnSwitchMode.vue](BtnSwitchMode.vue).

### Usage

```vue
<BtnSwitchMode />
```

---

## BtnSwitchModule

Switches between root and `/teacher` routes. File: [BtnSwitchModule.vue](BtnSwitchModule.vue).

### Props

- `type` — `'teacher'` (currently in teacher module) or `'main'`.

### Usage

```vue
<BtnSwitchModule type="main" />     <!-- click → go to /teacher -->
<BtnSwitchModule type="teacher" />  <!-- click → go to / -->
```

---

## BtnSwicthOnFooterNavbav

Footer navbar layout host. Provides four named slots. File: [BtnSwicthOnFooterNavbav.vue](BtnSwicthOnFooterNavbav.vue).

### Props

- `hasEvent` — boolean attribute; when truthy adds `hasEvent` class for interactive styling.

### Slots

`left`, `center`, `right`, `IconCollapsed`.

### Usage

```vue
<BtnSwicthOnFooterNavbav hasEvent>
  <template #left><BtnIcon icon="ri-home-2-line" /></template>
  <template #center><span>Main</span></template>
  <template #right><BtnIcon icon="ri-settings-3-line" /></template>
  <template #IconCollapsed><BtnCollapse toSmall /></template>
</BtnSwicthOnFooterNavbav>
```

---

## BtnShowUploadProcess

Cloud-upload popover anchored to a `BtnIcon`. Reads upload state from your upload store. File: [BtnShowUploadProcess.vue](BtnShowUploadProcess.vue). No props.

### Usage

```vue
<BtnShowUploadProcess />
```

---

## GroupImage

Stacked avatars; clicking shows a list of profiles in a popover. Used for "assigned employees", "students in class", etc. File: [GroupImage.vue](GroupImage.vue).

### Props

| Prop | Default | Notes |
|---|---|---|
| `items` | `[]` | List of records. |
| `path` | `'EmployeePathImage'` | Field on each item for the image URL. |
| `name` | `'EmployeeName'` | KH-name field. |
| `englishName` | `'EmployeeEnglishName'` | EN-name field. |
| `code` | `'EmployeeCode'` | Sub-title field. |
| `gender` | `'EmployeeGender'` | Drives default avatar. |
| `title` | — | Popover heading. |
| `placement` | `'bottom-start'` (or `'bottom'` if `use='flowbite'`) | |
| `use` | `'nuxtui'` | Popover backend. |
| `maxSrc` | `3` | Max stacked avatars. |
| `size` | — | Avatar size. |
| `background` | `'content'` | Stack background. |
| `isNotDefaultOne` | `false` | Force popover even when only one item. |

### Usage

```vue
<GroupImage
  :items="employees"
  title="Assigned Team"
  size="m"
/>

<!-- with custom field names -->
<GroupImage
  :items="students"
  path="StudentPhotoUrl"
  name="StudentNameKh"
  english-name="StudentNameEn"
  code="StudentCode"
  gender="Gender"
  :max-src="5"
  title="Students"
/>
```

---

## Picking the right component

```
┌─ Need a single labeled action with a known type? .........  OCBtn
├─ A toolbar row that should collapse responsively? .........  OCBtns
├─ Per-row actions inside a table? ..........................  OCTblBtns
├─ Icon trigger with a menu of choices? .....................  OCBtnDropdown (or BtnDropdown)
├─ Menu item used inside one of the above? ..................  OCBtnList
├─ Plain icon-only button (no menu)? ........................  BtnIcon
├─ Header chrome: theme/lang/logout/module switch ............  BtnHeaderAction
├─ "Back" affordance on a page header .......................  BtnBack
└─ List ↔ grid view toggle ..................................  OCBtnListGrid
```