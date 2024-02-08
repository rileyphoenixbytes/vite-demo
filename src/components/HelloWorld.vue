<template>
  <v-container class="fill-height">
    <v-responsive class="align-center text-center fill-height">
      <div class="text-h2 font-weight-light mb-n1">Vite + XchainJS example</div>

      <div class="py-14" />
      <v-row class="d-flex align-center justify-center">
        <v-col cols="12">
          <v-textarea
            variant="filled"
            label="Enter your mnemonic phrase to connect"
            v-model="phrase"
            :class="{ 'v-field--error': errors }"
            :error-messages="errorPhrase"
          ></v-textarea>
        </v-col>
        <v-col cols="auto">
          <v-btn min-width="164" variant="outlined" @click="connectWallet"> connect </v-btn>
        </v-col>
      </v-row>

      <div class="py-14" />

      <v-row class="d-flex align-center justify-center" v-if="thorAddress">
        <v-col cols="auto"> address: {{ thorAddress }} </v-col>

        <v-col cols="auto"> with balance: {{ thorBalance || 0 }} </v-col>
      </v-row>
      <v-row class="d-flex align-center justify-center" v-if="kujiAddress">
        <v-col cols="auto"> address: {{ kujiAddress }} </v-col>

        <v-col cols="auto"> with balance: {{ kujiBalance || 0 }} </v-col>
      </v-row>
      <v-row class="d-flex align-center justify-center">
        <v-col cols="auto">
          <v-btn
            href="https://github.com/xchainjs/xchainjs-lib/tree/master/examples/frameworks/vite-example"
            min-width="164"
            rel="noopener noreferrer"
            target="_blank"
            variant="text"
          >
            <v-icon icon="mdi-github" size="large" start />
            Repository
          </v-btn>
        </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script lang="ts" setup>
import { Client } from '@xchainjs/xchain-thorchain'
import { baseToAsset } from '@xchainjs/xchain-util'
import { computed, onMounted, ref, watch } from 'vue'
import { Client as KujiraClient } from '@xchainjs/xchain-kujira'
import {
  generatePhrase,
  validatePhrase,
  encryptToKeyStore,
  decryptFromKeystore,
} from '@xchainjs/xchain-crypto'

const thorAddress = ref('')
const thorBalance = ref<Number | string>('')
const kujiAddress = ref('')
const kujiBalance = ref<Number | string>('')
const errors = ref<string>('')
const errorPhrase = computed(() => {
  return errors.value ? 'invalid phrase' : ''
})
onMounted(async () => {
  console.log('mounted')


  const keystore =  {
    "crypto": {
      "cipher": "aes-128-ctr",
      "ciphertext": "87604970d2b8b465518683b3a04ae1db1870ebebabee9df5ff0b7504213600e6c52997f8bda265c3e9c11de68ad17d2d6379e3dfb6295c50309db863fab6e7b3b28b3eeacdb1",
      "cipherparams": {
        "iv": "76e78e49f15f61706329a551b7fa15c8"
      },
      "kdf": "pbkdf2",
      "kdfparams": {
        "prf": "hmac-sha256",
        "dklen": 32,
        "salt": "36a4c0ac54b8837633acafa33d8f00d901050e667f93eb27f25abcf381ee2412",
        "c": 262144
      },
      "mac": "c1f9ca36fafb6d091d9a521ec3320b5bf7f16a5c6bdce733df231be90deb1e9d"
    },
    "id": "073c062b-15be-42b4-8b9f-ef6bc4c1c94a",
    "version": 1,
    "meta": "xchain-keystore"
  }
  const decryptFromKeystoreResult = await decryptFromKeystore(keystore, "1234")
  console.log('decryptFromKeystoreResult :>> ', decryptFromKeystoreResult);
})
const phrase = ref<string>(import.meta.env.VITE_PHRASE || '')

// Create new instance of the client and query chain for balances.
const connectWallet = async () => {
  const isPhraseValid = validatePhrase(phrase.value)

  if (isPhraseValid) {
    const thorClient = new Client({ phrase: phrase.value })
    const kujiClient = new KujiraClient({ phrase: phrase.value })

    phrase.value = ''

    const address = await thorClient.getAddressAsync()
    const addressKuji = await kujiClient.getAddressAsync()
    thorAddress.value = address
    kujiAddress.value = addressKuji

    try {
      const balance = await thorClient.getBalance(address)
      const assetAmount = baseToAsset(balance[0].amount).amount()
      thorBalance.value = assetAmount.toNumber()
      phrase.value = ''
    } catch (error) {
      console.error(`Caught ${error}`)
    }

    try {
      const balance = await kujiClient.getBalance(addressKuji)
      const assetAmount = baseToAsset(balance[0].amount).amount()
      kujiBalance.value = assetAmount.toNumber()
    } catch (error) {
      console.error(`kuji balance ${error}`)
    }
  } else {
    errors.value = 'phrase invalid'
    setTimeout(() => {
      errors.value = ''
    }, 3000)
  }
}
watch(phrase, () => {
  errors.value = ''
})
if (phrase.value) connectWallet()
</script>
