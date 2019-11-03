<template>
  <div id="app">
    <layout />
  </div>
</template>

<script>
import Layout from './components/Layout.vue'

export default {
  name: 'app',
  components: {
    Layout
  }
}
</script>

<style lang="scss">
@import "~bootstrap/scss/bootstrap";

// FA styles
.fa-icon {
  display: inline-block;
  font-size: inherit;
  height: 1em;
  overflow: visible;
  vertical-align: -0.125em;
  width: 0.875em;
}

// Gutters to use with d-flex
.gutter, .gutter-y {
  margin-top: -1rem;

  > * {
    margin-top: 1rem;
  }
}
.gutter-x, .gutter {
  margin-left: -1rem;
  > * {
    margin-left: 1rem;
  }
}

.gutter-sm, .gutter-y-sm {
  margin-top: -.5rem;

  > * {
    margin-top: .5rem;
  }
}
.gutter-x-sm, .gutter-sm {
  margin-left: -.5rem;
  > * {
    margin-left: .5rem;
  }
}

.gutter-xs, .gutter-y-xs {
  margin-top: -.25rem;

  > * {
    margin-top: .25rem;
  }
}
.gutter-x-xs, .gutter-xs {
  margin-left: -.25rem;
  > * {
    margin-left: .25rem;
  }
}
</style>