<template>
  <form @submit.prevent="submitForm">
    <input type="text" v-model="name" placeholder="Name" required />
    <input type="number" v-model="price" placeholder="Price" required />
    <select v-model="locationType" required>
      <option disabled value="">Please select a location type</option>
      <option>tree</option>
      <option>ground</option>
    </select>
    <input
      type="text"
      v-model="animalType"
      placeholder="Animal Type"
      required
    />
    <button type="submit">Add Product</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      name: "",
      price: 0,
      locationType: "",
      animalType: "",
    };
  },
  methods: {
    submitForm() {
      const newProduct = {
        id: Date.now(),
        name: this.name,
        price: this.price,
        locationType: this.locationType,
        animalType: this.animalType,
      };

      this.$emit("new-product", newProduct);

      this.name = "";
      this.price = 0;
      this.locationType = "";
      this.animalType = "";
    },
  },
};
</script>
