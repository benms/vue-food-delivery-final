<template>
  <v-container>
    <v-snackbar v-model="snackbar" timeout="2000">
      {{ product.name }} added to basket.

      <template v-slot:action="{ attrs }">
        <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
          <router-link to="/cart">Checkout</router-link>
        </v-btn>
      </template>
    </v-snackbar>
    <v-dialog v-model="isOpen" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">{{ product.name }}</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12" sm="6">
                <v-select
                  :items="quantities"
                  label="Quantity"
                  required
                  v-model="quantity"
                ></v-select>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="removeFromCart(product)">
            {{ isProductInCart ? "Remove" : "Cancel" }}
          </v-btn>
          <v-btn color="blue darken-1" text @click="addToCart()"> Add </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-progress-circular
      indeterminate
      color="primary"
      v-if="loading"
    ></v-progress-circular>
    <v-alert v-else-if="error" dense outlined type="error">{{ error }}</v-alert>
    <v-row e-else dense>
      <v-col v-for="product in products" :key="product._id" cols="4">
        <v-card @click.stop="showModal(product)">
          <v-img :src="product.image"> </v-img>
          <v-card-title v-text="product.name"></v-card-title>
          <v-card-subtitle>{{ product.calorie }} Cal</v-card-subtitle>
          <v-card-text class="text-h5"> ${{ product.price }} </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    loading: true,
    error: "",
    quantity: "1",
    quantities: [...Array(10).keys()].map((x) => `${x + 1}`),
    product: {},
    products: [],
    isOpen: false,
    snackbar: false,
  }),
  computed: {
    isProductInCart() {
      return this.$store.state.Cart.cartItems.find(
        (x) => x.name === this.product.name
      );
    },
  },
  methods: {
    async fetch() {
      try {
        this.loading = true;
        const { data } = await axios.get(`/api/products`);
        this.products = data;
        this.loading = false;
      } catch (err) {
        this.error = err.message;
        this.loading = false;
      }
    },
    showModal(product) {
      this.product = product;
      this.isOpen = true;
    },
    addToCart() {
      this.$store.dispatch("Cart/add", {
        ...this.product,
        quantity: Number(this.quantity),
      });
      this.isOpen = false;
      this.snackbar = true;
    },
    removeFromCart() {
      this.$store.dispatch("Cart/remove", this.product);
      this.isOpen = false;
    },
  },
  mounted() {
    this.fetch();
  },
};
</script>
