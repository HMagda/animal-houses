<template>
  <div>
    <product-form @new-product="handleNewProduct"></product-form>
    <input type="text" v-model="searchText" placeholder="Search products..." />
    <table class="product-table">
      <thead>
        <tr>
          <th @click="sortProducts('name')">
            House Name
            <font-awesome-icon icon="sort" />
          </th>
          <th @click="sortProducts('price')">
            Price
            <font-awesome-icon icon="sort" />
          </th>
          <th @click="sortProducts('locationType')">
            Location
            <font-awesome-icon icon="sort" />
          </th>
          <th @click="sortProducts('animalType')">
            Animal
            <font-awesome-icon icon="sort" />
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in filteredProducts" :key="product.id">
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td @click="filterBy('location', product.locationType)">
            {{ product.locationType }}
          </td>
          <td @click="filterBy('animal', product.animalType)">
            {{ product.animalType }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { library } from "@fortawesome/fontawesome-svg-core";
import {
  faArrowUp,
  faArrowDown,
  faSort,
} from "@fortawesome/free-solid-svg-icons";

library.add(faArrowUp, faArrowDown, faSort);

import { ref, provide, reactive, computed, toRefs } from "vue";
import axios from "axios";

import ProductForm from "../ProductForm/ProductForm.vue";

import "./ProductList.scss";

library.add(faArrowUp, faArrowDown, faSort);

export default {
  components: {
    ProductForm,
    FontAwesomeIcon,
  },
  setup() {
    const products = ref([]);
    provide("products", products);

    const searchText = ref("");

    const sortParams = reactive({
      sortKey: "",
      sortOrder: 1,
    });

    const fetchProducts = async () => {
      const response = await axios.get("http://localhost:5000/products");
      products.value = response.data;
    };

    fetchProducts();

    const handleNewProduct = async (product) => {
      const response = await axios.post(
        "http://localhost:5000/products",
        product
      );
      products.value.push(response.data);
    };

    const showSortDropdown = ref(false);
    const toggleSortDropdown = () => {
      showSortDropdown.value = !showSortDropdown.value;
    };

    const filteredProducts = computed(() => {
      let sortedProducts = [...products.value];

      if (sortParams.sortKey) {
        sortedProducts.sort((a, b) => {
          const aValue = a[sortParams.sortKey];
          const bValue = b[sortParams.sortKey];

          if (aValue < bValue) return -sortParams.sortOrder;
          if (aValue > bValue) return sortParams.sortOrder;
          return 0;
        });
      }

      return sortedProducts.filter((product) => {
        return Object.values(product)
          .join(" ")
          .toLowerCase()
          .includes(searchText.value.toLowerCase());
      });
    });

    const sortProducts = (key) => {
      if (sortParams.sortKey === key) {
        sortParams.sortOrder *= -1;
      } else {
        sortParams.sortKey = key;
        sortParams.sortOrder = 1;
      }
    };

    const filterBy = (type, value) => {
      if (type === "animal") {
        searchText.value = value;
      } else if (type === "location") {
        searchText.value = value;
      }
    };

    return {
      ...toRefs(sortParams),
      searchText,
      handleNewProduct,
      filteredProducts,
      sortProducts,
      showSortDropdown,
      toggleSortDropdown,
      filterBy,
    };
  },
};
</script>
