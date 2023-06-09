<template>
  <div>
    <div class="container">
      <form class="mt-4">
        <div v-for="(field, index) in formFields" :key="index" class="mb-3">
          <label class="form-label">
            {{ field.label }}
            <span v-if="field.required" class="required">*</span>
          </label>
          <template v-if="field.type === 'text'">
            <input
              :type="field.type"
              v-model="formData[field.name]"
              class="form-control"
              :style="field.style"
            />
          </template>
          <template v-else-if="field.type === 'textarea'">
            <textarea
              v-model="formData[field.name]"
              class="form-control"
              :style="field.style"
            ></textarea>
          </template>
          <template v-else-if="field.type === 'number'">
            <input
              v-model="formData[field.name]"
              class="form-control"
              :style="field.style"
              type="number"
            />
          </template>
          <template v-else-if="field.type === 'select'">
            <select v-model="formData[field.name]" class="form-select" :style="field.style">
              <option v-for="option in field.options" :key="option.value" :value="option.value">
                {{ option.label }}
              </option>
            </select>
          </template>
        </div>
        <button @click.prevent="submitForm" class="btn btn-primary">Next</button>
      </form>
    </div>
  </div>
</template>

<script>
import Navbar from '@/components/Navbar.vue'
import Footer from '@/components/Footer.vue'
import axios from 'axios'

const FORM_CONFIGS = {
  datacenterView: [
    { name: 'name', label: 'Datacenter Name', type: 'text', required: true },
    { name: 'notes', label: 'notes', type: 'textarea', required: true },
    { name: 'location', label: 'location', type: 'text', required: true }
  ]
}

export default {
  components: {
    Navbar,
    Footer
  },
  props: {
    idDc: Number
  },
  data() {
    return {
      formData: {},
      formFields: [],
      endpoint: ''
    }
  },
  watch: {
    idDc: {
      immediate: true,
      handler(newIdDc) {
        this.loadData(newIdDc)
      }
    }
  },

  created() {
    const formConfig = FORM_CONFIGS.datacenterView
    this.formFields = formConfig || []

    this.endpoint = 'http://localhost:8080/api/v1/datacenters'
  },

  methods: {
    loadData(idDc) {
      if (idDc !== 0) {
        axios
          .get(`${this.endpoint}/${idDc}`)
          .then((response) => {
            this.formData = response.data
          })
          .catch((error) => {
            console.log(error)
          })
      }
    },
    submitForm() {
      // Check if required fields are empty
      for (const field of this.formFields) {
        if (field.required && !this.formData[field.name]) {
          alert(`${field.label} is required`)
          return
        }
      }

      axios
        .put(`${this.endpoint}/${this.idDc}`, this.formData)
        .then((response) => {
          console.log(response.data)
          alert('Datacenter' + this.formData.name + 'has been updated')
          window.location.reload()
          this.$router.push({ path: '/datacenters' })
        })
        .catch((error) => {
          console.log(error)
        })
    }
  }
}
</script>

<style scoped>
.required {
  color: red;
}
</style>

<style>
.form-label {
  font-weight: bold;
}

.form-control {
  margin-bottom: 1rem;
}

.btn-primary {
  margin-top: 1rem;
}
</style>
