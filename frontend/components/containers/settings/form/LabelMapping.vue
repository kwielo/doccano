<template>
  <v-data-table
    :headers="headers"
    :items="value"
  >
    <template v-slot:top>
      <v-dialog
        v-model="dialog"
        max-width="800px"
      >
        <template v-slot:activator="{ on, attrs }">
          <v-btn
            color="primary"
            dark
            class="text-none"
            v-bind="attrs"
            v-on="on"
          >
            Add
          </v-btn>
        </template>
        <v-card>
          <v-card-title>
            <span class="headline">Add a new mapping</span>
          </v-card-title>

          <v-card-text>
            <v-container>
              <v-form
                ref="form"
                v-model="valid"
              >
                <v-row>
                  <v-col
                    cols="12"
                    sm="12"
                    class="pa-0"
                  >
                    <v-text-field
                      v-model="editedItem.from"
                      label="From"
                      :rules="labelNameRules($t('rules.labelNameRules'))"
                      outlined
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    sm="12"
                    class="pa-0"
                  >
                    <v-select
                      v-model="editedItem.to"
                      :items="items"
                      :rules="labelNameRules($t('rules.labelNameRules'))"
                      label="To"
                      outlined
                    />
                  </v-col>
                </v-row>
              </v-form>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer />
            <v-btn
              color="blue darken-1"
              class="text-capitalize"
              text
              @click="close"
            >
              Cancel
            </v-btn>
            <v-btn
              :disabled="!valid"
              color="blue darken-1"
              class="text-capitalize"
              text
              @click="save"
            >
              Save
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </template>
    <template v-slot:[`item.actions`]="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
  </v-data-table>
</template>

<script lang="ts">
import Vue from 'vue'
import { FromApiLabelItemListRepository } from '@/repositories/label/api'
import { LabelApplicationService } from '@/services/application/label.service'
import { labelNameRules } from '@/rules/index'

export default Vue.extend({

  props: {
    value: {
      type: Array,
      default: () => [],
      required: true
    }
  },
  data() {
    return {
      dialog: false,
      headers: [
        {
          text: 'From',
          align: 'left',
          value: 'from',
          sortable: false
        },
        {
          text: 'To',
          align: 'left',
          value: 'to',
          sortable: false
        },
        {
          text: 'Actions',
          value: 'actions',
          sortable: false
        }
      ],
      valid: false,
      editedIndex: -1,
      editedItem: {
        'from': '',
        'to': ''
      },
      defaultItem: {
        'from': '',
        'to': ''
      },
      items: [] as string[],
      labelNameRules
    }
  },

  computed: {
    labelService(): LabelApplicationService {
      const repository = new FromApiLabelItemListRepository()
      const service = new LabelApplicationService(repository)
      return service
    }
  },

  async created() {
    const labels = await this.labelService.list(this.$route.params.id)
    this.items = labels.nameList
  },

  methods: {
    editItem(item: {'from': string, 'to': string}) {
      this.editedIndex = this.value.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item: {'from': string, 'to': string}) {
      this.editedIndex = this.value.indexOf(item)
      this.editedItem = Object.assign({}, item)
      const items = Object.assign([], this.value)
      items.splice(this.editedIndex, 1)
      this.editedItem = Object.assign({}, this.defaultItem)
      this.editedIndex = -1
      this.$emit('input', items)
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save() {
      const items = Object.assign([], this.value)
      if (this.editedIndex > -1) {
        Object.assign(items[this.editedIndex], this.editedItem)
      } else {
        items.push(this.editedItem)
      }
      this.$emit('input', items)
      this.close()
    }
  }
})
</script>
