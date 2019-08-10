<template>
  <v-card width="150" @click="onClick($event)">
    <v-img contain class="white--text" :aspect-ratio="16 / 9" :src="imageSource" />

    <v-card-text>
      <h5 class="description">{{ `${description}` }}</h5>
    </v-card-text>

    <v-card-text>
      <span class="pa-1 orange lighten-2">${{ price }}</span>
    </v-card-text>
  </v-card>
</template>

<script lang="ts">
import { Component, Prop, Vue, Emit } from "vue-property-decorator";

@Component({})
export default class ProductCard extends Vue {
  @Prop({ default: "N/A" }) description?: String;
  @Prop({ default: "N/A" }) imageSource?: String;
  @Prop({ default: "N/A" }) price?: String;

  @Emit("onButtonClick")
  onClick(e) {
    return;
  }
}
</script>

<style scoped>
.description {
  line-height: 1.2em;
  max-height: 3.6em;
  height: 1.4em;
}
</style>
