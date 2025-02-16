<template>
  <v-container fluid>
    <v-row>
      <v-col cols="12">
        <h2 class="mb-4">Courses Management</h2>
        <v-card outlined>
          <v-toolbar flat>
            <v-btn
              color="primary"
              @click="handleCreateItem"
            >
              <v-icon>mdi-plus</v-icon> Add
            </v-btn>
          </v-toolbar>
          <v-card-text>
            <v-data-table
              v-model="selected"
              :headers="headers"
              :items="tableData"
              :single-select="false"
              :options="tableOptions"
              item-key="id"
              :show-select="false"
              :mobile-breakpoint="1023"
              class="elevation-1 mt-4"
              @click:row="handleRowClick"
              @item-selected="handleSelectItem"
              @toggle-select-all="handleSelectItemAll"
            >
              <template v-slot:top>
                <v-dialog
                  v-model="dialog.new"
                  max-width="50vw"
                  max-height="70vh"
                  persistent
                >
                  <v-card>
                    <v-card-title>
                      <span class="headline">{{ form.id === '' ? dialog.newTitle : dialog.editTitle }}</span>
                    </v-card-title>
                    <v-card-text>
                      <v-container>
                        <v-row>
                          <v-col
                            cols="12"
                          >
                            <v-text-field
                              v-model="form.name"
                              label="Name"
                              counter="255"
                            ></v-text-field>
                          </v-col>
                          <v-col
                            cols="12"
                          >
                            <v-text-field
                              v-model="form.memo"
                              label="Memo"
                              counter="255"
                            ></v-text-field>
                          </v-col>
                          <v-col
                            cols="12"
                          >
                            <v-select
                              v-model="form.includedMenus"
                              :items="selectMenus"
                              label="Included Menus"
                              multiple
                            ></v-select>
                          </v-col>
                          <v-col
                            cols="12"
                            md="6"
                          >
                            Select Consisted Inventories
                            <v-select
                              v-model="form.inventory.current"
                              :items="selectInventories"
                              label="Inventory Item"
                            >
                            </v-select>
                          </v-col>
                          <v-col
                            cols="12"
                            md="4"
                            class="mt-sm-6"
                          >
                             <v-text-field
                              v-model="form.inventory.quantity"
                              label="Quantity"
                              :disabled="form.inventory.current === ''"
                              suffix="unit"
                            ></v-text-field>
                          </v-col>
                          <v-col
                            cols="12"
                            md="2"
                            class="mt-md-10"
                          >
                            <v-btn color="primary" @click="handleAddConsistedInventory" :disabled="form.inventory.current === ''">Add</v-btn>
                          </v-col>
                          <v-col
                            cols="12"
                          >
                            <v-data-table
                              :headers="[{ text: 'Item', value: 'text' }, { text: 'Quantity', value: 'quantity' }]"
                              :items="form.consistedInventories"
                              :disable-pagination="true"
                              :disable-sort="true"
                              :disable-filtering="true"
                              :hide-default-footer="true"
                            >
                            </v-data-table>
                          </v-col>
                        </v-row>
                      </v-container>
                    </v-card-text>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn
                        color="blue darken-1"
                        text
                        @click="handleCreateCancel"
                      >
                        Cancel
                      </v-btn>
                      <v-btn
                        color="blue darken-1"
                        text
                        @click="handleCreateSave"
                      >
                        Save
                      </v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
                <v-dialog v-model="dialog.delete" max-width="50vw">
                  <v-card>
                    <v-card-title class="headline">{{ dialog.deleteConfimationText }}</v-card-title>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn color="blue darken-1" text @click="handleDeleteCancel">Cancel</v-btn>
                      <v-btn color="blue darken-1" text @click="handleDeleteConfirm">Confirm</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
                <v-dialog v-model="dialog.pause" max-width="50vw">
                  <v-card>
                    <v-card-title class="headline">{{ dialog.pauseConfimationText }}</v-card-title>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn color="blue darken-1" text @click="handlePauseCancel">Cancel</v-btn>
                      <v-btn color="blue darken-1" text @click="handlePauseConfirm">Confirm</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
              </template>
              <template v-slot:item.misc="{ item }">
                <v-icon
                  small
                  @click.stop="handlePauseItem(item)"
                >
                  {{ item.active ? 'mdi-pause' : 'mdi-play' }}
                </v-icon>
                <v-icon
                  small
                  @click.stop="handleDeleteItem(item)"
                >
                  mdi-delete
                </v-icon>
              </template>
            </v-data-table>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'nuxt-property-decorator'
import { DateFactory } from '~/utils/date'
import { sysStore } from '~/store'

