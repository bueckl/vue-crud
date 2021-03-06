<template>
  <v-card>
    <v-card-text dark class="secondary white--text">
      <h4 style="text-transform:uppercase;">{{ title }}</h4>
    </v-card-text>
    <v-card-text class="details-container">
      <v-layout row wrap>
        <v-flex class="field" :class="field.grid" v-for="(field, i) in fields" :key="i" pr-2>
          <!-- divider -->
          <h3 v-if="field.type == 'divider'" class="text-xs-center section-header">{{ field.text }}</h3>
          <field
            v-else
            :field="field"
            :field-value="field.value"
            :reload="reload"
            @valueChanged="valueChanged"
          ></field>
        </v-flex>
      </v-layout>
    </v-card-text>
  </v-card>
</template>

<script>
import Vue from 'vue'
import Field from './Field.vue'
import {
  mapState,
  mapActions
} from 'vuex'

export default {
  components: {
    Field
  },
  name: 'item-details-extended',
  props: ['fieldsInfo', 'title'],
  data () {
    return {
      files: null,
      masks: {
        date: '####-##-##',
        time: '##:##',
        datetime: '####-##-## ##:##:##'
      },
      reload: false,
      fields: []
    }
  },
  mounted () {
    this.setFields()
  },
  watch: {
    item: {
      handler (val) {
        this.setFields()
        this.reload = true
        setTimeout(() => {
          this.reload = false
        }, 100)
      },
      deep: true
    }
  },
  computed: {
    ...mapState('crud', ['item', 'itemIdColumn', 'detailsDialog']),
    rules () {
      const self = this
      return {
        input: [v => !!v || self.$t('global.details.rules.required')],
        required: v => !!v || self.$t('global.details.rules.required')
      }
    }
  },
  methods: {
    ...mapActions('crud', ['updateItemDetail']),
    setFields () {
      const result = this.fieldsInfo.map((field) => {
        const rField = field
        rField.value = this.item[field.column]
        rField.oldValue = rField.value
        if (field.type === 'select') {
          const defaultVal = field.list.default || ''
          rField.value = (field.stringId
            ? this.item[field.column]
            : parseInt(this.item[field.column])) || defaultVal
        }
        if (field.type === 'checkbox') {
          rField.value = parseInt(this.item[field.column]) === 1
        }
        return rField
      })
      this.$set(this, 'fields', result)
    },
    update (field) {
      const obj = {}
      obj[field.column] = field.value
      this.updateItemDetail([
        this.item[this.itemIdColumn],
        obj,
        this.$t('global.alerts.updated')
      ])
    },
    fieldRules (field) {
      const rules = []
      const required = field.required !== undefined ? field.required : true
      if (required) {
        rules.push(this.rules.required)
      }
      return rules
    },
    remember (field) {
      field.oldValue = field.value
    },
    valueChanged (val, fieldColumn) {
      let field = this.fields[this.fields.findIndex(el => el.column === fieldColumn)]
      let oldVal = field.value
      if (oldVal !== val) {
        this.$set(field, 'value', val)
        Vue.set(this.item, field.column, field.value)
        this.update(field)
      }
    }
  }
}
</script>
<style scoped>
.details-container {
  max-height: calc(100vh - 150px);
  overflow-y: auto;
}
.section-header {
  padding-top: 20px;
  padding-bottom: 10px;
}
.checkbox-label {
  padding-left: 5px;
  color: rgba(0, 0, 0, 0.54);
  font-size: 16px;
}
.field {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
</style>
