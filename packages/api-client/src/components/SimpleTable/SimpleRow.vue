<template>
  <tr class="simple-row">
    <slot />
  </tr>
</template>
<style scoped>
.simple-row {
  all: unset;
  display: table-row;
  box-shadow: 0 -0.5px var(--scalar-border-color);
}
.simple-row:first-of-type {
  box-shadow: none;
}
</style>
