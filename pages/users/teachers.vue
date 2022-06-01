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
      <template v-slot:item.reg_date="{ item }">
        <span>{{ dateTimeFormater(item.reg_date) }}</span>
      </template>
      <template v-slot:top>
        <v-toolbar elevation="0">
          <v-toolbar-title>Teachers</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>

          <v-dialog v-model="dialog" max-width="600px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                @click="dialogAction(null, 'a')"
                color="green darken-2"
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
                    <v-col v-if="dialogType == 'a'" cols="12">
                      <v-text-field
                        v-model="editedItem.auth_id"
                        label="Auth ID"
                        dense
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col
                      v-if="dialogType == 'a'"
                      cols="12"
                      md="6"
                      lg="6"
                      sm="12"
                    >
                      <v-text-field
                        v-model="editedItem.teacher_id"
                        label="Teacher ID"
                        dense
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-text-field
                        v-model="editedItem.name"
                        label="Name"
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
                      <v-select
                        v-model="editedItem.medium"
                        :items="['Sinhala', 'Tamil', 'English']"
                        label="Medium"
                        dense
                        outlined
                      ></v-select>
                    </v-col>
                    <v-col
                      v-if="dialogType == 'e'"
                      cols="12"
                      md="6"
                      lg="6"
                      sm="12"
                    ></v-col>
                    <v-col
                      v-if="dialogType == 'a'"
                      cols="12"
                      md="6"
                      lg="6"
                      sm="12"
                    >
                      <v-text-field
                        v-model="editedItem.email"
                        label="Email"
                        dense
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col
                      v-if="dialogType == 'a'"
                      cols="12"
                      md="6"
                      lg="6"
                      sm="12"
                    >
                      <v-text-field
                        v-model="editedItem.password"
                        label="Password"
                        dense
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-textarea
                        v-model="editedItem.edu_qualifications"
                        label="Education Qualifications"
                        dense
                        outlined
                        class="text-area-max-height"
                        height="110"
                        no-resize
                      ></v-textarea>
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
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-select
                        :items="[true, false]"
                        v-model="editedItem.active"
                        label="Active"
                        dense
                        outlined
                      ></v-select>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
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
    <!-- </v-col> -->
  </div>
</template>

<script>
import { v4 as uuid } from "uuid";
var gradesRef;
var subjectsRef;
var teachersRef;

