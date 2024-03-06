<script setup lang="ts">
import fetch from "./fetch.vue";
import realtime from "./realtime.vue";
</script>

<template>
  <n-card title="Table" style="margin-bottom: 16px">
    <n-tabs type="line" animated>
      <n-tab-pane name="realtime" tab="realtime">
        <realtime />
      </n-tab-pane>
      <n-tab-pane name="fetch" tab="fetch">
        <fetch />
      </n-tab-pane>
    </n-tabs>
  </n-card>
</template>
