<template>
  <form @submit.prevent="submitForm">
    <input type="text" v-model="name" placeholder="Name" required />
    <input type="number" v-model="price" placeholder="Price" required />

    <div class="dropdown">
      <div class="dropdown-toggle" @click="toggleLocationDropdown">
        {{ selectedLocation || 'Select location' }}
      </div>
      <div class="dropdown-menu" :style="{ display: showLocationDropdown ? 'block' : 'none' }">
        <div class="dropdown-item">
          <input v-model="newLocation" placeholder="Type new location..." />
          <button @click.prevent.stop="addNewLocation">+</button>
        </div>
        <div class="dropdown-item" v-for="location in uniqueLocationTypes" :key="location" @click="selectLocation(location)">
          {{ location }}
        </div>
      </div>
    </div>

    <div class="dropdown">
      <div class="dropdown-toggle" @click="toggleAnimalDropdown">
        {{ selectedAnimal || 'Select animal' }}
      </div>
      <div class="dropdown-menu" :style="{ display: showAnimalDropdown ? 'block' : 'none' }">
        <div class="dropdown-item">
          <input v-model="newAnimal" placeholder="Type new animal..." />
          <button @click.prevent.stop="addNewAnimal">+</button>
        </div>
        <div class="dropdown-item" v-for="animal in uniqueAnimalTypes" :key="animal" @click="selectAnimal(animal)">
          {{ animal }}
        </div>
      </div>
    </div>

    <button type="submit">Add Product</button>
  </form>
</template>

<script>
import { ref, inject, computed } from 'vue';
import '../ProductForm/ProductForm.scss';

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

    const selectedLocation = ref('');
    const selectedAnimal = ref('');

    const newLocation = ref('');
    const newAnimal = ref('');

    const name = ref("");
    const price = ref(0);

    const selectLocation = (location) => {
      selectedLocation.value = location;
      newLocation.value = '';
      showLocationDropdown.value = false;
    };

    const selectAnimal = (animal) => {
      selectedAnimal.value = animal;
      newAnimal.value = '';
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

    const submitForm = () => {
      if (!selectedLocation.value || !selectedAnimal.value) {
        alert('Please select both location and animal');
        return;
      }

      const newProduct = {
        id: Date.now(),
        name: name.value,
        price: price.value,
        locationType: selectedLocation.value,
        animalType: selectedAnimal.value,
      };

      emit("new-product", newProduct);

      name.value = "";
      price.value = 0;
      selectedLocation.value = "";
      selectedAnimal.value = "";
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
      name,
      price
    };
  },
};
</script>

