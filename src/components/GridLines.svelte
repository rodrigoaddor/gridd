<script lang="ts" context="module">
  export type LineCallback = (lineIndex: number) => void;
  export type SquareData = boolean;

  enum ElementType {
    DOT = 'dot',
    HORIZONTAL_LINE = 'horizontal-line',
    VERTICAL_LINE = 'vertical-line',
    SQUARE = 'square',
  }
</script>

<script lang="ts">
  export let size: [number, number];

  export let lines: boolean[];
  export let squares: boolean[];

  let elements: { type: ElementType; active: boolean; handler?: () => void }[];

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

    elements = Array.from({ length: (size[0] * 2 + 1) * (size[1] * 2 + 1) }, (_, i) => {
      const col = i % (size[0] * 2 + 1);
      const line = Math.floor(i / (size[0] * 2 + 1));

      const type = getElementType(col, line);
      let active = false;
      let handler: (() => void) | undefined;

      if (type === ElementType.VERTICAL_LINE || type === ElementType.HORIZONTAL_LINE) {
        let thisLineIndex = lineIndex++;
        active = lines[thisLineIndex];
        lines[thisLineIndex] ??= false;
        handler = () => {
          // set clicked line to active
          lines[thisLineIndex] = true;

          // verify if a square can be activated
          getRelatedSquares(thisLineIndex).forEach(({ square, lines: relatedLines }) => {
            if (relatedLines.every((line) => lines[line])) {
              squares[square] = true;
            }
          });

          // getRelatedSquares(thisLineIndex).forEach(({ square, lines: relatedLines }) => {
          //   squares[square] = true;
          //   relatedLines.forEach((line) => (lines[line] = true));
          // });

          // trigger updates if needed
          squares = squares;
          lines = lines;
        };
      } else if (type === ElementType.SQUARE) {
        let thisSquareIndex = squareIndex++;
        active = squares[thisSquareIndex];
        squares[thisSquareIndex] ??= false;
      }

      return { type, active, handler };
    });
  }

  /*-- Reset grid on size change --*/

  let oldSize = size;

  $: {
    if (size.join('x') !== oldSize.join('x')) {
      lines = Array.from({ length: lines.length }, () => false);
      squares = Array.from({ length: squares.length }, () => false);
      oldSize = size;
    }
  }
</script>

<template>
  <div class="wrapper" style:--rows={size[1]} style:--columns={size[0]}>
    {#each elements as item, index}
      {#if item.type === ElementType.DOT}
        <div class="dot" />
      {:else if item.type === ElementType.HORIZONTAL_LINE}
        <div class="line horizontal hoverable" class:active={item.active} on:click={item.handler} />
      {:else if item.type === ElementType.VERTICAL_LINE}
        <div class="line vertical hoverable" class:active={item.active} on:click={item.handler} />
      {:else}
        <div class="square" class:active={item.active} />
      {/if}
    {/each}
  </div>
</template>

<style lang="sass">
  @use '@material/theme/color-palette' as palette

  :root
    --thickness: 4px
    --grid-color: white

  .wrapper
    display: grid
    grid-template-columns: repeat(var(--columns), 1rem 4rem) 1rem
    grid-template-rows: repeat(var(--rows), 1rem 4rem) 1rem
    margin: 2rem
    place-items: center

  .dot
    height: var(--thickness)
    width: var(--thickness)
    background-color: var(--grid-color)
    border-radius: 50%

  .line
    &::after
      display: block
      border-radius: var(--thickness)
      background-color: var(--grid-color)
      content: ' '

    &.horizontal, &.vertical
      display: grid
      place-items: center
      height: 100%
      width: 100%

    &.horizontal::after
      height: var(--thickness)
      width: 100%
    
    &.vertical::after
      width: var(--thickness)
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
    min-width: 4rem
    aspect-ratio: 1/1

    &.active
      background-color: transparentize(palette.$light-blue-500, 0.5)
</style>
