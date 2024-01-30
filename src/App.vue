<template>
  <div id="app" class="bpi-design">
    <img src="./assets/img/bank logo.png" alt="Bank Logo" class="bank-logo" />

    <p class="greeting">Hi, {{ accountHolder }}</p>
    <div class="pin-display">
      <span v-for="index in enteredPin.length" :key="index">*</span>
    </div>
    <div v-if="!isPinEntered" class="pin-entry">
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

      <button @click="clearPin" class="clear-button">Clear</button>
      <button @click="verifyPin" class="submit-button">Submit</button>
    </div>

    <div v-if="isPinEntered">
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
        <p class="account-balance">Account Balance: ₱{{ formattedBalance }}</p>
      </div>

      <div class="transaction-form">
        <label for="amount" class="transaction-label">Amount:</label>
        <input
          type="number"
          v-model="transactionAmount"
          class="transaction-input"
        />
        <button @click="deposit" class="transaction-button deposit">
          Deposit
        </button>
        <button @click="withdraw" class="transaction-button withdraw">
          Withdraw
        </button>
        <button @click="reset" class="transaction-button reset">Reset</button>
        <button
          @click="generateReceipt"
          class="transaction-button generate-receipt"
        >
          Generate Receipt
        </button>
        <button @click="logout" class="transaction-button logout">
          Logout
        </button>
        <button
          @click="newTransaction"
          class="transaction-button new-transaction"
        >
          New Transaction
        </button>
      </div>

      <div v-if="transactionHistory.length > 0" class="receipt">
        <h2>Bank Receipt</h2>
        <ul>
          <li v-for="(transaction, index) in transactionHistory" :key="index">
            {{ transaction.type }}: ₱{{ transaction.amount }}
          </li>
        </ul>
      </div>
    </div>
  </div>
  <audio ref="clickSound" src="./assets/audio/pinbeep.mp3"></audio>
</template>

<script>
export default {
  data() {
    return {
      bankName: "BPI",
      accountHolder: "John",
      selectedAccount: "savings",
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
    playClickSound() {
      this.$refs.clickSound.play();
    },
    deposit() {
      if (this.transactionAmount > 0) {
        this.balance[this.selectedAccount] += parseFloat(
          this.transactionAmount
        );
        this.recordTransaction("Deposit", this.transactionAmount);
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
      } else {
        alert("Insufficient funds!");
      }
    },
    reset() {
      this.transactionAmount = 0;
    },
    updateBalance() {
      // Add any additional logic if needed when the account type changes
    },
    enterDigit(digit) {
      this.playClickSound();
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
      this.transactionHistory = []; // Clear transaction history on logout
    },
    newTransaction() {
      this.transactionAmount = 0;
      this.transactionHistory = [];
    },
    generateReceipt() {
      // This function can be used to generate a receipt based on the current transaction history
      // In this example, we are simply logging the receipt to the console
      console.log("Bank Receipt:");
      this.transactionHistory.forEach((transaction, index) => {
        console.log(
          `${index + 1}. ${transaction.type}: ₱${transaction.amount}`
        );
      });
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
