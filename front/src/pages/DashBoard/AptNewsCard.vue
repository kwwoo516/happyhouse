<template>
  <md-card class="md-card-nav-tabs" style="height: auto;">
    <md-card-content>
      <slot name="content"></slot>
    </md-card-content>
  </md-card>
</template>

<script>
export default {
  name: "apt-news-card",
};
</script>
