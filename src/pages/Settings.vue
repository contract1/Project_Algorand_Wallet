<template>
  <main-layout>
    <h1>{{ $t("settings.title") }}</h1>
    <h2>{{ $t("settings.server") }}</h2>
    <label for="env">{{ $t("settings.environment") }}</label>
    <select id="env" v-model="env" class="form-control">
      <option value="mainnet">{{ $t("settings.mainnet") }}</option>
      <option value="testnet">{{ $t("settings.testnet") }}</option>
      <option value="sandbox">{{ $t("settings.sandbox") }}</option>
      <option value="devnet">{{ $t("settings.devnet") }}</option>
      <option value="custom">{{ $t("settings.custom") }}</option>
    </select>

    <table>
      <tr>
        <td>AlgoD {{ $t("settings.host") }}:</td>
        <td>
          <input
            type="text"
            :disabled="env != 'custom'"
            v-model="algodHost"
            class="form-control"
          />
        </td>
      </tr>
      <tr>
        <td>AlgoD {{ $t("settings.token") }}:</td>
        <td>
          <input
            type="text"
            :disabled="env != 'custom'"
            v-model="algodToken"
            class="form-control"
          />
        </td>
      </tr>
      <tr>
        <td>KMD {{ $t("settings.host") }}:</td>
        <td>
          <input
            type="text"
            :disabled="env != 'custom'"
            v-model="kmdHost"
            class="form-control"
          />
        </td>
      </tr>
      <tr>
        <td>KMD {{ $t("settings.token") }}:</td>
        <td>
          <input
            type="text"
            :disabled="env != 'custom'"
            v-model="kmdToken"
            class="form-control"
          />
        </td>
      </tr>
      <tr>
        <td>Indexer {{ $t("settings.host") }}:</td>
        <td>
          <input
            type="text"
            :disabled="env != 'custom'"
            v-model="indexerHost"
            class="form-control"
          />
        </td>
      </tr>
      <tr>
        <td>Indexer {{ $t("settings.token") }}:</td>
        <td>
          <input
            type="text"
            :disabled="env != 'custom'"
            v-model="indexerToken"
            class="form-control"
          />
        </td>
      </tr>
    </table>
    <div>
      <h2>{{ $t("settings.protocol_title") }}</h2>
      <button @click="registerProtocolClick" class="btn btn-light">
        {{ $t("settings.protocol_button") }}
      </button>
    </div>

    <h2>{{ $t("settings.language") }}</h2>

    <Dropdown
      v-model="$i18n.locale"
      :options="$store.state.config.languages"
      @change="languageUpdated"
      style="min-width: 100px"
    >
      <template #value="slotProps">
        <div v-if="slotProps.value" class="border-dark">
          <img
            :alt="slotProps.value"
            class="border-dark"
            :src="'/flags/3x2/' + slotProps.value + '.svg'"
            height="15"
          />
          <span class="m-1">{{ slotProps.value }}</span>
        </div>
        <span v-else>
          {{ slotProps.placeholder }}
        </span>
      </template>
      <template #option="slotProps">
        <div class="border-dark">
          <img
            :alt="slotProps.option"
            :src="'/flags/3x2/' + slotProps.option + '.svg'"
            height="15"
          />
          <span class="m-1">{{ slotProps.option }}</span>
        </div>
      </template>
    </Dropdown>
    <h2>{{ $t("settings.pass") }}</h2>
    <form @submit="changePasswordClick">
      <label>{{ $t("settings.oldpass") }}</label>
      <input type="password" class="form-control my-2" v-model="passw1" />
      <label
        >{{ $t("settings.newpass") }}
        <span v-if="strength" :class="strengthClass">{{
          strength
        }}</span></label
      >
      <input type="password" class="form-control my-2" v-model="passw2" />
      <label>{{ $t("settings.repeatpass") }}</label>
      <input type="password" class="form-control my-2" v-model="passw3" />
      <input type="submit" class="btn btn-light my-2" value="Update password" />
    </form>
    <h2>{{ $t("settings.backup") }}</h2>
    <p>{{ $t("settings.backup_help") }}</p>
    <p>
      <a v-if="!b64wallet" @click="makeBackupDataClick" class="btn btn-light">
        {{ $t("settings.create_backup") }}
      </a>
      <a
        v-if="b64wallet"
        :href="'data:image/png;base64,' + b64wallet"
        :download="downloadableWalletName"
        target="_blank"
        class="btn btn-primary"
      >
        {{ $t("settings.download") }}
      </a>
      <a
        v-if="b64wallet"
        @click="destroyWalletClick"
        class="btn btn-danger mx-2"
      >
        {{ $t("settings.delete") }}
      </a>
    </p></main-layout
  >
</template>

<script>
import { mapActions } from "vuex";
import MainLayout from "../layouts/Main.vue";
import { passwordStrength } from "check-password-strength";

