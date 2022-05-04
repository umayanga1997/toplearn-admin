<template>
  <div>
    <div class="search-container">
      <v-col cols="12" lg="4" md="4" sm="6" class="ma-0 pa-0 mb-2 mt-2">
        <v-text-field
          v-model="search"
          dense
          label="Search"
          outlined
          clearable
        ></v-text-field>
      </v-col>
    </div>
    <!-- <v-col cols="12" sm="12" md="4" lg="4" class="pa-0 ma-0"> -->
    <v-data-table
      :headers="headers"
      :items="items"
      :loading="loading"
      :search="search"
    >
      <template v-slot:top>
        <v-toolbar elevation="0">
          <v-toolbar-title>Students</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>

          <v-dialog v-model="dialog" max-width="600px">
            <!-- <template v-slot:activator="{ on, attrs }">
              <v-btn color="green darken-2" dark v-bind="attrs" v-on="on">
                New
              </v-btn>
            </template> -->
            <v-card v-if="dialogType != 'd'">
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-select
                        :items="[true, false]"
                        v-model="editedItem.active"
                        label="Active"
                        dense
                        outlined
                      ></v-select>
                    </v-col>
                    <!-- <v-col cols="12" md="6" lg="6" sm="12">
                      <v-text-field
                        v-model="editedItem.name_of_trustee"
                        label="Name of Trustee"
                        dense
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-text-field
                        v-model="editedItem.mobile_no"
                        label="Mobile No"
                        dense
                        outlined
                      ></v-text-field>
                    </v-col>

                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-text-field
                        v-model="editedItem.email"
                        label="Email"
                        dense
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-combobox
                        :items="gradesList"
                        v-model="editedItem.grade"
                        label="Grade"
                        dense
                        outlined
                      ></v-combobox>
                    </v-col>
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-combobox
                        :items="subjectsList"
                        v-model="editedItem.subject"
                        label="Subject"
                        dense
                        outlined
                      ></v-combobox>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.link"
                        label="Link"
                        dense
                        outlined
                      ></v-text-field>
                    </v-col> -->
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
                  Cancel
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="dialogType == 'a' ? saveData() : updateData()"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
            <v-card v-else>
              <v-card-title class="text-h5"
                >Are you sure you want to delete this item?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  :disabled="btnLoading"
                  color="blue darken-1"
                  text
                  @click="close"
                  >Cancel</v-btn
                >
                <v-btn
                  :disabled="btnLoading"
                  :loading="btnLoading"
                  color="blue darken-1"
                  text
                  @click="deleteData()"
                  >OK</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon small class="mr-2" @click="dialogAction(item, 'e')">
          mdi-pencil
        </v-icon>
        <v-icon small @click="dialogAction(item, 'd')"> mdi-delete </v-icon>
      </template>
      <!-- <template v-slot:no-data>
          <v-btn color="primary" @click="initialize"> Reset </v-btn>
        </template> -->
    </v-data-table>
    <!-- </v-col> -->
  </div>
</template>

<script>
import { v4 as uuid } from "uuid";
var studentsRef;

export default {
  name: "live_class_screen",
  data: () => ({
    dialog: false,
    dialogType: "a",
    loading: false,
    btnLoading: false,
    // gradesList: [],
    // subjectsList: [],
    tIdsList: [],
    search: "",
    headers: [
      {
        text: "Id",
        align: "start",
        sortable: false,
        value: "student_id",
      },
      { text: "Name of Student", value: "name" },
      { text: "Name of Trustee", value: "name_of_trustee" },
      { text: "Mobile No", value: "mobile_no" },
      { text: "Email", value: "email" },
      { text: "Grade", value: "grade" },
      { text: "Subject", value: "subject" },
      { text: "Active", value: "active" },
      { text: "Register Date", value: "reg_date" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    items: [],
    editedItem: [],
    editedIndex: -1,
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New" : "Edit";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
  },

  created() {
    studentsRef = this.$fire.firestore.collection("students");
    this.initialize();
  },

  methods: {
    initialize() {
      try {
        this.loading = true;
        studentsRef.onSnapshot((querySnapshot) => {
          this.items = [];
          querySnapshot.docs.forEach((doc) => {
            this.items.push(doc.data());
          });
          this.loading = false;
        });
      } catch (error) {
        console.log(error);
        this.loading = false;
      }
    },
    dialogAction(item, type) {
      this.editedIndex = this.items.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogType = type;
      this.dialog = true;
    },
    saveData() {
      try {
        this.btnLoading = true;
        var id = uuid();
        studentsRef
          .doc(id)
          .set({
            auth_id: null,
            student_id: id,
            name: this.editedItem.name,
            name_of_trustee: this.editedItem.name_of_trustee,
            mobile_no: this.editedItem.mobile_no,
            email: this.editedItem.email,
            password: this.editedItem.password,
            grade: this.editedItem.grade,
            subject: this.editedItem.subject,
            active: this.editedItem.active,
            reg_date: this.editedItem.reg_date,
          })
          .then(() => {
            this.$store.dispatch("alertState/message", [
              "Data added successfully.",
              "success",
            ]);
            this.btnLoading = false;
          });
      } catch (error) {
        console.log(error);
      }
    },
    updateData() {
      try {
        this.btnLoading = true;
        studentsRef
          .doc(this.editedItem.id)
          .update({
            // auth_id: null,
            // student_id: id,
            // name: this.editedItem.name,
            // name_of_trustee: this.editedItem.name_of_trustee,
            // mobile_no: this.editedItem.mobile_no,
            // email: this.editedItem.email,
            // password: this.editedItem.password,
            // grade: this.editedItem.grade,
            // subject: this.editedItem.subject,
            active: this.editedItem.active,
            // reg_date: this.editedItem.reg_date,
          })
          .then(() => {
            this.$store.dispatch("alertState/message", [
              "Data updated successfully.",
              "success",
            ]);
            this.btnLoading = false;
          });
      } catch (error) {
        console.log(error);
      }
    },
    deleteData() {
      try {
        this.btnLoading = true;
        studentsRef
          .doc(this.editedItem.id)
          .delete()
          .then(() => {
            this.$store.dispatch("alertState/message", [
              "Data deleted successfully.",
              "success",
            ]);
            this.btnLoading = false;
            this.close();
          });
      } catch (error) {
        console.log(error);
      }
    },
    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
      });
    },
  },
};
</script>

<style lang="scss">
</style>