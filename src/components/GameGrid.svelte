<script lang="ts" context="module">
  export type Cell = null | string;

  const size: [number, number] = [5, 5];
</script>

<script lang="ts">
  let cells: Cell[][] = Array.from({ length: size[0] }, () => Array.from({ length: size[1] }, () => 'hallo'));
</script>

<template lang="pug">
  .


  //- .rows
  //-   +each('cells as x')
  //-     .columns
  //-       +each('x as cell')
  //-         .cell {cell}
</template>

<style lang="sass">
  .rows
    display: flex
    flex-direction: column
  
  .columns
    display: flex
    flex-direction: row

  .cell
    flex: 1
    aspect-ratio: 1/1
</style>
