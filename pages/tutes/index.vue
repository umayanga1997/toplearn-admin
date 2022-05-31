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
      <template v-slot:item.tute_link="{ item }">
        <a :href="item.tute_link">Link</a>
      </template>
      <template v-slot:item.cover_img="{ item }">
        <a :href="item.cover_img" target="__blank">
          <img class="pa-2" :src="item.cover_img" width="100" alt="Cover Image"
        /></a>
      </template>
      <template v-slot:top>
        <v-toolbar elevation="0">
          <v-toolbar-title>Tutes</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>

          <v-dialog v-model="dialog" max-width="600px">
            <v-card>
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
        <!-- <v-icon small class="mr-2" @click="dialogAction(item, 'e')">
          mdi-pencil
        </v-icon> -->
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
var tutesRef;

export default {
  name: "tutes_screen",
  data: () => ({
    dialog: false,
    dialogType: "a",
    loading: false,
    btnLoading: false,
    search: "",
    headers: [
      //   {
      //     text: "Id",
      //     align: "start",
      //     sortable: false,
      //     value: "id",
      //   },
      { text: "Grade", value: "grade" },
      { text: "Subject", value: "subject" },
      { text: "Topic", value: "topic" },
      { text: "Description", value: "description" },
      { text: "Price", value: "price" },
      { text: "Teacher ID", value: "teacher_id" },
      { text: "Teacher Name", value: "teacher_name" },
      { text: "Medium", value: "medium" },
      { text: "Tute Link", value: "tute_link" },
      { text: "Cover Img URL", value: "cover_img" },
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
    tutesRef = this.$fire.firestore.collection("tutes");
    this.initialize();
  },

  methods: {
    initialize() {
      try {
        this.loading = true;
        tutesRef.onSnapshot((querySnapshot) => {
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

    deleteData() {
      try {
        this.btnLoading = true;
        tutesRef
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