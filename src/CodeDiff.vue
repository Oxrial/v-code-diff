<script setup lang="ts">
import { computed, onMounted } from "vue-demi";
import { createSplitDiff, createUnifiedDiff } from "./utils";
import UnifiedViewer from "./unified/UnifiedViewer.vue";
import SplitViewer from "./split/SplitViewer.vue";

import "./style.scss";

interface Props {
  theme: string;
  newString: string;
  oldString: string;
  language?: string;
  context?: number;
  diffStyle?: "word" | "char";
  outputFormat?: "line-by-line" | "side-by-side";
  trim?: boolean;
  noDiffLineFeed?: boolean;
  maxHeight?: number;
  filename?: string;
}

const props = withDefaults(defineProps<Props>(), {
  theme: "",
  language: "plaintext",
  context: 10,
  diffStyle: "word",
  outputFormat: "line-by-line",
  trim: false,
  noDiffLineFeed: false,
  maxHeight: undefined,
  filename: undefined,
});
onMounted(() => {
  importTheme();
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
});
const importTheme = (func: Function | null = null): any =>
  (func && func()) ||
  (function () {
    import("./style/monaco-yaml.scss");
  })();
defineExpose({ importTheme });
const isUnifiedViewer = computed(() => props.outputFormat === "line-by-line");

const oldString = computed(() => {
  const value = props.trim ? props.oldString.trim() : props.oldString;
  return props.noDiffLineFeed ? value.replace(/(\r\n)/g, "\n") : value;
});
const newString = computed(() => {
  const value = props.trim ? props.newString.trim() : props.newString;
  return props.noDiffLineFeed ? value.replace(/(\r\n)/g, "\n") : value;
});

const diffChange = computed(() =>
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
    <UnifiedViewer v-if="isUnifiedViewer" :diff-change="diffChange.changes" />
    <SplitViewer v-else :diff-change="diffChange.changes" />
  </div>
</template>

<style lang="scss"></style>
