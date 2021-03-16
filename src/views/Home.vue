<template>
  <v-container fluid>

    <h2>Heartbeat Monitors</h2>

    <!-- Card Form -->
    <v-card
      class="mt-2"
      elevation="2"
    >
      <v-card-title>Form</v-card-title>
      <v-card-subtitle>Insert Carefully !</v-card-subtitle>
      <v-card-text>
        <!-- Alert Jika Gagal -->
        <v-alert 
          :type="alertObject.type"
          v-model="alertObject.status"
          dismissible
        >
          <li
            v-for="(value, key) of alertObject.message"
            :key="key"
          >
            <template v-for="item in value"> {{item}} </template>
          </li>
        </v-alert>

        <!-- Baris 1 -->
        <v-row>
          <v-col
            cols="12"
            sm="6"
            md="3"
          >
            <v-select
              :items="['http', 'icmp', 'tcp']"
              label="Type"
              placeholder="Type"
              v-model="form.type"
            ></v-select>
          </v-col>

          <v-col
            cols="12"
            sm="6"
            md="3"
          >
            <v-text-field
              label="Name"
              placeholder="Name"
              v-model="form.name"
            ></v-text-field>
          </v-col>

          <v-col
            cols="12"
            sm="6"
            md="3"
          >
            <v-text-field
              label="ID"
              placeholder="ID"
              v-model="form.id"
            ></v-text-field>
            <p class="mt-n6">Jangan Ada Spasi !</p>
          </v-col>

          <v-col
            cols="12"
            sm="6"
            md="3"
          >
             <section v-if="form.type == 'icmp'">
              <v-select
                :items="['*/1 * * * *', '*/3 * * * *', '*/5 * * * *']"
                label="Schedule"
                placeholder="Schedule"
                v-model="form.schedule"
              ></v-select>
            </section>

            <section v-else>
               <v-select
                :items="['@every 3s', '@every 5s', '@every 7s']"
                label="Schedule"
                placeholder="Schedule"
                v-model="form.schedule"
              ></v-select>
            </section>
          </v-col>
        </v-row>

        <!-- Baris 2 -->
        <v-row>
          <v-col
            cols="12"
            sm="6"
            md="3"
          >
            <v-combobox
              v-model="form.hosts"
              hide-selected
              label="URL / Endpoint"
              placeholder="URL / Endpoint"
              multiple
              persistent-hint
              small-chips
            >
              <template v-slot:no-data>
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title>
                      Press <kbd>enter</kbd> to create a new one
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </template>
            </v-combobox>
          </v-col>

          <v-col
            cols="12"
            sm="6"
            md="3"
          >
            <v-text-field
              label="IPv6"
              placeholder="IPv6"
              v-model="form.ipv6_monitor"
              disabled
            ></v-text-field>
          </v-col>

          <v-col
            cols="12"
            sm="6"
            md="3"
          >
            <v-text-field
              label="IPv4"
              placeholder="IPv4"
              v-model="form.ipv4_monitor"
              disabled
            ></v-text-field>
          </v-col>

          <v-col
            cols="12"
            sm="6"
            md="3"
          >
            <v-text-field
              label="Mode"
              placeholder="Mode"
              v-model="form.mode"
              disabled
            ></v-text-field>
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-actions>
        <v-btn
          color="primary"
          class="mr-4"
          @click="submit"
        >
          Add
        </v-btn>
      </v-card-actions>
    </v-card>

    <v-card
      class="mt-5"
      elevation="2"
    >
      <v-card-title>
        Monitors
        <v-spacer></v-spacer>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          single-line
          hide-details
        ></v-text-field>
      </v-card-title>

      <v-data-table
        :headers="headers"
        :items="dataMonitors"
        :search="search"
        :loading="tableLoading"
        mobile-breakpoint="0"
      >
        <template v-slot:top>
          <v-btn
            color="primary"
            dark
            @click="getData"
            small
            class="ma-2"
          >
            <v-icon>
              mdi-reload
            </v-icon>
          </v-btn>
        </template>

        <template v-slot:item="row">
            <tr>
              <td>{{row.index + 1}}</td>
              <td>{{row.item.id}}</td>
              <td>{{row.item.type}}</td>
              <td>{{row.item.name}}</td>
              <td>{{row.item.schedule}}</td>
              <td>
                <li
                  v-for="(value, key) of row.item.hosts"
                  :key="key"
                >
                  {{value}}
                </li>
              </td>
              <td>
                  <v-btn
                    small
                    class="mx-2"
                    icon
                    color="red" 
                    @click="deleteData(row.item)"
                  >
                    <v-icon dark>mdi-trash-can-outline</v-icon>
                  </v-btn>
              </td>
            </tr>
        </template>
      </v-data-table>

    </v-card>

  </v-container>
