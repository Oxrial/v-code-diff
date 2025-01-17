<script setup lang="ts">
import { computed, ref, watch, onMounted } from "vue-demi";
import { createSplitDiff, createUnifiedDiff } from "./utils";
import UnifiedViewer from "./unified/UnifiedViewer.vue";
import SplitViewer from "./split/SplitViewer.vue";

import "./style.scss";

interface Props {
  newString: string;
  oldString: string;
  language?: string;
  context?: number;
  diffStyle?: "word" | "char";
  outputFormat?: "line-by-line" | "side-by-side";
  trim?: boolean;
  noDiffLineFeed?: boolean;
  maxHeight?: string;
  filename?: string;
}

const props = withDefaults(defineProps<Props>(), {
  language: "plaintext",
  context: 10,
  diffStyle: "word",
  outputFormat: "line-by-line",
  trim: false,
  noDiffLineFeed: false,
  maxHeight: undefined,
  filename: undefined,
});
onMounted(
  () => importTheme()
  //   switch (props.theme) {
  //     case 'vs':
  //       import('highlight.js/scss/vs.scss')
  //       break
  //     case 'monokai-sublime':
  //       import('highlight.js/scss/monokai-sublime.scss')
  //       break
  //     case 'synth-midnight-terminal-dark':
  //       import('highlight.js/scss/base16/synth-midnight-terminal-dark.scss')
  //       break
  //     case 'tomorrow-night':
  //       import('highlight.js/scss/base16/tomorrow-night.scss')
  //   }
);
const importTheme = (func: Function | null = null): any =>
  (func && func()) ||
  (function () {
    import("./style/monaco-yaml.scss");
  })();
defineExpose({ importTheme });
const isUnifiedViewer = computed(() => props.outputFormat === "line-by-line");

const oldString = computed(() => {
  let value = props.oldString || "";
  value = props.trim ? value.trim() : value;
  value = props.noDiffLineFeed ? value.replace(/(\r\n)/g, "\n") : value;
  return value;
});
const newString = computed(() => {
  let value = props.newString || "";
  value = props.trim ? value.trim() : value;
  value = props.noDiffLineFeed ? value.replace(/(\r\n)/g, "\n") : value;
  return value;
});

const raw = computed(() =>
  isUnifiedViewer.value
    ? createUnifiedDiff(
        oldString.value,
        newString.value,
        props.language,
        props.diffStyle,
        props.context
      )
    : createSplitDiff(
        oldString.value,
        newString.value,
        props.language,
        props.diffStyle,
        props.context
      )
);
const diffChange = ref(raw.value);
watch(
  () => props,
  () => {
    diffChange.value = raw.value;
  },
  { deep: true }
);
</script>

<template>
  <div class="code-diff-view" :style="{ maxHeight }">
    <div class="file-header">
      <div class="file-info">
        <span class="filename">{{ filename }}</span>
        <span class="diff-stat">
          <span class="diff-stat-added"
            >+{{ diffChange.stat.additionsNum }} additions</span
          >
          <span class="diff-stat-deleted" style="margin-left: 8px"
            >-{{ diffChange.stat.deletionsNum }} deletions</span
          >
        </span>
      </div>
    </div>
    <UnifiedViewer v-if="isUnifiedViewer" :diff-change="diffChange" />
    <SplitViewer v-else :diff-change="diffChange" />
  </div>
</template>

<style lang="scss"></style>
