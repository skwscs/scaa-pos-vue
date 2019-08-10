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
        <slot-top-product
          :item="item"
          @onFilter="handleFilter"
          @onPrint="handlePrint"
          @onRefresh="actionRefresh"
          @onCreate="actionCreate"
          @onCancel="actionCancel"
        />
      </template>

      <template v-slot:item.ImageId="{ item }">
        <product-image
          :imageId="item.ImageId"
          :item="item"
          :on-save="handleSaveImage"
          :version="item.version"
        ></product-image>
      </template>

      <template v-slot:item.DateTimeModified="{ item }">
        <format-date :dateStr="item.DateTimeModified"></format-date>
      </template>

      <template v-slot:item.Actions="{ item }">
        <actions-product
          :item="item"
          @onCancel="actionCancel"
          @onDelete="actionDelete"
          @onUpdate="actionUpdate"
        />
      </template>
    </v-data-table>
  </v-flex>
</template>

<script lang="ts">
import { Component, Watch, Vue } from "vue-property-decorator";
import { BASE_URL } from "@/utils/const";
import { IOptions, IProduct } from "@/types";
import moment from "moment";
import * as api from "@/services";
import _ from "lodash";

@Component({
  components: {
    ActionsProduct: () => import("@/components/datatable/actions-product.vue"),
    SlotTopProduct: () => import("@/components/datatable/slot-top-product.vue"),
    FormatDate: () => import("@/components/datatable/format-date.vue"),
    ProductImage: () => import("@/components/dialog/product-image-dialog.vue")
  }
})
export default class Products extends Vue {
  // datatable settings
  private loading = false;
  private items = [];
  private item: any = {};
  private filter = "";
  private itemsPerPage: number = 10;
  private serverItemsLength: number = 0;

  // datatable options
  private options: IOptions = { sortBy: ["ProductCode"], sortDesc: [], itemsPerPage: 10 };

  // footer options
  private footerProps = {
    showCurrentPage: true,
    showFirstLastPage: true,
    itemsPerPageOptions: [10, 25, 50, 100],
    itemsPerPageText: "每頁顯示"
  };

  get headers() {
    return [
      { text: this.$t("ProductCode"), width: "10vw", value: "ProductCode" },
      { text: "", width: "0px", divider: true, value: "ImageId", sortable: false },
      { text: this.$t("ProductName"), divider: true, value: "ProductName" },
      { text: this.$t("ProductCategory"), width: "10vw", divider: true, value: "ProductCategory" },
      {
        text: this.$t("ProductUnit"),
        width: "8vw",
        align: "right",
        divider: true,
        value: "ProductUnit"
      },
      {
        text: this.$t("UnitPrice"),
        width: "8vw",
        divider: true,
        align: "right",
        value: "UnitPrice"
      },
      { text: "Modified", value: "DateTimeModified", width: "8vw", divider: true },
      { text: "Actions", align: "center", value: "Actions", width: "8vw", sortable: false }
    ];
  }

  get lazyUrl() {
    return require("@/assets/img/empty1.png");
  }

  get ver() {
    return this.version;
  }

  version = 0;
  thumbnail(imageId) {
    let ver = this.version;
    return imageId == 0
      ? require("@/assets/img/empty1.png")
      : BASE_URL + `/api/V1/GetPosProductImage?imageId=` + imageId + "&ver=" + ver;
  }

  // private getImageUrl(id) {
  //   return BASE_URL + `/api/V1/GetPropertyAgencyImage?imageId=` + id;
  // }

  private async fetchData(pageNo, pageSize: number, sort?: string, filter?: string) {
    this.loading = true;
    try {
      const response = await api.getPosProducts(pageNo, pageSize, sort, filter);

      this.items = response.data;
      // let index = 0;
      this.items.forEach(item => {
        item["version"] = 0;
      });

      this.itemsPerPage = response.per_page;
      this.serverItemsLength = response.total;
    } catch (e) {
      console.log("fetchProduct failed..", e);
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

  private actionRefresh() {
    // this.incrementVersion();
    this.version = this.version + 1;
    console.log("version" + this.ver);
    this.item = Object.assign({}, {} as IProduct);
    console.log("options" + JSON.stringify(this.options));
    let sortParams = _.zipWith(this.options.sortBy, this.options.sortDesc, function(by, isDesc) {
      return isDesc ? by + "|desc" : by;
    }).join(",");
    this.fetchData(this.options.page, this.options.itemsPerPage, sortParams, this.filter);
  }

  private incrementVersion() {
    this.version = (this.version + 1) % 100;
  }

  private async actionRemoveImage(item: any) {}

  private handleFilter(filterText: string) {
    this.filter = filterText;
    this.actionRefresh();
  }

  private handlePrint(obj: any) {
    this.exportProduct();
  }

  private actionCancel() {
    this.actionRefresh();
  }

  private handleSaveImage(productCode: string, file: any) {
    this.actionSetImage(productCode, file);
  }

  private async actionSetImage(productCode, file) {
    let formData = new FormData();
    formData.append("ProductCode", productCode);
    formData.append("FormFile", file);

    this.loading = true;
    try {
      const response = await api.uploadProductImage(formData);
      console.log(JSON.stringify(response));
      this.items.forEach(item => {
        if(item.ProductCode == productCode){
          item.version += 1;
          item.ImageId = response.Id;
        }
      });
    } catch (e) {
      console.log("set product image failed..", e);
    } finally {
      // this.actionRefresh();
      this.loading = false;
    }
  }

  // CRUD
  private actionCreate(item: IProduct) {
    if (_.isEmpty(item)) {
      confirm("empty");
    } else {
      this.processApiCall(item, api.addProducts);
    }
  }

  private actionUpdate(item: any) {
    this.processApiCall(item, api.editProducts);
  }

  private actionDelete(item: any) {
    this.processApiCall(item, api.deleteProducts);
  }

  private async exportProduct() {
    this.loading = true;
    try {
      await api.exportPosProduct().then(response => this.forceFileDownload(response));
    } catch (e) {
      this.loading = false;
      console.log("export product failed..", e);
    } finally {
      this.loading = false;
    }
  }

  private async processApiCall(obj: any, func: Function) {
    this.loading = true;
    try {
      const response = await func(obj);
    } catch (e) {
      console.log("add product failed..", e);
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
    link.setAttribute("download", "product.xlsx"); //or any other extension
    document.body.appendChild(link);
    link.click();
  }
}
</script>
