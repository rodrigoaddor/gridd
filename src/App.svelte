<script lang="ts">
  import type { TopAppBarComponentDev } from '@smui/top-app-bar';
  import TopAppBar, { Row, Section, Title, AutoAdjust } from '@smui/top-app-bar';

  import TextField from '@smui/textfield';

  import GridLines, { Player } from './components/GridLines.svelte';

  let topAppBar: TopAppBarComponentDev;

  let size: [number, number] = [3, 3];
  let error: boolean = false;

  let sizeInput: string = size.join('x');
  let oldSizeInput = sizeInput;

  const isValid = (amount: any): boolean => {
    const num = parseInt(amount);
    if (isNaN(num)) return false;
    return amount > 0 && amount < 20;
  };

  const validateSize = (e: CustomEvent<any>) => {
    const { inputType, data: char } = e as any as InputEvent;
    const target = e.target as HTMLInputElement;

    if (/^\d*x\d*$/.test(target.value)) {
      const [x, y] = target.value.split('x').map(Number);
      error = !(isValid(x) && isValid(y));
      if (!error) {
        size = target.value.split('x').map(Number) as [number, number];
      }
      oldSizeInput = target.value;
    } else {
      setTimeout(() => target.setSelectionRange(target.selectionStart - 1, target.selectionStart - 1));
      sizeInput = '';
      sizeInput = oldSizeInput;

      const select = (pos: number) =>
        setTimeout(() => {
          target.setSelectionRange(target.selectionStart + pos, target.selectionStart + pos);
          console.log(target.selectionStart + pos);
        });

      console.log(char, isNaN(parseInt(char)));

      if (inputType === 'deleteContentBackward') {
        // select(-1);
      } else if (char !== null && isNaN(parseInt(char))) {
        select(1);
      }
    }
  };

  let squares: (Player | null)[] = [];
  let lines: (Player | null)[] = [];
  let player: Player = Player.A;
</script>

<template>
  <TopAppBar bind:this={topAppBar} variant="fixed" dense>
    <Row>
      <Section>
        <Title>Gridd</Title>
      </Section>
      <Section align="end" toolbar>
        <TextField
          class="size-field"
          value={sizeInput}
          on:input={validateSize}
          variant="outlined"
          prefix="Size"
          bind:error
          dense
        />
        <!-- iconbutton.material-icons(aria-label='Bookmark this page') bookmark -->
      </Section>
    </Row>
  </TopAppBar>
  <AutoAdjust {topAppBar}>
    <GridLines bind:size bind:squares bind:lines bind:player />
  </AutoAdjust>
</template>

<style lang="sass">
  @use '@material/theme/color-palette' as palette

  :global(.size-field)
    background-color: transparentize(#2a2a2e, 0.4)
    border: none
    border-radius: 4px
    height: 32px
    margin-right: 4px
    width: 12em
    transition: all 0.3s

  :global(.size-field[error='true'])
    background-color: palette.$red-900

  :global(.size-field .mdc-text-field__input)
    margin-left: 12px
      
  :global([class^='mdc-notched-outline'])
    border: none
</style>
