<template>
  <div class="account-wrapper">
    <h1 v-if="user" class="balance-title balance-name">
      {{ user.firstName }} {{ user.middleName }} {{ user.lastName }}
    </h1>
    <h1 class="balance-title" v-if="accounts[0]">
      Your {{ accounts[0].currency }} Balance
    </h1>
    <h1 class="balance" v-if="accounts[0]">
      ${{ formatMoney(accounts[0].balance) }}
    </h1>
    <div class="date">{{ formatDate() }}</div>
    <div class="accounts-holder">
      <div
        v-for="(account, int) in accounts"
        :key="account.id"
        class="each-account"
      >
        <div
          class="account-name check"
          :class="{ check: int == 1, current: int == 2 }"
        >
          {{ account.currency }}
        </div>
        <h1 class="each-account-balance">
          {{ account.symbol }}{{ formatMoney(Number(account.balance)) }}
        </h1>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // user: "",
      accounts: [],
    };
  },
  methods: {
    formatMoney(amount) {
      if (amount == 0) {
        return "0.00";
      }
      if (amount == "" || amount == null || amount == undefined) {
        return "0.00";
      } else {
        return amount.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
      }
    },

    formatDate() {
      const date = new Date().getTime(); // Note that month is zero-indexed in JavaScript
      const options = {
        year: "numeric",
        month: "long",
        day: "numeric",
        suffix: (day) => {
          if (day === 1 || day === 21 || day === 31) {
            return "st";
          } else if (day === 2 || day === 22) {
            return "nd";
          } else if (day === 3 || day === 23) {
            return "rd";
          } else {
            return "th";
          }
        },
      };

      return new Intl.DateTimeFormat("en-US", options).format(date);
    },

    reshuffleAccounts(accounts) {
      const shuffledArray = [...accounts];

      for (let i = shuffledArray.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));

        [shuffledArray[i], shuffledArray[j]] = [
          shuffledArray[j],
          shuffledArray[i],
        ];
      }

      return shuffledArray;
    },

    async getAccount() {
      try {
        const result = await this.$axios.get(
          `/accounts/user-accounts/?username=${this.user.username}`
        );
        this.accounts = this.reshuffleAccounts(result.data.data);
      } catch (err) {
        console.log(err.response.data.message);
      }
    },
  },
  mounted() {
    this.getAccount();
  },
  computed: {
    user() {
      return this.$store.state.auth.user;
    },
  },
};
</script>

<style>
h1.balance-title {
  margin-top: 10px;
}
h1.balance-name {
  font-size: 20px;
  color: #e524c5;
  margin-bottom: 0px;
}
</style>
