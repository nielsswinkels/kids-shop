<template>
  <q-page class="q-pa-md">
    <q-input
      v-model="barcode"
      ref="barcodeInput"
      @keyup.enter="handleBarcodeSubmit"
      label="Skanna streckkod"
      autofocus
    />
    <q-list bordered class="q-mt-md">
      <q-item v-for="(item, index) in scannedItems" :key="index">
        <q-item-section avatar>
          <q-img :src="item.image?.url() || placeholderImage" :alt="item.name" style="width: 50px; height: 50px;" />
        </q-item-section>
        <q-item-section>
          <q-item-label>{{ item.name }}</q-item-label>
          <q-item-label caption>{{ item.price }} SEK</q-item-label>
        </q-item-section>
        <q-item-section side>
          <q-btn icon="delete" @click="removeItem(index)" />
        </q-item-section>
      </q-item>
    </q-list>
    <div class="q-mt-md">
      <q-btn color="primary" @click="checkout">Kassa</q-btn>
    </div>
    <div v-if="showReceipt" class="q-mt-md">
      <q-list bordered>
        <q-item v-for="(item, index) in scannedItems" :key="index">
          <q-item-section>
            <q-item-label>{{ item.name }}</q-item-label>
            <q-item-label caption>{{ item.price }} SEK</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
      <div class="q-mt-md">
        <q-item-label>Total: {{ totalAmount }} SEK</q-item-label>
      </div>
    </div>
  </q-page>
</template>

<script>

import parseUtil from 'src/js/parseUtil';

export default {
  data() {
    return {
      barcode: '',
      scannedItems: [],
      showReceipt: false,
      placeholderImage: 'path/to/placeholder.png',
    };
  },
  computed: {
    totalAmount() {
      return this.scannedItems.reduce((total, item) => total + item.price, 0);
    },
  },
  methods: {
    handleBarcodeSubmit() {
      this.fetchItem(this.barcode);
      this.barcode = '';
      this.$nextTick(() => {
        this.$refs.barcodeInput.focus();
      });
    },
    async fetchItem(barcode) {
      console.log('Going to fetch barcode', barcode)
      const result = await parseUtil.getProductByBarcode(barcode)
      if (result) {
        const item = {
          name: result.get('name'),
          image: result.get('image'),
          price: result.get('price'),
        };
        this.scannedItems.push(item);
      }
    },
    removeItem(index) {
      this.scannedItems.splice(index, 1);
    },
    checkout() {
      this.showReceipt = true;
    },
  },
  watch: {
    showReceipt(newVal) {
      if (newVal) {
        this.$nextTick(() => {
          this.$refs.barcodeInput.focus();
        });
      }
    },
  },
};
</script>

<style scoped>
.q-page {
  max-width: 600px;
  margin: auto;
}
</style>