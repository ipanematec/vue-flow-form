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

      @input="doSearch"
      @keydown.down.prevent="onArrowDown"
      @keydown.up.prevent="onArrowUp"

      autocomplete="off"
      autocorrect="off"
      autocapitalize="off"
      spellcheck="false" />

    <transition-group
      name="autocomplete-transitions"
      :css="false"

      tag="ul"
      v-if="isLoading || results"
      class="autocomplete-results"
      @mouseout="resetSelectionIndex" >
      <li
        v-if="isLoading"
        :key="'loading'"
        class="autocomplete-result" >
        {{ this.language.autocompleteLoading }}
      </li>
      <li
        v-else-if="!isValidSearch"
        :key="'invalid'"
        class="autocomplete-result f-invalid" >
        {{ this.language.autocompleteInvalidSearch }}
      </li>
      <li
        v-else-if="results.length === 0"
        :key="'empty'"
        class="autocomplete-result f-invalid" >
        {{ this.language.autocompleteNoResults }}
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
    </transition-group>
  </div>
</template>

<script>
  import BaseType from './BaseType.vue';
  import { QuestionType } from '../../models/QuestionModel';
  import LanguageModel from '../../models/LanguageModel';
  // import debounce from 'vue-debounce/src/debounce';

  // TODO Consider using https://github.com/foxbenjaminfox/vue-async-computed

  export default {
    extends: BaseType,
    name: QuestionType.Autocomplete,
    data() {
      return {
        // BaseType API:
        canReceiveFocus: true,

        results: null,
        isLoading: false,
        isValidSearch: null,
        currentSelectionIndex: -1
        // debouncedDoSearch: null
      };
    },
    methods: {
      async doSearch() {
        // BaseType API:
        this.onChange({ target: { value: '' } });

        this.resetSelectionIndex();

        const timeout = setTimeout(() => { this.isLoading = true }, 800);
        this.results = await this.question.searchFunction(this.$refs.input.value) ?? [];
        clearTimeout(timeout);

        if (this.results === 'invalid') {
          this.isValidSearch = false;
          this.results = [];
        } else {
          this.isValidSearch = true;
        }

        this.isLoading = false;
      },
      selectResult(result) {
        this.$refs.input.value = result.label;

        this.results = null;
        this.resetSelectionIndex();

        // BaseType API:
        this.onChange({ target: { value: result.value } });
      },

      // Handling currentSelectionIndex:

      onMouseOver(index) {
        this.currentSelectionIndex = index;
      },
      onArrowDown() {
        if (this.currentSelectionIndex < this.results?.length - 1)
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
        if (this.currentSelectionIndex < 0 || this.currentSelectionIndex >= this.results?.length)
          return;
        this.selectResult(this.results[this.currentSelectionIndex]);
      }

      // TODO Allow client app to use GSAP ( https://v3.vuejs.org/guide/transitions-list.html#staggering-list-transitions )
      // beforeEnter(el) {
      //   el.style.opacity = 0
      //   el.style.height = 0
      // },
      // enter(el, done) {
      //   gsap.to(el, {
      //     duration: 0.1,
      //     opacity: 1,
      //     height: '1.6em',
      //     onComplete: done
      //   })
      // },
      // leave(el, done) {
      //   gsap.to(el, {
      //     duration: 0.1,
      //     opacity: 0,
      //     height: 0,
      //     onComplete: done
      //   })
      // }
    },
    computed: {
      placeholder() {
        return this.question.placeholder || this.language.autocompletePlaceholder
      }
    }
    // created() {
    //   this.debouncedDoSearch = debounce(this.doSearch.bind(this), '200ms');
    // }
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

  .autocomplete-result.f-invalid {
    margin-top: 0.5rem;
    cursor: inherit;
    font-size: inherit;
  }

  .autocomplete-result.currently-selected {
    font-weight: bold;
  }

  /* TODO Allow client app to use GSAP ( https://v3.vuejs.org/guide/transitions-list.html#staggering-list-transitions ) */
  /* .autocomplete-transitions-move {
    transition: all 0.1s ease;
  }
  .autocomplete-transitions-enter-active,
  .autocomplete-transitions-leave-active {
    display: none;
  } */
</style>
