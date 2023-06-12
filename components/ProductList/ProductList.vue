<template>
  <div>
    <div class="options-container">
      <button @click="showForm = !showForm">Add Product</button>
      <product-form
        v-if="showForm"
        @new-product="handleNewProduct"
        @close-form="closeForm"
      ></product-form>
      <div class="currency-container">
        Currency:
        <select v-model="selectedCurrency">
          <option disabled value="">Please select a currency</option>
          <option
            v-for="(rate, currency) in currencies"
            :key="currency"
            :value="currency"
          >
            {{ currency }}
          </option>
        </select>
      </div>
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
          <td>{{ convertPrice(product.price) }}</td>
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
import { ref, provide, reactive, computed, toRefs, watch } from "vue";
import axios from "axios";
import ProductForm from "../ProductForm/ProductForm.vue";
import "./ProductList.scss";

library.add(faArrowUp, faArrowDown, faSort);

const BASE_API_URL = "http://localhost:5000";
const EXCHANGE_RATE_API = "https://api.exchangerate-api.com/v4/latest";

const currencyFormats = {
  PLN: { symbol: "zł", position: "after", separator: "," },
  USD: { symbol: "$", position: "before", separator: "." },
  EUR: { symbol: "€", position: "after", separator: "." },
};

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
    const selectedCurrency = ref("PLN");
    const isLoading = ref(false);
    const currencies = ref({ PLN: 1, USD: null, EUR: null });

    const sortParams = reactive({ sortKey: "", sortOrder: 1 });

    const fetchProducts = async () => {
      const { data } = await axios.get(`${BASE_API_URL}/products`);
      products.value = data;
    };

    const handleNewProduct = async (product) => {
      const { data } = await axios.post(`${BASE_API_URL}/products`, product);
      products.value.push(data);
    };

    const filterBy = (type, value) => {
      searchText.value = value;
    };

    const clearFilters = () => {
      searchText.value = "";
    };

    const closeForm = () => {
      showForm.value = false;
    };

    const fetchExchangeRates = async () => {
      isLoading.value = true;
      try {
        const { data } = await axios.get(`${EXCHANGE_RATE_API}/PLN`);
        currencies.value = {
          ...currencies.value,
          USD: data.rates.USD,
          EUR: data.rates.EUR,
        };
      } catch (error) {
        console.log(error);
      }
      isLoading.value = false;
    };

    const convertPrice = (price) => {
      const currencyValue =
        price * (currencies.value[selectedCurrency.value] || 1);
      const formattedValue = new Intl.NumberFormat("en-US", {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
        useGrouping: false,
      }).format(currencyValue);

      const { symbol, separator } = currencyFormats[selectedCurrency.value];
      return symbol === "zł"
        ? `${formattedValue.replace(".", separator)} ${symbol}`
        : `${symbol}${formattedValue}`;
    };

    const sortProducts = (key) => {
      if (sortParams.sortKey === key) {
        sortParams.sortOrder *= -1;
      } else {
        sortParams.sortKey = key;
        sortParams.sortOrder = 1;
      }
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

    watch(
      selectedCurrency,
      () => {
        if (selectedCurrency.value !== "PLN") fetchExchangeRates();
      },
      { immediate: true }
    );

    fetchProducts();

    return {
      ...toRefs(sortParams),
      searchText,
      products,
      currencies,
      handleNewProduct,
      filteredProducts,
      sortProducts,
      filterBy,
      clearFilters,
      closeForm,
      showForm,
      selectedCurrency,
      isLoading,
      convertPrice,
    };
  },
};
</script>
