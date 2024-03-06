<script setup lang="ts">
import {
  NBadge,
  NTag,
  type DataTableColumns,
  type DataTableInst,
} from "naive-ui";
import type { RowData } from "naive-ui/es/data-table/src/interface";
import { ref } from "vue";

const props = defineProps({
  USER_ID: {
    required: true,
  },
});

const loading = ref(true);
const supabase = useSupabaseClient();
const { data: supabaseData } = await useAsyncData("contacts", async () => {
  const { data } = await supabase.rpc("get_contacts_table", {
    userid: props.USER_ID,
  });
  return data;
});

const settingsMapper = {
  UNKNOWN: {
    color: "grey",
    tooltip: "Unknown",
  },
  VALID: {
    color: "green",
    tooltip: "Valid email address",
  },
  INVALID: {
    color: "red",
    tooltip: "Invalid email address",
  },
  RISKY: {
    color: "orange",
    tooltip: "Risky email address",
  },
};
function getColor(status: string) {
  if (!status) {
    return;
  }
  return settingsMapper[status].color;
}

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
    {
      title: "status",
      key: "status",
      sorter: "default",
      render(row) {
        if (!row.status) return;
        return h(NBadge, {
          color: getColor(row.status as string),
          dot: true,
        });
      },
    },
  ];
};

const rows = ref<RowData[]>(supabaseData);

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

const pagination = reactive({
  page: 1,
  pageSize: 150,
  showSizePicker: true,
  pageSizes: [150, 500, 10000],
  onChange: (page: number) => {
    pagination.page = page;
  },
  onUpdatePageSize: (pageSize: number) => {
    pagination.pageSize = pageSize;
    pagination.page = 1;
  },
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
    :pagination="pagination"
  />
</template>
