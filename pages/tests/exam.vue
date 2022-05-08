<template>
  <div>
    <h4>Exam : {{ this.$route.query.topic }}</h4>
    <v-row justify="center" class="ma-0 pa-0" dense>
      <exam-card
        v-for="item in items"
        :key="item.id"
        :item="item"
        @deleteFunction="dialogAction(item)"
      />
    </v-row>

    <v-dialog persistent v-model="dialog" max-width="600px">
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
  </div>
</template>

<script>
var testsRef;
var testID;

export default {
  name: "exam_screen",
  data: () => ({
    dialog: false,
    // dialogType: "a",
    loading: false,
    btnLoading: false,
    search: "",
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
    testID = this.$route.query.id;
    testsRef = this.$fire.firestore.collection("tests");
    this.initialize();
  },

  methods: {
    initialize() {
      try {
        this.loading = true;
        testsRef
          .doc(testID)
          .collection("questions")
          .onSnapshot((querySnapshot) => {
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
    dialogAction(item) {
      this.editedIndex = this.items.indexOf(item);
      this.editedItem = Object.assign({}, item);
      //   this.dialogType = type;
      this.dialog = true;
    },

    deleteData() {
      try {
        this.btnLoading = true;
        testsRef
          .doc(testID)
          .collection("questions")
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
