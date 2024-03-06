<script setup lang="ts">
import fetch from "./fetch.vue";
import realtime from "./realtime.vue";
const USER_ID = "9e217eca-0358-4b09-8a69-7a5269b2d864";
</script>

<template>
  <n-card title="Table" style="margin-bottom: 16px">
    <n-tabs type="line" animated>
      <n-tab-pane name="realtime" tab="realtime">
        <realtime :USER_ID="USER_ID" />
      </n-tab-pane>
      <n-tab-pane name="fetch" tab="fetch">
        <fetch :USER_ID="USER_ID" />
      </n-tab-pane>
    </n-tabs>
  </n-card>
</template>
