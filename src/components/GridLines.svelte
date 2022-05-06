<script lang="ts">
  export let size: [number, number];

  const hLines: boolean[][] = Array.from({ length: size[1] + 1 }, () => Array.from({ length: size[0] }, () => false));
  const vLines: boolean[][] = Array.from({ length: size[0] + 1 }, () => Array.from({ length: size[1] }, () => false));

  const hDots: boolean[][] = Array.from({ length: size[1] + 1 }, () =>
    Array.from({ length: size[0] + 1 }, () => false)
  );

  const vDots: boolean[][] = Array.from({ length: size[0] + 1 }, () =>
    Array.from({ length: size[1] + 1 }, () => false)
  );

  const toggleLine = (x, line) => {
    return () => console.log({ x, line });
  };
</script>

<template lang="pug">
  .outer-container
    .inner-container
      .hWrapper
        +each('hLines as x')
          .hLines
            +each('x as line')
              .hLine(on:click='{toggleLine(x, line)}')

      .vWrapper
        +each('vLines as y')
          .vLines
            +each('y as line')
              .vLine
      
      .hDotsWrapper
        +each('hDots as x')
          .hDotsLines
            +each('x as dot')
              .hDot
                .dot +
</template>

<style lang="sass">
  :global(#app)
    height: 100vh
    width: 100vw

  .outer-container
    height: 100%
    width: 100%
    padding: 2rem
  
  .inner-container
    height: 100%
    width: 100%
    position: relative

  .hWrapper
    position: absolute
    top: 0
    left: 0
    bottom: 0
    right: 0
    display: flex
    flex-direction: column
    justify-content: space-between
    aspect-ratio: 1/1
    pointer-events: none
  
    .hLines
      display: flex
      flex-direction: row

      .hLine
        flex: 1
        height: 1px
        background-color: white
        margin: 0 10px
        cursor: pointer
        pointer-events: auto
        opacity: 0

        &:hover
          opacity :1
  
  .vWrapper
    position: absolute
    top: 0
    left: 0
    bottom: 0
    right: 0
    display: flex
    flex-direction: row
    justify-content: space-between
    aspect-ratio: 1/1
    pointer-events: none
  
    .vLines
      display: flex
      flex-direction: column

      .vLine
        flex: 1
        width: 1px
        background-color: white
        margin: 10px 0
        pointer-events: auto
        opacity: 0

        &:hover
          opacity: 1

  .dot
    content: '+'
    transform: translateY(-1.5px)
  
  .hDotsWrapper
    position: absolute
    top: 0
    left: 0
    bottom: 0
    right: 0
    display: flex
    flex-direction: column
    justify-content: space-between
    aspect-ratio: 1/1
    pointer-events: none

    .hDotsLines
      display: flex
      flex-direction: row
      justify-content: space-between

      .hDot
        display: flex
        justify-content: center
        align-items: center
        height: 1px
        width: 1px
        background-color: white
        pointer-events: auto
</style>
