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
    <v-col cols="12" sm="12" md="8" lg="8" class="pa-0 ma-0">
      <v-data-table
        :headers="headers"
        :items="items"
        :loading="loading"
        :search="search"
      >
        <template v-slot:top>
          <v-toolbar elevation="0">
            <v-toolbar-title>Topics</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>

            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  color="green darken-2"
                  @click="dialogAction(null, 'a')"
                  dark
                  v-bind="attrs"
                  v-on="on"
                >
                  New
                </v-btn>
              </template>
              <v-card v-if="dialogType != 'd'">
                <v-card-title>
                  <span class="text-h5">{{ formTitle }}</span>
                </v-card-title>
                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col>
                        <v-text-field
                          v-model="editedItem.topic"
                          label="Topic"
                          dense
                          outlined
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12">
                        <v-textarea
                          v-model="editedItem.description"
                          label="Description"
                          dense
                          outlined
                          class="text-area-max-height"
                          height="110"
                          no-resize
                        ></v-textarea>
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
                    </v-row>
                  </v-container>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                    :disabled="btnLoading"
                    color="blue darken-1"
                    text
                    @click="close"
                  >
                    Cancel
                  </v-btn>
                  <v-btn
                    :disabled="btnLoading"
                    :loading="btnLoading"
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
    </v-col>
  </div>
</template>

<script>
import { v4 as uuid } from "uuid";
var topicsRef;
var gradesRef;
var subjectsRef;

export default {
  name: "grades_screen",
  data: () => ({
    dialog: false,
    dialogType: "a",
    loading: false,
    btnLoading: false,
    gradesList: [],
    subjectsList: [],
    search: "",
    headers: [
      {
        text: "Id",
        align: "start",
        sortable: false,
        value: "id",
      },
      { text: "Grade", value: "grade" },
      { text: "Subject", value: "subject" },
      { text: "Topic", value: "topic" },
      { text: "Description", value: "description" },
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
    gradesRef = this.$fire.firestore.collection("grades");
    subjectsRef = this.$fire.firestore.collection("subjects");
    topicsRef = this.$fire.firestore.collection("topics");
    this.initialize();
  },

  methods: {
    initialize() {
      try {
        this.loading = true;
        topicsRef.onSnapshot((querySnapshot) => {
          this.items = [];
          querySnapshot.docs.forEach((doc) => {
            this.items.push(doc.data());
          });
          this.loading = false;
        });
        gradesRef.onSnapshot((querySnapshot) => {
          this.gradesList = [];
          querySnapshot.docs.forEach((doc) => {
            this.gradesList.push(doc.data()["grade_name"]);
          });
        });
        subjectsRef.onSnapshot((querySnapshot) => {
          this.subjectsList = [];
          querySnapshot.docs.forEach((doc) => {
            this.subjectsList.push(doc.data()["subject"]);
          });
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
        topicsRef
          .doc(id)
          .set({
            id: id,
            grade: this.editedItem.grade,
            subject: this.editedItem.subject,
            topic: this.editedItem.topic,
            description: this.editedItem.description,
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
        topicsRef
          .doc(this.editedItem.id)
          .update({
            grade: this.editedItem.grade,
            subject: this.editedItem.subject,
            topic: this.editedItem.topic,
            description: this.editedItem.description,
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
        topicsRef
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