<template>
  <v-flex xs12d class="pa-3">
    <v-data-table
      class="elevation-1 pb-3"
      :footer-props="footerProps"
      :headers="headers"
      :items="items"
      :items-per-page="itemsPerPage"
      :loading="loading"
      :options.sync="options"
      :server-items-length="serverItemsLength"
      @update:options="handleUpdateOptions"
    >
      <!-- Top Slot -->
      <template v-slot:top>
        <v-toolbar text color="grey lighten-5" elevation="0">
          <search-text-field @onFilter="handleFilter"></search-text-field>
          <v-spacer class="hidden-xs-only"></v-spacer>

          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on }">
              <v-btn rounded v-on="on" class="secondary text-capitalize white--text">
                <v-icon color="white">add</v-icon>
                <span class="mx-2"> {{ $t("button-text-add") }} </span>
              </v-btn>
            </template>
            <customer-form
              :item="editedItem"
              :title="formTitle"
              @onSave="handleSave"
              @onCancel="handleCancel"
            ></customer-form>
          </v-dialog>

          <print-button @click="handlePrint"> </print-button>
          <refresh-button @click="handleRefresh"> </refresh-button>
        </v-toolbar>
      </template>

      <!-- Action Slot -->
      <template v-slot:item.Actions="{ item }">
        <edit-button @click="editItem(item)"> </edit-button>
        <delete-dialog
          :item="editedItem"
          :title="$t('dialog-title-delete-customer')"
          @onDelete="handleDelete"
        />
      </template>

      <template v-slot:item.CustomerAddress="{ item }">
        <div>
          <div v-if="item.address1 != '' && item.address1 != null">
            <p>{{ item.address1 }}</p>
            <br />
          </div>
          <div v-if="item.address2 != '' && item.address2 != null">
            <p>{{ item.address2 }}</p>
            <br />
          </div>
          <div v-if="item.address3 != '' && item.address3 != null">
            <p>{{ item.address3 }}</p>
            <br />
          </div>
          <div v-if="item.address4 != '' && item.address4 != null">
            <p>{{ item.address4 }}</p>
            <br />
          </div>
        </div>
      </template>
    </v-data-table>
  </v-flex>
</template>

<script lang="ts">
import { Component, Watch, Vue } from "vue-property-decorator";
import { IOptions, ICustomer1 } from "@/types";
import moment from "moment";
import * as api from "@/services";
import _ from "lodash";

@Component({
  components: {
    CustomerForm: () => import("@/components/input/customer-form.vue"),
    FormatDate: () => import("@/components/datatable/format-date.vue"),
    EditButton: () => import("@/components/button/edit-button.vue"),
    PrintButton: () => import("@/components/button/print-button.vue"),
    RefreshButton: () => import("@/components/button/refresh-button.vue"),
    SearchTextField: () => import("@/components/input/search-text-field.vue"),
    DeleteDialog: () => import("@/components/dialog/delete-dialog.vue")
  }
})
export default class Customer extends Vue {
  get headers() {
    return [
      { text: this.$t("CustomerCode"), width: "8vw", divider: true, value: "CustomerCode" },
      { text: this.$t("CustomerName"), width: "10vw", value: "CustomerName", divider: true },
      { text: this.$t("PhoneNumber"), width: "8vw", value: "PhoneNumber", divider: true },
      { text: this.$t("email"), width: "8vw", value: "email", divider: true },
      { text: this.$t("CustomerAddress"), width: "8vw", value: "CustomerAddress", divider: true },
      { text: this.$t("CustomerCountry"), width: "8vw", value: "country", divider: true },
      { text: "Actions", width: "8vw", align: "center", value: "Actions", sortable: false }
    ];
  }
  // datatable settings
  private loading = false;
  private items = [];
  private item: any = {};
  private filter = "";
  private itemsPerPage: number = 10;
  private serverItemsLength: number = 0;

  // datatable options
  private options: IOptions = { sortBy: ["CustomerCode"], sortDesc: [], itemsPerPage: 10 };

  // footer options
  private footerProps = {
    showCurrentPage: true,
    showFirstLastPage: true,
    itemsPerPageOptions: [10, 25, 50, 100],
    itemsPerPageText: "每頁顯示"
  };

