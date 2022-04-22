<template>
  <div>
    <b-navbar type="is-primary" transparent :mobile-burger="false">
      <template slot="brand">
        <b-navbar-item>
          Artnet Wifi node '<i>{{ nodeName }}</i>' 
        </b-navbar-item>
      </template>
    </b-navbar>
    <div class="container">
      <form @submit.prevent="updateSettings">
        <b-field label="SSID">
          <b-input v-model="ssid" required icon="wifi"></b-input>
        </b-field>
        <b-field label="Password">
          <b-input
            v-model="password"
            icon="key"
            type="password"
            password-reveal
          ></b-input>
        </b-field>
        <b-field label="Node name">
          <b-input v-model="nodeName" required icon="pencil"></b-input>
        </b-field>
        <b-field label="Start Universe">
          <b-numberinput
            v-model="startUniverse"
            required
            :min="startUniverseMin"
          ></b-numberinput>
        </b-field>
        <b-field label="Pixel Count">
          <b-numberinput
            v-model="pixelCount"
            required
            :min="pixelCountMin"
            :max="pixelCountMax"
          ></b-numberinput>
        </b-field>
        <div class="buttons">
          <b-button type="is-primary" native-type="submit" :disabled="!valid">
            Update
          </b-button>
          <b-button type="is-danger" @click.prevent="reset" :disabled="!valid">
            Reset
          </b-button>
        </div>
      </form>
    </div>
    <b-loading class="loader-layer" :active="!loaded"></b-loading>
    <b-loading class="toast-layer" :active="toast"></b-loading>
  </div>
</template>

<script>
export default {
  name: 'App',
  data: () => ({
    valid: true,
    toast: false,
    pixelCountMin: 1,
    pixelCountMax: 300,
    startUniverseMin: 0,
    loaded: false,
    serverUrl:
      process.env.NODE_ENV === 'production'
        ? window.location.href
        : 'http://localhost:3000/',
    // v-model default values
    // will be overwritten by the settings api call
    ssid: '',
    password: '',
    nodeName: 'artnet-wifi-node',
    pixelCount: 100,
    startUniverse: 0,
  }),
  mounted() {
    this.init()
  },
  watch: {
    startUniverse() {
      this.validate()
    },
    nodeName() {
      this.validate()
    },
  },
  methods: {
    validate() {
      if (
        this.ssid &&
        this.nodeName &&
        this.startUniverse !== null &&
        this.pixelCount > 0 
      )
        this.valid = true
      else this.valid = false
    },
    async init() {
      const res = await fetch(`${this.serverUrl}settings`)
      const {
        ssid,
        password,
        nodeName,
        pixelCount,
        startUniverse,
      } = await res.json()
      this.ssid = ssid
      this.password = password
      this.nodeName = nodeName
      this.pixelCount = pixelCount
      this.startUniverse = startUniverse
      setTimeout(() => {
        this.loaded = true
      }, 1000)
    },
    async reset() {
      this.ssid = ''
      this.password = ''
      this.nodeName = 'wifi-node'
      this.pixelCount = 170
      this.startUniverse = 0

      await this.updateSettings()
    },
    async updateSettings() {
      await fetch(`${this.serverUrl}update`, {
        method: 'POST',
        headers: {
          Accept: 'application/json',
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          ssid: this.ssid,
          password: this.password,
          nodeName: this.nodeName,
          pixelCount: this.pixelCount,
          startUniverse: this.startUniverse,
        }),
      })
      this.toast = true
      this.$buefy.toast.open({
        duration: 20000,
        message: `${this.nodeName} will restart! (this page will reload in 15s)`,
        position: 'is-bottom',
        type: 'is-danger',
      })
      setTimeout(() => {
        window.location.reload()
      }, 15000)
    },
  },
}
</script>

<style lang="css" scoped>
.container {
  margin-top: 30px;
  padding: 0 10px;
}
.buttons {
  display: flex;
  justify-content: flex-end;
}
.loader-layer,
.toast-layer {
  background-color: #ffffff;
  z-index: 5;
}
.toast-layer {
  opacity: 0.7;
}
</style>
