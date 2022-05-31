<template>
  <div>
    <loading-compo v-if="loading" />
    <v-col v-else cols="12">
      <v-row
        justify="center"
        align="start"
        align-content="start"
        class="ma-0 pa-0"
      >
        <dashoard-card
          headline="Studets"
          :title="'Count : ' + count_of_students"
          :head_color="'yellow--text'"
          :title_color="'white--text'"
          :subtitle_color="'white--text'"
          :bg_color="'indigo  darken-3'"
        ></dashoard-card>
        <dashoard-card
          headline="Teachers"
          :title="'Count : ' + count_of_teachers"
          :head_color="'yellow--text'"
          :title_color="'white--text'"
          :subtitle_color="'white--text'"
          :bg_color="'pink darken-4 darken-3'"
        ></dashoard-card>
        <dashoard-card
          headline="Videos"
          :title="'S.Count : ' + count_of_videos_sale"
          :subtitle="'S.Amount : ' + amount_of_videos_sale.toFixed(2)"
          :head_color="'yellow--text'"
          :title_color="'white--text'"
          :subtitle_color="'white--text'"
          :bg_color="'blue darken-3'"
        ></dashoard-card>
        <dashoard-card
          headline="Tests"
          :title="'S.Count : ' + count_of_tests_sale"
          :subtitle="'S.Amount : ' + amount_of_tests_sale.toFixed(2)"
          :head_color="'yellow--text'"
          :title_color="'white--text'"
          :subtitle_color="'white--text'"
          :bg_color="'purple darken-3'"
        ></dashoard-card>
        <dashoard-card
          headline="Tutes"
          :title="'S.Count : ' + count_of_tutes"
          :subtitle="'S.Amount : ' + amount_of_tutes.toFixed(2)"
          :head_color="'yellow--text'"
          :title_color="'white--text'"
          :subtitle_color="'white--text'"
          :bg_color="'deep-purple darken-3'"
        ></dashoard-card>
        <dashoard-card
          headline="Total of Income"
          :title="'Amount : ' + total_income.toFixed(2)"
          :head_color="'yellow--text'"
          :title_color="'white--text'"
          :subtitle_color="'white--text'"
          :bg_color="'teal darken-3'"
        ></dashoard-card>
      </v-row>
      <br />
      <v-card elevation="4">
        <v-card-title class="orange--text">
          Transaction - Sold Items <v-spacer> </v-spacer>
          <v-col class="pa-0 ma-0" cols="12" lg="4" md="5" xl="4" sm="12">
            <v-text-field
              v-model="search"
              label="Search"
              dense
              outlined
              clearable
            ></v-text-field>
          </v-col>
        </v-card-title>
        <v-data-table
          :headers="headers"
          :items="items"
          :search="search"
          item-key="s_id"
          sort-by="sold_date"
        >
          <template v-slot:item.sold_date="{ item }">
            <span>{{ dateTimeFormater(item.sold_date) }}</span>
          </template>
          <template v-slot:item.exp_date="{ item }">
            <span>{{
              item.exp_date != "" ? dateTimeFormater(item.exp_date) : ""
            }}</span>
          </template>
        </v-data-table>
      </v-card>
    </v-col>
  </div>
</template>

<script>
var teachersRef;
var studentsRef;

export default {
  name: "home_screen",
  data() {
    return {
      loading: false,
      search: "",
      count_of_students: 0,
      count_of_teachers: 0,
      count_of_videos_sale: 0,
      amount_of_videos_sale: 0,
      count_of_tests_sale: 0,
      amount_of_tests_sale: 0,
      count_of_tutes: 0,
      amount_of_tutes: 0,
      total_income: 0,
      headers: [
        // {
        //   text: "Item ID",
        //   value: "id",
        //   align: "start",
        //   sortable: false,
        //   width: 100,
        // },
        { text: "Item Type", value: "type" },
        {
          text: "Teacher ID",
          align: "start",
          sortable: false,
          value: "teacher_id",
        },
        {
          text: "Teacher Name",
          align: "start",
          sortable: false,
          value: "teacher_name",
        },
        {
          text: "Student ID",
          align: "start",
          sortable: false,
          value: "student_id",
        },
        { text: "Student Name", value: "student_name" },
        { text: "Price", value: "price" },
        { text: "Transaction ID", value: "transaction_id" },
        { text: "Sold Date", value: "sold_date" },
        { text: "Expire Date", value: "exp_date" },
      ],
      items: [],
    };
  },
  created() {
    teachersRef = this.$fire.firestore.collection("teachers");
    studentsRef = this.$fire.firestore.collection("students");
    this.initialize();
  },

  methods: {
    initialize() {
      try {
        this.loading = true;
        teachersRef
          .get()
          .then(async (querySnapshotMain) => {
            this.items = [];
            this.count_of_teachers = querySnapshotMain.docs?.length;
            for (const key in querySnapshotMain.docs) {
              await querySnapshotMain.docs[key].ref
                .collection("sold_videos")
                .get()
                .then((querySnapshot) => {
                  querySnapshot.docs?.forEach((element) => {
                    this.count_of_videos_sale += 1;
                    this.amount_of_videos_sale += element.data()["price"] ?? 0;
                    this.items.push({
                      ...element.data(),
                      teacher_id:
                        querySnapshotMain.docs[key].data()["teacher_id"],
                      teacher_name:
                        querySnapshotMain.docs[key].data()["teacher_name"],
                    });
                  });
                });
              await querySnapshotMain.docs[key].ref
                .collection("sold_tests")
                .get()
                .then((querySnapshot) => {
                  querySnapshot.docs?.forEach((element) => {
                    this.count_of_tests_sale += 1;
                    this.amount_of_tests_sale += element.data()["price"] ?? 0;
                    this.items.push({
                      ...element.data(),
                      teacher_id:
                        querySnapshotMain.docs[key].data()["teacher_id"],
                      teacher_name:
                        querySnapshotMain.docs[key].data()["teacher_name"],
                    });
                  });
                });
              await querySnapshotMain.docs[key].ref
                .collection("sold_tutes")
                .get()
                .then((querySnapshot) => {
                  querySnapshot.docs?.forEach((element) => {
                    this.count_of_tutes += 1;
                    this.amount_of_tutes += element.data()["price"] ?? 0;
                    this.items.push({
                      ...element.data(),
                      teacher_id:
                        querySnapshotMain.docs[key].data()["teacher_id"],
                      teacher_name:
                        querySnapshotMain.docs[key].data()["teacher_name"],
                    });
                  });
                });
            }
          })
          .then(() => {
            // Students
            studentsRef
              .get()
              .then((querySnapshot) => {
                this.count_of_students = querySnapshot.docs?.length;
              })
              .then(() => {
                // Calculate full amount
                this.total_income =
                  this.amount_of_videos_sale +
                  this.amount_of_tests_sale +
                  this.amount_of_tutes;
                //
                this.loading = false;
              })
              .catch((error) => {
                this.btnLoading = false;
                this.$store.dispatch("alertState/message", [error, "error"]);
              });
          })
          .catch((error) => {
            this.btnLoading = false;
            this.$store.dispatch("alertState/message", [error, "error"]);
          });
      } catch (error) {
        this.loading = false;
        console.log(error);
      }
    },
  },
};
</script>

<style lang="scss">
</style>