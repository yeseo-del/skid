<template>
  <div id="terminal-logs" class="w-full sm:w-2/3 md:w-1/2 px-4 py-6 font-mono" :style="{ height: logsHeight }" />
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';

import { RootState } from '@/store/state';
import { commandMutations } from '@/store/command/command.mutations';

@Component({})
export default class TerminalLogs extends Vue {
  private unsubscribe!: () => any;

  private logsHeight: string = '100%';

  public mounted(): void {
    this.calculateHeight();

    this.unsubscribe = this.$store.subscribe((mutation, state: RootState) => {
      if (mutation.type === commandMutations.addLog) {
        this.addLog(mutation.payload);
      }

      if (mutation.type === commandMutations.removeLogs) {
        this.clearLogs();
      }
    });
  }

  public beforeDestroy(): void {
    this.unsubscribe();
  }

  private calculateHeight(): void {
    const terminalInput = document.getElementById('terminal-input') as HTMLDivElement;

    this.logsHeight = `calc(100% - ${terminalInput.offsetHeight}px)`;
  }

  /**
   * Called when receiving a `addLog` mutation, add a log in the terminal.
   */
  private addLog(text: string): void {
    const span = document.createElement('span');
    span.setAttribute('class', 'block text-gray-200 mb-2');

    span.innerHTML = text;

    this.$el.appendChild(span);
  }

  /**
   * Remove all logs from both the DOM and the store state.
   */
  private clearLogs(): void {
    while (this.$el.firstChild) {
      this.$el.removeChild(this.$el.firstChild);
    }
  }
}
</script>
