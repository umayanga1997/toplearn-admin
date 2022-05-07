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
      <template v-slot:item.create_date="{ item }">
        <span>{{ dateTimeFormater(item.create_date) }}</span>
      </template>
      <template v-slot:item.last_update_date="{ item }">
        <span>{{ dateTimeFormater(item.last_update_date) }}</span>
      </template>
      <template v-slot:top>
        <v-toolbar elevation="0">
          <v-toolbar-title>Live Classes</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>

          <v-dialog v-model="dialog" max-width="600px">
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
                    <!-- <v-col cols="12" md="6" lg="6" sm="12">
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
                    </v-col> -->
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-combobox
                        :items="tIdsList"
                        v-model="editedItem.teacher_id"
                        label="Teacher ID"
                        @change="loadTopics"
                        dense
                        outlined
                      ></v-combobox>
                    </v-col>
                    <v-col cols="12" md="6" lg="6" sm="12"> </v-col>
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-menu
                        v-model="dateMenu"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                      >
                        <template v-slot:activator="{ on, attrs }">
                          <v-text-field
                            v-model="editedItem.schedule_date"
                            label="Schedule Date"
                            prepend-icon="mdi-calendar"
                            readonly
                            dense
                            outlined
                            v-bind="attrs"
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          v-model="editedItem.schedule_date"
                          @input="dateMenu = false"
                        ></v-date-picker>
                      </v-menu>
                    </v-col>
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-menu
                        v-model="timeMenu"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                      >
                        <template v-slot:activator="{ on, attrs }">
                          <v-text-field
                            v-model="editedItem.schedule_time"
                            label="Schedule Time"
                            prepend-icon="mdi-clock-time-four-outline"
                            readonly
                            dense
                            outlined
                            v-bind="attrs"
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-time-picker
                          v-model="editedItem.schedule_time"
                          @input="timeMenu = false"
                        ></v-time-picker>
                      </v-menu>
                    </v-col>
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-select
                        :items="topicList"
                        v-model="editedItem.topic"
                        label="Topic"
                        dense
                        outlined
                      ></v-select>
                    </v-col>
                    <v-col cols="12" md="6" lg="6" sm="12">
                      <v-text-field
                        v-model="editedItem.price"
                        label="Price"
                        dense
                        outlined
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="editedItem.link"
                        label="Link"
                        dense
                        outlined
                      ></v-text-field>
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
                  color="blue darken-1"
                  :disabled="btnLoading"
                  :loading="btnLoading"
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
var liveClassRef;
var teachersRef;
var topicsRef;

export default {
  name: "live_class_screen",
  data: () => ({
    dialog: false,
    dialogType: "a",
    loading: false,
    btnLoading: false,
    dateMenu: false,
    timeMenu: false,
    tIdsList: [],
    tDataList: [],
    topicList: [],
    search: "",
    headers: [
      {
        text: "Id",
        align: "start",
        sortable: false,
        value: "id",
      },
      { text: "Grade", value: "grade_name" },
      { text: "Subject", value: "subject" },
      { text: "Topic", value: "topic" },
      { text: "Price", value: "price" },
      { text: "Teacher ID", value: "teacher_id" },
      { text: "Teacher Name", value: "teacher_name" },
      { text: "Medium", value: "medium" },
      { text: "link", value: "link" },
      { text: "Schedule Date", value: "schedule_date" },
      { text: "Schedule Time", value: "schedule_time" },
      { text: "Create Date", value: "create_date" },
      { text: "Last Update Date", value: "last_update_date" },
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
    liveClassRef = this.$fire.firestore.collection("live_classes");
    teachersRef = this.$fire.firestore.collection("teachers");
    topicsRef = this.$fire.firestore.collection("topics");
    this.initialize();
  },

  methods: {
    initialize() {
      try {
        this.loading = true;
        liveClassRef.onSnapshot((querySnapshot) => {
          this.items = [];
          querySnapshot.docs.forEach((doc) => {
            this.items.push(doc.data());
          });
          this.loading = false;
        });
        teachersRef.onSnapshot((querySnapshot) => {
          this.tIdsList = [];
          this.tDataList = [];
          querySnapshot.docs.forEach((doc) => {
            this.tDataList.push(doc.data());
            this.tIdsList.push(doc.data()["teacher_id"]);
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
    loadTopics(data) {
      try {
        var selectedData = this.tDataList.find((value) => {
          return value.teacher_id == data;
        });
        topicsRef
          .where("grade", "==", selectedData.grade)
          .where("subject", "==", selectedData.subject)
          .onSnapshot((querySnapshot) => {
            this.topicList = [];
            querySnapshot.docs.forEach((doc) => {
              this.topicList.push(doc.data()["topic"]);
            });
          });
      } catch (error) {
        console.log(error);
      }
    },
    saveData() {
      try {
        this.btnLoading = true;
        var id = uuid();
        // Get Full details from items list [] using selected stock id
        var selectedData = this.tDataList.find((value) => {
          return value.teacher_id == this.editedItem.teacher_id;
        });

        liveClassRef
          .doc(id)
          .set({
            id: id,
            grade: selectedData.grade,
            subject: selectedData.subject,
            teacher_id: this.editedItem.teacher_id,
            teacher_name: selectedData.name,
            topic: this.editedItem.topic,
            price: Number(this.editedItem?.price),
            medium: selectedData.medium,
            link: this.editedItem.link,
            schedule_date: this.editedItem.schedule_date,
            schedule_time: this.editedItem.schedule_time,
            create_date: new Date(),
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
        // Get Full details from items list [] using selected stock id
        var selectedData = this.tDataList.find((value) => {
          return value.teacher_id == this.editedItem.teacher_id;
        });
        liveClassRef
          .doc(this.editedItem.id)
          .update({
            grade: selectedData.grade,
            subject: selectedData.subject,
            teacher_id: this.editedItem.teacher_id,
            teacher_name: selectedData.name,
            topic: this.editedItem.topic,
            price: Number(this.editedItem?.price),
            medium: selectedData.medium,
            link: this.editedItem.link,
            schedule_date: this.editedItem.schedule_date,
            schedule_time: this.editedItem.schedule_time,
            last_update_date: new Date(),
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
        liveClassRef
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