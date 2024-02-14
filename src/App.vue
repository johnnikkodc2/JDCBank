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
            <option value="savings">029324</option>
            <option value="checking">049212</option>
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
import Swal from "sweetalert2";
import Receipt from "./Receipt.vue";

export default {
  components: {
    Receipt,
  },
  data() {
    return {
      accountHolder: "John Nikko",
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
      if (this.transactionAmount <= 0) {
        this.showNotification("error", "Please insert money!");
        return;
      }

      if (this.transactionAmount > 1) {
        Swal.fire({
          icon: "question",
          title: "Confirmation",
          text: "Are you sure you want to deposit?",
          showCancelButton: true,
          confirmButtonText: "Deposit",
          cancelButtonText: "Cancel",
          reverseButtons: true,
        }).then((result) => {
          if (result.isConfirmed) {
            this.balance[this.selectedAccount] += parseFloat(
              this.transactionAmount
            );
            this.recordTransaction("Deposit", this.transactionAmount);
            this.showReceipt = true;
            this.showNotification("success", "Deposit successful!");
          }
        });
      } else {
        this.balance[this.selectedAccount] += parseFloat(
          this.transactionAmount
        );
        this.recordTransaction("Deposit", this.transactionAmount);
        this.showReceipt = true;
        this.showNotification("success", "Deposit successful!");
      }
    },
    withdraw() {
      if (this.transactionAmount <= 0) {
        this.showNotification(
          "error",
          "Invalid withdrawal amount or insufficient funds!"
        );
        return;
      }

      if (this.transactionAmount > 1) {
        Swal.fire({
          icon: "question",
          title: "Confirmation",
          text: "Are you sure you want to withdraw?",
          showCancelButton: true,
          confirmButtonText: "Withdraw",
          cancelButtonText: "Cancel",
          reverseButtons: true,
        }).then((result) => {
          if (result.isConfirmed) {
            if (!(this.selectedAccount in this.balance)) {
              this.balance[this.selectedAccount] = 0;
            }
            if (this.transactionAmount <= this.balance[this.selectedAccount]) {
              this.balance[this.selectedAccount] -= parseFloat(
                this.transactionAmount
              );
              this.recordTransaction("Withdrawal", this.transactionAmount);
              this.showReceipt = true;
              this.showNotification("success", "Withdrawal successful!");
              this.generateReceipt();
            } else {
              this.showNotification("error", "Insufficient funds!");
            }
          }
        });
      } else {
        if (!(this.selectedAccount in this.balance)) {
          this.balance[this.selectedAccount] = 0;
        }
        if (this.transactionAmount <= this.balance[this.selectedAccount]) {
          this.balance[this.selectedAccount] -= parseFloat(
            this.transactionAmount
          );
          this.recordTransaction("Withdrawal", this.transactionAmount);
          this.showReceipt = true;
          this.showNotification("success", "Withdrawal successful!");
          this.generateReceipt();
        } else {
          this.showNotification("error", "Insufficient funds!");
        }
      }
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
        this.showNotification("error", "Incorrect PIN. Please try again.");
        this.clearPin();
      }
    },
    logout() {
      Swal.fire({
        icon: "question",
        title: "Logout Confirmation",
        text: "Are you sure you want to logout?",
        showCancelButton: true,
        confirmButtonText: "Logout",
        cancelButtonText: "Cancel",
        reverseButtons: true,
      }).then((result) => {
        if (result.isConfirmed) {
          this.isPinEntered = false;
          this.clearPin();
          this.transactionHistory = [];
          this.showNotification("error", "Logged out successfully.");
        }
      });
    },

    recordTransaction(type, amount) {
      this.transactionHistory.push({
        type,
        amount,
      });
    },

    showNotification(type, message) {
      Swal.fire({
        icon: type,
        title: message,
        showConfirmButton: false,
        timer: 1500,
      });
    },
  },
};
</script>
