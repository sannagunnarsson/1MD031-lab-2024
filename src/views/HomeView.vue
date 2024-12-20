<template>
  <div>
    <header class="header-container">
      <h1>Välkommen till BurgerOnline</h1>
    </header>
  </div>

  <!-- Sektion för att visa burgarmenyn -->
  <section class="burgers">
    <div class="menu-header"></div>
    <div>
      <h2>Menu alternatives</h2>
      <p>This is where you execute burger selection</p>
    </div>

    <!-- Dynamiskt generera burgare -->
    <div class="burger-grid">
      <OneBurger v-for="burger in burgers" v-bind:burger="burger" v-bind:key="burger.name" v-bind:reset="resetSignal"
        v-on:updateOrder="addToOrder" />
    </div>
  </section>

  <!-- Sektion för kundinformation -->
  <main>
    <section class="information">
      <h2>Customer information</h2>
      <form>
        <p>
          <label for="fullname">Full Name</label><br>
          <input type="text" id="fullname" v-model="customer.name" required placeholder="First- and Last name">
        </p>
        <p>
          <label for="email">E-mail</label><br>
          <input type="email" id="email" v-model="customer.email" required placeholder="E-mail address">
        </p>
        <p>
          <label for="payment">Payment options</label><br>
          <select id="payment" v-model="customer.payment">
            <option disabled value="">Credit Card</option>
            <option v-for="option in paymentOptions" :key="option" :value="option">
              {{ option }}
            </option>
          </select>
        </p>

        <p>
          Please indicate point of delivery:
        </p>

        <!-- Kartfunktion för att välja leveransplats -->
        <div class="map-container">
          <div id="map" v-on:click="setLocation">
          </div>
          <div class="target" v-if="clickPosition.x !== 0 || clickPosition.y !== 0"
            :style="{ left: clickPosition.x + 'px', top: clickPosition.y + 'px' }">
          </div>
        </div>

        <p>Gender</p>
        <p>
          <input type="radio" id="male" name="gender" value="male" v-model="customer.gender">
          <label for="male">Male</label>
        </p>
        <p>
          <input type="radio" id="female" name="gender" value="female" v-model="customer.gender">
          <label for="female">Female</label>
        </p>
        <p>
          <input type="radio" id="nb" name="gender" value="nb" v-model="customer.gender">
          <label for="nb">Non-binary</label>
        </p>
        <p>
          <input type="radio" id="undisclosed" name="gender" value="undisclosed" v-model="customer.gender" checked>
          <label for="undisclosed">Undisclosed</label>
        </p>
      </form>
    </section>
  </main>


  <!-- Knappen för att skicka beställning -->
  <section class="order-button">

    <!-- Felmeddelanden -->
    <section class="validation-message" v-if="validationMessage">
      <p class="error-message">{{ validationMessage }}</p>
    </section>

    <!-- Framgångsmeddelanden -->
    <section class="success-message" v-if="successMessage">
      <p class="success-message-text">{{ successMessage }}</p>
    </section>
    <button id="button" v-on:click="addOrder">
      <img src='/img/Food-delivery-man.png' alt="Delivery Icon" style="width: 50px; height: auto;">
      Place my order!
    </button>
  </section>


  <div>
    <footer>
      <hr />
      <p>&copy; 2024 Sannas burgare AB.</p>
    </footer>
  </div>
</template>

<script>
import OneBurger from '../components/OneBurger.vue';
import io from 'socket.io-client';
import menu from '../assets/menu.json';


const socket = io("localhost:3000");

export default {
  name: 'HomeView',
  components: { OneBurger },
  data: function () {
    return {
      burgers: menu,    // Lista över tillgängliga burgare
      orderedBurgers: {},    // Objekt för att hålla beställda burgare
      clickPosition: { x: 0, y: 0 },    // Position där användaren klickar på kartan

      // Kundinformation
      customer: {
        name: '',
        email: '',
        payment: 'Credit Card',
        gender: 'undisclosed',
      },

      // Tillgängliga betalningsalternativ
      paymentOptions: ["Credit card", "Swish", "PayPal", "Debit card", "Apple Pay"],
      resetSignal: false, // Signal för att nollställa antalet burgare
      validationMessage: '',    // Felmeddelanden
      successMessage: '' // Meddelande för godkänd order

    };
  },
  methods: {
    // Visar kundinformation och vald burgare i konsolen
    getOrderInformation: function () {
      console.log("Order details:");
      console.log("Customer Name:", this.customer.name);
      console.log("Email:", this.customer.email);
      console.log("Payment:", this.customer.payment);
      console.log("Gender:", this.customer.gender);
      console.log("Ordered Burgers:", this.orderedBurgers);
    },

    getOrderNumber: function () {
      return Math.floor(Math.random() * 100);
    },

    // Funktion för att skicka beställning
    addOrder: function (event) {

      // Kontrollerar att alla obligatoriska fält är ifyllda genom olika kontroller
      //Återger olika felmeddelanden

      if (Object.keys(this.orderedBurgers).length === 0) {
        this.validationMessage = "You need to add at least one burger to your order!";
        this.successMessage = ''; // Rensa framgångsmeddelandet

        return;
      }
      if (!this.customer.name) {
        this.validationMessage = "Please enter your full name.";
        this.successMessage = '';

        return;
      }

      if (!this.customer.email) {
        this.validationMessage = "Please enter a valid email address.";
        this.successMessage = '';
        return;
      }

      if (!this.customer.payment) {
        this.validationMessage = "Please select a payment method.";
        this.successMessage = '';
        return;
      }

      if (this.clickPosition.x === 0 && this.clickPosition.y === 0) {
        this.validationMessage = "Please select a location on the map.";
        this.successMessage = '';
        return;
      }

      // Om allt är korrekt ifyllt, skickas beställningen
      this.validationMessage = ''; // Rensar eventuella gamla felmeddelanden
      this.successMessage = "Your order has been placed!"; // Skicka meddelande för genomförd order

      // Om validering godkänns, skicka beställning via socket
      socket.emit("addOrder", {
        orderId: this.getOrderNumber(),
        details: this.clickPosition,
        orderItems: this.orderedBurgers,
        customer: this.customer
      });
      this.resetForm();
    },

    // Funktion för att spara platsen där användaren klickar på kartan
    setLocation: function (event) {
      var offset = {
        x: event.currentTarget.getBoundingClientRect().left,
        y: event.currentTarget.getBoundingClientRect().top
      };

      this.clickPosition = {
        x: event.clientX - 10 - offset.x,
        y: event.clientY - 10 - offset.y
      }
    },

    // Funktion för att lägga till burgare i beställningen
    addToOrder: function (event) {
      this.orderedBurgers[event.name] = event.amount;
    },

    // Återställer alla fält i formuläret
    resetForm: function () {
      this.customer = {
        name: '',
        email: '',
        payment: 'Credit card', // Sätt förvalt betalningsmetod
        gender: 'undisclosed', // Återställ kön till "undisclosed"
      };
      this.orderedBurgers = {}; // Töm beställningslistan
      this.clickPosition = { x: 0, y: 0 }; // Återställ kartposition

      this.resetSignal = true;
      this.$nextTick(() => {
        this.resetSignal = false; // Återställ signalen
      });
    }
  }
}

