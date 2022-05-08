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
          <v-toolbar-title>Tests</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <v-card>
              <v-card-title class="text-h5"
                >Are you sure you want to delete this item?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="blue darken-1"
                  :disabled="btnLoading"
                  text
                  @click="close"
                  >Cancel</v-btn
                >
                <v-btn
                  color="blue darken-1"
                  :disabled="btnLoading"
                  :loading="btnLoading"
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
        <v-icon small class="mr-2" @click="navigate(item)">
          mdi-note-edit
        </v-icon>
        <v-icon small @click="dialogAction(item)"> mdi-delete </v-icon>
      </template>
      <!-- <template v-slot:no-data>
          <v-btn color="primary" @click="initialize"> Reset </v-btn>
        </template> -->
    </v-data-table>
    <!-- </v-col> -->
  </div>
</template>

<script>
var testsRef;

export default {
  name: "tests_screen",
  data: () => ({
    dialog: false,
    // dialogType: "a",
    loading: false,
    btnLoading: false,
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
      { text: "Teacher ID", value: "teacher_id" },
      { text: "Medium", value: "medium" },
      { text: "Topic", value: "topic" },
      { text: "Price", value: "price" },
      { text: "Description", value: "description" },
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
    testsRef = this.$fire.firestore.collection("tests");
    this.initialize();
  },

  methods: {
    initialize() {
      try {
        this.loading = true;
        testsRef.onSnapshot((querySnapshot) => {
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

    save() {},

    navigate(item) {
      this.$router.push({
        path: "tests/exam",
        query: { id: item.id, topic: item.topic },
      });
    },

    dialogAction(item) {
      this.editedIndex = this.items.indexOf(item);
      this.editedItem = Object.assign({}, item);
      // this.dialogType = type;
      this.dialog = true;
    },
    deleteData() {
      try {
        this.btnLoading = true;
        testsRef
          .doc(this.editedItem.id)
          .collection("questions")
          .get()
          .then((querySnapshot) => {
            querySnapshot.docs?.forEach((snapshot) => {
              snapshot.ref.delete();
            });
          })
          .then(() => {
            testsRef
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
          })
          .catch((e) => {
            console.log(e);
            this.btnLoading = false;
          });
      } catch (error) {
        console.log(error);
        this.btnLoading = false;
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