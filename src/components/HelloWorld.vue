<script setup>
import { ref } from 'vue';
import bs58 from 'bs58';
import {
  Connection,
  LAMPORTS_PER_SOL,
  SystemProgram,
  Transaction,
  PublicKey
} from "@solana/web3.js";
import { Buffer } from 'buffer';

const account = ref(null);
const nativeBalance = ref(null);
const signedMessage = ref(null);


const connectWallet = async () => {
  try {

    if (account.value) {
      account.value = null;
      const provider = window.phantom.solana;
      return await provider.disconnect();
      
    }
    const provider = window.phantom.solana;
    if (!provider) {
      return console.log('Install Phantom Wallet first');
    }
    const connect = await provider.connect();
    const accounts = connect.publicKey.toString();
    if (accounts) {
      account.value = accounts;
      await checkNativeBalance(connect.publicKey);
    }
  } catch (error) {
    console.log(error.message)
  }

}

const checkNativeBalance = async (address) => {
  try {
    const endpoint = "https://api.mainnet-beta.solana.com";
    const connection = new Connection(endpoint);
    const balance = await connection.getBalance(address);
    nativeBalance.value = balance / LAMPORTS_PER_SOL;
  } catch (error) {
    console.log(error.message)
  }
}

const signMessage = async () => {
  try {
    const provider = window.phantom.solana;
    const message = 'Sign this test message';
    const encodedMessage = new TextEncoder().encode(message);
    const signedMessageUTF8 = await provider.signMessage(encodedMessage, "utf8");
    signedMessage.value = bs58.encode(signedMessageUTF8.signature);

  } catch (error) {
    console.log(error.message)
  }
}

const sendTransaction = async () => {
  try {
    window.Buffer = Buffer;
    const endpoint = "https://api.mainnet-beta.solana.com";
    const connection = new Connection(endpoint);
    console.log(window.phantom.solana)

    const provider = window.phantom?.solana;
    const fromPubkey = provider.publicKey;
    const toPubkey = "4eYoFfe3kjDotWdb5zxG4kyogM4sTZwCj1WMLd9QGHYf";
    const amount = 0.01;
    const blockhash = (await connection.getLatestBlockhash("finalized")).blockhash;
    const transaction = new Transaction().add(
      SystemProgram.transfer({
        fromPubkey,
        toPubkey,
        lamports: amount * LAMPORTS_PER_SOL,
      })
    );

    transaction.feePayer = fromPubkey;
    transaction.recentBlockhash = blockhash;
    const signedTransaction = await provider.signAndSendTransaction(
      transaction
    );

    console.log(signedTransaction.signature)
  } catch (error) {
    console.log(error.message)
  }
}
</script>

<template>
  <div class="greetings">
    <div v-if="account"> Address: {{ account }} </div>
    <div v-if="nativeBalance && account"> Native Balance: {{ nativeBalance }} </div>
    <div v-if="signedMessage && account"> Sign Message: {{ signedMessage }} </div>
    <div class="connectWallet">
      <button v-if="account" class="button" @click="signMessage()">Sign Message</button>
      <button v-if="account" class="button" @click="sendTransaction()">Send Transaction</button>
      <button class="button" @click="connectWallet()">{{ account ? 'Disconnect' : 'Connect Wallet' }}</button>
    </div>
  </div>
</template>

<style scoped>
.address {
  margin-bottom: 30px
}

.connectWallet {
  margin-top: 30px
}

.button {
  margin-right: 5px;
}
</style>
