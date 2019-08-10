<template>
  <v-toolbar text color="grey lighten-5" elevation="0">
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

    <staff-dialog
      :item="item"
      :title="$t('dialog-title-new-staff')"
      @onSave="handleSave"
      @onCancel="handleCancel"
    />

    <v-btn text icon @click="handlePrint"> <v-icon>print</v-icon> </v-btn>
    <v-btn text icon @click="handleRefresh"> <v-icon>refresh</v-icon> </v-btn>
  </v-toolbar>
</template>

<script lang="ts">
import { Component, Vue, Emit, Prop } from "vue-property-decorator";
import { IStaff } from "@/types";

@Component({
  components: {
    StaffDialog: () => import("@/components/dialog/staff-dialog.vue")
  }
})
export default class SlotTopStaff extends Vue {
  private filterText: string = "";

  @Prop({ default: {} as IStaff }) item?: IStaff;

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
  private handleSave(item: IStaff) {
    return item;
  }

  @Emit("onCancel")
  private handleCancel(item: IStaff) {
    return item;
  }

  @Emit("onFilter")
  handleReset() {
    this.filterText = ""; // clear the text in text input
    return this.filterText;
  }
}
</script>
