<script lang="ts" context="module">
  export type LineCallback = (lineIndex: number) => void;
  export type SquareData = boolean;
  export enum Player {
    A,
    B,
  }

  enum ElementType {
    DOT = 'dot',
    HORIZONTAL_LINE = 'horizontal-line',
    VERTICAL_LINE = 'vertical-line',
    SQUARE = 'square',
  }

  interface ElementData {
    type: ElementType;
    owner?: Player;
    handler?: () => void;
  }
</script>

<script lang="ts">
  export let size: [number, number];

  export let lines: (Player | null)[];
  export let squares: (Player | null)[];

  export let player: Player;

  let elements: ElementData[];

  const getElementType = (col: number, line: number): ElementType => {
    if (col % 2 == 0 && line % 2 == 0) return ElementType.DOT;
    else if (col % 2 == 0 && line % 2) return ElementType.VERTICAL_LINE;
    else if (col % 2 && line % 2 == 0) return ElementType.HORIZONTAL_LINE;
    else if (col % 2 && line % 2) return ElementType.SQUARE;
    else throw Error('Invalid state: invalid column and line combination');
  };

  const findGroup = (n: number, ...groups: number[]) => {
    let cur = 0;
    let group = 0;
    let groupsAmount: number[] = Array.from({ length: groups.length }, () => 0);

    do {
      const curGroup = group % groups.length;

      if (n >= cur && n < cur + groups[curGroup]) {
        return {
          index: n - cur, // element's index in the group instance the element is in
          group: groups[curGroup], // group the element is in
          groupNumber: groupsAmount[curGroup], // index of the group instance the element is in
          groupIndex: groupsAmount[curGroup] * groups[curGroup] + (n - cur), // element's index in the group the element is in, cummulative
        };
      }

      cur += groups[curGroup];
      group++;
      groupsAmount[curGroup]++;
      if (group >= groups.length) group = 0;
    } while (cur <= n);
  };

  const checkCompletion = (lineIndex: number) => {
    const { index, group, groupIndex } = findGroup(lineIndex, size[0], size[0] + 1);
    const lineType = group === size[0] ? ElementType.HORIZONTAL_LINE : ElementType.VERTICAL_LINE;
  };

  const getRelatedSquares = (lineIndex: number) => {
    const result: { square: number; lines: number[] }[] = [];
    const getLines = (...offsets: number[]) => offsets.map((offset) => lineIndex + offset);

    const xOffset = size[0];

    const { index, group, groupIndex, groupNumber } = findGroup(lineIndex, xOffset, xOffset + 1);
    const lineType = group === xOffset ? ElementType.HORIZONTAL_LINE : ElementType.VERTICAL_LINE;

    if (lineType === ElementType.HORIZONTAL_LINE) {
      if (groupNumber === 0) {
        // top row, only one square (below)
        result.push({ square: groupIndex, lines: getLines(0, xOffset, xOffset + 1, xOffset * 2 + 1) });
      } else if (groupNumber === size[1]) {
        // bottom row, only one square (above)
        result.push({ square: groupIndex - xOffset, lines: getLines(0, -xOffset, -xOffset - 1, -xOffset * 2 - 1) });
      } else {
        // middle rows, two squares (above and below)
        result.push({ square: groupIndex - xOffset, lines: getLines(0, -xOffset, -xOffset - 1, -xOffset * 2 - 1) });
        result.push({ square: groupIndex, lines: getLines(0, xOffset, xOffset + 1, xOffset * 2 + 1) });
      }
    } else {
      if (index === 0) {
        // start column, only one square (right)
        result.push({ square: groupNumber * xOffset, lines: getLines(0, -xOffset, 1, xOffset + 1) });
      } else if (index === xOffset) {
        // end colunm, only one square (left)
        result.push({ square: (groupNumber + 1) * xOffset - 1, lines: getLines(0, -xOffset - 1, -1, xOffset) });
      } else {
        // middle columns, two squares (left and right)
        result.push({ square: groupNumber * xOffset + index, lines: getLines(0, -xOffset, 1, xOffset + 1) });
        result.push({ square: groupNumber * xOffset + index - 1, lines: getLines(0, -xOffset - 1, -1, xOffset) });
      }
    }

    return result;
  };

  const getRelatedLines = (lineIndex: number): number[] => {
    const result: number[] = [];

    const addResults = (...offsets: number[]) => offsets.forEach((offset) => result.push(lineIndex + offset));

    const { index, group } = findGroup(lineIndex, size[0], size[0] + 1);
    const lineType = group === size[0] ? ElementType.HORIZONTAL_LINE : ElementType.VERTICAL_LINE;

    if (lineType === ElementType.HORIZONTAL_LINE) {
      addResults(-size[0] * 2 - 1, size[0] * 2 + 1, -size[0], -size[0] - 1, size[0], size[0] + 1);
      // addResults(-7, -4, -size[0], size[0], 4, 7);
    } else if (index === 0) {
      addResults(-size[0], 1, size[0] + 1);
    } else if (index === size[0]) {
      addResults(-size[0] - 1, -1, size[0]);
    } else {
      addResults(-size[0] - 1, -size[0], -1, 1, size[0], size[0] + 1);
    }

    return result.filter((index) => index >= 0 && index < lines.length);
  };

  $: {
    let lineIndex = 0;
    let squareIndex = 0;

    elements = Array.from<number, ElementData>({ length: (size[0] * 2 + 1) * (size[1] * 2 + 1) }, (_, i) => {
      const col = i % (size[0] * 2 + 1);
      const line = Math.floor(i / (size[0] * 2 + 1));

      const type = getElementType(col, line);
      let owner: Player | undefined;
      let handler: (() => void) | undefined;

      if (type === ElementType.VERTICAL_LINE || type === ElementType.HORIZONTAL_LINE) {
        let thisLineIndex = lineIndex++;
        owner = lines[thisLineIndex];
        lines[thisLineIndex] ??= null;
        handler = () => {
          // set clicked line to active
          lines[thisLineIndex] = player;

          let squareFilled = false;

          // verify if a square can be activated
          getRelatedSquares(thisLineIndex).forEach(({ square, lines: relatedLines }) => {
            if (relatedLines.every((line) => lines[line] !== null)) {
              squares[square] = player;
              squareFilled = true;
            }
          });

          if (!squareFilled) {
            player = player === Player.A ? Player.B : Player.A;
          }

          // trigger updates if needed
          squares = squares;
          lines = lines;
        };
      } else if (type === ElementType.SQUARE) {
        let thisSquareIndex = squareIndex++;
        owner = squares[thisSquareIndex];
        squares[thisSquareIndex] ??= null;
      }

      return { type, owner, handler };
    });
  }

  /*-- Reset grid on size change --*/

  let oldSize = size;

  $: {
    if (size.join('x') !== oldSize.join('x')) {
      lines = Array.from({ length: lines.length }, () => null);
      squares = Array.from({ length: squares.length }, () => null);
      oldSize = size;
    }
  }
