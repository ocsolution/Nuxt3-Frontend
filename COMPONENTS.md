# `components/` — Component reference

Every Vue component shipped from `KEDTEC_MODULE/src/runtime/components/` documented in one place.

For each component you'll find: **props**, **events**, **slots**, **v-models**, **exposed methods** (where applicable) and one runnable usage snippet.

> Two folders have their own dedicated docs:
> - **Buttons** → see [btn/OCBtn.md](btn/OCBtn.md) for all 17 button components
> - **OCFormCreateModal** has an extended doc at [form/OCFormCreateModal.md](form/OCFormCreateModal.md)

---

## Table of contents

### Top-level components

| Display & layout | Modals & overlays | Tables / lists | Inputs & forms (top-level) |
|---|---|---|---|
| [OCAvatar](#ocavatar) | [OCActionConfirmModal](#ocactionconfirmmodal) | [OCSelect](#ocselect) | [OCColorPicker](#occolorpicker) |
| [OCBreadcrumb](#ocbreadcrumb) | [OCConfirm](#occonfirm) | [OCSlide](#ocslide) | [OCSwitch](#ocswitch) |
| [OCCollapse](#occollapse) | [OCDrawer](#ocdrawer) | [OCTab](#octab) | [SwitchDate](#switchdate) |
| [OCContent](#occontent) | [OCModal](#ocmodal) | [OCTblChip](#octblchip) | [SwitchDatabase](#switchdatabase) |
| [OCHead](#ochead) | [OCModalNew](#ocmodalnew) | [OCTblFilter](#octblfilter) | |
| [OCIcon](#ocicon) | [OCPopover](#ocpopover) | [DropdownChip](#dropdownchip) | |
| [OCIframe](#ociframe) | [OCPreviewFile](#ocpreviewfile) | | |
| [OCLabelValue](#oclabelvalue) | [OCReport](#ocreport) | | |
| [OCLoading](#ocloading) | [OCToast](#octoast) | | |
| [OCMessage](#ocmessage) | [OCTooltip](#octooltip) | | |
| [OCProfileInfo](#ocprofileinfo) | [PreviewImage](#previewimage) | | |
| [OCProfileUpload](#ocprofileupload) | [OCFilterDropDownByIcon](#ocfilterdropdownbyicon) | | |
| [OCSkeleton](#ocskeleton) | | | |
| [OCStatus](#ocstatus) | | | |
| [OCTitle](#octitle) | | | |
| [OCTruncateText](#octruncatetext) | | | |
| [OCVideo](#ocvideo) | | | |
| [OCViewInfo](#ocviewinfo) | | | |
| [OCWrapperPage](#ocwrapperpage) | | | |
| [OCWrapperCollapseLeftRight](#ocwrappercollapseleftright) | | | |
| [AccountingPeriod](#accountingperiod) | | | |
| [CardMenu](#cardmenu) | | | |
| [Currency](#currency) | | | |
| [ImgToBase64](#imgtobase64) | | | |
| [InternetConnectStatus](#internetconnectstatus) | | | |
| [Logo](#logo) | | | |
| [Notification](#notification) | | | |
| [OCFilter](#ocfilter) | | | |
| [ProgressCircle](#progresscircle) | | | |
| [ResizeHandle](#resizehandle) | | | |
| [UserDetail](#userdetail) | | | |
| [Version](#version) | | | |
| [WelcomeImage](#welcomeimage) | | | |

### Subfolders

- [AttachmentFile/](#attachmentfile) — AttachFile, UploadImage
- [btn/](btn/OCBtn.md) — buttons (dedicated doc)
- [card/](#card) — OCCard, OCCardSelect
- [checkRadio/](#checkradio) — OCRadio, OCRadios
- [checkbox/](#checkbox) — OCCheckbox, OCCheckboxes
- [contact/](#contact) — OCModalStudentContact
- [currency/](#currency-folder) — CurrencyMain, CurrencyView, CreateUpdateCurrencyNote
- [dateRangePicker/](#daterangepicker) — OCDatePicker, OCDateRangePicker, OCTimePicker
- [exchangeRate/](#exchangerate) — CardExchangeRate, CardExchangeRateSkeleton, ExchangeRateMainPage, ExchangeRateSwitchDate
- [files/](#files) — FileUpload
- [form/](#form) — OCForm, OCFormCreate, OCFormCreateModal, OCFormGroup, OCInputForm
- [input/](#input) — OCInput, OCInputColor, OCInputSearch, OCQuill, OCTextEditor, OCTextarea
- [layout/](#layout) — HeaderNav, Layout, OCHeaderNav
- [menu/](#menu) — ListMenu, MenuActions, MenuNavDialog, MenuNavLeft, ModalLogOut, NavMain
- [ocReportModule/](#ocreportmodule) — OCReportCard, OCReportModule
- [profile/](#profile) — FeatureSignature, ListProfileDetail, ModalEditEmail, ModalEditPassword, ModalEditPhone, ModalEditProfile, ModalRight, OCSignature
- [setting/](#setting) — DeviceACView, DeviceDisplayPoper
- [tables/](#tables) — OCBtnReload, OCPagination, OCSelectRecord, OCTableFooter, OCTableHeader
- [treetable/](#treetable) — OCRowTreeTable, OCTreeTable
- [uploadMananger/](#uploadmananger) — OCCamera, OCUploadManager, OCUploadProfile
- [wrapperCollapseLeftRight/](#wrappercollapseleftright) — WrapperCollapseLeftRight

---

## Top-level components

### AccountingPeriod

Inline label showing the current accounting period's date range; wrapped in a tooltip with the full start/end dates.

| Prop | Notes |
|---|---|
| `prevent` | Disables the tooltip when truthy. |

```vue
<AccountingPeriod />
<AccountingPeriod :prevent="hideOnPrint" />
```

---

### CardMenu

Renders a clickable menu card (linked via `NuxtLink`) for a `items` group with a colored icon.

**Props:** `items` (`{ title, items: [{ href, color, icon, label }] }`)
**Events:** `pushed` — fires when a card is clicked.

```vue
<CardMenu :items="{
  title: 'Setup',
  items: [
    { href: '/setup/branch',   icon: 'ri-building-line',     label: 'Branch',   color: '#3b82f6' },
    { href: '/setup/currency', icon: 'ri-coins-line',         label: 'Currency', color: '#22c55e' },
  ],
}" @pushed="onPushed" />
```

---

### Currency

Inline number with leading/trailing currency markers, automatic negative-sign placement, and decimal styling.

**Props:** `value`, `leading`, `trailing`, `class`, plus styling props.
**Slots:** `#leading`, `#trailing`.

```vue
<Currency :value="123.45" leading="$" />
<Currency :value="-89.5" trailing="៛">
  <template #leading><i class="ri-money-dollar-circle-line" /></template>
</Currency>
```

---

### DropdownChip

Chip-style dropdown that supports single/multi selection with searchable items.

**Props (object form):** `items`, `selected`, `placeholder`, `multiple`, `searchable`, `pk`, `displayKey`, …
**Events:** `update:selected`

```vue
<DropdownChip
  :items="branches"
  :selected="selected"
  pk="Id"
  displayKey="Name"
  multiple
  @update:selected="(v) => selected = v"
/>
```

---

### ImgToBase64

Image-file → base64 converter UI (drag-drop, preview, copy-to-clipboard).

**Props (object form):** `accept`, `maxSizeMB`, `class` …
**Events:** `base64-changed(value)`, `image-converted(meta)`

```vue
<ImgToBase64 @base64-changed="payload = $event" />
```

---

### InternetConnectStatus

Slide-in banner that shows online/offline state from `useOnline()`. No props.

```vue
<InternetConnectStatus />
```

---

### Logo

App logo + optional name. Switches dark/light asset, supports navbar/login/header layouts.

**Props:** `type` (`'login' | 'header' | 'navbar'`), `isCollapsed`

```vue
<Logo type="navbar" :isCollapsed="sidebarSmall" />
<Logo type="login" />
```

---

### Notification

Notification icon + popover panel (system notifications). Self-contained — no props.

```vue
<Notification />
```

---

### OCActionConfirmModal

Reusable confirm modal for irreversible actions (delete / void / approve). Renders a label, a custom message, and one or more action buttons.

**Props:** `btns` (array of `{ type, label }`), `type`, `data`, `message`, `loading`
**v-model:** `open` (boolean)
**Events:** `eventBtn(type)` — `'cancel'` is emitted on dismiss.
**Slots:** default — replace the message body.

```vue
<OCActionConfirmModal
  v-model="open"
  :btns="[{ type: 'delete', label: $t('delete') }]"
  type="delete"
  :data="row"
  :message="$t('confirm_delete')"
  :loading="busy"
  @eventBtn="onAction"
/>
```

---

### OCAvatar

Single or stacked avatar group. Handles src arrays, error fallback, gender-based default, hex border.

**Props (object form):** `src` (string | array), `size`, `round`, `border`, `borderColor`, `borderHexColor`, `gender`, `errorType`, `background`, `newClass`, `isUrl`.
**Slots:** default (overlay content, e.g., upload icon).

```vue
<OCAvatar :src="user.ImagePath" size="md" />
<OCAvatar :src="[a.img, b.img, c.img]" :max-src="3" border />
```

---

### OCBreadcrumb

Lightweight breadcrumb trail with chevron separators.

**Props (object form):** `items` (`[{ label, to, icon? }]`)
**Events:** `click(item)`

```vue
<OCBreadcrumb
  :items="[{ label: 'Home', to: '/' }, { label: 'Orders', to: '/orders' }, { label: code }]"
  @click="(i) => router.push(i.to)"
/>
```

---

### OCCollapse

Accordion wrapper around `UAccordion`. Supports multiple-open and per-item slot rendering.

**Props:** `items` (`[{ label, slot, ... }]`), `color`, `variant`, `trailing`, `multiple`
**Events:** `onOpen`, `onClose`
**Slots:** dynamic by `item.slot` name.

```vue
<OCCollapse
  :items="[
    { label: 'Profile', slot: 'profile' },
    { label: 'Address', slot: 'address' },
  ]"
  multiple
>
  <template #profile>…</template>
  <template #address>…</template>
</OCCollapse>
```

---

### OCColorPicker

Eye-dropper-style color picker (uses native `EyeDropper` API when available).

**v-model:** color value
**Props (object form):** `disabled`, `defaultColor`, …
**Events:** `onChange(hex)`, `onPicker(hex)`

```vue
<OCColorPicker v-model="brand" @onChange="updateTheme" />
```

---

### OCConfirm

Imperative confirm dialog that reads from `useConfirmStore()`. Mount once at the app root; trigger from anywhere with the store.

```vue
<OCConfirm />

<script setup>
const confirm = useConfirmStore()
confirm.open({ title: 'Delete?', type: 'delete', onConfirm: () => doDelete() })
</script>
```

---

### OCContent

Page-content wrapper — adds `.oc-content` skeleton with optional header slot and dynamic height.

**Props:** `class`, `color`, `height`
**Slots:** `#header`, default

```vue
<OCContent height="600px">
  <template #header><h2>Inventory</h2></template>
  <OCTable ... />
</OCContent>
```

---

### OCDrawer

Right-side slide-over (`USlideover` wrapper) with header/footer slots.

**Props:** `side`, `ui`, `transition`, `overlay`, `preventClose`, `appear`
**v-model:** `Boolean`
**Events:** `closed`
**Slots:** `#header`, default, `#footer`

```vue
<OCDrawer v-model="open" preventClose @closed="reset">
  <template #header>{{ $t('details') }}</template>
  <div>…</div>
  <template #footer>
    <OCBtn type="cancel" @click="open = false" />
    <OCBtn type="save" @click="save" />
  </template>
</OCDrawer>
```

---

### OCFilter

Marquee filter banner using SwiperJS (auto-cycling slides). Self-contained promo banner — no props.

```vue
<OCFilter />
```

---

### OCFilterDropDownByIcon

Positioned popover that lists filter items beside an icon trigger.

**Props:** `items`, `NameClassElement`, `isSelectItemFirst`
**v-model:** `Boolean` (open state)
**Events:** `selectItem(item)`

```vue
<OCFilterDropDownByIcon
  v-model="isFilterOpen"
  :items="filterOptions"
  NameClassElement=".my-filter-anchor"
  @selectItem="applyFilter"
/>
```

---

### OCHead

`<Head>` wrapper that sets title, description, OG tags, favicon, keywords.

**Props (object form):** `title`, `description`, `img`, `url`, `icon`, `keyword`
**Slots:** default — extra `<Meta>` / `<Link>` injections.

```vue
<OCHead title="Orders" description="All orders" img="/og.png" />
```

---

### OCIcon

Square icon tile with configurable size, background and icon color.

**Props:** `icon` (Remix class or inline SVG), `size`, `bgColor`, `iconColor`, `class`

```vue
<OCIcon icon="ri-printer-line" size="m" bgColor="var(--ocs-c-orange)" iconColor="#fff" />
```

---

### OCIframe

Embedded iframe with loading overlay and `postMessage` bridge.

**Props:** `path`, `querystring`
**Events:** `onMessage(payload)`
**Exposed:** `reload()`, ref accessors

```vue
<OCIframe :path="reportPath" :querystring="filterQS" @onMessage="onIframeMsg" />
```

---

### OCLabelValue

Two-line label / value pair used inside detail panels.

**Props:** `label`, `value`, `ui`, `isTruncate`
**Slots:** `#value` — replace the value side.

```vue
<OCLabelValue :label="$t('amount')" :value="ocNumber({ number: row.Amount })" />
<OCLabelValue :label="$t('status')">
  <template #value><OCStatus :text="row.Status" color="approve" /></template>
</OCLabelValue>
```

---

### OCLoading

Centered SVG spinner. Drop into any container to gate UI on `loading`.

**Props:** `loading` (`Boolean`), `fullscreen` (`Boolean`)

```vue
<OCLoading :loading="busy" />
<OCLoading :loading="busy" fullscreen />
```

---

### OCMessage

Colored inline message box with leading icon (info/success/warning/error).

**Props:** `color`, `icon`, `title`, `message`

```vue
<OCMessage color="green" icon="ri-check-line" title="Saved" message="Changes were saved." />
```

---

### OCModal

App-standard modal. Header / sub-title / center / right / footer slots; supports fullscreen mode.

**Props (selected):** `ui`, `transition`, `preventClose`, `fullscreen`, `appear`, `overlay`, `overflowAuto`
**v-model:** `Boolean`
**Events:** `onClose`
**Slots:** `#header`, `#headerCenter`, `#headerRight`, `#subTitle`, default, `#footer`

```vue
<OCModal v-model="open" :ui="{ width: '!max-w-[480px]' }" preventClose>
  <template #header>{{ $t('edit') }}</template>
  <div>…</div>
  <template #footer>
    <OCBtn type="cancel" @click="open = false" />
    <OCBtn type="edit"   @click="save" />
  </template>
</OCModal>
```

---

### OCModalNew

Headless modal scaffold (no UModal dependency). Useful when you need custom transitions or to opt out of NuxtUI styles.

**Props (object form):** `ui`, `class`
**v-model:** `Boolean`
**Events:** `close`
**Slots:** `#header`, default, `#footer`

```vue
<OCModalNew v-model="open" @close="open = false">
  <template #header>Custom</template>
  <div>…</div>
</OCModalNew>
```

---

### OCPopover

Unified popover wrapper supporting three back-ends: NuxtUI, Flowbite, Headless UI.

**Props:** `mode`, `closeDelay`, `use` (`'nuxtui' | 'flowbite' | 'headless'`), `placement`, `class`, `closeInside`, `offset`, `ui`
**v-model:** `Boolean`
**Events:** `click`
**Slots:** `#trigger`, default (`close` argument provided)

```vue
<OCPopover use="nuxtui" placement="bottom-end">
  <template #trigger><BtnIcon icon="ri-more-2-fill" /></template>
  <div class="p-3">Menu content</div>
</OCPopover>
```

---

### OCPreviewFile

Full-screen file preview (image/PDF/text/video, with multi-page navigation).

**Props:** `pathUrl`, `typeFile`, `isBGClose`, `errorType`, `isMulti`, `selected`, `page`, `extension`, `src`
**v-model:** `Boolean`
**Events:** `onClickBackGroundBlur`

```vue
<OCPreviewFile v-model="show" :pathUrl="file.url" :typeFile="file.type" :extension="file.ext" />
```

---

### OCProfileInfo

Avatar + name + sub-title row, commonly used in lists and popovers.

**Props (selected):** `title`, `subTitle`, `src`, `size`, `round`, `border`, `gender`, `errorType`, `background`, `newClass`
**Slots:** `#title`, `#subTitle`

```vue
<OCProfileInfo :title="emp.Name" :subTitle="emp.Position" :src="emp.PhotoUrl" gender="male" />
```

---

### OCProfileUpload

Avatar with hover-to-upload affordance.

**Props:** `getImage` (current URL)
**Events:** `onDone(file)`

```vue
<OCProfileUpload :getImage="form.Image" @onDone="form.Image = $event" />
```

---

### OCReport

Mounted once at app root. Reads `useReportStore()` and renders a modal with the active report.

```vue
<OCReport />
```

---

### OCSelect

Searchable select (single or multi) with remote API support, virtualization, leading/option slots, sticky create button.

**Props (selected):** `localData`, `api`, `pk`, `label`, `multiple`, `placeholder`, `required`, `disabled`, `template`, `mapData`, `isNotAllowRemoteSelect`, …
**v-models:** default (selected value), `search`, `loading`
**Events:** `selected`, `mapData`, `onSearch`, `onFocus`
**Slots:** `#iconLeading`, `#leading`, `#option`, …
**Exposed:** various — see source

```vue
<OCSelect
  v-model="form.CurrencyCode"
  :api="{ url: 'api/currency/list', method: 'POST' }"
  pk="Id"
  label="Currency"
  required
/>
```

---

### OCSkeleton

Skeleton placeholder block (NuxtUI USkeleton wrapper).

**Props (object form):** `ui`, `outContent`, `class`

```vue
<OCSkeleton :ui="{ rounded: 'rounded-md' }" class="h-6 w-32" />
```

---

### OCSlide

Card-grid / slide view with built-in toolbar (search, reload, filter). Used as an alternative to OCTable.

**Props (selected):** `options`, `data`, `api`, `pk`, `selectable`, …
**v-models:** default (selected), `loading`
**Events:** `mapData`, `selected`, `unSelect`, `onSaveState`
**Slots:** `#headerLeft`, `#headerCenter`, `#headerApplyFilter`, `#headerRight`, default (per item), `#loading`
**Exposed:** `remoteSelect`, `removeSelect`, `replaceItem`

```vue
<OCSlide :api="{ url: 'api/student/list', method: 'POST' }" pk="StudentCode">
  <template #default="{ data }">
    <Card :student="data" />
  </template>
</OCSlide>
```

---

### OCStatus

Pill-style status indicator.

**Props:** `icon`, `color`, `text`, `variant` (`'solid' | 'outline'`), `size`, `textColor`, `iconColor`

```vue
<OCStatus text="Active"  color="green" variant="solid" />
<OCStatus text="Expired" color="red"   variant="outline" icon="ri-error-warning-line" />
```

---

### OCSwitch

Toggle switch.

**Props (object form):** `checked`, `label`, `disabled`
**Events:** `update:checked`

```vue
<OCSwitch v-model:checked="form.IsActive" label="Active" />
```

---

### OCTab

Tabs with default/item/per-tab slots, switchable orientation.

**Props (selected):** `items`, `orientation`, `defaultIndex`, `ui`, `variant`
**v-model:** active index
**Events:** `onChange(item, index)`
**Slots:** `#default`, `#item`, dynamic per tab key
**Exposed:** `jumpTo(index)`, `disabled(index, bool)`, …

```vue
<OCTab v-model="tab" :items="[{ key:'a', label:'A' }, { key:'b', label:'B' }]">
  <template #a>Tab A content</template>
  <template #b>Tab B content</template>
</OCTab>
```

---

### OCTblChip

Renders a row of chips representing active filter values, with remove buttons. Pairs with `OCTblFilter`.

**v-model:** items array
**Props:** `stateKey`
**Events:** `onRemove(item)`

```vue
<OCTblChip v-model="filterChips" stateKey="orders.list" @onRemove="clearOne" />
```

---

### OCTblFilter

Filter popover used in table headers. Configurable per-field inputs (selects, dates, ranges, checkboxes).

**Props (object form):** `inputs`, `state`, `title`, …
**v-model:** filter state object
**Events:** `onClick`, `onCancel`, `onOpen`
**Exposed:** various — see source

```vue
<OCTblFilter
  v-model="filter"
  :inputs="[
    { type:'select', name:'Status', label:'Status', options: statuses },
    { type:'date',   name:'FromDate', label:'From' },
  ]"
  @onClick="applyFilter"
/>
```

---

### OCTitle

Page section title with `#left` / `#right` slots for action buttons.

**Props:** `label`, `class`
**Slots:** `#left`, `#right`

```vue
<OCTitle :label="$t('orders')">
  <template #right><OCBtn type="create" @click="open=true" /></template>
</OCTitle>
```

---

### OCToast

App-wide toast container. Mount once at the root; trigger via `useToast()`.

```vue
<OCToast />
```

---

### OCTooltip

Tooltip wrapper supporting both Popper-driven and CSS hover modes.

**Props (selected):** `text`, `isPopper`, `placement`, `isGlobal`, `popper`, `ui`, `bgColor`, `prevent`
**Slots:** default, `#text`

```vue
<OCTooltip text="Edit user" placement="top">
  <BtnIcon icon="ri-pencil-line" />
</OCTooltip>
```

---

### OCTruncateText

Single-line truncated text with tooltip showing full content and optional copy-to-clipboard.

**Props:** `text`, `class`, `isCopyable`, `bgColor`
**Slots:** default — override rendered text.

```vue
<OCTruncateText :text="row.LongDescription" isCopyable />
```

---

### OCVideo

Video player with custom play button, autoplay flag, and optional event muting.

**Props:** `path`, `VideoStyle`, `BtnStyle`, `IconStyle`, `autoPlay`, `eventNone`
**Exposed:** `container`

```vue
<OCVideo :path="clipUrl" />
```

---

### OCViewInfo

Display block with title + sub-title; used by `OCProfileInfo`.

**Props:** `title`, `subTitle`, `size`, `colorTitle`, …
**Slots:** `#title`, `#subTitle`

```vue
<OCViewInfo title="John Doe" subTitle="Admin" size="m" />
```

---

### OCWrapperPage

Top-level page wrapper that applies app padding/scroll rules.

**Props:** `class`
**Slots:** default

```vue
<OCWrapperPage>
  <OCContent>…</OCContent>
</OCWrapperPage>
```

---

### OCWrapperCollapseLeftRight

Two-pane layout with a draggable / collapsible left side.

**Props (object form):** `widthCollapse`, `isClearPaddingOutSize`, `ui`, …
**Events:** `onCollapseExpand(state)`
**Slots:** `#headerContent`, `#headerCollapse`, `#leftContent(collapse, isOverModal)`, `#rightContent(collapse, isOverModal)`
**Exposed:** `fnOnOpenCollapse`, `fnOnCloseCollapse`

```vue
<OCWrapperCollapseLeftRight>
  <template #headerContent><PageTitle /></template>
  <template #leftContent="{ collapse }">List…</template>
  <template #rightContent="{ collapse }">Detail…</template>
</OCWrapperCollapseLeftRight>
```

---

### PreviewImage

Lightweight image-preview modal for a single file/blob.

**Props:** `file`
**v-model:** `Boolean`

```vue
<PreviewImage v-model="open" :file="selectedFile" />
```

---

### ProgressCircle

Animated SVG progress ring.

**Props (object form):** `value` (0–100), `size`, `strokeWidth`, `color`

```vue
<ProgressCircle :value="uploadProgress" />
```

---

### ResizeHandle

Drag handle to resize the navbar width (records via mouse events). No props.

```vue
<ResizeHandle />
```

---

### SwitchDatabase

Database/branch switcher in the footer navbar (driven by `useDatabaseStore()`).

**Props:** `layoutType` (`'main' | 'teacher' | …`)

```vue
<SwitchDatabase layoutType="main" />
```

---

### SwitchDate

Date picker with prev/next arrows, used as a one-tap range mover.

**v-model:** `Date`
**Props:** `variant` (`'solid' | 'outline' | 'none'`), `attrs`, `disabled`
**Events:** `actioned(date)`

```vue
<SwitchDate v-model="date" variant="outline" @actioned="reload" />
```

---

### UserDetail

User profile popover triggered by username; fetches & shows user details.

**Props:** `username`, `size`
**Slots:** default (custom trigger), `#panel({ data })` (custom panel)

```vue
<UserDetail :username="row.CreatedBy" size="s" />
```

---

### Version

App version footer block with copyright + link.

**Props:** `v`, `name`

```vue
<Version v="1.2.3" name="KEDTEC" />
```

---

### WelcomeImage

Inline SVG illustration used on empty/welcome screens. No props.

```vue
<WelcomeImage />
```

---

## AttachmentFile/

### AttachFile

Image attachment grid with browse-to-add cells, delete buttons, fixed-square sizing.

**v-model:** `Array<File | { ImagePath }>`
**Props:** `showIconDelete`, `height`, `columns`, `image`, `fixedSquareSize`

```vue
<AttachFile v-model="images" showIconDelete columns="grid-cols-4" />
```

### UploadImage

Single image upload tile with default placeholder.

**Props:** `defaultImage`
**v-model:** image data
**Events:** `onDoneUpload(payload)`

```vue
<UploadImage v-model="profileImg" defaultImage="/img/default.png" @onDoneUpload="onUploaded" />
```

---

## card/

### OCCard

Container card with optional header/footer.

**Props:** `ui`, `header`, `footer`, `classes`
**Slots:** `#header`, default, `#footer`

```vue
<OCCard classes="p-4">
  <template #header>Title</template>
  Body
  <template #footer><OCBtn type="save" @click="save" /></template>
</OCCard>
```

### OCCardSelect

Selectable card grid with search/filter/pagination — same toolbar as OCTable, but renders items via a default slot instead of rows.

**Props (selected):** `api`, `localData`, `options`, `pk`, `selectable`, …
**v-models:** default (selected[]), `searching`
**Events:** `mapData`, `selected`, `unSelect`, `onSaveState`, `onSearching`, `onReload`, …
**Slots:** `#headerLeft`, `#headerCenter`, `#headerApplyFilter`, `#headerRight`, default (per item), `#loading`, `#ShowDetailAfterTableOne`, `#ShowDetailAfterTableTwo`, `#footerLeft`, `#footerCenter`, `#footer`
**Exposed:** various — see source

```vue
<OCCardSelect
  :api="{ url: 'api/product/list', method: 'POST' }"
  pk="Id"
  v-model="selectedProducts"
>
  <template #default="{ data, active }">
    <ProductCard :item="data" :active="active" />
  </template>
</OCCardSelect>
```

---

## checkRadio/

### OCRadio

Single radio button.

**v-model:** value
**Props:** `value`, `label`, `name`, `disabled`, `help`, `ui`

```vue
<OCRadio v-model="gender" value="male" label="Male" />
<OCRadio v-model="gender" value="female" label="Female" />
```

### OCRadios

Radio group.

**v-model:** selected value
**Props (selected):** `options` (`[{ value, label, disabled? }]`), `name`, `legend`, `ui`, `uiRadio`, `orientation`
**Events:** `onChange(value)`
**Slots:** `#label({ option })`

```vue
<OCRadios v-model="payment" :options="[
  { value: 'cash', label: 'Cash' },
  { value: 'card', label: 'Card' },
]" @onChange="updatePreview" />
```

---

## checkbox/

### OCCheckbox

Single checkbox.

**v-model:** `Boolean | Array` (when used as a group member)
**Props:** `ui`, `label`, `name`, `help`, `required`, `value`, `disabled`, `id`
**Events:** `onChange(value)`
**Slots:** `#label`

```vue
<OCCheckbox v-model="form.AgreeToTerms" label="I agree to the terms" required />
```

### OCCheckboxes

Group of `OCCheckbox` from an options array.

**v-model:** array of selected values
**Props:** `options`, `disabled`, `name`, `ui`, `isBorder`, `vertical`
**Events:** `onChange(value)`
**Slots:** `#label({ option })`

```vue
<OCCheckboxes
  v-model="features"
  :options="[
    { label: 'A', value: 'a' },
    { label: 'B', value: 'b' },
  ]"
  isBorder vertical
/>
```

---

## contact/

### OCModalStudentContact

Modal showing a student's contact list (parents/guardians) by student code.

**v-model:** `Boolean`
**Props:** `studentCode`

```vue
<OCModalStudentContact v-model="open" :studentCode="row.StudentCode" />
```

---

## currency/

### CurrencyMain

Page-level currency list table — uses `OCTable` internally, hooked to `api/currency/list`.

**Props (object form):** see file; typically self-contained.

```vue
<CurrencyMain />
```

### CurrencyView

Read-only currency view page (no create/edit) — uses `OCTable`.

```vue
<CurrencyView />
```

### CreateUpdateCurrencyNote

Drawer-based form to create/update a currency note row.

**v-models:** `open` (Boolean), `data` (currency-note object)
**Props (object form):** various — see file
**Events:** `onSuccess(payload)`

```vue
<CreateUpdateCurrencyNote
  v-model:open="openDrawer"
  v-model:data="noteModel"
  @onSuccess="reload"
/>
```

---

## dateRangePicker/

### OCDatePicker

Single or range date picker built on `VDatePicker`.

**Props (object form):** `dateRangePicker`, `attributes`, `minDate`, `maxDate`, `columns`, `placeholder`, `masks`, `keyDate`, `attrs`, …
**Events:** `update:model-value(value)`, `close`
**Exposed:** `datePicker`, `fnReset`, `fnUpdateValue`, `fnResetInput`

```vue
<OCDatePicker v-model="date" placeholder="Pick a date" />
<OCDatePicker v-model="range" :date-range-picker="true" />
```

### OCDateRangePicker

Date-range picker with predefined ranges (today, this week, last month, etc.).

**v-model:** `{ start: Date, end: Date }`
**Props (object form):** `attributes`, `minDate`, `maxDate`, `columns`, `placeholder`, `predefined`, `minwidth`, …
**Exposed:** `dt()`

```vue
<OCDateRangePicker v-model="range" predefined />
```

### OCTimePicker

Time-only picker (hours + minutes, 12 or 24 hour).

**v-model:** time value
**Props (object form):** `is24hr`, `minDate`, `maxDate`, `popover`, `attrs`, …
**Events:** `update:model-value`, `close`, `clear`
**Exposed:** internal `input_time` ref

```vue
<OCTimePicker v-model="startTime" :is24hr="true" />
```

---

## exchangeRate/

### CardExchangeRate

List of currency exchange-rate cards.

**v-model:** array of currency-rate rows
**Props (object form):** styling props, see file.

```vue
<CardExchangeRate v-model="rates" />
```

### CardExchangeRateSkeleton

Skeleton placeholder rows (5 items). No props.

```vue
<CardExchangeRateSkeleton />
```

### ExchangeRateMainPage

Full exchange-rate dashboard page combining a switch date, cards, and edit drawer.

```vue
<ExchangeRateMainPage />
```

### ExchangeRateSwitchDate

Compact date selector (prev / today / next) used inside the exchange-rate page.

**v-model:** `Date`

```vue
<ExchangeRateSwitchDate v-model="date" />
```

---

## files/

### FileUpload

Drag-and-drop file upload area; v-models a list of files (existing + newly uploaded).

**v-model:** `Array<File | { Path, Name, ... }>`
**Props (selected):** `accept`, `maxSize`, `multiple`, `disabled`, …
**Events:** `onDeleteFile(file)`

```vue
<FileUpload v-model="files" multiple @onDeleteFile="markDeleted" />
```

---

## form/

### OCForm

Thin wrapper around `UForm` exposing schema/validate/state with `onSubmit`/`onError` events.

**Props:** `schema`, `validate`, `state`
**Events:** `onSubmit(payload)`, `onInput`, `onError(errors)`
**Exposed:** `ocForm` (ref), `submit()`, `clearValidate()`

```vue
<OCForm :state="state" :schema="zSchema" @onSubmit="save">
  <OCInput v-model="state.Name" name="Name" />
  <OCBtn type="save" typeButton="submit" />
</OCForm>
```

### OCFormCreate

Configurable form builder. Accepts an `inputs` array (or `tabs` for tabbed forms) and a `form` config object describing API endpoints + behavior. Used as the body of [OCFormCreateModal](#ocformcreatemodal). See [extended doc](form/OCFormCreateModal.md) for the full input shape.

**Props (object form):** `form`, `inputs`, `tabs`
**v-models:** `data`, `disabled`
**Events:** `onSubmitted`, `onCancel`, `onReset`, `onLoading`, `onError`, `onSuccess`
**Exposed:** `onsubmit()`, `clearForm()`, `resetForm()`, `refInput()`, `jumpTo(i)`, `disabledTab(i, b)`
**Slots:** `#custom`

```vue
<OCFormCreate
  v-model:data="form"
  :inputs="[
    { type: 'text', name: 'Code', label: 'Code', required: true },
    { type: 'text', name: 'Name', label: 'Name', required: true },
  ]"
  :form="{ api: { create: 'api/room/create', update: 'api/room/update' } }"
  @onSuccess="reload"
/>
```

### OCFormCreateModal

Drawer-based wrapper around `OCFormCreate`. **Full documentation:** [form/OCFormCreateModal.md](form/OCFormCreateModal.md).

```vue
<OCFormCreateModal
  v-model:show="open"
  v-model:data="model"
  :inputs="inputs"
  :form="form"
  @onSuccess="reload"
/>
```

### OCFormGroup

Form-field group wrapper providing label/error/help text/required indicator. NuxtUI `UFormGroup` wrapper.

**Props:** `label`, `name`, `help`, `required`, `size`, `description`, `error`, `hint`, `ui`, …
**Slots:** `#label`, `#description`, `#help`, `#hint`, `#error`

```vue
<OCFormGroup name="Email" label="Email" required>
  <OCInput v-model="state.Email" type="email" />
</OCFormGroup>
```

### OCInputForm

Router component used by `OCFormCreate` — chooses the right input (text/select/date/radio/checkbox/textarea/quill/etc.) based on `input.type`.

**v-models:** `state`, `dataUpdate`
**Props:** `input` (single input definition)
**Exposed:** ref to the underlying input.

You generally don't use this directly — pass inputs through `OCFormCreate`.

---

## input/

### OCInput

Text-style input. NuxtUI `UInput` wrapper.

**v-model:** value
**Props (selected):** `name`, `type`, `placeholder`, `loading`, `disabled`, `icon`, `leadingIcon`, `trailingIcon`, `padded`, `autofocus`, `autofocusDelay`, `autocomplete`, `ui`, `size`
**Events:** `onFocus`, `onBlur`, `onInput`, `onAction`
**Slots:** `#leading`, `#trailing`
**Exposed:** `ocInput`

```vue
<OCInput v-model="form.Name" placeholder="Name" leadingIcon="ri-user-line" />
```

### OCInputColor

Color input with popover swatch & hex preview.

**v-model:** hex string
**Props:** `disabled`
**Events:** `onChange(hex)`, `onPicker(hex)`

```vue
<OCInputColor v-model="brand" />
```

### OCInputSearch

Standalone search input with debounce + clear button.

**v-model:** search query
**Props:** `autofocus`, `class`, `wrapper`, `placeholder`
**Events:** `onInput(value)`, `onCancel`
**Exposed:** `remoteData`, `ocInput`

```vue
<OCInputSearch v-model="q" @onInput="refetch" />
```

### OCQuill

Quill rich-text editor.

**Props (object form):** `modelValue`, `placeholder`, `disabled`, `maxHeight`, `textareaClass`, …
**Events:** `update:modelValue`, `onFocus`, `onBlur`, `onInput`
**Exposed:** focus / blur / format helpers

```vue
<OCQuill v-model="body" maxHeight="300px" />
```

### OCTextEditor

Alternate rich-text editor (custom toolbar + content area).

**Props (object form):** `modelValue`, `disabled`, `showToolbar`, …
**Events:** `update:modelValue`, `onFocus`, `onBlur`, `onInput`
**Slots:** default — extra toolbar buttons
**Exposed:** content / toolbar APIs

```vue
<OCTextEditor v-model="body" />
```

### OCTextarea

Auto-grow textarea. NuxtUI `UTextarea` wrapper.

**v-model:** value
**Props (selected):** `rows`, `maxRows`, `placeholder`, `disabled`, `autofocus`, `resize`, `ui`
**Events:** `onFocus`, `onBlur`, `onInput`
**Slots:** default
**Exposed:** focus / refs

```vue
<OCTextarea v-model="form.Description" :rows="3" autofocus />
```

---

## layout/

### HeaderNav

Fixed-position page header with `#left` / `#center` / `#right` slots.

**Props:** `background` (CSS class)
**Slots:** `#left`, `#center`, `#right`

```vue
<HeaderNav background="bg-white">
  <template #left><Logo type="header" /></template>
  <template #right><BtnHeaderAction /></template>
</HeaderNav>
```

### Layout

Top-level app layout (sidebar + main content + footer). Driven by `useScreenStore`.

**Props:** `type`
**Slots:** default (page content)

```vue
<Layout type="main"><RouterView /></Layout>
```

### OCHeaderNav

Alternate header bar with `#logo`, `#left`, `#center`, `#right` slots. No props.

```vue
<OCHeaderNav>
  <template #logo><Logo type="header" /></template>
  <template #left>Menu</template>
  <template #right>User</template>
</OCHeaderNav>
```

---

## menu/

### ListMenu

Single navigation menu entry (icon + label + collapse-aware tooltip).

**Props:** `items` (`{ title, href, icon, children? }`), `title`, `isCollapsed`
**Events:** `clicked(item)`

```vue
<ListMenu :items="menu" :isCollapsed="sidebarSmall" @clicked="track" />
```

### MenuActions

Action list (icon + label rows).

**Props:** `items` (`[{ type, icon, label, color, disabled, permission, showIfAllowed }]`)
**Events:** `actionClick(type)`

```vue
<MenuActions
  :items="[{ type:'edit', icon:'ri-pencil-line', label:'Edit' }]"
  @actionClick="onAct"
/>
```

### MenuNavDialog

Mobile/tablet nav dialog (`navbar-left` overlay).

**Props:** `navWidth`
**Events:** `closedMenu`

```vue
<MenuNavDialog :navWidth="navWidth" @closedMenu="open = false" />
```

### MenuNavLeft

Desktop left-sidebar nav with collapse toggle.

**Props:** `isCollapsed`, `isForMobile`
**v-model:** `Boolean`
**Events:** `showCollapsed(state)`

```vue
<MenuNavLeft v-model="open" :isCollapsed="sidebarSmall" @showCollapsed="setSmall" />
```

### ModalLogOut

Logout confirmation popover anchored to the header.

**Props:** `open`, `layoutType`
**Events:** `update:open`
**Slots:** default (custom body)

```vue
<ModalLogOut v-model:open="open" layoutType="main">
  Are you sure you want to log out?
</ModalLogOut>
```

### NavMain

Top header section: brand + navigation triggers + slot for actions.

**Props (selected):** `isCollapsed`, `isOpen`, `title`, `breadcrumb`, …
**Events:** `hideCollapsed`, `openMenu`
**Slots:** default — actions on the right side.

```vue
<NavMain :title="$t('orders')" :breadcrumb="bc">
  <OCBtn type="create" @click="open=true" />
</NavMain>
```

---

## ocReportModule/

### OCReportCard

Single report card with skeleton-mode loading state.

**Props:** `data`, `isSkeleton`, `loadings`
**Events:** `onClick(report)`

```vue
<OCReportCard :data="report" @onClick="openReport" />
<OCReportCard :isSkeleton="true" :loadings="6" />
```

### OCReportModule

Report-listing page with built-in filter side-panel (uses `OCTable` with `showTemplate`).

**v-model:** drawing data
**Props:** `reportGroup`, `filterForm`, `modalOverflow`
**Events:** `open`, `onAction`, `onClose`
**Slots:** `#filter({ data })`

```vue
<OCReportModule :reportGroup="group" :filterForm="filterDef" @open="loadReport">
  <template #filter="{ data }">
    <FilterForm :data="data" />
  </template>
</OCReportModule>
```

---

## profile/

### FeatureSignature

Placeholder showing the user's signature. No props.

```vue
<FeatureSignature />
```

### ListProfileDetail

Two-column label / value row for profile detail pages.

**Props:** `label`, `value`
**Slots:** default — replace the value side.

```vue
<ListProfileDetail label="Email" :value="user.Email" />
<ListProfileDetail label="Status">
  <OCStatus :text="user.Status" color="green" />
</ListProfileDetail>
```

### ModalEditEmail

Drawer/modal to change the current user's email.

**Props:** `isOpen`
**Events:** `ChangeEmail(newEmail)`, `Open`, `Close`

```vue
<ModalEditEmail :isOpen="open" @Open="open=true" @Close="open=false" @ChangeEmail="apply" />
```

### ModalEditPassword

Drawer/modal to change the current user's password.

**Props:** `isOpen`
**Events:** `ChangePassword({ old, new })`, `Open`, `Close`

```vue
<ModalEditPassword :isOpen="open" @ChangePassword="apply" @Close="open=false" />
```

### ModalEditPhone

Drawer/modal to change the user's phone number.

**v-model:** `Boolean`
**Events:** `ChangePhone(newPhone)`, `Open`, `Close`

```vue
<ModalEditPhone v-model="open" @ChangePhone="apply" />
```

### ModalEditProfile

Drawer/modal to edit profile (name + photo).

**v-model:** `Boolean`
**Props:** `data` (current profile)
**Events:** `ChangeProfile(payload)`, `Open`, `Close`

```vue
<ModalEditProfile v-model="open" :data="me" @ChangeProfile="save" />
```

### ModalRight

Generic right-side slideover with title + icon header.

**Props:** `isOpen`, `title`, `icon`
**Events:** `Close`
**Slots:** default

```vue
<ModalRight :isOpen="open" title="Edit" icon="ri-pencil-line" @Close="open=false">
  Form…
</ModalRight>
```

### OCSignature

Signature capture pad (canvas) wrapped in a modal.

**v-models:** `open` (Boolean), default (image data URL)
**Props:** `loading`
**Events:** `onSave(dataUrl)`

```vue
<OCSignature v-model:open="open" v-model="sigUrl" :loading="busy" @onSave="save" />
```

---

## setting/

### DeviceACView

Drawer that shows AC device details (serial, model, location, etc.).

**v-model:** `Boolean` (drawer open)
**Props:** `data` (device record)

```vue
<DeviceACView v-model="open" :data="device" />
```

### DeviceDisplayPoper

Popover that shows the rest of a long device list when only a few are visible inline.

**Props (object form):** `items`, `showAll`, `clickableMore`, `previewCount`, `use`

```vue
<DeviceDisplayPoper :items="devices" :previewCount="3" />
```

---

## tables/

### OCBtnReload

Round reload icon button with tooltip.

**Events:** `onClick`

```vue
<OCBtnReload @onClick="reload" />
```

### OCPagination

Pager control (NuxtUI `UPagination` wrapper).

**v-model:** `page`
**Props:** `table`, `pageCount`, `total`

```vue
<OCPagination v-model:page="page" :pageCount="size" :total="total" />
```

### OCSelectRecord

Dropdown to pick records-per-page (`lengthMenu`).

**v-model:** current selection
**Props:** `loading`, `lengthMenu`
**Events:** `select(value)`

```vue
<OCSelectRecord v-model="size" :lengthMenu="[10, 25, 50, 100]" @select="reload" />
```

### OCTableFooter

Footer used by `OCTable` (records count + pager).

**Props:** `pageCount`, `total`, `options`, `totalPages`, `loading`
**v-models:** `page`, `selected`
**Events:** `changePagination(page)`, `selectRecords(value)`
**Slots:** `#footerLeft`, `#footerCenter`

> Internal — used by `OCTable`. Usually you don't compose this directly.

### OCTableHeader

Header used by `OCTable` (search box, filter chips, reload button, custom toolbar slots).

**Props:** `options`
**v-models:** `selected`, `searching`
**Events:** see source — `search-table`, `clear-search`, `reload`, …
**Slots:** `#headerLeft`, `#headerCenter`, `#headerApplyFilter`, `#headerRight`

> Internal — used by `OCTable`. Use through `OCTable` slots when you need to customize.

---

## treetable/

### OCRowTreeTable

Single row renderer for `OCTreeTable` (handles indent, expand/collapse, custom cell slots).

**Props (object form):** `item`, `columns`, `pk`, `parentId`, `option`, `level`, …
**Events:** `fnRowOnClick`, `fnColumnOnClick`, `fnRowOnHover`, `fnRowOnLeave`
**Slots:** dynamic per column.

> Internal — composed inside `OCTreeTable`.

### OCTreeTable

Hierarchical table with parent/child rows, expand/collapse, search.

**Props:** `options`, `columns`, `localData`, `api`, `pk`, `parentId`, `keySearch`, `searchLocal`
**v-model:** `loading`
**Events:** `mapData`, `fnRowOnClick`, `fnRowOnHover`, `fnRowOnLeave`
**Slots:** `#headerLeft`, `#headerRight`, dynamic cell slots, `#ShowDetailAfterTableOne`
**Exposed:** `reload`, `expandAll`, `collapseAll`, `calcHeight`, `container`

```vue
<OCTreeTable
  :api="{ url:'api/account/tree', method:'POST' }"
  :columns="cols"
  pk="Id"
  parentId="ParentId"
  keySearch="Name"
/>
```

---

## uploadMananger/

### OCCamera

Modal that opens the device camera, captures a frame, optionally crops it.

**v-model:** `Boolean`
**Props:** `width`, `height`, `isBack`, `notCrop`
**Events:** `capture(dataUrl)`, `onBack`

```vue
<OCCamera v-model="camOpen" :width="480" :height="640" @capture="onCapture" />
```

### OCUploadManager

Upload-management modal: file pick / camera / crop combined.

**v-model:** `uploadProfile` (Boolean)
**Props (selected):** `width`, `height`, …
**Events:** `getFile(file)`

```vue
<OCUploadManager v-model="open" @getFile="upload" />
```

### OCUploadProfile

Avatar + upload affordance combining `OCAvatar`, `OCCamera`, and `OCUploadManager`.

**v-models:** default (image), `open` (Boolean)
**Props:** `loading`, `round`, `size`, `border`, `gender`, `errorType`, `cameraLeft`, `iconSize`, `iconBg`, `isSquare`, `height`, `width`, `hideCircle`
**Events:** `onDone(file)`, `onDelete`, `onUpload`

```vue
<OCUploadProfile
  v-model="form.Image"
  size="lg"
  gender="male"
  @onDone="save"
/>
```

---

## wrapperCollapseLeftRight/

### WrapperCollapseLeftRight

Older variant of `OCWrapperCollapseLeftRight` — same two-pane layout with collapse.

**Props (selected):** `widthCollapse`, `isClearPaddingOutSize`, `classes`, …
**v-model:** `isRemovePaddingAndBackground` (Boolean)
**Slots:** `#headerContent`, `#leftContent`, `#rightContent`, `#headerCollapse`

```vue
<WrapperCollapseLeftRight>
  <template #headerContent><PageTitle /></template>
  <template #leftContent>List</template>
  <template #rightContent>Detail</template>
</WrapperCollapseLeftRight>
```

> For new code prefer [OCWrapperCollapseLeftRight](#ocwrappercollapseleftright).

---

## Conventions used in this doc

- **Props (object form)** means the component uses `defineProps({ ... })` with types. Refer to the source file for full types if you need to be precise — only the prop *names* are listed here for brevity.
- **v-model** entries note bindings registered via `defineModel(...)`. The default `v-model` (no name) is shown as `v-model`; named ones as `v-model:name`.
- **Exposed** lists methods returned by `defineExpose(...)` — usable through a `ref()` on the component.
- Where a component has a dedicated long-form doc (see `btn/OCBtn.md`, `form/OCFormCreateModal.md`), this file just links to it.