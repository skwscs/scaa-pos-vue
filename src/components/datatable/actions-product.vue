<template>
  <div class="text-no-wrap">
    <product-dialog
      isEdit="true"
      :item="item"
      :title="$t('dialog-title-edit-product')"
      @onSave="handleSave"
      @onCancel="handleCancel"
    />
    <delete-dialog
      isEdit="true"
      :item="item"
      :title="$t('dialog-title-delete-product')"
      @onDelete="handleDelete"
    />
  </div>
</template>

<script lang="ts">
import { Component, Vue, Emit, Prop } from "vue-property-decorator";
import { IProduct } from "@/types";

@Component({
  components: {
    DeleteDialog: () => import("@/components/dialog/delete-dialog.vue"),
    ProductDialog: () => import("@/components/dialog/product-dialog.vue")
  }
})
export default class ActionsProduct extends Vue {
  @Prop({ default: {} as IProduct }) item?: IProduct;

  @Emit("onUpdate")
  private handleSave(item: IProduct) {
    return item;
  }

  @Emit("onCancel")
  private handleCancel(item: IProduct) {
    return item;
  }

  @Emit("onDelete")
  private handleDelete(item: IProduct) {
    return item;
  }
}
</script>
