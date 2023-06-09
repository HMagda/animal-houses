<template>
  <div>
    <div class="options-container">
      <button @click="showForm = !showForm">Add Product</button>
      <product-form
        v-if="showForm"
        @new-product="handleNewProduct"
        @close-form="closeForm"
      ></product-form>
      <div class="search-container">
        Search:
        <input
          type="text"
          v-model="searchText"
          placeholder="Search products..."
        />
        <button @click="clearFilters">Clear filters</button>
      </div>
    </div>

    <table class="product-table">
      <thead>
        <tr>
          <th @click="sortProducts('name')">
            House Name
            <font-awesome-icon
              :icon="
                sortKey === 'name'
                  ? sortOrder === 1
                    ? 'sort-up'
                    : 'sort-down'
                  : 'sort'
              "
            />
          </th>
          <th @click="sortProducts('price')">
            Price
            <font-awesome-icon
              :icon="
                sortKey === 'price'
                  ? sortOrder === 1
                    ? 'sort-up'
                    : 'sort-down'
                  : 'sort'
              "
            />
          </th>
          <th @click="sortProducts('locationType')">
            Location
            <font-awesome-icon
              :icon="
                sortKey === 'locationType'
                  ? sortOrder === 1
                    ? 'sort-up'
                    : 'sort-down'
                  : 'sort'
              "
            />
          </th>
          <th @click="sortProducts('animalType')">
            Animal
            <font-awesome-icon
              :icon="
                sortKey === 'animalType'
                  ? sortOrder === 1
                    ? 'sort-up'
                    : 'sort-down'
                  : 'sort'
              "
            />
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in filteredProducts" :key="product.id">
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td class="location-value">
            <span
              class="clickable table-content"
              @click="filterBy('location', product.locationType)"
              >{{ product.locationType }}</span
            >
          </td>
          <td class="animal-value">
            <span
              class="clickable table-content"
              @click="filterBy('animal', product.animalType)"
              >{{ product.animalType }}</span
            >
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
    const showForm = ref(false);

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
          let aValue = a[sortParams.sortKey];
          let bValue = b[sortParams.sortKey];

          if (sortParams.sortKey === "price") {
            aValue = parseFloat(aValue);
            bValue = parseFloat(bValue);
          } else {
            aValue = aValue.toLowerCase();
            bValue = bValue.toLowerCase();
          }

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

    const clearFilters = () => {
      searchText.value = "";
    };

    const closeForm = () => {
      showForm.value = false;
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
      clearFilters,
      closeForm,
      showForm,
    };
  },
};
</script>