</template>

<script>
import { mapActions } from 'vuex'
import axios from 'axios'
import 'vue-sweetalert2';

export default {
  name: 'home',
  data() {
    return {
      api_url: process.env.VUE_APP_API_ENDPOINT,
      form: {
        type: 'http',
        name: '',
        id: '',
        schedule: '',
        hosts: [],
        ipv6_monitor: true,
        ipv4_monitor: true,
        mode: 'any'
      },

      tableLoading: false,
      search: '',
      dataMonitors: [],
      headers: [
        { text: 'No', value: 'no', sortable: false },
        { text: 'ID', value: 'id' },
        { text: 'Type', value: 'type' },
        { text: 'Nama', value: 'name' },
        { text: 'Schedule', value: 'schedule' },
        { text: 'Endpoint', value: 'hosts' },
        { text: 'Actions', value: 'controls', sortable: false },
      ],

      alertObject: {
        type: 'success',
        message: {},
        status: false
      },
    }
  },
  mounted() {
    this.getData()
  },
  methods: {
     ...mapActions({
      setAlert  : 'alert/set',
      setAuth   : 'auth/set',
      setDialog : 'dialog/set'
    }),

    async getData() {
      try {
        
        this.tableLoading = true

        const response = await axios.get(this.api_url + '/coba-ambil')

        this.dataMonitors = response.data.data

        this.tableLoading = false

      } catch (error) {
        this.tableLoading = true
        
        this.setAlert({
          status : true,
          color  : 'error',
          text  : error,
        })
      }
    },

    async submit(e) {
      e.preventDefault()

      try {
        this.setDialog({
          status : true,
        })

        let formData = new FormData()

        formData.append('type', this.form.type)
        formData.append('name', this.form.name)
        formData.append('id', this.form.id)
        formData.append('schedule', this.form.schedule)

        this.form.hosts.forEach(element => {
          formData.append('hosts[]', element)
        });
        //formData.append('hosts[]', this.form.hosts)

        formData.append('ipv6_monitor', this.form.ipv6_monitor)
        formData.append('ipv4_monitor', this.form.ipv4_monitor)
        formData.append('mode', this.form.mode)        

        const response = await axios.post(this.api_url + '/coba-tambah', formData)


        this.hasil = response.data.data

        await this.getData()
        this.resetForm()

        this.setDialog({
          status : false,
        })

        this.setAlert({
          status : true,
          color  : 'success',
          text  : response.data.message,
        })

      } catch (error) {
        this.setDialog({
          status : false,
        })

        if (error.response.status === 406) {
          this.setAlert({
            status : true,
            color  : 'error',
            text  : error.response.data.message,
          })
        } else {
          this.setAlert({
            status : true,
            color  : 'error',
            text  : 'Inputan Tidak Valid !',
          })

          this.alertObject.status = true
          this.alertObject.type = 'error'
          this.alertObject.message = error.response.data
        }

      }
    },

    async deleteData(e) {
      const sw = await this.$swal.fire({
        title: 'Are you sure?',
        text: `Delete "${e.id}" ?`,
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes, delete it!'
      })
      
      if (sw.value) {
        try {
          
          this.setDialog({
            status : true,
          })

          const response = await axios.delete(this.api_url + '/coba-hapus/' + e.id)

          this.setDialog({
            status : false,
          })

          this.getData()

          this.setAlert({
            status : true,
            color  : 'success',
            text  : response.data.message,
          })
          

        } catch (error) {

          this.setDialog({
            status : false,
          })
          
          this.setAlert({
            status : true,
            color  : 'error',
            text  : error.response.data.message,
          })

        }
      } 

    },

    resetForm() {
      this.form.type = ''
      this.form.name = ''
      this.form.id = ''
      this.form.schedule = ''
      this.form.hosts = []
    }
  }
}
</script>

<style>

</style>
