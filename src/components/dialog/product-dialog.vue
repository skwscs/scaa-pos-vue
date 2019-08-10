<template>
  <v-dialog v-model="showDialog" width="500">
    <template v-slot:activator="{ on }">
      <template v-if="isEdit">
        <v-btn slot="activator" v-on="on" text icon color="primary">
          <v-icon>edit</v-icon>
        </v-btn>
      </template>
      <template v-else>
        <v-btn
          depressed
          slot="activator"
          v-on="on"
          class="secondary text-capitalize white--text"
          rounded
          text
        >
          <v-icon color="white" class="mr-2">add</v-icon>
          <span class="mr-2"> {{ $t("button-text-add") }} </span>
        </v-btn>
      </template>
    </template>

    <v-form v-model="validInput">
      <v-card>
        <v-card-title primary-title>{{ title }}</v-card-title>
        <v-divider></v-divider>
        <v-container grid-list-md>
          <v-layout wrap>
            <v-flex xs12>
              <v-text-field
                v-model="item.ProductCode"
                v-mask="maskProductCode"
                :disabled="isEdit"
                :label="$t('label-product-code')"
                :rules="[rules.required]"
                onkeyup="this.value = this.value.toUpperCase();"
              >
              </v-text-field>
            </v-flex>
            <v-flex xs12>
              <v-text-field
                v-model="item.ProductName"
                :label="$t('label-product-name')"
                :rules="[rules.required]"
              />
            </v-flex>
            <v-flex xs12>
              <v-text-field
                v-model="item.ProductUnit"
                :label="$t('label-unit')"
                :rules="[rules.required]"
              />
            </v-flex>
            <!-- v-money="money" -->
            <v-flex xs12>
              <v-text-field
                type="number"
                v-model.lazy="item.UnitPrice"
                :label="$t('label-unit-price')"
                :rules="[rules.required]"
              />
              <!-- <money v-model="item.UnitPrice" v-bind="money"></money> -->
            </v-flex>
          </v-layout>
        </v-container>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="handleCancel">{{ $t("button-text-cancel") }}</v-btn>
          <v-btn color="primary" text @click="handleSave" :disabled="validInput == false">{{
            $t("button-text-confirm")
          }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-form>
  </v-dialog>
</template>

<script lang="ts">
import { Component, Vue, Emit, Prop } from "vue-property-decorator";
import { IProduct } from "@/types";
import { MASK_16A } from "@/utils/const";
import { TheMask } from "vue-the-mask";
// import { VMoney } from "v-money";

@Component({
  // directives: { money: VMoney },
  components: { TheMask }
})
export default class ProductDialog extends Vue {
  private showDialog = false;

  private rules = { required: value => !!value || "Required" };
  private validInput: boolean = false;
  private maskProductCode = MASK_16A;
  private money = {
    masked: false /* doesn't work with directive */
  };

  @Prop({ default: {} as IProduct }) item?: IProduct;
  @Prop({ default: false }) isEdit?: boolean;
  @Prop({ default: "" }) title?: string;

  @Emit("onSave")
  handleSave() {
    this.showDialog = false;
    return this.item;
  }

  @Emit("onCancel")
  handleCancel() {
    this.showDialog = false;
    return this.item;
  }

  @Emit("update:item")
  private closeDialog() {
    return {} as IProduct;
  }

  reset() {
    this.item = {} as IProduct;
  }
}
</script>
<style scoped>
.capitalize-text input {
  text-transform: uppercase !important;
}
input[type="number"] {
  -moz-appearance: textfield;
}
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
}
</style>
