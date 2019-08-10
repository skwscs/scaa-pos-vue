<template>
  <div>
    <staff-dialog
      isEdit="true"
      :item="item"
      :title="$t('dialog-title-edit-staff')"
      @onSave="handleSave"
      @onCancel="handleCancel"
    />
    <delete-dialog
      isEdit="true"
      :item="item"
      :title="$t('dialog-title-delete-staff')"
      @onDelete="handleDelete"
    />
  </div>
</template>

<script lang="ts">
import { Component, Vue, Emit, Prop } from "vue-property-decorator";
import { IStaff } from "@/types";

@Component({
  components: {
    DeleteDialog: () => import("@/components/dialog/delete-dialog.vue"),
    StaffDialog: () => import("@/components/dialog/staff-dialog.vue")
  }
})
export default class ActionsStaff extends Vue {
  @Prop({ default: {} as IStaff }) item?: IStaff;

  @Emit("onDelete")
  private handleDelete(item: IStaff) {
    return item;
  }

  @Emit("onCancel")
  private handleCancel(item: IStaff) {
    return item;
  }

  @Emit("onUpdate")
  private handleSave(item: IStaff) {
    return item;
  }
}
</script>