</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Agbalumo&family=Cormorant:wght@700&display=swap');

.map-container {
  width: 100%;
  height: 500px;
  overflow: scroll;
  border: 1px solid #ccc;
  position: relative;
}

#map {
  width: 1920px;
  height: 1078px;
  background: url('/img/polacks.jpg');
  background-size: cover;
  background-position: center;
}

.target {
  position: absolute;
  background-image: url('/img/pin.png');
  background-size: cover;
  width: 35px;
  height: 35px;
  transform: translate(-20%, -65%);
}

body {
  font-size: 12pt;
  font-family: Lexend Deca, sans-serif;
}

p {
  color: rgb(0, 0, 0);
}

h1 {
  font-family: 'Agbalumo', sans-serif;
  font-size: 36pt;
}

main,
header,
footer,
nav ul {
  min-width: 40rem;
  margin: 0 20px 0 20px;
}

main {
  background-color: rgb(176, 195, 172);
  border-style: dashed;
  border-width: 4px;
  border-color: rgb(103, 120, 99);
}

header h1 {
  font-family: 'Agbalumo', sans-serif;
  font-size: 36pt;
  color: black;
  margin: 4px;
  position: absolute;
  margin-top: 10px;
  margin-left: 20%;
}

nav ul {
  display: grid;
  grid-template-columns: repeat(auto-fill, 9.25em);
  gap: 1em;
  padding: 0;
}

nav li {
  display: block;
  background-color: grey;
  padding: 1em;
}

.burger div {
  padding: 40px;
  border-width: 2px;
  border-style: groove;
  border-color: rgb(88, 132, 87);
}

section {
  margin: 20px;
}

.Very-good {
  color: green;
}

.Master {
  color: green;
  font-weight: bold;
}

.header-container {
  width: 100%;
  height: 150px;
  margin: 20px 0;
  border: 1px solid #ccc;
  position: relative;
  background-image: url('/img/headeruppsala.jpeg');
  background-size: cover;
  background-position: center 35%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.header-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(255, 255, 255);
  opacity: 0.5;
  z-index: 0;
}



.header-container h1 {
  font-family: 'Agbalumo', sans-serif;
  font-size: 36pt;
  color: black;
  margin: 0;
  z-index: 1;
}

.burgers {
  background-color: rgb(145, 167, 141);
  color: rgb(255, 255, 255);
  position: relative;

}

.menu-header {
  margin-bottom: 20px;
  text-align: center;
  padding-left: 30px;
}

.burger-grid {
  display: grid;
  grid-gap: 10px;
  grid-template-columns: 32% 32% 32%;
  padding-left: 20px;
  padding-bottom: 20px;
  position: relative;

  #free {
    color: rgb(200, 227, 200);
    font-weight: bold;
  }

  #contains {
    color: rgb(160, 7, 7);
    font-weight: bold;
  }
}

.burger-item img {
  max-width: 100%;
  height: auto;
}

.burgers h2 {
  font-weight: bold;
  font-size: 28pt;
}

.burgers p {
  font-size: 12pt;
  font-weight: bold;
  color: rgb(255, 255, 255);
}

.burger-item {
  background-color: #dae2d8;
  padding: 5px;
  border-radius: 8px;
  border-style: groove;
  border-width: 10px;
  text-align: center;
  position: relative;
  color: black;
}

.a {
  grid-column: 1;
}

.b {
  grid-column: 2;
}

.c {
  grid-column: 3;
}

.information {
  padding: 2px;
  margin-top: 2px;
  font-weight: bold;
  position: relative;

}

button:hover {
  background-color: green;
}

button {
  margin: 40px;
}

.validation-message p {
  color: red;
  font-weight: bold;
  margin-top: 10px;
  font-size: 16px;
}

.success-message p {
  color: green;
  font-weight: bold;
  margin-top: 10px;
  font-size: 16px;
}
</style>
