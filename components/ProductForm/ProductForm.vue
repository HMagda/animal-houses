<template>
  <div class="product-form-wrapper" @click="closeForm">
    <div class="product-form" @click.stop>
      <span class="form-close" @click="closeForm">x</span>
      <h2 class="form-title">Add New Product</h2>

      <form @submit.prevent="submitForm">
        House Name
        <input type="text" v-model="name" placeholder="Name" required />

        Price
        <input
          type="number"
          v-model="price"
          min="0.01"
          step="0.01"
          placeholder="Price"
          required
        />

        Location
        <div class="dropdown">
          <div class="dropdown-toggle" @click="toggleLocationDropdown">
            {{ selectedLocation || "Select location" }}
          </div>
          <div
            class="dropdown-menu"
            :style="{ display: showLocationDropdown ? 'block' : 'none' }"
          >
            <div class="dropdown-item">
              <input v-model="newLocation" placeholder="Type new location..." />
              <button @click.prevent.stop="addNewLocation">+</button>
            </div>
            <div
              class="dropdown-item"
              v-for="location in uniqueLocationTypes"
              :key="location"
              @click="selectLocation(location)"
            >
              {{ location }}
            </div>
          </div>
        </div>

        Animal
        <div class="dropdown">
          <div class="dropdown-toggle" @click="toggleAnimalDropdown">
            {{ selectedAnimal || "Select animal" }}
          </div>
          <div
            class="dropdown-menu"
            :style="{ display: showAnimalDropdown ? 'block' : 'none' }"
          >
            <div class="dropdown-item">
              <input v-model="newAnimal" placeholder="Type new animal..." />
              <button @click.prevent.stop="addNewAnimal">+</button>
            </div>
            <div
              class="dropdown-item"
              v-for="animal in uniqueAnimalTypes"
              :key="animal"
              @click="selectAnimal(animal)"
            >
              {{ animal }}
            </div>
          </div>
        </div>

        <button class="btn-submit" type="submit">Add Product</button>
      </form>
    </div>
  </div>
</template>

<script>
import { ref, inject, computed } from "vue";
import "../ProductForm/ProductForm.scss";

export default {
  setup(_, { emit }) {
    const products = inject("products");

    const uniqueLocationTypes = computed(() => {
      const locationTypes = products.value.map(
        (product) => product.locationType
      );
      return [...new Set(locationTypes)];
    });

    const uniqueAnimalTypes = computed(() => {
      const animalTypes = products.value.map((product) => product.animalType);
      return [...new Set(animalTypes)];
    });

    const showLocationDropdown = ref(false);
    const showAnimalDropdown = ref(false);

    const selectedLocation = ref("");
    const selectedAnimal = ref("");

    const newLocation = ref("");
    const newAnimal = ref("");

    const name = ref("");
    const price = ref(0);

    const selectLocation = (location) => {
      selectedLocation.value = location;
      newLocation.value = "";
      showLocationDropdown.value = false;
    };

    const selectAnimal = (animal) => {
      selectedAnimal.value = animal;
      newAnimal.value = "";
      showAnimalDropdown.value = false;
    };

    const addNewLocation = () => {
      if (newLocation.value) {
        uniqueLocationTypes.value.push(newLocation.value);
        selectLocation(newLocation.value);
      }
    };

    const addNewAnimal = () => {
      if (newAnimal.value) {
        uniqueAnimalTypes.value.push(newAnimal.value);
        selectAnimal(newAnimal.value);
      }
    };

    const toggleLocationDropdown = () => {
      showLocationDropdown.value = !showLocationDropdown.value;
      showAnimalDropdown.value = false;
    };

    const toggleAnimalDropdown = () => {
      showAnimalDropdown.value = !showAnimalDropdown.value;
      showLocationDropdown.value = false;
    };

    const closeForm = () => {
      emit("close-form");
    };

    const submitForm = () => {
      if (!selectedLocation.value || !selectedAnimal.value) {
        alert("Please select both location and animal");
        return;
      }

      const newProduct = {
        id: Date.now(),
        name: name.value,
        price: price.value,
        locationType: selectedLocation.value,
        animalType: selectedAnimal.value,
      };

      name.value = "";
      price.value = 0;
      selectedLocation.value = "";
      selectedAnimal.value = "";

      emit("new-product", newProduct);
      closeForm();
    };

    return {
      uniqueLocationTypes,
      uniqueAnimalTypes,
      showLocationDropdown,
      showAnimalDropdown,
      selectedLocation,
      selectedAnimal,
      newLocation,
      newAnimal,
      selectLocation,
      selectAnimal,
      addNewLocation,
      addNewAnimal,
      toggleLocationDropdown,
      toggleAnimalDropdown,
      submitForm,
      closeForm,
      name,
      price,
    };
  },
};
</script>
