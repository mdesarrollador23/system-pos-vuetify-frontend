<template>
  <v-container class="ma-0 pa-0" fluid>
    <v-card>
      <v-data-table
        :headers="headers"
        :items="providers"
        item-key="id"
        :sort-by="['namecompany']"
        :sort-desc="[false, true]"
        multi-sort
        :search="search"
        :loading="tableLoading"
        loading-text="Por favor... espere un momento."
        class="elevation-1"
        dense
        :items-per-page="20"
        :footer-props="{
          itemsPerPageAllText: 'Todo',
          itemsPerPageText: 'Filas por página',
          itemsPerPageOptions: [10, 20, 50, 100, -1],
          pageText: '{0}-{1} de {2}',
        }"
      >
        <template v-slot:top>
          <v-toolbar flat dense>
            <v-toolbar-title>Proveedores</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-dialog
              @keydown.esc="close"
              @keydown.enter="save"
              persistent
              v-model="dialog"
              max-width="900px"
            >
              <template v-slot:activator="{ on, attrs }">
                <div class="d-flex justify-center align-center">
                  <v-text-field
                    class="mx-4"
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Buscar"
                    single-line
                    hide-details
                    solo
                    dense
                  ></v-text-field>
                  <v-tooltip bottom>
                    <template #activator="{ on: onTooltip }">
                      <v-btn
                        class="mx-2"
                        color="success"
                        dark
                        v-bind="attrs"
                        v-on="{ ...on, ...onTooltip }"
                      >
                        <v-icon>mdi-plus-circle</v-icon>
                      </v-btn>
                    </template>
                    <span>Nuevo Proveedor</span>
                  </v-tooltip>
                </div>
              </template>
              <v-card>
                <v-toolbar dark color="primary" dense>
                  <v-btn icon dark @click="close" small>
                    <v-icon>mdi-close</v-icon>
                  </v-btn>
                  <v-toolbar-title v-text="formTitle"></v-toolbar-title>
                  <v-spacer></v-spacer>
                  <v-toolbar-items>
                    <v-btn
                      :disabled="$v.$invalid"
                      :loading="btnLoading"
                      dark
                      text
                      @click="save"
                      small
                    >
                      Guardar
                    </v-btn>
                  </v-toolbar-items>
                </v-toolbar>
                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col cols="12" sm="3" md="3" lg="3" xl="3">
                        <label class="font-weight-bold" for=""
                          >Nombre <span class="error--text">*</span></label
                        >
                        <v-text-field
                          solo
                          required
                          v-model="editedItem.namecompany"
                          :hide-details="hideDetails(namecompanyErrors.length)"
                          :error-messages="namecompanyErrors"
                          @input="$v.editedItem.namecompany.$touch()"
                          @blur="$v.editedItem.namecompany.$touch()"
                          label="Nombre Empresa"
                          type="text"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="3" md="3" lg="3" xl="3">
                        <label class="font-weight-bold" for="">RNC</label>
                        <v-text-field
                          solo
                          v-model="editedItem.rnc"
                          :hide-details="hideDetails(rncErrors.length)"
                          :error-messages="rncErrors"
                          @input="$v.editedItem.rnc.$touch()"
                          @blur="$v.editedItem.rnc.$touch()"
                          v-mask="'###-###-###'"
                          label="999-999-999"
                          type="text"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="3" md="3" lg="3" xl="3">
                        <label class="font-weight-bold" for="">Cedula</label>
                        <v-text-field
                          solo
                          v-model="editedItem.aid"
                          :hide-details="hideDetails(aidErrors.length)"
                          :error-messages="aidErrors"
                          @input="$v.editedItem.aid.$touch()"
                          @blur="$v.editedItem.aid.$touch()"
                          v-mask="'###-#######-#'"
                          label="999-9999999-9"
                          type="text"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="3" md="3" lg="3" xl="3">
                        <label class="font-weight-bold" for="">Correo</label>
                        <v-text-field
                          solo
                          v-model="editedItem.email"
                          :hide-details="hideDetails(emailErrors.length)"
                          :error-messages="emailErrors"
                          @input="$v.editedItem.email.$touch()"
                          @blur="$v.editedItem.email.$touch()"
                          label="Correo"
                          type="email"
                        ></v-text-field>
                      </v-col>

                      <v-col cols="12" sm="3" md="3" lg="3" xl="3">
                        <label class="font-weight-bold" for="">Telefono</label>
                        <v-text-field
                          solo
                          v-model="editedItem.phone1"
                          :hide-details="hideDetails(phone1Errors.length)"
                          :error-messages="phone1Errors"
                          @input="$v.editedItem.phone1.$touch()"
                          @blur="$v.editedItem.phone1.$touch()"
                          v-mask="'(###) ###-####'"
                          label="(999)-999-9999"
                          type="tel"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="3" md="3" lg="3" xl="3">
                        <label class="font-weight-bold" for="">Celuar</label>
                        <v-text-field
                          solo
                          v-model="editedItem.phone2"
                          :hide-details="hideDetails(phone2Errors.length)"
                          :error-messages="phone2Errors"
                          @input="$v.editedItem.phone2.$touch()"
                          @blur="$v.editedItem.phone2.$touch()"
                          v-mask="'(###) ###-####'"
                          label="(999)-999-9999"
                          type="tel"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="6" lg="6" xl="6">
                        <label class="font-weight-bold" for="">Direccion</label>
                        <v-text-field
                          solo
                          v-model="editedItem.address"
                          :hide-details="hideDetails(addressErrors.length)"
                          :error-messages="addressErrors"
                          @input="$v.editedItem.address.$touch()"
                          @blur="$v.editedItem.address.$touch()"
                          label="Direccion"
                          type="text"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="12" md="12" lg="12" xl="12">
                        <label class="font-weight-bold" for=""
                          >Descripcion</label
                        >
                        <v-textarea
                          :hide-details="hideDetails(descriptionErrors.length)"
                          :error-messages="descriptionErrors"
                          @input="$v.editedItem.description.$touch()"
                          @blur="$v.editedItem.description.$touch()"
                          v-model="editedItem.description"
                          solo
                          label="Descripciona"
                          type="text"
                        ></v-textarea>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>
              </v-card>
            </v-dialog>
            <v-dialog
              @keydown.esc="closeDelete"
              @keydown.enter="deleteItemConfirm"
              persistent
              v-model="dialogDelete"
              max-width="600px"
            >
              <v-card>
                <v-toolbar dark color="primary" dense>
                  <v-btn icon dark @click="closeDelete" small>
                    <v-icon>mdi-close</v-icon>
                  </v-btn>
                  <v-toolbar-title
                    >Seguro que desea deshabilitar este
                    registro?</v-toolbar-title
                  >
                  <v-spacer></v-spacer>
                  <v-toolbar-items>
                    <v-btn
                      dark
                      text
                      :loading="btnDLoading"
                      @click="deleteItemConfirm"
                      samall
                    >
                      Save
                    </v-btn>
                  </v-toolbar-items>
                </v-toolbar>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>

        <template v-slot:[`item.options`]="{ item }">
          <v-tooltip top>
            <template v-slot:activator="{ on, attrs }">
              <v-icon
                color="info"
                v-bind="attrs"
                v-on="on"
                small
                class="mr-2"
                @click="editItem(item)"
              >
                mdi-pencil
              </v-icon>
            </template>
            <span>Editar</span>
          </v-tooltip>
          <v-tooltip top>
            <template v-slot:activator="{ on, attrs }">
              <v-icon
                color="error"
                v-bind="attrs"
                v-on="on"
                small
                @click="deleteItem(item)"
              >
                mdi-delete
              </v-icon>
            </template>
            <span>Borrar</span>
          </v-tooltip>
        </template>
        <template v-slot:no-data>
          <v-btn color="tertiary white--text" @click="getProviders">
            Actualizar
          </v-btn>
        </template>
        <template v-slot:no-results> Lo que esta buscando no aparece </template>
      </v-data-table>
    </v-card>
  </v-container>
