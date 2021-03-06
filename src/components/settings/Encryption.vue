<template>
  <section>
    <h4 class="font-weight-normal mt-3 mt-md-5">Client-side Encryption</h4>
    <hr>
    <div class="form-group">
      <label for="tags-search">Documents are encrypted using a <a href="https://en.wikipedia.org/wiki/Hybrid_cryptosystem" target="_blank" rel="noopener noreferrer">hybrid cryptosystem</a>. You may provide your own keys or generate a new set here. If you choose to generate a new set, be sure to <strong>make a secure backup of your keys</strong>. If you lose your private key, you will not be able to recover any data that is encrypted.</label>
    </div>
    <div class="form-group">
      <div>
        <label class="btn btn-primary btn-toggle">
          <div class="custom-control custom-checkbox d-flex align-items-center">
            <input v-model="toggleCrypto" :disabled="!allowCrypto || togglingCrypto" type="checkbox" class="custom-control-input d-flex">
            <span class="custom-control-label d-flex">Enable Encryption</span>
          </div>
        </label>
      </div>
      <small class="text-muted">Note: Toggling encryption will encrypt/decrypt all existing documents. <span v-if="!allowCrypto">Enabling encryption <strong>requires</strong> private/public keys. Generate or supply them below to enable.</span></small>
    </div>
    <div class="form-group">
      <label for="tags-search">Private Key</label>
      <textarea v-model="privateKey" class="form-control" rows="5" placeholder="Private key" autocomplete="off"></textarea>
      <small class="text-muted">This key is used to <em>decrypt</em> documents. It <strong>will not</strong> be synced across devices when signed in.</small>
    </div>
    <div class="form-group">
      <label for="tags-search">Public Key</label>
      <textarea v-model="publicKey" class="form-control" rows="5" placeholder="Public key" autocomplete="off"></textarea>
      <small class="text-muted">This key is used to <em>encrypt</em> documents. It <strong>will</strong> be synced across devices when signed in.</small>
    </div>
    <div class="form-group">
      <button @click="generateKeys" class="btn btn-secondary">Generate Keys</button>
    </div>
  </section>
</template>

<script>
import { exportKeys, generateKeys } from '@/common/crypto/asymmetric';

import { TOUCH_DOCUMENT } from '@/store/actions';

import {
  SET_CRYPTO_ENABLED,
  SET_CRYPTO_KEYS,
} from '@/store/modules/settings';

export default {
  name: 'Encryption',
  data() {
    return {
      togglingCrypto: false,
    };
  },
  computed: {
    allowCrypto() {
      return this.privateKey && this.publicKey;
    },
    privateKey: {
      get() {
        return this.$store.state.settings.crypto.privateKey;
      },
      set(value) {
        this.$store.dispatch(SET_CRYPTO_KEYS, {
          privateKey: value,
        });
      },
    },
    publicKey: {
      get() {
        return this.$store.state.settings.crypto.publicKey;
      },
      set(value) {
        this.$store.dispatch(SET_CRYPTO_KEYS, {
          publicKey: value,
        });
      },
    },
    toggleCrypto: {
      get() {
        return this.$store.state.settings.crypto.enabled;
      },
      async set(value) {
        this.togglingCrypto = true;

        await this.$store.dispatch(SET_CRYPTO_ENABLED, value);
        await Promise.all(
          this.$store.getters.decrypted.map((doc) => {
            return this.$store.dispatch(TOUCH_DOCUMENT, doc);
          })
        );

        this.togglingCrypto = false;
      },
    },
  },
  methods: {
    async generateKeys() {
      const keys = await generateKeys();
      const { privateKey, publicKey } = await exportKeys(keys);

      this.privateKey = privateKey;
      this.publicKey = publicKey;
    },
  },
};
</script>
