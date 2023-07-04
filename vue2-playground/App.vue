<script setup lang="ts">
import { onMounted, reactive, ref, version } from "vue-demi";
import hljs from "highlight.js";

const form = reactive({
  oldString: "a:\n b: c",
  newString: "a:\n bc: \n c: d",
  language: "yaml",
  diffStyle: "word",
});
const code = ref(null);
onMounted(() => {});
</script>

<template>
  <div>
    <p align="center">Vue version: {{ version }}</p>
    <textarea v-model="form.oldString" style="width: 48vw" :rows="20" />
    <textarea
      v-model="form.newString"
      style="width: 48vw; margin-left: 20px"
      :rows="20"
    />
    <CodeDiff
      ref="code"
      :old-string="form.oldString"
      theme="tomorrow-night"
      :new-string="form.newString"
      :language="form.language"
      :diff-style="form.diffStyle"
      output-format="side-by-side"
    />
  </div>
</template>

<style>
body > div {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
