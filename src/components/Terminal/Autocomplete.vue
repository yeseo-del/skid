<template>
  <div
    id="autocomplete-tooltip"
    class="z-10 absolute rounded py-2 px-4 opacity-0 pointer-events-none text-grey-darkest bg-white shadow-md"
  >
    <div v-if="suggestions.length">
      <p v-for="(suggestion, index) in suggestions" :key="'suggestion-' + index">{{suggestion}}</p>
    </div>

    <span v-else>No suggestions</span>

    <!--
      Thoses are fake elements, theyr are needed to calculate both the width of
      the text input and the height of the suggestions. It doesn't interfere
      with other DOM elements.
    -->
    <span ref="text-input-ruler" class="absolute opacity-0 pointer-events-none"></span>
    <span ref="suggestions-ruler" class="absolute opacity-0 pointer-events-none"></span>
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
import { autocomplete } from '@totominc/command-parser';

import { Command } from '@/models/command.model';
import { CommandState } from '@/store/command/command.state';
import { commandMutations } from '@/store/command/command.mutations';

/**
 * The autocomplete component must be used as a child of the Input component,
 * otherwise it won't work because it depends of this parent (using Input's
 * `$refs`).
 */
@Component({})
export default class TerminalAutocomplete extends Vue {
  public suggestions: string[] = [];

  public get commands(): CommandState {
    return this.$store.state.commands;
  }

  public mounted(): void {
    this.$store.subscribe(({ type, payload }) => {
      if (type === commandMutations.toggleAutocompletion) {
        this.commands.isInAutocomplete ? this.showAutocomplete() : this.hideAutocomplete();
      }
    });
  }

  /**
   * Call the `autocomplete` function from the command-parser to retrieve and
   * store an array of all possibilities.
   */
  private setSuggestions(): string[] {
    const suggestions = autocomplete<Command>(this.commands.inputContent, this.commands.commands);

    this.suggestions = suggestions;

    return suggestions;
  }

  /**
   * Calculate the top position which depends on the tooltip suggestions height
   * and the terminal text-input height.
   */
  private calculateTopPosition(): number {
    const terminalInputEl = this.$parent.$refs['terminal-input'] as HTMLDivElement;
    const suggestionsRulerEl = this.$refs['suggestions-ruler'] as HTMLSpanElement;

    const noSuggestionsHeight = this.suggestions.length ? 0 : 16;

    suggestionsRulerEl.innerHTML = this.suggestions.join('<br>');

    return suggestionsRulerEl.offsetHeight + terminalInputEl.offsetHeight + noSuggestionsHeight;
  }

  /**
   * Calculate the width of the text inside the terminal text-input using the
   * ruler element.
   */
  private calculateTextInputLength(): number {
    const terminalInputEl = this.$parent.$refs['terminal-input'] as HTMLDivElement;
    const textInputRulerEl = this.$refs['text-input-ruler'] as HTMLSpanElement;

    textInputRulerEl.innerText = terminalInputEl.innerText;

    return textInputRulerEl.offsetWidth;
  }

  /**
   * Calculate the left position of the tooltip which depends on the terminal
   * name width, the width of the text inside the terminal text-input and the
   * width of the autocomplete element.
   */
  private calculateLeftPosition(): number {
    const terminalNameEl = this.$parent.$refs['terminal-name'] as HTMLSpanElement;
    const terminalInputEl = this.$parent.$refs['terminal-input'] as HTMLDivElement;
    const autocompleteEl = this.$el as HTMLDivElement;

    const terminalNameWidth = terminalNameEl.offsetWidth + 24;
    const terminalTextWidth = this.calculateTextInputLength();
    const autocompleteWidth = autocompleteEl.offsetWidth;

    return terminalNameWidth + terminalTextWidth - autocompleteWidth / 2;
  }

  private showAutocomplete(): void {
    this.setSuggestions();

    const parent = this.$parent;
    const autocompleteEl = this.$el as HTMLDivElement;

    autocompleteEl.style.transition = 'all 0s ease-in-out';
    autocompleteEl.style.left = `${this.calculateLeftPosition()}px`;
    autocompleteEl.style.top = `-${this.calculateTopPosition()}px`;

    autocompleteEl.style.transition = 'all 0.25s ease-in-out';
    autocompleteEl.style.transform = 'translateY(6px)';
    autocompleteEl.style.pointerEvents = 'auto';
    autocompleteEl.style.opacity = '1';
  }

  private hideAutocomplete(): void {
    const autocompleteEl = this.$el as HTMLDivElement;

    autocompleteEl.style.transform = 'translateY(-6px)';
    autocompleteEl.style.pointerEvents = 'none';
    autocompleteEl.style.opacity = '0';
  }
}
</script>

<style scoped>
#autocomplete-tooltip:after {
  top: 100%;
  left: 50%;
  border: solid transparent;
  content: ' ';
  position: absolute;
  pointer-events: none;
  border-color: rgba(255, 255, 255, 0);
  border-top-color: #ffffff;
  border-width: 6px;
  margin-left: -6px;
}
</style>