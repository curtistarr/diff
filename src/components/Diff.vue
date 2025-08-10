<template>
  <div class="container">
    <div class="text-area-container">
      <div class="panel card">
        <h3>Left</h3>
        <textarea class="text-area" v-model="leftText"/>
      </div>

      <div class="panel card">
        <h3>Right</h3>
        <textarea class="text-area" v-model="rightText"/>
      </div>  
    </div>

    <div class="controls card">
      <label><input type="checkbox" v-model="newlineIsToken"> Newline is Token</label>
      <label><input type="checkbox" v-model="ignoreCase"> Ignore Case</label>
      <label><input type="checkbox" v-model="ignoreWhitespace"> Ignore Whitespace</label>
    </div>

    <div class="diff-output card">
      <pre v-html="highlightedDifferences"></pre>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { createTwoFilesPatch, PatchOptions } from 'diff';
import hljs from 'highlight.js';
import 'highlight.js/styles/github-dark-dimmed.css';

export default defineComponent({
  data() {
    return {
      leftText: 'Hello Left World\n',
      rightText: 'Hello Right World\n',
      newlineIsToken: false,
      ignoreCase: false,
      ignoreWhitespace: false,
    };
  },
  computed: {
    differences(): string {
      const options: PatchOptions = {
        newlineIsToken: this.newlineIsToken,
        ignoreCase: this.ignoreCase,
        ignoreWhitespace: this.ignoreWhitespace,
      };
      return createTwoFilesPatch('Left', 'Right', this.leftText, this.rightText, '', '', options);
    },
    highlightedDifferences(): string {
      return hljs.highlight('diff', this.differences).value;
    },
  },
});
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  gap: var(--space-8);
}

.text-area-container {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--space-4);
}
@media (min-width: 900px) {
  .text-area-container {
    grid-template-columns: 1fr 1fr;
  }
}

.panel {
  padding: var(--space-4);
}
.panel h3 {
  margin-bottom: var(--space-3);
}

.text-area {
  width: 100%;
  min-height: clamp(220px, 35vh, 380px);
  resize: vertical;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-3) var(--space-6);
  align-items: center;
  padding: var(--space-4);
  border-radius: var(--radius);
}
.controls label {
  display: inline-flex;
  gap: var(--space-2);
  align-items: center;
  color: var(--muted);
}

.diff-output {
  padding: 0; /* pre has its own padding from global styles */
  text-align: left;
}
.diff-output :deep(pre) {
  max-height: 50vh;
}
</style>