@Component({
  layout: 'default'
})
export default class MenusEdit extends Vue {
  private headers: Array<any> = [
    { text: 'Name', value: 'name', align: 'start', sortable: true },
    { text: 'Memo', value: 'memo', align: 'start', sortable: false },
    { text: '', value: 'misc', align: 'start', sortable: false }
  ]

  private get tableData(): Array<any> {
    return [
      {
        id: '123',
        name: 'Cooked Porked 001',
        memo: '',
        active: true,
        consistedInventories: [
          {
            text: 'Pork 1',
            value: '1',
            quantity: 5
          }
        ],
        includedMenus: [
          '1', '2'
        ],
        date: new DateFactory(new Date()).yymmddhhmmss()
      }
    ]
  }

  private tableOptions: any = {
      page: 1,
      itemsPerPage: 10,
      sortBy: ['date'],
      sortDesc: [true],
      // groupBy: string[],
      // groupDesc: boolean[],
      multiSort: true,
      mustSort: false
  }

  private selected: Array<any> = []

  private singleSelect: boolean = false

  private selectInventories: Array<any> = [
    {
      text: 'Pork 1',
      value: '1'
    },
    {
      text: 'Steak 1',
      value: '2'
    },
    {
      text: 'Pork 2',
      value: '3'
    }
  ]

  private selectMenus: Array<any> = [
    {
      text: 'Menu 1',
      value: '1'
    },
    {
      text: 'Menu 2',
      value: '2'
    },
    {
      text: 'Menu 3',
      value: '3'
    }
  ]

  private form: any = {
    id: '',
    name: '',
    consistedInventories: [],
    includedMenus: '',
    memo: '',
    inventory: {
      current: '',
      quantity: 1
    }
  }

  private dialog: any = {
    deleteConfimationText: 'Are you sure you want to delete this course?',
    delete: false,
    pauseConfimationText: 'Are you sure you want to hide this course from all menus?',
    pause: false,
    edit: false,
    editTitle: 'Edit Course',
    new: false,
    newTitle: 'Create Course',
    error: false,
    errorTitle: 'Server Error. Please try again later.'
  }

  private handleAddConsistedInventory(): void {
    this.form.consistedInventories = [
      ...this.form.consistedInventories,
      {
        text: this.selectInventories.find(item => item.value === this.form.inventory.current).text,
        value: this.form.inventory.current,
        quantity: this.form.inventory.quantity
      }
    ]
    this.form.inventory = {
      current: '',
      quantity: 1
    }
  }

  private handleSelectItem(obj: any): void {
    const { value, item } = obj
    if(value === true) {
      this.selected = [...item]
    } else {
      this.selected = []
    }
  }

  private handleSelectItemAll(obj: any): void {
    const { value, items } = obj
    if(value === true) {
      this.selected = [...items]
    } else {
      this.selected = []
    }
  }

  private handleRowClick(row: any, col: any): void {
    this.handleEditItem(row)
  }

  private clearForm(): void {
    this.form = {
      id: '',
      name: '',
      consistedInventories: [],
      includedMenus: '',
      memo: '',
      inventory: {
        current: '',
        quantity: 1
      }
    }
  }

  private handleCreateItem(): void {
    this.dialog.new = true
  }

  private handleEditItem(item: any): void {
    const { id, name, memo, includedMenus, consistedInventories } = item
    this.form = {
      ...this.form,
      id,
      name,
      memo,
      includedMenus,
      consistedInventories
    }
    this.dialog.new = true
  }

  private handlePauseItem(item: any): void {
    const { id, name, memo, includedMenus, consistedInventories } = item
    this.form = {
      ...this.form,
      id,
      name,
      memo,
      includedMenus,
      consistedInventories
    }
    this.dialog.pause = true
  }

  private handleDeleteItem(item: any): void {
    const { id, name, memo, includedMenus, consistedInventories } = item
    this.form = {
      ...this.form,
      id,
      name,
      memo,
      includedMenus,
      consistedInventories
    }
    this.dialog.delete = true
  }

  private handleCreateCancel(): void {
    this.dialog.new = false
    this.clearForm()
  }

  private handleCreateSave(): void {
    this.dialog.new = false
    this.clearForm()
  }

  private handlePauseConfirm(): void {
    this.dialog.pause = false
    this.clearForm()
  }

  private handlePauseCancel(): void {
    this.dialog.pause = false
    this.clearForm()
  }

  private handleDeleteConfirm(): void {
    this.dialog.delete = false
    this.clearForm()
  }

  private handleDeleteCancel(): void {
    this.dialog.delete = false
    this.clearForm()
  }

  private handleEditCancel(): void {
    this.dialog.new = false
    this.clearForm()
  }

  private handleEditSave(): void {
    this.dialog.new = false
    this.clearForm()
  }

  private async created() {
  }
}
</script>
