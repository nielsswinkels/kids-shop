<template>
  <q-page class="q-pa-md">
    <q-input
      v-model="barcode"
      ref="barcodeInput"
      @keyup.enter="handleBarcodeSubmit"
      label="Skanna streckkod"
      autofocus
    />
    <q-list bordered class="q-mt-md" v-if="!this.showReceipt">
      <q-item v-for="(item, index) in scannedItems" :key="index">
        <q-item-section avatar>
          <q-img :src="item.image?.url() || placeholderImage" :alt="item.name" style="width: 50px; height: 50px;" />
        </q-item-section>
        <q-item-section>
          <q-item-label>{{ item.name }}</q-item-label>
        </q-item-section>
        <q-item-section side >
          <q-item-label class="text-h6">{{ item.price }}</q-item-label>
        </q-item-section>
        <q-item-section side>
          <q-btn icon="delete" @click="removeItem(index)" />
        </q-item-section>
      </q-item>
    </q-list>
    <div class="q-mt-md" v-if="!this.showReceipt">
        <q-item-label class="text-h4">Total: {{ totalAmount }} SEK</q-item-label>
      </div>
    <div class="q-mt-md" v-if="!this.showReceipt">
      <q-btn color="primary" @click="checkout">Betala</q-btn>
    </div>
    <div v-if="showReceipt" class="q-mt-md">
      <q-list bordered>
        <q-item v-for="(item, index) in scannedItems" :key="index">
          <q-item-section>
            <q-item-label>{{ item.name }}</q-item-label>
          </q-item-section>
          <q-item-section side >
            <q-item-label caption>{{ item.price }}</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
      <div class="q-mt-md">
        <q-item-label class="text-h4">Total: {{ totalAmount }} SEK</q-item-label>
      </div>
      <q-list bordered>
        <q-item v-for="(item, index) in paymentcards" :key="index">
          <q-item-section>
            <q-item-label>{{ item.name }}</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
      <div class="q-mt-md">
        <q-btn color="primary" @click="reset">Nästa</q-btn>
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
      paymentcards: [],
      showReceipt: false,
      paymentDone: false,
      placeholderImage: 'path/to/placeholder.png',
      focusInterval: null,
    };
  },
  computed: {
    totalAmount() {
      return this.scannedItems.reduce((total, item) => total + item.price, 0);
    },
  },
  methods: {
    handleBarcodeSubmit() {
      if (!this.showReceipt) {
        this.fetchItem(this.barcode);
      } else {
        this.fetchCard(this.barcode);
      }
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
    async fetchCard(barcode) {
      console.log('Going to fetch barcode', barcode)
      const result = await parseUtil.getPaymentcardByBarcode(barcode)
      if (result) {
        const item = {
          name: result.get('name'),
        };
        this.paymentcards.push(item);
        this.paymentDone = true;
      }
    },
    removeItem(index) {
      this.scannedItems.splice(index, 1);
    },
    checkout() {
      this.showReceipt = true;
      this.$refs.barcodeInput.focus();
    },
    reset() {
      this.barcode = '';
      this.scannedItems = [];
      this.paymentcards = [];
      this.showReceipt = false;
      this.paymentDone = false;
      this.$refs.barcodeInput.focus();
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
  mounted() {
    this.focusInterval = setInterval(() => {
      this.$refs.barcodeInput.focus();
    }, 2000);
  },
  beforeUnmount() {
    clearInterval(this.focusInterval);
  },
};
</script>

<style scoped>
.q-page {
  max-width: 600px;
  margin: auto;
}
</style>