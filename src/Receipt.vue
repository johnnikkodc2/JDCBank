<template>
  <div class="bpi-design">
    <img src="./assets/img/bank logo.png" alt="Bank Logo" class="bank-logo" />

    <div class="receipt">
      <h2>Bank Receipt</h2>
      <p>Name: {{ accountHolder }}</p>
      <p>Date: {{ getCurrentDate() }}</p>
      <p>Time: {{ getCurrentTime() }}</p>
      <p>
        Account number:
        {{ selectedAccount === "savings" ? "029324" : "049212" }}
      </p>
      <p>Balance: ₱{{ formattedBalance }}</p>
      <ul>
        <li v-for="(transaction, index) in transactionHistory" :key="index">
          {{ transaction.type }}: ₱{{ transaction.amount }}
        </li>
      </ul>
      <div class="pin-buttons">
        <button @click="printReceipt" class="print-button">
          Print Receipt
        </button>
        <span class="button-space"></span>
        <button @click="makeAnotherTransaction" class="transaction-button2">
          Make Another Transaction
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import Swal from "sweetalert2";
export default {
  props: {
    accountHolder: {
      type: String,
      default: "John Nikko",
    },
    selectedAccount: {
      type: String,
      default: "savings",
    },
    formattedBalance: {
      type: String,
      default: "0",
    },
    transactionHistory: {
      type: Array,
      default: () => [],
    },
  },
  methods: {
    getCurrentDate() {
      const now = new Date();
      return now.toLocaleDateString();
    },
    getCurrentTime() {
      const now = new Date();
      return now.toLocaleTimeString();
    },
    printReceipt() {
      window.print();
    },
    makeAnotherTransaction() {
      Swal.fire({
        icon: "info",
        title: "Generating New Transaction",
        timer: 2000,
        timerProgressBar: true,
        showConfirmButton: false,
        willClose: () => {
          window.location.reload();
        },
      });
    },
  },
};
</script>

<style>
.print-button,
.transaction-button2 {
  margin-top: 10px;
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
}
.button-space {
  margin-right: 10px;
}
</style>