export default {
  data() {
    return {
      env: "mainnet",
      passw1: "",
      passw2: "",
      passw3: "",
      b64wallet: "",
      algodHost: "",
      algodToken: "",
      kmdHost: "",
      kmdToken: "",
      indexerHost: "",
      indexerToken: "",
    };
  },

  watch: {
    env() {
      if (this.env == "mainnet") {
        this.setHosts({
          env: "mainnet",
          algod: "https://algoexplorerapi.io",
          kmd: "?",
          indexer: "https://algoexplorerapi.io/idx2",
        });
      }
      if (this.env == "testnet") {
        this.setHosts({
          env: "testnet",
          algod: "https://testnet.algoexplorerapi.io",
          kmd: "?",
          indexer: "https://testnet.algoexplorerapi.io/idx2",
        });
      }
      if (this.env == "devnet") {
        this.setHosts({
          env: "devnet",
          algod: "http://localhost:4180",
          kmd: "http://localhost:4002",
          indexer: "http://localhost:8980",
          algodToken:
            "c87f5580d7a866317b4bfe9e8b8d1dda955636ccebfa88c12b414db208dd9705",
          indexerToken: "reach-devnet",
        });
      }
      if (this.env == "sandbox") {
        this.setHosts({
          env: "sandbox",
          algod: "http://localhost:4001",
          kmd: "http://localhost:4002",
          indexer: "http://localhost:8980",
          algodToken:
            "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
          kmdToken:
            "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
          indexerToken:
            "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
        });
      }
      localStorage.setItem("env", this.env);
    },
    algodHostConfig() {
      if (this.algodHost != this.algodHostConfig)
        this.algodHost = this.algodHostConfig;
    },
    algodTokenConfig() {
      if (this.algodToken != this.algodTokenConfig)
        this.algodToken = this.algodTokenConfig;
    },
    kmdHostConfig() {
      if (this.kmdHost != this.kmdHostConfig) this.kmdHost = this.kmdHostConfig;
    },
    kmdTokenConfig() {
      if (this.kmdToken != this.kmdTokenConfig)
        this.kmdToken = this.kmdTokenConfig;
    },
    indexerHostConfig() {
      if (this.indexerHost != this.indexerHostConfig)
        this.indexerHost = this.indexerHostConfig;
    },
    indexerTokenConfig() {
      if (this.indexerToken != this.indexerTokenConfig)
        this.indexerToken = this.indexerTokenConfig;
    },
    algodHost() {
      if (this.algodHost != this.algodHostConfig) this.updateConfig();
    },
    algodToken() {
      if (this.algodToken != this.algodTokenConfig) this.updateConfig();
    },
    kmdHost() {
      if (this.kmdHost != this.kmdHostConfig) this.updateConfig();
    },
    kmdToken() {
      if (this.kmdToken != this.kmdTokenConfig) this.updateConfig();
    },
    indexerHost() {
      if (this.indexerHost != this.indexerHostConfig) this.updateConfig();
    },
    indexerToken() {
      if (this.indexerToken != this.indexerTokenConfig) this.updateConfig();
    },
  },
  computed: {
    envConfig() {
      return this.$store.state.config.env;
    },
    algodHostConfig() {
      return this.$store.state.config.algod;
    },
    algodTokenConfig() {
      return this.$store.state.config.algodToken;
    },
    kmdHostConfig() {
      return this.$store.state.config.kmd;
    },
    kmdTokenConfig() {
      return this.$store.state.config.kmdToken;
    },
    indexerHostConfig() {
      return this.$store.state.config.indexer;
    },
    indexerTokenConfig() {
      return this.$store.state.config.indexerToken;
    },
    downloadableWalletName() {
      return (
        this.$store.state.wallet.name.replace(" ", "") +
        ".algow." +
        new Date().toISOString().slice(0, 10) +
        ".dat"
      );
    },
    strengthClass() {
      if (!this.passw2) return "";
      const ret = passwordStrength(this.passw2);
      if (ret.id <= 0) return "badge bg-danger";
      if (ret.id <= 1) return "badge bg-warning text-dark";
      return "badge bg-success";
    },
    strength() {
      if (!this.passw2) return "";
      const ret = passwordStrength(this.passw2);
      return this.$t("strength") + ": " + ret.value;
    },
  },
  components: {
    MainLayout,
  },
  mounted() {
    if (this.envConfig) {
      this.env = this.envConfig;
    }
    this.algodHost = this.algodHostConfig;
    this.algodToken = this.algodTokenConfig;
    this.kmdHost = this.kmdHostConfig;
    this.kmdToken = this.kmdTokenConfig;
    this.indexerHost = this.indexerHostConfig;
    this.indexerToken = this.indexerTokenConfig;
  },
  methods: {
    ...mapActions({
      setHosts: "config/setHosts",
      changePassword: "wallet/changePassword",
      backupWallet: "wallet/backupWallet",
      destroyWallet: "wallet/destroyWallet",
      openError: "toast/openError",
      openSuccess: "toast/openSuccess",
    }),
    changePasswordClick(e) {
      e.preventDefault();
      const result = this.changePassword({
        passw1: this.passw1,
        passw2: this.passw2,
        passw3: this.passw3,
      });
      if (result) {
        alert(this.$t("settings.updated_password"));
      }
    },
    async makeBackupDataClick() {
      this.b64wallet = await this.backupWallet();
    },
    async destroyWalletClick() {
      await this.destroyWallet();
    },
    languageUpdated() {
      localStorage.setItem("lang", this.$i18n.locale);
    },
    updateConfig() {
      console.log("update", {
        env: this.env,
        algod: this.algodHost,
        kmd: this.kmdHost,
        indexer: this.indexerHost,
        algodToken: this.algodToken,
        kmdToken: this.kmdToken,
        indexerToken: this.indexerToken,
      });
      this.setHosts({
        env: this.env,
        algod: this.algodHost,
        kmd: this.kmdHost,
        indexer: this.indexerHost,
        algodToken: this.algodToken,
        kmdToken: this.kmdToken,
        indexerToken: this.indexerToken,
      });
    },
    registerProtocolClick(e) {
      e.preventDefault();
      try {
        navigator.registerProtocolHandler(
          "web+algorand",
          location.origin + "/pay/%s",
          "A Wallet"
        );
        this.openSuccess(this.$t("settings.protocol_change_success"));
      } catch (exc) {
        this.openError(exc.message);
      }
    },
  },
};
</script>
