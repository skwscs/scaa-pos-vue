<template>
  <v-container>
    <v-layout row wrap>
      <v-flex xs12 lg6 class="pa-3">
        <v-card class="v-sheet theme--dark cyan darken-3 pa-5">
          <div class="silver--text subheading">Opening Balance</div>
          <input
            class="silver--text display-1"
            v-model="openingBalance"
            label="Opening Balance"
            style="width:90%;"
          />
          <v-icon dark>create</v-icon>
        </v-card>
      </v-flex>
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
          <template v-slot:top>
            <slot-top-order
              :item="item"
              @onFilter="handleFilter"
              @onRefresh="actionRefresh"
              @onCreate="actionCreate"
              @onCancel="actionCancel"
            />
          </template>

          <template v-slot:item.PostingDate="{ item }">
            <format-date :dateStr="item.PostingDate"></format-date>
          </template>

          <template v-slot:item.PostingCloseDate="{ item }">
            <format-date :dateStr="item.PostingCloseDate"></format-date>
          </template>

          <template v-slot:item.action="{ item }">
            <actions-order :postItem="item" @onUpdate="actionUpdate" @onCancel="actionCancel" />
          </template>
        </v-data-table>
      </v-flex>
    </v-layout>
  </v-container>
</template>
<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import { IOptions, IPosPosting } from "@/types";
import * as api from "@/services";
import _ from "lodash";

@Component({
  components: {
    ActionsOrder: () => import("@/components/datatable/actions-order.vue"),
    SlotTopOrder: () => import("@/components/datatable/slot-top-order.vue"),
    FormatDate: () => import("@/components/datatable/format-date.vue")
  }
})
export default class Orders extends Vue {
  // private items = ["Foo", "Bar", "Fizz", "Buzz"];

  private headers = [
    { text: "Posting Id", value: "PostingId", divider: true },
    { text: "Open Date", value: "PostingDate", divider: true },
    { text: "Opening", value: "Opening", align: "right", divider: true },
    { text: "Posting Amount", value: "PostingAmount", align: "right", divider: true },
    { text: "Closing", value: "Closing", align: "right", divider: true },
    { text: "Close Date", value: "PostingCloseDate", divider: true },
    { text: "Invoice From", value: "InvoiceFrom", divider: true },
    { text: "Invoice To", value: "InvoiceTo", divider: true },
    { text: "Staff Code", value: "StaffCode", divider: true },
    { text: "Posting Flag", align: "right", value: "PostingFlag", divider: true },
    { text: "", align: "left", value: "action", sortable: false }
  ];

  private showDetail = false;
  private selectedItem: any = null;

  // datatable options
  private options: IOptions = { sortBy: ["PostingId"], sortDesc: [true], itemsPerPage: 10 };

  // footer options
  private footerProps = {
    showCurrentPage: true,
    showFirstLastPage: true,
    itemsPerPageOptions: [10, 25, 50, 100],
    itemsPerPageText: "每頁顯示"
  };
  // datatable settings
  private loading = false;
  private items = [];
  private item: any = {};
  private filter = "";
  private itemsPerPage: number = 10;
  private serverItemsLength: number = 0;

  private handleUpdateOptions(opt) {
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
  }

  private openingBalance = 10000;

  private handleFilter(filterText: string) {
    alert("to filter");
  }

  private actionCancel() {
    this.actionRefresh();
    // alert(this.item.CustomerCode);
  }

  private actionRefresh() {
    this.item = Object.assign({}, {} as IPosPosting);
    console.log("options" + JSON.stringify(this.options));
    let sortParams = _.zipWith(this.options.sortBy, this.options.sortDesc, function(by, isDesc) {
      return isDesc ? by + "|desc" : by;
    }).join(",");
    this.fetchData(this.options.page, this.options.itemsPerPage, sortParams, this.filter);
  }

  // CRUD

  private actionCreate(item: IPosPosting) {
    if (_.isEmpty(item)) {
      // confirm("empty");
    } else {
      this.fetchADdData(item);
    }
  }

  private actionUpdate(item: any) {
    this.doEodPosting(item);
  }

  private async fetchData(pageNo, pageSize: number, sort?: string, filter?: string) {
    this.loading = true;
    try {
      // const response = await api.getPostings(pageNo, pageSize, sort, filter);
      const response = await api.getPosPostings(pageNo, pageSize, sort, filter);

      this.items = response.data;
      this.itemsPerPage = response.per_page;
      this.serverItemsLength = response.total;
    } catch (e) {
      console.log("fetchPostings failed..", e);
    } finally {
      this.loading = false;
    }
  }

  private async fetchADdData(obj: any) {
    this.loading = true;
    try {
      let objStaff = { StaffCode: obj.StaffCode };
      const response = await api.addSession(objStaff);
      // alert("Add - response.data.Status");
    } catch (e) {
      console.log("add session failed..", e);
    } finally {
      this.actionRefresh();
      this.loading = false;
    }
  }

  private async doEodPosting(obj: any) {
    this.loading = true;
    try {
      const response = await api.editPosPosting(obj);
      // alert("Edit - response.data.Status");
    } catch (e) {
      console.log("edit Posting failed..", e);
    } finally {
      this.actionRefresh();
      this.loading = false;
    }
  }
}
</script>

<style>
.wrapperUtils {
  display: flex;
  justify-content: flex-end;
}
</style>
