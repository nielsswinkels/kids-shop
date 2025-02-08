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
          <q-img :src="item.image?.url() || placeholderImage" :alt="item.name" style="width: 100px; height: 100px;" />
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
    <div class="q-mt-md row justify-between" v-if="!this.showReceipt">
      <div class="q-mt-md col row justify-begin">
        <q-btn color="primary" @click="checkout">Betala</q-btn> 
      <q-img src="QR-BETALA.png" style="width: 100px; height: 100px;" />
      </div>
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
      <div class="q-mt-md row justify-between">
        <div class="q-mt-md col row justify-begin">
          <q-btn  @click="backToShopping">Förregående</q-btn>
          <q-img src="QR-FÖRREGÅENDE.png" style="width: 100px; height: 100px;" />
        </div>
        <div class="q-mt-md col row justify-end">
          <q-btn color="primary" @click="reset">Nästa</q-btn>
          <q-img src="QR-NASTA.png" style="width: 100px; height: 100px;" />
        </div>
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
        if (this.barcode === 'BETALA') {
          this.checkout();
        } else {
          this.fetchItem(this.barcode);
        }
      } else {
        if (this.barcode === 'NÄSTA') {
          this.reset();
        } else if (this.barcode === 'FÖRREGÅENDE') {
          this.backToShopping();
        } else {
          this.fetchCard(this.barcode);
        }
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
    backToShopping() {
      this.barcode = '';
      this.showReceipt = false;
      this.paymentDone = false;
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