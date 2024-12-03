<template>
  <div id="orders">
    <div id="orderList">
      <div v-for="(order, key) in orders" :key="'order' + key" class="order-item">
        <div class="order-header">

          <!-- Ordernummer på sidan -->
          <span class="order-number">#{{ key }}</span>

          <!-- Lista över antalet burgare -->
          <div class="order-details">
            <ul>
              <li v-for="(amount, burger) in order.orderItems" :key="burger">
                {{ burger }} (x{{ amount }})
              </li>
            </ul>
          </div>
        </div>

        <!-- Kundinformationen under antalet burgare -->
        <div class="customer-details">
          <p><em>
              {{ order.customer.name }}
              ({{ order.customer.email }}, {{ order.customer.payment }}, {{ order.customer.gender }})
            </em></p>
        </div>
      </div>
      <button id="clearButton" v-on:click="clearQueue">Clear Queue</button>
    </div>
    <div id="dots">
      <div v-for="(order, key) in orders" v-bind:style="{ left: order.details.x + 'px', top: order.details.y + 'px' }"
        v-bind:key="'dots' + key">
        {{ key }}
      </div>
    </div>
  </div>
</template>
<script>
import io from 'socket.io-client'
const socket = io("localhost:3000");

export default {
  name: 'DispatcherView',
  data: function () {
    return {
      orders: null,
    }
  },
  created: function () {
    socket.on('currentQueue', data =>
      this.orders = data.orders);
  },
  methods: {
    clearQueue: function () {
      socket.emit('clearQueue');
    },
    changeStatus: function (orderId) {
      socket.emit('changeStatus', { orderId: orderId, status: "Annan status" });

    }
  }
}
</script>
<style>
#orderList {
  top: 1em;
  left: 1em;
  position: absolute;
  z-index: 2;
  background: rgba(255, 255, 255, 0.9);
  padding: 0.5em;
  font-size: 10pt;
  width: 350px;
  border-radius: 5px;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
}

.order-item {
  margin-bottom: 0.5em;
  padding: 0.5em;
  border-bottom: 1px solid #ddd;
}

.order-header {
  display: flex;
  align-items: flex-start;
}

.order-number {
  font-size: 12pt;
  font-weight: bold;
  color: #333;
  margin-right: 10px;
  flex-shrink: 0;
}

.order-details ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.order-details ul li {
  font-size: 10pt;
  margin: 0;
}

.customer-details p {
  margin: 0.5em 0 0;
  font-size: 9pt;
  color: #555;
  font-style: italic;
}

#dots {
  position: relative;
  margin: 0;
  padding: 0;
  background-repeat: no-repeat;
  width: 1920px;
  height: 1078px;
  cursor: crosshair;
  background-image: url('/img/polacks.jpg');
}

#dots div {
  position: absolute;
  background: red;
  color: black;
  border-radius: 15px;
  width: 22px;
  height: 22px;
  text-align: center;
}

#clearButton {
  background-color: green;
}

#clearButton:hover {
  background-color: lightgreen;
}
</style>