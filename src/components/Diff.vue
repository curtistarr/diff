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
      <label class="display-mode" for="display-mode">Display mode
        <select id="display-mode" v-model="displayMode" aria-label="Select diff display mode">
          <option value="combined">Combined</option>
          <option value="split">Split</option>
        </select>
      </label>
    </div>

    <div class="diff-output card">
      <pre v-if="displayMode === 'combined'" v-html="highlightedDifferences"></pre>

      <div v-else class="split-diff" aria-label="Split diff view">
        <div class="split-header">
          <div>Left</div>
          <div>Right</div>
        </div>
        <div class="split-body" role="table" aria-label="Side by side diff">
          <div
            v-for="(row, index) in splitRows"
            :key="`${index}-${row.leftText}-${row.rightText}`"
            class="split-row"
            role="row"
          >
            <div :class="['cell', row.leftType]" role="cell">{{ row.leftText }}</div>
            <div :class="['cell', row.rightType]" role="cell">{{ row.rightText }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { createTwoFilesPatch, diffLines, Change, PatchOptions } from 'diff';
import hljs from 'highlight.js';

type SplitRow = {
  leftText: string;
  rightText: string;
  leftType: 'added' | 'removed' | 'unchanged' | 'empty';
  rightType: 'added' | 'removed' | 'unchanged' | 'empty';
};

export default defineComponent({
  data() {
    return {
      leftText: 'Hello Left World\n',
      rightText: 'Hello Right World\n',
      newlineIsToken: false,
      ignoreCase: false,
      ignoreWhitespace: false,
      displayMode: 'combined' as 'combined' | 'split',
    };
  },
  computed: {
    diffOptions(): PatchOptions {
      return {
        newlineIsToken: this.newlineIsToken,
        ignoreCase: this.ignoreCase,
        ignoreWhitespace: this.ignoreWhitespace,
      };
    },
    differences(): string {
      return createTwoFilesPatch('Left', 'Right', this.leftText, this.rightText, '', '', this.diffOptions);
    },
    highlightedDifferences(): string {
      return hljs.highlight('diff', this.differences).value;
    },
    splitRows(): SplitRow[] {
      const chunks: Change[] = diffLines(this.leftText, this.rightText, this.diffOptions);
      const rows: SplitRow[] = [];

      const chunkLines = (value: string): string[] => {
        const lines = value.split('\n');
        if (lines[lines.length - 1] === '') {
          lines.pop();
        }
        return lines;
      };

      let index = 0;
      while (index < chunks.length) {
        const current = chunks[index];
        const next = chunks[index + 1];

        if (current.removed && next?.added) {
          const leftLines = chunkLines(current.value);
          const rightLines = chunkLines(next.value);
          const maxLines = Math.max(leftLines.length, rightLines.length);

          for (let i = 0; i < maxLines; i += 1) {
            rows.push({
              leftText: leftLines[i] ?? '',
              rightText: rightLines[i] ?? '',
              leftType: leftLines[i] !== undefined ? 'removed' : 'empty',
              rightType: rightLines[i] !== undefined ? 'added' : 'empty',
            });
          }
          index += 2;
          continue;
        }

        const lines = chunkLines(current.value);
        if (current.added) {
          lines.forEach((line) => {
            rows.push({ leftText: '', rightText: line, leftType: 'empty', rightType: 'added' });
          });
        } else if (current.removed) {
          lines.forEach((line) => {
            rows.push({ leftText: line, rightText: '', leftType: 'removed', rightType: 'empty' });
          });
        } else {
          lines.forEach((line) => {
            rows.push({ leftText: line, rightText: line, leftType: 'unchanged', rightType: 'unchanged' });
          });
        }
        index += 1;
      }

      return rows;
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
.controls .display-mode {
  gap: var(--space-3);
}
.controls select {
  padding: 0.35rem 0.5rem;
  border-radius: var(--radius);
  border: 1px solid var(--border-subtle);
  background-color: var(--surface-1);
  color: var(--text);
}

.diff-output {
  padding: 0; /* pre has its own padding from global styles */
  text-align: left;
  max-width: 90vw;
}
.diff-output :deep(pre) {
  max-height: 50vh;
}

.split-diff {
  display: grid;
  grid-template-rows: auto 1fr;
  gap: var(--space-2);
  overflow-x: auto;
}

.split-header {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  gap: var(--space-1);
  padding: 0 var(--space-3);
  font-weight: 600;
  color: var(--muted);
}

.split-body {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  gap: var(--space-1);
  align-items: start;
  width: 100%;
}

.split-row {
  display: contents;
}

.cell {
  padding: 0.35rem 0.75rem;
  white-space: pre;
  border-radius: var(--radius-sm);
  min-height: 1.75rem;
}

.cell.added {
  background-color: rgba(46, 160, 67, 0.15);
}

.cell.removed {
  background-color: rgba(248, 81, 73, 0.15);
}

.cell.unchanged {
  background-color: rgba(110, 118, 129, 0.08);
}

.cell.empty {
  background-color: transparent;
}

@media (max-width: 720px) {
  .split-body,
  .split-header {
    grid-template-columns: 1fr;
  }

  .split-row {
    display: grid;
    grid-template-columns: 1fr;
  }
}
</style>
