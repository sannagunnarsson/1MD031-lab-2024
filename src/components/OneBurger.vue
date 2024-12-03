<template>
  <div class="burger-item">
    <h3>{{ burger.name }}</h3>
    <img :src="burger.url" :alt="burger.name" style="width: 200px;">
    <ul>
      <li v-for="(ingredient, index) in burger.ingredients" :key="index">{{ ingredient }}</li>
      <li v-if="burger.gluten"><span id="contains">Contains gluten</span></li>
      <li v-else><span id="free">Gluten free</span></li>
      <li v-if="burger.lactose"><span id="contains">Contains lactose</span></li>
      <li v-else><span id="free">Lactose free</span></li>
    </ul>

<!-- Visa antal beställda -->
<p>Amount: </p>

<button @click="decreaseAmount" :disabled="amountOrdered === 0">-</button>

{{ amountOrdered }}

<!-- Knapp för att öka antalet -->
<button @click="increaseAmount">+</button>

</div>
</template>

<script>
export default {
  name: 'OneBurger',
  props: {
    burger: Object,
    reset: Boolean
  },
  data: function() {
    return {
      amountOrdered: 0 // Håll reda på hur många burgare som är beställda
    };
  },
  methods: {
    increaseAmount: function() {
      this.amountOrdered += 1;
      console.log(`Increased ${this.burger.name} to ${this.amountOrdered}`);

      this.$emit('updateOrder', { name:   this.burger.name, 
                                amount: this.amountOrdered 
                              }
  );

    },
    decreaseAmount: function() {
      if (this.amountOrdered > 0) {
      this.amountOrdered--;
      console.log(`Decreased ${this.burger.name} to ${this.amountOrdered}`);

      this.$emit('updateOrder', { 
        name: this.burger.name, 
        amount: this.amountOrdered 
      });
    }
  },
  resetAmount: function() {
      this.amountOrdered = 0; // Nollställ antal
      console.log(`Reset ${this.burger.name} to ${this.amountOrdered}`);

    }
 },
 updated: function() {
    // Kontrollera om reset har blivit true och nollställ i så fall
    if (this.reset) {
      this.resetAmount();
    }
}
};
</script>


<style scoped>
.burger-item {
  background-color: #6b8a6e;
  padding: 5px;
  border-radius: 8px;
  text-align: center;
}
#free {
  color: green;
  font-weight: bold;
}
#contains {
  color: red;
  font-weight: bold;
}
</style>
