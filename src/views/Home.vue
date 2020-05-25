<template>
  <Center>
    <div class="block mb-6">
      <img src="~/@/assets/logo.svg" class="mb-2" />
      <h1 class="mb-4">Potion DAI faucet</h1>
      <p v-if="tx" style="max-width: 400px;" class="mb-4">
        Success! Some fake DAI are on the way to your wallet.
        <a
          :href="`https://ropsten.etherscan.io/tx/${tx.hash}`"
          v-text="'Open on Etherscan'"
          target="_blank"
        />
      </p>
      <form v-else @submit.prevent="handleSubmit" class="form">
        <p class="mb-6">
          Mint fake DAI and play with it on
          <a href="https://potion.finance" target="_blank">Potion</a>.
        </p>
        <div class="mb-4">
          <input
            type="text"
            class="input mb-4"
            placeholder="Your address"
            step="0.001"
            v-model="address"
            :required="logged"
          />
        </div>
        <button type="submit" class="button button-primary mb-2">
          <template v-if="!logged">Connect wallet</template>
          <template v-else>Mint</template>
        </button>
      </form>
    </div>
  </Center>
</template>

<script>
import { ethers } from 'ethers';
import provider from '@/helpers/provider';
import { abi as testnetErc20Abi } from '@/helpers/abi/TestnetERC20.json';

const parseEther = ethers.utils.parseEther;

export default {
  data() {
    return {
      logged: false,
      address: '',
      tx: false
    };
  },
  methods: {
    handleSubmit() {
      if (!this.logged) return this.login();
      this.mint();
    },
    async login() {
      if (!provider) return console.error('This website require MetaMask');
      try {
        await window.ethereum.enable();
        const signer = provider.getSigner();
        this.address = await signer.getAddress();
        this.logged = true;
      } catch (error) {
        console.error(error);
      }
    },
    async mint() {
      const signer = provider.getSigner();
      const address = process.env.VUE_APP_DAI_ADDRESS;
      const erc20 = new ethers.Contract(address, testnetErc20Abi, provider);
      const erc20WithSigner = erc20.connect(signer);
      this.tx = await erc20WithSigner.allocateTo(this.address, parseEther((1e9).toString()));
    }
  }
};
</script>