  private filterText = "";
  private handleReset() {}

  private handleSave(item) {
    //confirm(JSON.stringify(item));

    if (this.editedIndex == -1) {
      //  new record
      this.actionCreate(item);
    } else {
      this.actionUpdate(item);
    }
    this.closeDialog();
  }

  private actionCancel() {}
  dialog = false;
  editedIndex = -1;
  editedItem = {} as ICustomer1;

  handleDelete() {}
  get formTitle() {
    return this.editedIndex === -1 ? this.$t("dialog-title-new-customer") : "Edit Item";
  }

  private async fetchData(pageNo, pageSize: number, sort?: string, filter?: string) {
    this.loading = true;
    try {
      const response = await api.getCustomers(pageNo, pageSize, sort, filter);

      this.items = response.data;
      this.itemsPerPage = response.per_page;
      this.serverItemsLength = response.total;
    } catch (e) {
      console.log("fetchDepots failed..", e);
    } finally {
      this.loading = false;
    }
  }

  private handleUpdateOptions = opt => {
    const { page, itemsPerPage, sortBy, sortDesc } = opt;
    this.itemsPerPage = itemsPerPage;

    if (sortBy.length > 0 && this.options.sortBy[0] != sortBy[0]) {
      this.options.sortBy = sortBy;
    }
    if (sortDesc.length > 0 && this.options.sortDesc[0] != sortDesc[0]) {
      this.options.sortDesc = sortDesc;
    } else if (sortBy.length <= 0) {
      this.options.sortDesc = this.options.sortDesc ? [false] : [true];
      this.options.sortBy = sortBy;
      this.options.sortDesc = sortDesc;
      console.log(JSON.stringify(this.options));
    }
    this.actionRefresh();
  };

  private handleCancel() {
    this.closeDialog();
    this.actionRefresh();
  }

  editItem(item) {
    this.editedIndex = this.items.indexOf(item);
    this.editedItem = Object.assign({}, item);
    // confirm(JSON.stringify(this.editedIndex));
    this.dialog = true;
  }

  private closeDialog() {
    this.dialog = false;
    setTimeout(() => {
      this.editedItem = Object.assign({}, {} as ICustomer1);
      this.editedIndex = -1;
    }, 300);
  }

  private actionRefresh() {
    this.item = Object.assign({}, {} as ICustomer1);
    console.log("options" + JSON.stringify(this.options));
    let sortParams = _.zipWith(this.options.sortBy, this.options.sortDesc, function(by, isDesc) {
      return isDesc ? by + "|desc" : by;
    }).join(",");
    this.fetchData(this.options.page, this.options.itemsPerPage, sortParams, this.filter);
  }

  private handleFilter(filterText: string) {
    this.filter = filterText;
    this.actionRefresh();
  }

  private handlePrint(obj: any) {
    this.exportCustomer();
  }

  private handleRefresh() {
    this.actionRefresh();
  }

  // CRUD
  private actionCreate(item: ICustomer1) {
    if (_.isEmpty(item)) {
      // Do something
    } else {
      this.processApiCall(item, api.addCustomer);
    }
  }

  private actionUpdate(item: any) {
    this.processApiCall(item, api.editCustomer);
  }

  private actionDelete(item: any) {
    this.processApiCall(item, api.deleteCustomer);
  }

  private async exportCustomer() {
    this.loading = true;
    try {
      await api.exportPosCustomer().then(response => this.forceFileDownload(response));
    } catch (e) {
      this.loading = false;
      console.log("export customer failed..", e);
    } finally {
      this.loading = false;
    }
  }

  private async processApiCall(obj: any, func: Function) {
    this.loading = true;
    try {
      const response = await func(obj);
    } catch (e) {
      // console.log("add product failed..", e);
    } finally {
      this.actionRefresh();
      this.loading = false;
    }
  }

  private forceFileDownload(response) {
    var blob = new Blob([response], {
      type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
    });
    var link = document.createElement("a");
    link.href = window.URL.createObjectURL(blob);
    link.setAttribute("download", "customer.xlsx"); //or any other extension
    document.body.appendChild(link);
    link.click();
  }
}
</script>
