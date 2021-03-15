<template>
  <div class="about">
    <section v-if="loading">
      <h2>Loading</h2>
    </section>

    <h2>Form</h2>

    <input
      v-model="form.type"
      placeholder="Type"
    />

    <input
      v-model="form.name"
      placeholder="Nama"
    />
    
    <input
      v-model="form.id"
      placeholder="ID"
    />

    <input
      v-model="form.schedule"
      placeholder="Schedule"
    />

    {{form.urls}}
    <input
      v-model="form.urls"
      placeholder="URL"
    />

    <input
      v-model="form.ipv6_monitor"
      placeholder="IP v6"
      disabled
    />

    <input
      v-model="form.ipv4_monitor"
      placeholder="IPv4"
      disabled
    />

    <input
      v-model="form.mode"
      placeholder="Mode"
      disabled
    />

    <br>

    <button
      @click="submit"
    >
      Add
    </button>

    <br>

    {{hasil}}

    <br>

    {{dataMonitors}}

  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'about',
  data() {
    return {
      apiUrl: 'http://localhost:8000',
      form: {
        type: '',
        name: '',
        id: '',
        schedule: '',
        urls: [],
        ipv6_monitor: true,
        ipv4_monitor: true,
        mode: 'any'
      },
      loading: false,
      hasil: '',

      dataMonitors: []
    }
  },
  mounted() {
    this.getData()
  },
  methods: {
    async getData() {
      try {
        
        this.loading = true

        const response = await axios.get(this.apiUrl + '/coba-ambil')

        this.dataMonitors = response.data.data

        this.loading = false

      } catch (error) {
        this.loading = false
        console.log(error)
      }
    },

    async submit(e) {
      e.preventDefault()

      try {
        let formData = new FormData()

        formData.append('type', this.form.type)
        formData.append('name', this.form.name)
        formData.append('id', this.form.id)
        formData.append('schedule', this.form.schedule)

        // this.urls.forEach(element => {
        //   formData.append('urls[]', element)
        // });
        formData.append('urls[]', this.form.urls)

        formData.append('ipv6_monitor', this.form.ipv6_monitor)
        formData.append('ipv4_monitor', this.form.ipv4_monitor)
        formData.append('mode', this.form.mode)

        this.loading = true

        const response = await axios.post(this.apiUrl + '/coba', formData)

        this.loading = false

        this.hasil = response.data.data

      } catch (error) {
        this.loading = false

        console.log(error)
      }
    }
  }
}
</script>

<style>

</style>
