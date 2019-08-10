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
      <template v-slot:top>
        <slot-top-staff
          :item="item"
          @onFilter="handleFilter"
          @onPrint="handlePrint"
          @onRefresh="actionRefresh"
          @onCreate="actionCreate"
          @onCancel="actionCancel"
        />
      </template>

      <template v-slot:item.StaffAddress="{ item }">
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

      <template v-slot:item.Actions="{ item }">
        <actions-staff
          :item="item"
          @onDelete="actionDelete"
          @onUpdate="actionUpdate"
          @onCancel="actionCancel"
        />
      </template>
    </v-data-table>
  </v-flex>
</template>

<script lang="ts">
import { Component, Watch, Vue } from "vue-property-decorator";
import { IOptions, IStaff } from "@/types";
import moment from "moment";
import * as api from "@/services";
import _ from "lodash";

const name = "staff";

@Component({
  components: {
    ActionsStaff: () => import("@/components/datatable/actions-staff.vue"),
    SlotTopStaff: () => import("@/components/datatable/slot-top-staff.vue"),
    FormatDate: () => import("@/components/datatable/format-date.vue")
  }
})
export default class Staff extends Vue {
  // datatable settings
  private loading = false;
  private items = [];
  private item: any = {};
  private filter = "";
  private itemsPerPage: number = 10;
  private serverItemsLength: number = 0;

  // datatable options
  private options: IOptions = { sortBy: ["StaffCode"], sortDesc: [], itemsPerPage: 10 };

  // footer options
  private footerProps = {
    showCurrentPage: true,
    showFirstLastPage: true,
    itemsPerPageOptions: [10, 25, 50, 100],
    itemsPerPageText: "每頁顯示"
  };

  get headers() {
    return [
      { text: this.$t("StaffCode"), width: "8vw", divider: true, value: "StaffCode" },
      {
        text: this.$t("DepartmentCode"),
        width: "8vw",
        align: "left",
        divider: true,
        value: "DepartmentCode"
      },
      {
        text: this.$t("StaffName"),
        width: "10vw",
        align: "left",
        value: "StaffName",
        divider: true
      },
      {
        text: this.$t("PhoneNumber"),
        width: "8vw",
        align: "left",
        value: "PhoneNumber",
        divider: true
      },
      {
        text: this.$t("email"),
        width: "8vw",
        align: "left",
        value: "email",
        divider: true
      },
      {
        text: this.$t("StaffAddress"),
        width: "8vw",
        align: "left",
        value: "StaffAddress",
        divider: true
      },
      {
        text: this.$t("StaffCountry"),
        width: "8vw",
        align: "center",
        value: "country",
        divider: true
      },
      { text: "Actions", align: "center", value: "Actions", width: "8vw", sortable: false }
    ];
  }

  private async fetchData(pageNo, pageSize: number, sort?: string, filter?: string) {
    this.loading = true;
    try {
      const response = await api.getStaff(pageNo, pageSize, sort, filter);

      this.items = response.data;
      this.itemsPerPage = response.per_page;
      this.serverItemsLength = response.total;
    } catch (e) {
      console.log("fetchStaff failed..", e);
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

  private actionCancel() {
    this.actionRefresh();
  }

  private actionRefresh() {
    this.item = Object.assign({}, {} as IStaff);
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
    this.exportStaff();
  }

  // CRUD

  private actionCreate(item: IStaff) {
    if (_.isEmpty(item)) {
      // confirm("empty");
    } else {
      this.fetchADdData(item);
    }
  }

  private actionUpdate(item: any) {
    this.fetchEditData(item);
  }

  private actionDelete(item: any) {
    this.fetchDeleteData(item);
  }

  private async exportStaff() {
    this.loading = true;
    try {
      const response = await api.exportPosStaff();

      this.forceFileDownload(response);
    } catch (e) {
      this.loading = false;
      console.log("export product failed..", e);
    } finally {
      this.loading = false;
    }
  }

  private async fetchADdData(obj: any) {
    this.loading = true;
    try {
      const response = await api.addStaff(obj);
      // alert("Add - response.data.Status");
    } catch (e) {
      console.log("add staff failed..", e);
    } finally {
      this.actionRefresh();
      this.loading = false;
    }
  }

  private async fetchEditData(obj: any) {
    this.loading = true;
    try {
      const response = await api.editStaff(obj);
      // alert("Edit - response.data.Status");
    } catch (e) {
      console.log("edit staff failed..", e);
    } finally {
      this.actionRefresh();
      this.loading = false;
    }
  }

  private async fetchDeleteData(obj: any) {
    this.loading = true;
    try {
      const response = await api.deleteStaff(obj);

      // alert("Delete - response.data.Status");
    } catch (e) {
      console.log("edit staff failed..", e);
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
    // link.download = "file.xlsx";
    link.setAttribute("download", "staff.xlsx"); //or any other extension
    document.body.appendChild(link);
    link.click();
  }
}

// <style scoped>
// >>> .v-data-table-header th {
//   position: relative;
// }
// >>> .v-data-table-header th.sortable .v-icon {
//   position: absolute;
//   right: 10px;
// }
// </style>
</script>
