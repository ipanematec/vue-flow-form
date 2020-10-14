<template>
  <div class="autocomplete">
    <!-- The first attributes are for BaseType API -->
    <!-- v-model is not working properly on Android Chrome (problems with autosuggest / autocorrect) -->
    <input
      ref="input"
      type="text"
      :required="question.required"
      :placeholder="placeholder"
      :focus="setFocus"
      :blur="unsetFocus"

      @input="debouncedDoSearch"
      @keydown.down.prevent="onArrowDown"
      @keydown.up.prevent="onArrowUp"

      autocomplete="off"
      autocorrect="off"
      autocapitalize="off"
      spellcheck="false" />

    <ul
      v-if="isLoading || results.length > 0"
      class="autocomplete-results"
      @mouseout="resetSelectionIndex" >
      <li
        v-if="isLoading"
        class="autocomplete-result" >
        {{ this.language.loadingIndicator }}
      </li>
      <template v-else>
        <li
          v-for="(result, index) in results"
          :key="result.value"
          @click="selectResult(result)"
          @mouseover="onMouseOver(index)"
          class="autocomplete-result"
          :class="{ 'currently-selected': index === currentSelectionIndex }" >
          {{ result.label }}
        </li>
      </template>
    </ul>
  </div>
</template>

<script>
  import BaseType from './BaseType.vue';
  import { QuestionType } from '../../models/QuestionModel';
  import LanguageModel from '../../models/LanguageModel';
  import { debounce } from 'vue-debounce';

  export default {
    extends: BaseType,
    name: QuestionType.Autocomplete,
    data() {
      return {
        // BaseType API:
        canReceiveFocus: true,

        results: [],
        isLoading: false,
        currentSelectionIndex: -1,
        debouncedDoSearch: null
      };
    },
    methods: {
      async doSearch() {
        // BaseType API:
        this.onChange({ target: { value: '' } });

        this.resetSelectionIndex();

        const timeout = setTimeout(() => { this.isLoading = true }, 200);
        this.results = await this.question.searchFunction(this.$refs.input.value) ?? [];
        clearTimeout(timeout);
        this.isLoading = false;
      },
      selectResult(result) {
        this.$refs.input.value = result.label;

        this.resetSelectionIndex();

        // BaseType API:
        this.onChange({ target: { value: result.value } });
      },

      // Handling currentSelectionIndex:

      onMouseOver(index) {
        this.currentSelectionIndex = index;
      },
      onArrowDown() {
        if (this.currentSelectionIndex < this.results.length - 1)
          ++this.currentSelectionIndex;
      },
      onArrowUp() {
        if (this.currentSelectionIndex >= 0)
          --this.currentSelectionIndex;
      },
      resetSelectionIndex() {
        this.currentSelectionIndex = -1;
      },

      // Called by BaseType API:
      onEnter() {
        if (this.currentSelectionIndex === -1)
          return;
        this.selectResult(this.results[this.currentSelectionIndex]);
      }
    },
    computed: {
      placeholder() {
        return this.question.placeholder || this.language.placeholderAutocomplete
      }
    },
    created() {
      this.debouncedDoSearch = debounce(this.doSearch.bind(this), '200ms');
    }
  };
</script>

<style scoped>
  .autocomplete-results {
    padding: 0;
    margin-top: 0.2rem;
  }

  .autocomplete-result {
    padding-left: 0.2rem;
    padding-right: 0.2rem;
    list-style: none;
    cursor: pointer;
    font-size: 1.7rem;
    font-weight: normal;
  }

  .autocomplete-result.currently-selected {
    font-weight: bold;
  }
</style>
