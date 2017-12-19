<template>
  <b-container>
    <header class="header clearfix">
      <b-navbar toggleable="md" type="dark" variant="info">
        <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>
        <b-navbar-brand>Risks dashboard</b-navbar-brand>
        <b-collapse is-nav id="nav_collapse">
          <b-navbar-nav>
            <b-nav-item href="#"
                        :key="schema.id"
                        v-bind:id="schema.id"
                        v-for="schema in schemas"
                        v-on:click="switchSchema">
              {{ schema.name }}
            </b-nav-item>
          </b-navbar-nav>
        </b-collapse>
      </b-navbar>
    </header>
    <b-container style="width:80%">
      <b-jumbotron v-if="instanceEmptyState" lead="Choose your risk" > </b-jumbotron>
      <b-jumbotron v-if="schemaEmptyState" lead="There are no risk types defined" > </b-jumbotron>
      <b-form>
        <vue-form-generator :schema="schema" :model="model" :options="formOptions"></vue-form-generator>

      </b-form>
    </b-container>
  </b-container>
</template>

<script>

import Vue from 'vue'
import BootstrapVue from 'bootstrap-vue'
import VueFormGenerator, { abstractField } from 'vue-form-generator'
import axios from 'axios'
import Datepicker from 'vuejs-datepicker'
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'
import 'vue-form-generator/dist/vfg.css'

Vue.use(BootstrapVue)
Vue.use(VueFormGenerator)

Vue.component('fieldDate', {
  mixins: [ abstractField ],
  template: '<datepicker  v-model="value" format="yyyy-MM-dd" :bootstrapStyling="true"></datepicker>',
  components: { Datepicker }
})

const API_URL = process.env.API_URL

export default {
  components: {
    Datepicker
  },
  data () {
    return {
      model: {},
      schemas: [],
      schema: {
        fields: []
      },
      formOptions: {
        validateAfterLoad: true,
        validateAfterChanged: true
      },
      schemaEmptyState: true,
      instanceEmptyState: false
    }
  },
  methods: {
    switchSchema: function (event) {
      let schemaId = parseInt(event.target.parentElement.id)

      axios.get(`${API_URL}/schemas/${schemaId}/`)
      .then(function (response) {
        this.schema.fields = this.getFormFieldDefinitions(response.data.field_definitions)
        this.instanceEmptyState = false
      }.bind(this))
    },
    getFormFieldDefinitions: function (modelFieldDefinitions) {
      return modelFieldDefinitions.map(function (item) {
        var result = {
          type: 'input',
          model: item.name,
          label: item.label,
          required: !item.blank,
          values: item.choices
        }

        if (item.type === 'enum') {
          result.type = 'select'
        }
        if (item.type === 'number') {
          result.validator = 'integer'
          result.inputType = 'number'
        }
        if (item.type === 'date') {
          result.type = 'date'
        }
        return result
      })
    }

  },
  beforeCreate: function () {
    axios.get(`${API_URL}/schemas/`)
      .then(function (response) {
        this.schemas = response.data

        if (response.data.length) {
          this.schemaEmptyState = false
          this.instanceEmptyState = true
        }
      }.bind(this))
  }
}
</script>
