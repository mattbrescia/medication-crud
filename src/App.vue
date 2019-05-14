<template>
  <v-app id="inspire">
      <v-toolbar flat color="white">
        <v-toolbar-title>Philips Medication Crud</v-toolbar-title>
        <v-divider
          class="mx-2"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <section>
          <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="primary" dark class="mb-2" v-on="on">Add</v-btn>
              <v-btn color="primary" dark class="mb-2" @click="detailsItem">View</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>
  
            <v-card-text> 
              <v-container grid-list-md>
                <v-layout wrap>
                  <v-flex xs12 sm6 md6>
                    <v-text-field v-model="editedItem.name" label="Medication Name"></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md6>
                        <v-select
                          v-model="editedItem.dosage"
                          id="dosage"
                          :items="dosages"
                          label="Dosage"
                          flat
                      ></v-select>
                  </v-flex>
                   <v-flex xs12 sm12 md12>
                    <v-text-field v-model="editedItem.dueDate" label="Due Date"
                     mask="##/##/####" placeholder="dd/mm/yyyy"></v-text-field>
                  </v-flex>
                </v-layout>
              </v-container>
                <v-container grid-list-md>
              <v-layout row wrap>
                  <v-flex xs12 lg6>
                    <v-menu
                      ref="menu1"
                      v-model="menu1"
                      :close-on-content-click="false"
                      :nudge-right="40"
                      lazy
                      transition="scale-transition"
                      offset-y
                      full-width
                      max-width="290px"
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on }">
                      </template>
                      
                    </v-menu>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card-text>
  
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" flat @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        </section>
    <section>
        <v-dialog v-model="dialogView" max-width="500px">
          <template v-slot:activator="{ on }">
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ viewTitle }}</span>
            </v-card-title>
            <v-card-text>
              <v-container grid-list-md>
                <v-layout wrap>
                  <v-flex xs12 sm6 md6>
                    <v-text-field disabled v-model="viewItem.name" label="Medication Name"></v-text-field>
                  </v-flex>
                  <v-flex xs12 sm6 md6>
                       <v-select
                          disabled
                          v-model="viewItem.dosage"
                          id="dosage"
                            :items="dosages"
                            label="Dosage"
                            flat
                        ></v-select>
                  </v-flex>
                   <v-flex xs12 sm12 md12>
                    <v-text-field disabled v-model="viewItem.dueDate" label="Due Date"></v-text-field>
                  </v-flex>
                </v-layout>
              </v-container>
                <v-container grid-list-md>
                  <v-layout row wrap>
                    <v-flex xs12 lg6>
                      <v-menu
                        ref="menu1"
                        v-model="menu1"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        lazy
                        transition="scale-transition"
                        offset-y
                        full-width
                        max-width="290px"
                        min-width="290px"
                      >
                        <template v-slot:activator="{ on }">
                        </template>
                      
                      </v-menu>
                    </v-flex>
                  </v-layout>
              </v-container>
            </v-card-text>
  
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" flat @click="closeView">Cancel</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        </section>
      </v-toolbar>
        <v-data-table
            :headers="headers"
            :items="medications"
            class="elevation-1"
          >
          <template v-slot:items="props">
              <v-checkbox
              @click="view(props.item)"
              v-model="props.selected"
              :value="selected.checked"
              primary
              hide-details
            ></v-checkbox>
              <td>{{ props.item.name }}</td>
              <td class="">{{ props.item.dosage }}</td>
              <td class="">{{ props.item.dueDate }}</td>
              <td class="justify-center layout px-0">
                <v-icon
                  small
                  class="mr-2"
                  @click="editItem(props.item)"
                >
                  edit
                </v-icon>
                <v-icon
                  small
                  @click="deleteItem(props.item)"
                >
                  delete
                </v-icon>
              </td>
           </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="initialize">Reset</v-btn>
        </template>
      </v-data-table>
  </v-app>
</template>

<script>
  export default {
    data: () => ({
      dialog: false,
      dialogView: false,
      selected: {
        checked: false,
      },
       dosages: [
      { text: 'ml' },
      { text: 'tablet' },
      { text: 'pill' }
      ], 
      headers: [
        {
          text: 'Select',
          value: false,
        },
        {
          text: 'New Medication',
          align: 'left',
          sortable: false,
          value: 'name'
        },
        { text: 'Dosage', value: 'ml' },
        { text: 'Due Date', value: '' },
        ],
        menu1: false,
        menu2: false,
        date: new Date().toISOString().substr(0, 10),
        medications: [],

      editedIndex: -1,
      viewItem: {
        name: '',
        dosage: '',
        dueDate: '',
      },
      editedItem: {
        name: '',
        dosage: '',
        dueDate: '',
      },
      defaultItem: {
        name: '',
        dosage: '',
        dueDate: '',
      }
    }),

    computed: {
      viewTitle(){
        return this.editedIndex === this.editedIndex ? 'View Item' : 'Edit Item'
      },

      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      },
      computedDateFormatted () {
        return this.formatDate(this.date)
      }
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
      date () {
        this.dateFormatted = this.formatDate(this.date)
      }
    },

    created () {
      this.initialize()
    },

    methods: {
      initialize () {
        this.medications = [
          {
            name: 'Paracetamol',
            dosage: 'pill',
            dueDate: '25/10/1930',
           },
          {
            name: 'Dorflex',
            dosage: 'pill',
            dueDate: '27/06/2028',
          },
          {
            name: 'Ritalina',
            dosage: 'tablet',
            dueDate: '12/08/2019',
          },
          {
            name: 'Rivotril',
            dosage: 'ml',
            dueDate: '16/05/2019',
          },
        ]
      },

      editItem (item) {
        this.editedIndex = this.medications.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      detailsItem(){
        this.dialogView = true
      },

      deleteItem (item) {
        const index = this.medications.indexOf(item)
        confirm('Are you sure you want to delete this item?') && this.medications.splice(index, 1)
      },

        close () {
          this.dialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
          }, 300)
        },

        closeView(){
          this.dialogView = false;
        },

        view(item) {
          this.editedIndex = this.medications.indexOf(item)
          this.viewItem = Object.assign({}, item)
          this.selected.checked = true;
        },

      save () {
        if (this.editedIndex > -1) {
          Object.assign(this.medications[this.editedIndex], this.editedItem)
        } else {
          this.medications.push(this.editedItem)
        }
        this.close()
        },
       formatDate (date) {
      if (!date) return null

      const [year, month, day] = date.split('-')
      return `${month}/${day}/${year}`
      },
    parseDate (date) {
      if (!date) return null

      const [month, day, year] = date.split('/')
      return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`
      },
    }
  }
</script>

