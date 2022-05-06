<script lang="ts">
  import type { TopAppBarComponentDev } from '@smui/top-app-bar';
  import TopAppBar, { Row, Section, Title, AutoAdjust } from '@smui/top-app-bar';

  // import GameGrid from './components/GameGrid.svelte';
  import GridLines from './components/GridLines.svelte';

  let topAppBar: TopAppBarComponentDev;
</script>

<template lang="pug">
  TopAppBar(bind:this='{topAppBar}' variant='fixed' dense)
    Row
      Section
        Title Gridd
      //- Section(align='end' toolbar)
        //- iconbutton.material-icons(aria-label='Bookmark this page') bookmark
  AutoAdjust(topAppBar='{topAppBar}')
    GridLines(size='{[5,5]}')
</template>
