<template>
  <v-toolbar text elevation="0">
    <v-text-field
      v-model="filterText"
      :placeholder="$t('placeholder-text-search')"
      append-icon="search"
      clear-icon="clear"
      clearable
      hide-details
      @keyup.enter="handleFilter"
      @click:append="handleFilter"
      @click:clear="handleReset"
    ></v-text-field>

    <v-spacer class="hidden-xs-only"></v-spacer>

    <product-dialog
      :item="item"
      :title="$t('dialog-title-new-product')"
      @onSave="handleSave"
      @onCancel="handleCancel"
    />

    <v-btn text icon @click="handlePrint"> <v-icon>print</v-icon> </v-btn>
    <v-btn text icon @click="handleRefresh"> <v-icon>refresh</v-icon> </v-btn>
  </v-toolbar>
</template>

<script lang="ts">
import { Component, Vue, Emit, Prop } from "vue-property-decorator";
import { IProduct } from "@/types";

@Component({
  components: {
    ProductDialog: () => import("@/components/dialog/product-dialog.vue")
  }
})
export default class SlotTopProduct extends Vue {
  private filterText: string = "";

  @Prop({ default: {} as IProduct }) item?: IProduct;

  @Emit("onFilter")
  handleFilter() {
    return this.filterText;
  }

  @Emit("onPrint")
  private handlePrint() {
    return true;
  }

  @Emit("onRefresh")
  private handleRefresh() {
    return false;
  }

  @Emit("onCreate")
  private handleSave(item: IProduct) {
    return item;
  }

  @Emit("onCancel")
  private handleCancel(item: IProduct) {
    return item;
  }

  @Emit("onFilter")
  handleReset() {
    this.filterText = ""; // clear the text in text input
    return this.filterText;
  }
}
</script>
