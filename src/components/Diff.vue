<template>
  <div class="container">
    <div class="text-area-container">
      <div>
        <h3>Left</h3>
        <textarea class="text-area" v-model="leftText"/>
      </div>

      <div>
        <h3>Right</h3>
        <textarea class="text-area" v-model="rightText"/>
      </div>  
    </div>

    <div>
      <input type="checkbox" v-model="newlineIsToken"> Newline is Token
      <input type="checkbox" v-model="ignoreCase"> Ignore Case
      <input type="checkbox" v-model="ignoreWhitespace"> Ignore Whitespace
    </div>

    <div class="diff-output">
      <pre v-html="highlightedDifferences"></pre>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { createTwoFilesPatch, PatchOptions } from 'diff';
import hljs from 'highlight.js';
import 'highlight.js/styles/github.css';

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
  align-items: center;
  height: 100vh;
}

.text-area-container {
  display: flex;
  gap: 20px;
}

.text-area {
  width: 40vw;
  height: 40vh;
}

.diff-output {
  margin-top: 20px;
  padding: 20px;
  background-color: rgb(0, 0, 0);
  width: 80vw;
  text-align: left;
}
</style>