<template>
  <div>
    <product-form @new-product="handleNewProduct"></product-form>
    <input type="text" v-model="searchText" placeholder="Search products..." />
    <div v-for="product in filteredProducts" :key="product.id">
      <p>{{ product.name }}</p>
      <p>{{ product.price }}</p>
      <p>{{ product.locationType }}</p>
      <p>{{ product.animalType }}</p>
    </div>
  </div>
</template>

<script>
import ProductForm from "./ProductForm.vue";
import { ref, computed } from "vue";

let axios;

import("axios").then((module) => {
  axios = module.default;
});

export default {
  components: {
    ProductForm,
  },

  setup() {
    const products = ref([]);
    const searchText = ref("");

    const fetchProducts = async () => {
      if (!axios) {
        await import("axios").then((module) => {
          axios = module.default;
        });
      }

      const response = await axios.get("http://localhost:5000/products");
      products.value = response.data;
    };

    fetchProducts();

    const handleNewProduct = async (product) => {
      if (!axios) {
        await import("axios").then((module) => {
          axios = module.default;
        });
      }

      const response = await axios.post(
        "http://localhost:5000/products",
        product
      );
      products.value.push(response.data);
    };

    const filteredProducts = computed(() => {
      return products.value.filter((product) => {
        return Object.values(product)
          .join(" ")
          .toLowerCase()
          .includes(searchText.value.toLowerCase());
      });
    });

    return { searchText, handleNewProduct, filteredProducts };
  },
};
</script>
