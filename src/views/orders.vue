<template>
  <v-container id="dashboard" fluid grid-list-lg class="mx-0 pa-3">
    <v-layout row wrap>
      <!-- Summary Section -->
      <v-flex lg4>
        <v-layout row wrap>
          <v-flex xs12>
            <fiscal-year-balance-card
              :fiscalBalance="fiscalBalance"
              @onSave="actionSave"
            ></fiscal-year-balance-card>
          </v-flex>
        </v-layout>
      </v-flex>
      <!-- Datatable section -->
      <v-flex lg8>
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
              @onPrint="handlePrint"
              @onRefresh="actionRefresh"
            />
          </template>

          <template v-slot:item.PostingDate="{ item }">
            <format-date :dateStr="item.PostingDate" format="YYYY-MM-DD"></format-date>
          </template>

          <template v-slot:item.PostingCloseDate="{ item }">
            <format-date :dateStr="item.PostingCloseDate" format="YYYY-MM-DD"></format-date>
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
import { IOptions, IPosPosting, IFiscalBalance } from "@/types";
import * as api from "@/services";
import _ from "lodash";

@Component({
  components: {
    ActionsOrder: () => import("@/components/datatable/actions-order.vue"),
    SlotTopOrder: () => import("@/components/datatable/slot-top-order.vue"),
    FormatDate: () => import("@/components/datatable/format-date.vue"),
    FiscalYearBalanceCard: () => import("@/components/card/fiscal-year-balance-card.vue")
  }
})
export default class Orders extends Vue {
  private title = "FY2019";
  private data = 100000;
  private btnText = "Set Open Balance";
  private amount = 789;

  private fiscalBalance: IFiscalBalance = {} as IFiscalBalance;

  private headers = [
    { text: "Posting Id", value: "PostingId", width: "6vw", divider: true },
    { text: "Open Date", value: "PostingDate", divider: true },
    { text: "Opening", value: "Opening", align: "right", divider: true },
    { text: "Posting", value: "PostingAmount", align: "right", divider: true },
    { text: "Closing", value: "Closing", align: "right", divider: true },
    { text: "Close Date", value: "PostingCloseDate", divider: true },
    { text: "Inv. From", value: "InvoiceFrom", divider: true, align: "right" },
    { text: "Inv. To", value: "InvoiceTo", divider: true, align: "right" },
    { text: "Staff Code", value: "StaffCode", divider: true },
    { text: "Status", align: "right", value: "PostingFlag", width: "5vw", divider: true },
    { text: "", align: "left", value: "action", sortable: false }
  ];

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

  get fiscalYearOpenBalance() {
    return this.$n(this.fiscalBalance.Opening);
  }
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

  private handleFilter(filterText: string) {
    this.filter = filterText;
    this.actionRefresh();
  }

  private handlePrint(obj: any) {
    this.exportPosPosting();
  }

  private async exportPosPosting() {
    this.loading = true;
    try {
      const response = await api.exportPosPosting();

      this.forceFileDownload(response);
    } catch (e) {
      this.loading = false;
      console.log("export posting failed..", e);
    } finally {
      this.loading = false;
    }
  }

  private actionCancel() {
    this.actionRefresh();
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
  private actionSave(result: any) {
    // confirm(result.Opening.toString());
    this.setFiscalBalance(result);
  }

  private async setFiscalBalance(obj: any) {
    this.loading = true;
    try {
      let objFiscalBalance = {
        StaffCode: obj.StaffCode,
        OpeningDate: obj.OpeningDate,
        Opening: obj.Opening
      };
      const response = await api.setFiscalYearBalance(objFiscalBalance);
    } catch (e) {
      this.fetchFiscalBalance();
      this.actionRefresh();
      // console.log("add session failed..", e);
    } finally {
      this.fetchFiscalBalance();
      this.actionRefresh();
      this.loading = false;
    }
  }

  private actionCreate(item: IPosPosting) {
    if (_.isEmpty(item) == false) {
      this.fetchADdData(item);
    }
  }

  private actionPrint(item: any) {}

  private actionUpdate(item: any) {
    this.doEodPosting(item);
  }

  private async fetchData(pageNo, pageSize: number, sort?: string, filter?: string) {
    this.loading = true;
    try {
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

  private async fetchFiscalBalance() {
    this.loading = true;

    try {
      const response = await api.getFiscalBalance();

      this.fiscalBalance = response.FiscalBalance;
    } catch (e) {
      console.log("fetchInvoice outstanding amount failed..", e);
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

  private forceFileDownload(response) {
    var blob = new Blob([response], {
      type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
    });
    var link = document.createElement("a");
    link.href = window.URL.createObjectURL(blob);
    link.setAttribute("download", "posting.xlsx"); //or any other extension
    document.body.appendChild(link);
    link.click();
  }

  created() {
    this.fetchFiscalBalance();
  }
}
</script>

<style>
.wrapperUtils {
  display: flex;
  justify-content: flex-end;
}
</style>
