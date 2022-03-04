<script setup>
import { ethers } from 'ethers'
import { ref, onMounted } from 'vue'
import { abi } from "@/json/WavePortal.json"

const CONTRACT_ADDRESS = '0x25ca19ee23ea295b43e96d7e82038b309231ad4e'
const CONTRACT_ABI = abi
const currentAccount = ref('')

const checkIfWalletIsConnected = async () => {
  if (!window.ethereum) return
  try {
    const accounts = await ethereum.request({ method: "eth_accounts" })
    if (accounts.length) {
      currentAccount.value = accounts[0]
      console.log("Found an authorized account:", currentAccount.value)
    }
  } catch (error) { console.log(error) }
}

const connectWallet = async () => {
  if (!window.ethereum) return
  try {
    const accounts = await ethereum.request({ method: "eth_requestAccounts" })
    currentAccount.value = accounts[0]
  } catch (error) { console.log(error) }
}

const wave = async () => {
  if (!window.ethereum) return

  try {
    const provider = new ethers.providers.Web3Provider(ethereum)
    console.log(provider)
    const signer = provider.getSigner()
    console.log(signer)
    const wavePortalContract = new ethers.Contract(CONTRACT_ADDRESS, CONTRACT_ABI, signer)

    let count = await wavePortalContract.getTotalWaves()
    console.log("Retrieved total wave count...", count.toNumber())

    const waveTxn = await wavePortalContract.wave()
    console.log("Mining...", waveTxn.hash)

    await waveTxn.wait()
    console.log("Mined -- ", waveTxn.hash)

    count = await wavePortalContract.getTotalWaves()
    console.log("Retrieved total wave count...", count.toNumber())
  } catch (error) { console.log(error) }
}

onMounted(() => {
  checkIfWalletIsConnected()
})

</script>

<template>
  <div class="mainContainer">
    <div class="dataContainer">
      <div class="header">👋 Hey there!</div>
      <div
        class="bio"
      >I am farza and I worked on self-driving cars so that's pretty cool right? Connect your Ethereum wallet and wave at me!</div>
      <button class="waveButton" @click="wave">Wave at Me</button>
      <button v-if="!currentAccount" class="waveButton" @click="connectWallet">Connect Wallet</button>
    </div>
  </div>
</template>

<style lang="less">
</style>
