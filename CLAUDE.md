This project uses Nuxt 3 with NuxtUI v2 (@nuxt/ui v2.x).

Tech stack:
- Nuxt 3 (latest)
- NuxtUI v2 (not v3) — components prefixed with U (UButton, UCard, etc.)
- Tailwind CSS (via NuxtUI)
- JavaScript (not TypeScript)
- SCSS
- Vue 3 Composition API (<script setup lang="js"> syntax)
- Pinia for state management
- kedtec-module — shared component/composable/util layer

Page
- Don't add NavMain in page
- Wrap all in <OCWrapperPage> and don't wrapp it in <OCContent>
- For data past across the page using queryParams
- For data across the component using Pinia Store
- For delete with confirm using const 
```
confirm = useConfirmStore();
confirm.show({
    message: `${data.rowData.Code} ${t("do_you_want_to_delete")}`,
    type: "delete",
    onConfirm: () => {
    deleteData(data.rowData.Id);
    },
});
```

Form
- Create or Edit form component naming pageName + Form

Coding rules:
- Always use <script setup lang="js"> (never TypeScript)
- Always use <style scoped lang="scss">
- Use auto-imports — never manually import Vue/Nuxt composables
- Never call composables that access $nuxt (like useBreadCrumb) at the top level of <script setup> — always call them inside onMounted()
- Use the mounted() util only for DOM measurement; use onMounted() for composable/store calls
- Pages go in /pages using snake_case filenames (e.g. currency_list.vue)
- Components go in /components using PascalCase filenames (e.g. FormModal.vue) inside camelCase folders
- Stores go in /stores using camelCase (e.g. useCount.ts), one file per store, no subfolder
- All kedtec-module components (OCTable, OCBtn, OCFormCreateModal, etc.) are globally auto-registered
- All composables (useHttp, useBreadCrumb, useConfirmStore, ocdate, isEmpty, etc.) are auto-imported
- For API calls always use useHttp()
- For breadcrumb always call useBreadCrumb() inside onMounted()
- definePageMeta must include layout and middleware

OCTable slot naming rule:
- When using render: '#SlotName', the slot name MUST be different from the column's data field name
  (e.g. data: 'CreatedBy', render: '#CreatedBySlot') to avoid DataTables internal collision
- Table actions @action-click="(action) => fnTableAction(action, cellData) param action is string not object
- Create position it in
 ```
 <template #headerRight>
        <OCBtn
          type="create"
          color="create"
          :label="$t('create')"
          icon="ri-add-circle-line"
          @click="fnClickBtnHeader"
        />
</template>
 ```
- Table actions
```
<template #actions="{ cellData }">
                <OCTblBtns :items="[
                    { type: 'view' },
                    { type: 'edit' },
                    { type: 'delete' },
                ]" @action-click="(action) => fnTableAction(action, cellData)" />
            </template>
```

Column render pattern:
  { title: 'Base', data: 'IsBase', render: '#IsBase' }        // OK — IsBase is not a common DT field
  { title: 'Created By', data: 'CreatedBy', render: '#CreatedBySlot' }  // use Slot suffix

Standard page structure:
  definePageMeta({ layout: 'navbar', middleware: ['auth'], pageTitle: '...' })
  onMounted(() => { useBreadCrumb().add({ label: '...' }) })

Translate Usage
- Don't use $t in setup script
- On setup script use this const { t } = useI18n()