</script>

<template>
  <div class="wrapper" style:--rows={size[1]} style:--columns={size[0]}>
    {#each elements as item, index}
      {#if item.type === ElementType.DOT}
        <div class="dot" />
      {:else if item.type === ElementType.HORIZONTAL_LINE || item.type === ElementType.VERTICAL_LINE}
        <div
          class="line hoverable"
          class:horizontal={item.type === ElementType.HORIZONTAL_LINE}
          class:vertical={item.type === ElementType.VERTICAL_LINE}
          class:active={item.owner !== null}
          class:player-a={item.owner === Player.A}
          class:player-b={item.owner === Player.B}
          on:click={item.handler}
        />
      {:else}
        <div
          class="square"
          class:active={item.owner !== null}
          class:player-a={item.owner === Player.A}
          class:player-b={item.owner === Player.B}
        />
      {/if}
    {/each}
  </div>
</template>

<style lang="sass">
  @use '@material/theme/color-palette' as palette

  $thickness: .25rem
  $grid-color: white
  $player-a: palette.$red-500
  $player-b: palette.$blue-500

  .wrapper
    display: grid
    grid-template-columns: repeat(var(--columns), 16px 64px) 16px
    grid-template-rows: repeat(var(--rows), 16px 64px) 16px
    margin: 32px
    place-items: center

  .dot
    height: $thickness
    width: $thickness
    background-color: $grid-color
    border-radius: 50%

  .line
    &::after
      display: block
      border-radius: $thickness
      background-color: $grid-color
      content: ' '

    &.player-a::after
      background-color: $player-a
    
    &.player-b::after
      background-color: $player-b

    &.horizontal, &.vertical
      display: grid
      place-items: center
      height: 100%
      width: 100%

    &.horizontal::after
      height: $thickness
      width: 100%
    
    &.vertical::after
      width: $thickness
      height: 100%

    &.hoverable
      opacity: 0
      cursor: pointer

    &.hoverable:hover, &.active
      opacity: 1

    &.active
      cursor: auto
      pointer-events: none
    
  .square
    min-width: 64px
    aspect-ratio: 1/1

    &.active.player-a
      background-color: $player-a
    
    &.active.player-b
      background-color: $player-b
</style>
