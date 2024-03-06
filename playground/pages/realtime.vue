<script setup lang="ts">
import { NTag, type DataTableColumns, type DataTableInst } from "naive-ui";
import { ref } from "vue";

const USER_ID = "8697d21d-0cd4-4818-83f7-eb46639d6c14";
const loading = ref(true);
const supabase = useSupabaseClient();
const createColumns = (): DataTableColumns => {
  return [
    {
      title: "Email",
      key: "email",
      sorter: "default",
    },
    {
      title: "Name",
      key: "name",
      sorter: "default",
    },
    {
      title: "Recency",
      key: "recency",
      sorter: "default",
    },
    {
      title: "Occurrence",
      key: "occurrence",
      sorter: "default",
    },
    {
      title: "tag",
      key: "tags",
      defaultFilterOptionValues: ["personal", "professional"],
      filterOptions: [
        {
          label: "Personal",
          value: "personal",
        },
        {
          label: "Professional",
          value: "professional",
        },
      ],
      filter(value, row) {
        if (!row.tags) return true;
        return row.tags?.includes(value);
      },
      render(row) {
        if (!row.tags) return;
        return row.tags.map((tagKey) =>
          h(
            NTag,
            {
              style: { marginRight: "6px" },
              type: "info",
              bordered: false,
            },
            () => tagKey,
          ),
        );
      },
    },
  ];
};

const rows = ref([]);

loading.value = false;
const columns = ref<DataTableColumns>(createColumns());
const tableRef = ref<DataTableInst>();

const downloadCsv = () =>
  tableRef.value?.downloadCsv({ fileName: "data-table" });

const exportSorterAndFilterCsv = () =>
  tableRef.value?.downloadCsv({
    fileName: "sorter-filter",
    keepOriginalData: false,
  });
const contactsCache = new Map();

onNuxtReady(() => {
  setInterval(() => {
    rows.value = Array.from(contactsCache.values());
    console.log(rows.value);
  }, 5000);
  supabase
    .channel("*")
    .on(
      "postgres_changes",
      {
        event: "*",
        schema: "public",
        table: "persons",
        filter: `user_id=eq.${USER_ID}`,
      },
      (payload) => {
        const newContact = payload.new as Contact;
        if (newContact) {
          contactsCache.set(newContact.email, newContact);
        }
      },
    )
    .subscribe();
});
</script>

<template>
  <n-space>
    <n-button @click="downloadCsv"> Export CSV (original data) </n-button>
    <n-button @click="exportSorterAndFilterCsv">
      Export CSV (displayed data)
    </n-button>
  </n-space>

  <n-data-table
    ref="tableRef"
    :bordered="false"
    :single-line="false"
    :columns="columns"
    :data="rows"
    :max-height="700"
    virtual-scroll
    :loading="loading"
  />
</template>
