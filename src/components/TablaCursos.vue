<template>
  <v-data-table
    :headers="headers"
    :items="cursos"
    sort-by="clave"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar flat color="white">
        <v-toolbar-title>Cursos</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="#F1C40F"  dark class="mb-2" v-on="on">Agregar Curso</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.clave" label="Clave"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.nombre" label="Nombre"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.creditos" label="Creditos"></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
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
    <template v-slot:no-data>
      <!-- <v-btn color="primary" @click="mostrarEstudiantes">Reset</v-btn> -->
    </template>
  </v-data-table>
</template>

<script>
  export default {
      name:"Cursos-Tabla",
    data: () => ({
      dialog: false,
      headers: [
        {
          text: 'Clave',
          align: 'start',
          sortable: false,
          value: 'clave',
        },
        { text: 'Nombre', value: 'nombre' },
        { text: 'Creditos', value: 'creditos' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      cursos: [],
      editedIndex: -1,
      editedItem: {
        nombre: '',
        creditosCursados: 0
      },
      defaultItem: {
        clave:0,
        nombre: '',
        creditos: '',
      },
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'Nuevo Curso' : 'Editar curso'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
    },

    created () {
      this.mostrarCursos()
    },

    methods: {
       mostrarCursos () {
        this.axios.get("/cursos").then((res)=>{
            this.cursos=res.data;
        }).catch((err)=>{
            console.log(err);
        })
        },

      editItem (item) {
        this.editedIndex = this.cursos.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        const index = this.cursos.indexOf(item)
        let confirmacion=confirm('¿Esta seguro que desea eliminar este curso?') && this.cursos.splice(index, 1)
        if (confirmacion) {
            this.axios.delete(`/cursos/${item.clave}`).then((res) => {
            console.log(res.data);
          }).catch((err) => {
            console.log(err);
          });
        }
      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      save () {//aqui me quede 
        if (this.editedIndex > -1) {
            const estudianteModificar={
                clave:this.editedItem.clave,
                nombre: this.editedItem.nombre,
                creditos: this.editedItem.creditos
            };
            console.log(this.editedItem.clave);
            this.axios.put(`/cursos/`,estudianteModificar).then((res)=>{
                 Object.assign(this.cursos[this.editedIndex], this.editedItem)
                 console.log(res.data);
            }).catch((err)=>{
                console.log(err);
            });
         
        } else {
            this.axios.post("/cursos/", this.editedItem).then((res) => {
            this.cursos.push(this.editedItem);
                console.log(res.data);
            }).catch((err) => {
                console.log(err);
            });
          
          console.log("se fue al else");
        }
        this.close()
      },
    },
  }
</script>