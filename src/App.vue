<template>
  <div v-if="!showReceipt">
    <div id="app" class="bpi-design">
      <img src="./assets/img/bank logo.png" alt="Bank Logo" class="bank-logo" />

      <p class="greeting">Hi, {{ accountHolder }}</p>

      <div v-if="!isPinEntered" class="pin-entry">
        <div class="pin-display">
          <span v-for="index in enteredPin.length" :key="index">*</span>
        </div>

        <div class="numeric-keypad">
          <button
            v-for="digit in pinDigits"
            :key="digit"
            @click="enterDigit(digit)"
            class="digit-button"
          >
            {{ digit }}
          </button>
        </div>

        <div class="pin-buttons">
          <button @click="clearPin" class="transaction-button">Clear</button>
          <span class="button-space"></span>
          <button @click="verifyPin" class="transaction-button">Submit</button>
        </div>
      </div>

      <div v-else>
        <div class="account-type">
          <label for="accountType" class="account-type-label"
            >Choose Account:</label
          >
          <select
            v-model="selectedAccount"
            @change="updateBalance"
            class="account-type-dropdown"
          >
            <option value="savings">Savings Account</option>
            <option value="checking">Checking Account</option>
          </select>
        </div>

        <div class="account-info">
          <p class="account-holder">Account Holder: {{ accountHolder }}</p>
          <p class="account-balance">
            Account Balance: ₱{{ formattedBalance }}
          </p>
        </div>

        <div class="transaction-form">
          <label for="amount" class="transaction-label">Amount:</label>
          <input
            type="number"
            v-model="transactionAmount"
            class="transaction-input"
          />

          <button @click="deposit" class="transaction-button">Deposit</button>
          <button @click="withdraw" class="transaction-button">Withdraw</button>
          <button @click="logout" class="transaction-button">Logout</button>
        </div>
      </div>
    </div>
  </div>
  <div v-if="showReceipt">
    <Receipt
      :accountHolder="accountHolder"
      :selectedAccount="selectedAccount"
      :formattedBalance="formattedBalance"
      :transactionHistory="transactionHistory"
    />
  </div>
</template>

<script>
import Receipt from "./Receipt.vue";
export default {
  components: {
    Receipt,
  },
  data() {
    return {
      bankName: "BPI",
      accountHolder: "John",
      selectedAccount: "savings",
      showReceipt: false,
      balance: {
        savings: 1000,
        checking: 5000,
      },
      transactionAmount: 0,
      defaultPin: "1234",
      enteredPin: "",
      isPinEntered: false,
      pinDigits: [1, 2, 3, 4, 5, 6, 7, 8, 9, 0],
      transactionHistory: [],
    };
  },
  computed: {
    formattedBalance() {
      return this.balance[this.selectedAccount].toLocaleString();
    },
  },
  methods: {
    deposit() {
      if (this.transactionAmount > 0) {
        this.balance[this.selectedAccount] += parseFloat(
          this.transactionAmount
        );
        this.recordTransaction("Deposit", this.transactionAmount);
        this.showReceipt = true;
      }
    },
    withdraw() {
      if (
        this.transactionAmount > 0 &&
        this.transactionAmount <= this.balance[this.selectedAccount]
      ) {
        this.balance[this.selectedAccount] -= parseFloat(
          this.transactionAmount
        );
        this.recordTransaction("Withdrawal", this.transactionAmount);
        this.showReceipt = true;
        this.generateReceipt();
      } else {
        alert("Insufficient funds!");
      }
    },
    reset() {
      this.transactionAmount = 0;
    },
    enterDigit(digit) {
      if (this.enteredPin.length < 4) {
        this.enteredPin += digit.toString();
      }
    },
    clearPin() {
      this.enteredPin = "";
    },
    verifyPin() {
      if (this.enteredPin === this.defaultPin) {
        this.isPinEntered = true;
      } else {
        alert("Incorrect PIN. Please try again.");
        this.clearPin();
      }
    },
    logout() {
      this.isPinEntered = false;
      this.clearPin();
      this.transactionHistory = [];
    },
    newTransaction() {
      this.transactionAmount = 0;
      this.transactionHistory = [];
    },
    generateReceipt() {
      console.log("Bank Receipt:");
      this.transactionHistory.forEach((transaction, index) => {
        console.log(
          `${index + 1}. ${transaction.type}: ₱${transaction.amount}`
        );
      });

      // Reset values after generating the receipt
      this.selectedAccount = ""; // Clear the selected account
      this.transactionAmount = 0; // Reset transaction amount
      // Add more data reset as needed
    },
    recordTransaction(type, amount) {
      this.transactionHistory.push({
        type,
        amount,
      });
    },
  },
};
</script>