</template>
<script>
import moment from "moment";
import { validationMixin } from "vuelidate";
import {
  required,
  maxLength,
  minLength,
  email,
  // helpers,
} from "vuelidate/lib/validators";
// const alpha = helpers.regex("alpha", /^[a-zA-Z-0-9- ]*$/);

export default {
  mixins: [validationMixin],
  validations: {
    editedItem: {
      namecompany: {
        required,
        maxLength: maxLength(100),
        minLength: minLength(1),
      },
      rnc: {
        maxLength: maxLength(25),
      },
      aid: {
        maxLength: maxLength(25),
      },
      email: {
        email,
      },
      phone1: {
        maxLength: maxLength(25),
      },
      phone2: {
        maxLength: maxLength(25),
      },
      address: {
        maxLength: maxLength(50),
        // alpha,
      },
      description: {
        maxLength: maxLength(100),
        // alpha,
      },
    },
  },
  name: "Dashboard-Provider",
  data: () => ({
    search: null,
    tableLoading: false,
    btnLoading: false,
    btnDLoading: false,
    headers: [
      {
        text: "Nombre",
        align: "start",
        sortable: true,
        value: "namecompany",
      },
      { text: "RNC", value: "rnc" },
      { text: "Cedula", value: "aid" },
      { text: "Correo", value: "email" },
      { text: "Telefono", value: "phone1" },
      { text: "Telefono 2", value: "phone2" },
      { text: "Direccion", value: "address" },
      // { text: "Descripción", value: "description" },
      // { text: "Fecha de Creacion", value: "createdDate" },
      { text: "Opciones", align: "end", value: "options", sortable: false },
    ],
    providers: [],
    dialog: false,
    dialogDelete: false,
    editedIndex: -1,
    editedItem: {
      namecompany: null,
      description: null,
      rnc: null,
      aid: null,
      email: null,
      phone1: null,
      phone2: null,
      address: null,
    },
    defaultItem: {
      namecompany: null,
      description: null,
      rnc: null,
      aid: null,
      email: null,
      phone1: null,
      phone2: null,
      address: null,
    },
  }),
  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },
  created() {
    this.getProviders();
  },
  methods: {
    editItem(item) {
      this.editedIndex = this.providers.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    deleteItem(item) {
      this.editedIndex = this.providers.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },
    deleteItemConfirm() {
      this.deleteProvider(this.editedItem);
    },
    close() {
      this.$v.$reset();
      this.btnLoading = false;
      this.tableLoading = false;
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },
    closeDelete() {
      this.tableLoading = false;
      this.btnDLoading = false;
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },
    save() {
      if (!this.$v.$invalid) {
        if (this.editedIndex > -1) this.updateProvider(this.editedItem);
        else this.createProvider(this.editedItem);
      }
    },

    async getProviders() {
      this.providers = [];
      this.tableLoading = true;
      await this.axios
        .get("/provider")
        .then((res) => {
          res.data[0].forEach((e) => {
            e.createdDate = moment(e.createdDate).format("yyyy-MM-DD");
            this.providers.push(e);
          });

          this.tableLoading = false;
        })
        .catch((err) => {
          this.tableLoading = false;
          this.btnLoading = false;
          if (err.response.status == 401) {
            this.$swal({
              icon: "error",
              title: "Oops...",
              text: "La sesion expiro, debe iniciar sesión otra vez.",
            });
          } else {
            err.response.status == 500
              ? this.$swal({
                  icon: "error",
                  title: "Oops...",
                  text: `
                Error interno en el servidor,
                Valor: ${err.response.data.message.value}. \n
                path: ${err.response.data.message.path} \n
                `,
                })
              : this.$swal({
                  icon: "error",
                  title: "Oops...",
                  text: err.response.data.message,
                });
          }
        });
    },
    async createProvider(data) {
      this.tableLoading = true;
      this.btnLoading = true;
      this.$v.$touch();
      if (!this.$v.$invalid) {
        await this.axios
          .post("/provider", data)
          .then((res) => {
            res.data[0].forEach((e) => {
              e.createdDate = moment(e.createdDate).format("yyyy-MM-DD");
              this.providers.push(e);
            });

            this.close();
          })
          .catch((err) => {
            this.tableLoading = false;
            this.btnLoading = false;
            if (err.response.status == 401) {
              this.$swal({
                icon: "error",
                title: "Oops...",
                text: "La sesion expiro, debe iniciar sesión otra vez.",
              });
            } else {
              err.response.status == 500
                ? this.$swal({
                    icon: "error",
                    title: "Oops...",
                    text: `
                Error interno en el servidor,
                Valor: ${err.response.data.message.value}. \n
                path: ${err.response.data.message.path} \n
                `,
                  })
                : this.$swal({
                    icon: "error",
                    title: "Oops...",
                    text: err.response.data.message,
                  });
            }
          });
      }
    },
    async updateProvider(data) {
      this.tableLoading = true;
      this.btnLoading = true;
      this.$v.$touch();
      if (!this.$v.$invalid) {
        await this.axios
          .put(`/provider/${data._id}`, data)
          .then((res) => {
            Object.assign(this.providers[this.editedIndex], res.data.item);
            this.close();
          })
          .catch((err) => {
            this.tableLoading = false;
            this.btnLoading = false;
            if (err.response.status == 401) {
              this.$swal({
                icon: "error",
                title: "Oops...",
                text: "La sesion expiro, debe iniciar sesión otra vez.",
              });
            } else {
              err.response.status == 500
                ? this.$swal({
                    icon: "error",
                    title: "Oops...",
                    text: `
                Error interno en el servidor,
                Valor: ${err.response.data.message.value}. \n
                path: ${err.response.data.message.path} \n
                `,
                  })
                : this.$swal({
                    icon: "error",
                    title: "Oops...",
                    text: err.response.data.message,
                  });
            }
          });
      }
    },
    async deleteProvider(data) {
      this.tableLoading = true;
      this.btnDLoading = true;
      await this.axios
        .delete(`/provider/${data._id}`)
        .then(() => {
          this.providers.splice(this.editedIndex, 1);
          this.closeDelete();
        })
        .catch((err) => {
          this.tableLoading = false;
          this.btnDLoading = false;
          if (err.response.status == 401) {
            this.$swal({
              icon: "error",
              title: "Oops...",
              text: "La sesion expiro, debe iniciar sesión otra vez.",
            });
          } else {
            err.response.status == 500
              ? this.$swal({
                  icon: "error",
                  title: "Oops...",
                  text: `
                Error interno en el servidor,
                Valor: ${err.response.data.message.value}. \n
                path: ${err.response.data.message.path} \n
                `,
                })
              : this.$swal({
                  icon: "error",
                  title: "Oops...",
                  text: err.response.data.message,
                });
          }
        });
    },
    hideDetails(val) {
      return val <= 0 ? true : false;
    },
  },
  computed: {
    addressErrors() {
      const errors = [];
      if (!this.$v.editedItem.address.$dirty) return errors;
      !this.$v.editedItem.address.maxLength &&
        errors.push("La direccion debe tener 100 caracteres como máximo.");
      return errors;
    },
    phone2Errors() {
      const errors = [];
      if (!this.$v.editedItem.phone2.$dirty) return errors;
      !this.$v.editedItem.phone2.maxLength &&
        errors.push("El numero debe tener 25 caracteres como máximo.");
      return errors;
    },
    phone1Errors() {
      const errors = [];
      if (!this.$v.editedItem.phone1.$dirty) return errors;
      !this.$v.editedItem.phone1.maxLength &&
        errors.push("El numero debe tener 25 caracteres como máximo.");
      return errors;
    },
    emailErrors() {
      const errors = [];
      if (!this.$v.editedItem.email.$dirty) return errors;
      !this.$v.editedItem.email.email && errors.push("El correo no es valido.");
      return errors;
    },
    aidErrors() {
      const errors = [];
      if (!this.$v.editedItem.aid.$dirty) return errors;
      !this.$v.editedItem.aid.maxLength &&
        errors.push("La cedula debe tener 25 caracteres como máximo.");
      return errors;
    },
    rncErrors() {
      const errors = [];
      if (!this.$v.editedItem.rnc.$dirty) return errors;
      !this.$v.editedItem.rnc.maxLength &&
        errors.push("El RNC debe tener 25 caracteres como máximo.");
      return errors;
    },
    descriptionErrors() {
      const errors = [];
      if (!this.$v.editedItem.description.$dirty) return errors;
      !this.$v.editedItem.description.maxLength &&
        errors.push("La descripcion debe tener 100 caracteres como máximo.");
      return errors;
    },
    namecompanyErrors() {
      const errors = [];
      if (!this.$v.editedItem.namecompany.$dirty) return errors;
      !this.$v.editedItem.namecompany.required &&
        errors.push("El nombre es requerido.");
      !this.$v.editedItem.namecompany.maxLength &&
        errors.push("El nombre debe tener 100 caracteres como máximo.");
      !this.$v.editedItem.namecompany.minLength &&
        errors.push("El nombre debe tener 1 caracteres como minimo.");
      return errors;
    },
    formTitle() {
      return this.editedIndex === -1 ? "Nuevo Proveedor" : "Editar Proveedor";
    },
  },
};
</script>