export default {
  name: "teachers_screen",
  data: () => ({
    dialog: false,
    dialogType: "a",
    loading: false,
    btnLoading: false,
    gradesList: [],
    gradesListData: [],
    subjectsList: [],
    subjectsListData: [],
    search: "",
    headers: [
      {
        text: "Id",
        align: "start",
        sortable: false,
        value: "teacher_id",
      },
      { text: "Name", value: "name" },
      { text: "Mobile No", value: "mobile_no" },
      { text: "Medium", value: "medium" },
      { text: "Email", value: "email" },
      { text: "Education Qualifications", value: "edu_qualifications" },
      { text: "Description", value: "description" },
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
    gradesRef = this.$fire.firestore.collection("grades");
    subjectsRef = this.$fire.firestore.collection("subjects");
    teachersRef = this.$fire.firestore.collection("teachers");
    this.initialize();
  },

  methods: {
    initialize() {
      try {
        this.loading = true;
        teachersRef.onSnapshot((querySnapshot) => {
          this.items = [];
          querySnapshot.docs.forEach((doc) => {
            this.items.push(doc.data());
          });
          this.loading = false;
        });
        gradesRef.onSnapshot((querySnapshot) => {
          this.gradesList = [];
          this.gradesListData = [];
          querySnapshot.docs.forEach((doc) => {
            this.gradesList.push(doc.data()["grade"]);
            this.gradesListData.push(doc.data());
          });
        });
        subjectsRef.onSnapshot((querySnapshot) => {
          this.subjectsList = [];
          this.subjectsListData = [];
          querySnapshot.docs.forEach((doc) => {
            this.subjectsList.push(doc.data()["subject"]);
            this.subjectsListData.push(doc.data());
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
        if (this.editedItem.auth_id == null || this.editedItem.auth_id == "") {
          this.$store.dispatch("alertState/message", [
            "Please enter Auth ID",
            "error",
          ]);
        } else if (
          this.editedItem.teacher_id == null ||
          this.editedItem.teacher_id == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please enter Teacher ID",
            "error",
          ]);
        } else if (this.editedItem.name == null || this.editedItem.name == "") {
          this.$store.dispatch("alertState/message", [
            "Please enter Teacher Name",
            "error",
          ]);
        } else if (
          this.editedItem.mobile_no == null ||
          this.editedItem.mobile_no == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please enter Mobile No",
            "error",
          ]);
        } else if (
          this.editedItem.medium == null ||
          this.editedItem.medium == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please select Medium",
            "error",
          ]);
        } else if (
          this.editedItem.email == null ||
          this.editedItem.email == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please enter Email Address",
            "error",
          ]);
        } else if (
          this.editedItem.password == null ||
          this.editedItem.password == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please enter Password",
            "error",
          ]);
        } else if (
          this.editedItem.edu_qualifications == null ||
          this.editedItem.edu_qualifications == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please enter Education Qualifications",
            "error",
          ]);
        } else if (
          this.editedItem.grade == null ||
          this.editedItem.grade == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please select Grade",
            "error",
          ]);
        } else if (
          this.editedItem.subject == null ||
          this.editedItem.subject == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please select Subject",
            "error",
          ]);
        } else if (
          this.editedItem.active == null ||
          this.editedItem.active == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please select Active Status",
            "error",
          ]);
        } else {
          this.btnLoading = true;

          this.editedItem.teacher_id =
            "T" + this.editedItem.teacher_id.replace("T", "");

          var gradeData = this.gradesListData.find(
            (element) => element.grade == this.editedItem.grade
          );

          var subjectData = this.subjectsListData.find(
            (element) => element.subject == this.editedItem.subject
          );

          teachersRef
            .doc(this.editedItem.auth_id)
            .set({
              auth_id: this.editedItem.auth_id,
              teacher_id: this.editedItem.teacher_id,
              name: this.editedItem.name,
              mobile_no: this.editedItem.mobile_no,
              medium: this.editedItem.medium,
              email: this.editedItem.email,
              password: this.editedItem.password,
              edu_qualifications: this.editedItem.edu_qualifications,
              description: this.editedItem.description ?? "",
              grade_id: gradeData?.id,
              grade: this.editedItem.grade,
              subject: this.editedItem.subject,
              subject_id: subjectData?.id,
              active: this.editedItem.active,
              reg_date: new Date(),
            })
            .then(() => {
              this.$store.dispatch("alertState/message", [
                "Data added successfully.",
                "success",
              ]);
              this.btnLoading = false;
            })
            .catch((error) => {
              this.btnLoading = false;
              this.$store.dispatch("alertState/message", [error, "error"]);
            });
        }
      } catch (error) {
        console.log(error);
        this.btnLoading = false;
      }
    },
    updateData() {
      try {
        if (this.editedItem.name == null || this.editedItem.name == "") {
          this.$store.dispatch("alertState/message", [
            "Please enter Teacher Name",
            "error",
          ]);
        } else if (
          this.editedItem.mobile_no == null ||
          this.editedItem.mobile_no == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please enter Mobile No",
            "error",
          ]);
        } else if (
          this.editedItem.medium == null ||
          this.editedItem.medium == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please select Medium",
            "error",
          ]);
        } else if (
          this.editedItem.edu_qualifications == null ||
          this.editedItem.edu_qualifications == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please enter Education Qualifications",
            "error",
          ]);
        } else if (
          this.editedItem.grade == null ||
          this.editedItem.grade == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please select Grade",
            "error",
          ]);
        } else if (
          this.editedItem.subject == null ||
          this.editedItem.subject == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please select Subject",
            "error",
          ]);
        } else if (
          this.editedItem.active == null ||
          this.editedItem.active == ""
        ) {
          this.$store.dispatch("alertState/message", [
            "Please select Active Status",
            "error",
          ]);
        } else {
          this.btnLoading = true;

          // Find new selected items data
          var gradeData = this.gradesListData.find(
            (element) => element.grade == this.editedItem.grade
          );
          var subjectData = this.subjectsListData.find(
            (element) => element.subject == this.editedItem.subject
          );

          teachersRef
            .doc(this.editedItem.auth_id)
            .update({
              // auth_id: null,
              // teacher_id: id,
              name: this.editedItem.name,
              mobile_no: this.editedItem.mobile_no,
              medium: this.editedItem.medium,
              // email: this.editedItem.email,
              // password: this.editedItem.password,
              edu_qualifications: this.editedItem.edu_qualifications,
              description: this.editedItem.description ?? "",
              grade_id: gradeData?.id,
              grade: this.editedItem.grade,
              subject: this.editedItem.subject,
              subject_id: subjectData?.id,
              active: this.editedItem.active,
              // reg_date: this.editedItem.reg_date,
            })
            .then(async () => {
              await this.videosUpdate(
                "teacher_id",
                this.editedItem.teacher_id,
                "teacher_name",
                this.editedItem.name
              );
              await this.testsUpdate(
                "teacher_id",
                this.editedItem.teacher_id,
                "teacher_name",
                this.editedItem.name
              );
              // await this.liveClassesUpdate(
              //   "teacher_id",
              //   this.editedItem.teacher_id,
              //   "teacher_name",
              //   this.editedItem.name
              // );
              await this.tutesUpdate(
                "teacher_id",
                this.editedItem.teacher_id,
                "teacher_name",
                this.editedItem.name
              );
            })
            .then(() => {
              this.$store.dispatch("alertState/message", [
                "Data updated successfully.",
                "success",
              ]);
              this.btnLoading = false;
            })
            .catch((error) => {
              this.btnLoading = false;
              this.$store.dispatch("alertState/message", [error, "error"]);
            });
        }
      } catch (error) {
        console.log(error);
      }
    },
    deleteData() {
      try {
        this.btnLoading = true;
        teachersRef
          .doc(this.editedItem.auth_id)
          .get()
          .then((snapshot) => {
            snapshot.ref
              .collection("sold_live_classes")
              .get((querySnapshot) => {
                querySnapshot.dosc?.forEach((element) => {
                  element.ref.delete();
                });
              });
            snapshot.ref.collection("sold_tests").get((querySnapshot) => {
              querySnapshot.dosc?.forEach((element) => {
                element.ref.delete();
              });
            });
            snapshot.ref.collection("sold_videos").get((querySnapshot) => {
              querySnapshot.dosc?.forEach((element) => {
                element.ref.delete();
              });
            });
            snapshot.ref.delete();
          })
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