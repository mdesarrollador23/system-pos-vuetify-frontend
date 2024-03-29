<template>
  <v-container class="ma-0 pa-0" fluid>
    <v-card>
      <v-data-table
        :headers="headers"
        :items="users"
        item-key="id"
        :sort-by="['username']"
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
            <v-toolbar-title>Usuarios</v-toolbar-title>
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
                    <span>Nuevo Usuario</span>
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
                      <v-col
                        cols="12"
                        sm="4"
                        md="4"
                        lg="4"
                        xl="4"
                        v-if="editedIndex === -1"
                      >
                        <label class="font-weight-bold" for=""
                          >Nombre de Usuario
                          <span class="error--text">*</span></label
                        >
                        <v-text-field
                          solo
                          required
                          v-model="editedItem.username"
                          :hide-details="hideDetails(usernameErrors.length)"
                          :error-messages="usernameErrors"
                          @input="$v.editedItem.username.$touch()"
                          @blur="$v.editedItem.username.$touch()"
                          v-mask="'NNNNNNNNNNNNNNNNNNNNNNNNN'"
                          label="Nombre de Usuario"
                          type="text"
                        ></v-text-field>
                      </v-col>
                      <v-col
                        cols="12"
                        sm="4"
                        md="4"
                        lg="4"
                        xl="4"
                        v-if="editedIndex === -1"
                      >
                        <label class="font-weight-bold" for=""
                          >Contraseña <span class="error--text">*</span></label
                        >
                        <v-text-field
                          solo
                          required
                          v-model="editedItem.password"
                          :hide-details="hideDetails(passwordErrors.length)"
                          :error-messages="passwordErrors"
                          @input="$v.editedItem.password.$touch()"
                          @blur="$v.editedItem.password.$touch()"
                          v-mask="'NNNNNNNNNNNNNNNNNNNNNNNNN'"
                          label="Contraseña"
                          :type="eyeShow ? 'text' : 'password'"
                          @click:append="eyeShow = !eyeShow"
                          :append-icon="eyeShow ? 'mdi-eye' : 'mdi-eye-off'"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="4" md="4" lg="4" xl="4">
                        <label class="font-weight-bold" for=""
                          >Correo <span class="error--text">*</span></label
                        >
                        <v-text-field
                          solo
                          required
                          v-model="editedItem.email"
                          :hide-details="hideDetails(emailErrors.length)"
                          :error-messages="emailErrors"
                          @input="$v.editedItem.email.$touch()"
                          @blur="$v.editedItem.email.$touch()"
                          label="Correo"
                          type="email"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="4" md="4" lg="4" xl="4">
                        <label class="font-weight-bold" for=""
                          >Nombre Completo <span class="error--text">*</span>
                        </label>
                        <v-text-field
                          solo
                          required
                          v-model="editedItem.fullname"
                          :hide-details="hideDetails(fullnameErrors.length)"
                          :error-messages="fullnameErrors"
                          @input="$v.editedItem.fullname.$touch()"
                          @blur="$v.editedItem.fullname.$touch()"
                          label="Nombre Completo"
                          type="text"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="4" md="4" lg="4" xl="4">
                        <label class="font-weight-bold" for="">Telefono </label>
                        <v-text-field
                          solo
                          v-model="editedItem.phonenumber"
                          hide-details
                          v-mask="'(###) ###-####'"
                          label="(999)-999-9999"
                          type="tel"
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="4" md="4" lg="4" xl="4">
                        <label class="font-weight-bold" for="">Cedula</label>

                        <v-text-field
                          solo
                          v-model="editedItem.aid"
                          hide-details
                          v-mask="'###-#######-#'"
                          label="999-9999999-9"
                          type="text"
                        ></v-text-field>
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
                      small
                    >
                      Guardar
                    </v-btn>
                  </v-toolbar-items>
                </v-toolbar>
              </v-card>
            </v-dialog>
            <v-dialog
              @keydown.esc="closePermission"
              @keydown.enter="savePermission"
              persistent
              v-model="dialogPermission"
              max-width="900px"
              scrollable
            >
              <v-card>
                <v-toolbar dark color="primary" dense>
                  <v-btn icon dark @click="closePermission" small>
                    <v-icon>mdi-close</v-icon>
                  </v-btn>
                  <v-toolbar-title>Editar Permisos</v-toolbar-title>
                  <v-spacer></v-spacer>
                  <v-toolbar-items>
                    <v-btn
                      :disabled="$v.$invalid"
                      :loading="btnLoading"
                      dark
                      text
                      @click="savePermission"
                      small
                    >
                      Guardar
                    </v-btn>
                  </v-toolbar-items>
                </v-toolbar>
                <v-card-text>
                  <v-container>
                    <v-data-table
                      :search="searchPermission"
                      :headers="headersPermission"
                      :items="permissionGet"
                      item-key="_id"
                      :sort-by="['upgrade']"
                      :sort-desc="[true]"
                      class="elevation-1"
                      :loading="tableLoading"
                      loading-text="Por favor... espere un momento."
                      hide-default-footer
                      :items-per-page="255"
                      :footer-props="{
                        itemsPerPageAllText: 'Todo',
                        itemsPerPageText: 'Filas por página',
                        itemsPerPageOptions: [255, -1],
                        pageText: '{0}-{1} de {2}',
                      }"
                      dense
                    >
                      <template v-slot:top>
                        <v-toolbar flat dense>
                          <v-toolbar-title
                            >Permisos de,
                            <strong>
                              {{ editedItem.fullname }}</strong
                            ></v-toolbar-title
                          >
                          <v-divider class="mx-4" inset vertical></v-divider>
                          <v-spacer></v-spacer>
                          <v-text-field
                            class="mx-4"
                            v-model="searchPermission"
                            append-icon="mdi-magnify"
                            label="Buscar descripcion"
                            single-line
                            hide-details
                            solo
                            dense
                          ></v-text-field> </v-toolbar
                      ></template>
                      <template v-slot:[`item.checkbox`]="{ item }">
                        <v-checkbox
                          dense
                          v-model="selected"
                          :value="item.upgrade"
                          color="tertiary"
                        ></v-checkbox>
                      </template>
                      <template v-slot:[`item.permission`]="{ item }">
                        <v-icon
                          v-if="checkPermission(item)"
                          class="success--text"
                          >mdi-check</v-icon
                        >
                        <v-icon v-else class="error--text">mdi-cancel</v-icon>
                      </template>
                      <template v-slot:no-data>
                        <v-btn
                          color="tertiary white--text"
                          @click="getPermission"
                        >
                          Actualizar
                        </v-btn>
                      </template>
                      <template v-slot:no-results>
                        Lo que esta buscando no aparece
                      </template>
                    </v-data-table>
                  </v-container>
                </v-card-text>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>
        <template v-slot:[`item.options`]="{ item }">
          <v-tooltip top>
            <template v-slot:activator="{ on, attrs }">
              <v-icon
                color="success"
                v-bind="attrs"
                v-on="on"
                small
                @click="permission(item)"
                class="mr-2"
              >
                mdi-shield-key
              </v-icon>
            </template>
            <span>Permisos</span>
          </v-tooltip>
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
          <v-btn color="primary" @click="getUsers"> Actualizar </v-btn>
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
  email,
  required,
  maxLength,
  minLength,
  helpers,
} from "vuelidate/lib/validators";
const alpha = helpers.regex("alpha", /^[a-zA-Z-0-9- ]*$/);
export default {
  mixins: [validationMixin],
  validations: {
    editedItem: {
      username: {
        required,
        alpha,
        maxLength: maxLength(25),
        minLength: minLength(3),
      },
      password: {
        required,
        minLength: minLength(6),
      },
      email: {
        required,
        email,
      },
      fullname: {
        required,
        alpha,
        maxLength: maxLength(30),
      },
    },
  },
  name: "Dashboard-Users",
  data: () => ({
    searchPermission: null,
    search: null,
    tableLoading: false,
    btnLoading: false,
    btnDLoading: false,
    singleSelect: false,
    selected: [],
    eyeShow: false,
    headersPermission: [
      {
        text: "",
        value: "checkbox",
      },
      {
        text: "Descripcion",
        align: "start",
        sortable: false,
        value: "description",
      },
      { text: "Permisos", value: "permission" },
    ],
    headers: [
      {
        text: "Nombre de Usuario",
        align: "start",
        value: "username",
      },
      { text: "Correo", value: "email" },
      { text: "Nombre Completo", value: "fullname" },
      { text: "Telefono", value: "phonenumber" },
      { text: "Cedula", value: "aid" },
      { text: "Opciones", align: "end", value: "options", sortable: false },
    ],
    permissionGet: [],
    users: [],
    dialogPermission: false,
    dialog: false,
    dialogDelete: false,
    dialogSaveLoading: false,
    editedIndex: -1,
    editedItem: {
      username: null,
      password: null,
      email: null,
      fullname: null,
      phonenumber: null,
      aid: null,
      claims: [],
    },
    defaultItem: {
      username: null,
      password: null,
      email: null,
      fullname: null,
      phonenumber: null,
      aid: null,
      claims: [],
    },
  }),
  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
    dialogPermission(val) {
      val || this.closePermission();
    },
  },
  created() {
    this.getUsers();
  },
  methods: {
    checkPermission(data) {
      if (this.editedIndex > -1) {
        return this.editedItem.claims.includes(data.upgrade);
      }
    },
    async updatePermission(data) {
      const claims = [];
      this.selected.forEach((e) => {
        claims.push(e);
      });
      await this.axios
        .put(`/users/permission/${data._id}`, { claims: claims })
        .then((res) => {
          Object.assign(this.users[this.editedIndex], {
            claims: res.data[0],
          });
          this.claims = [];
          this.closePermission();
        })
        .catch((err) => {
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
    async getPermission() {
      this.permissionGet = [];
      await this.axios
        .get("/users/permission")
        .then((res) => {
          this.permissionGet = res.data[0];
        })
        .catch((err) => {
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
    async getUsers() {
      this.users = [];
      this.tableLoading = true;
      await this.axios
        .get("/users")
        .then((res) => {
          res.data[0].forEach((e) => {
            e.createdDate = moment(e.createdDate).format("yyyy-MM-DD");
            this.users.push(e);
          });
          this.tableLoading = false;
        })
        .catch((err) => {
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
    async createUsers(data) {
      this.$v.$touch();
      if (!this.$v.$invalid) {
        await this.axios
          .post("/users", data)
          .then((res) => {
            res.data[0].forEach((e) => {
              e.createdDate = moment(e.createdDate).format("yyyy-MM-DD");
              this.users.push(e);
            });
            this.close();
          })
          .catch((err) => {
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
    async updateUsers(data) {
      this.tableLoading = true;
      this.btnLoading = true;
      this.$v.$touch();
      if (!this.$v.$invalid) {
        await this.axios
          .put(`/users/${data._id}`, data)
          .then((res) => {
            Object.assign(this.users[this.editedIndex], res.data.user);
            this.close();
          })
          .catch((err) => {
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
    async deleteUsers(data) {
      this.tableLoading = true;
      this.btnDLoading = true;
      await this.axios
        .delete(`/users/${data._id}`)
        .then(() => {
          this.users.splice(this.editedIndex, 1);
          this.closeDelete();
        })
        .catch((err) => {
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
    save() {
      if (!this.$v.$invalid) {
        if (this.editedIndex > -1) this.updateUsers(this.editedItem);
        else this.createUsers(this.editedItem);
      }
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
    deleteItemConfirm() {
      this.deleteUsers(this.editedItem);
    },
    deleteItem(item) {
      this.editedIndex = this.users.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },
    editItem(item) {
      this.editedIndex = this.users.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    savePermission() {
      this.updatePermission(this.editedItem);
    },
    closePermission() {
      this.selected = [];
      this.btnLoading = false;
      this.tableLoading = false;
      this.dialogPermission = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },
    permission(item) {
      this.getPermission();
      this.editedIndex = this.users.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.selected = this.editedItem.claims;
      this.dialogPermission = true;
    },
  },
  computed: {
    fullnameErrors() {
      const errors = [];
      if (!this.$v.editedItem.fullname.$dirty) return errors;
      !this.$v.editedItem.fullname.required && errors.push("Es requerido.");
      !this.$v.editedItem.fullname.maxLength &&
        errors.push("Solo permite 30 caracteres maximo.");
      !this.$v.editedItem.fullname.alpha &&
        errors.push("Solo es permitido a-z-A-Z-0-9.");
      return errors;
    },
    emailErrors() {
      const errors = [];
      if (!this.$v.editedItem.email.$dirty) return errors;
      !this.$v.editedItem.email.required && errors.push("Es requerido.");
      !this.$v.editedItem.email.email && errors.push("No es correcto.");
      return errors;
    },
    passwordErrors() {
      const errors = [];
      if (!this.$v.editedItem.password.$dirty) return errors;
      !this.$v.editedItem.password.required && errors.push("Es requerido.");
      !this.$v.editedItem.password.minLength &&
        errors.push("Solo permite 6 caracteres como minimo.");
      return errors;
    },
    usernameErrors() {
      const errors = [];
      if (!this.$v.editedItem.username.$dirty) return errors;
      !this.$v.editedItem.username.required && errors.push("Es requerido.");
      !this.$v.editedItem.username.maxLength &&
        errors.push("Solo permite 25 caracteres maximo.");
      !this.$v.editedItem.username.minLength &&
        errors.push("Solo permite 3 caracteres como minimo.");
      !this.$v.editedItem.username.alpha &&
        errors.push("Solo es permitido a-z-A-Z-0-9.");
      return errors;
    },
    formTitle() {
      return this.editedIndex === -1 ? "Nuevo Usuario" : "Editar Usuario";
    },
  },
};
</script>