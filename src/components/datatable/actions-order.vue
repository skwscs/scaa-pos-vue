<template>
  <div>
    <invoice-dialog :postItem="postItem" />
    <eod-closing-dialog
      v-if="postItem.PostingFlag == 0"
      isEdit="true"
      :item="postItem"
      @onSave="handleSave"
      @onCancel="handleCancel"
    />
  </div>
</template>

<script lang="ts">
import { Component, Vue, Emit, Prop } from "vue-property-decorator";
import { IPosPosting } from "@/types";

@Component({
  components: {
    InvoiceDialog: () => import("@/components/dialog/invoice-dialog.vue"),
    EodClosingDialog: () => import("@/components/dialog/eod-closing-dialog.vue"),
    NewPostingDialog: () => import("@/components/dialog/new-posting-dialog.vue")
  }
})
export default class ActionsOrder extends Vue {
  private isEdit = true;
  @Prop({ default: {} as IPosPosting }) postItem?: IPosPosting;

  @Emit("onUpdate")
  private handleSave(item: IPosPosting) {
    return item;
  }

  @Emit("onCancel")
  private handleCancel(item: IPosPosting) {
    return item;
  }
}
</script>
