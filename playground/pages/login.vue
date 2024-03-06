<script setup lang="ts">
import { NTag, type DataTableColumns, type DataTableInst } from 'naive-ui';

const loading = ref(true);
const client = useSupabaseClient();
const { data: supabaseData } = await useAsyncData("contacts", async () => {
  const { data } = await client.rpc("get_contacts_table", {
    userid: "9e217eca-0358-4b09-8a69-7a5269b2d864",
  });
  return data;
});

type RowData = {
  key: number;
  email: string;
  name: string;
  recency: Date;
  occurence: number;
  tags: string;
};

const createColumns = (): DataTableColumns<RowData> => {
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
      defaultFilterOptionValues: ['personal', 'professional'],
      filterOptions: [
        {
          label: 'Personal',
          value: 'personal'
        },
        {
          label: 'Professional',
          value: 'professional'
        }
      ],
      filter(value, row) {
        return row.tags.includes(value)
      },
      render(row) {
        return row.tags.map((tagKey) =>
          h(
            NTag,
            {
              style: { marginRight: "6px" },
              type: "info",
              bordered: false,
            },
            () => tagKey
          )
        );
      },
    },
  ];
};

const data = ref<RowData[]>(supabaseData);
console.log(supabaseData.value);
loading.value= false;
const columns = ref<DataTableColumns<RowData>>(createColumns());
  const tableRef = ref<DataTableInst>()

  const downloadCsv = () =>
      tableRef.value?.downloadCsv({ fileName: 'data-table' })

  const exportSorterAndFilterCsv = () =>
      tableRef.value?.downloadCsv({
        fileName: 'sorter-filter',
        keepOriginalData: false
      })

</script>

<template>
  <n-space vertical :size="12">
    <n-space>
      <n-button @click="downloadCsv">
        Export CSV (original data)
      </n-button>
      <n-button @click="exportSorterAndFilterCsv">
        Export CSV (displayed data)
      </n-button>
    </n-space>
    <n-data-table
      ref="tableRef"
      :bordered="false"
      :single-line="false"
      :columns="columns"
      :data="data"
      :max-height="700"
      virtual-scroll
      :loading="loading"
    />
  </n-space>
</template>
