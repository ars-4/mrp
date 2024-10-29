<template>
  <div class="home">

    <SaleComponent v-on:processed_sales="process_sales" />

    <BillOfMaterialsComponent v-on:processed_bom="process_bom" v-if="sales.length > 0" />

    <ConsumptionComponent v-if="bom.length > 0 && sales.length > 0" :products="bom" :sales="sales"
      v-on:processed_consumption="process_consumption" />

    <POGenerator v-if="consumption_processed" :consumption="consumption" />

  </div>
</template>

<style>
.home {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
}

.home>div {
  width: 600px;
}
</style>

<script>
import SaleComponent from '@/components/SalePlan.vue';
import BillOfMaterialsComponent from '@/components/BillOfMaterials.vue';
import ConsumptionComponent from '@/components/ConsumptionComponent.vue';
import POGenerator from '@/components/POGenerator.vue';

export default {

  name: 'HomeView',

  data() {
    return {
      sales: [],
      bom: [],
      consumption : [],
      consumption_processed : false
    }
  },

  components: {
    SaleComponent,
    BillOfMaterialsComponent,
    ConsumptionComponent,
    POGenerator
  },

  methods: {

    process_sales: function (sales) {
      this.sales = sales
      // console.log(this.sales.reduce((a, b) => a + b.sale, 0))
    },

    process_bom: function (bom) {
      this.bom = bom
      // console.log(bom)
    },

    process_consumption: function (consumption) {
      this.consumption = consumption
      this.consumption_processed = true
      console.log("Consumption processed")
    }

  }

}
</script>