<template>
  <v-app>
    <v-main style="background-color: #eceff1">
      <v-container>
        <v-row>
          <v-col cols="12">
            <h1 class="h1 text-center">
              Extended vuetify date-picker component
            </h1>
          </v-col>
          <v-col cols="12">
            <v-sheet class="ma-5 mx-auto">
              <h4>dates:</h4>
              <pre>{{ dates }}</pre>
            </v-sheet>
          </v-col>
          <v-sheet class="ma-5 mx-auto" elevation="4" rounded>
            <date-picker-extended
              :dates.sync="dates"
              @set-dates="setDates"
            ></date-picker-extended>
          </v-sheet>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
  import moment from 'moment';
  import DatePickerExtended from './components/DatePickerExtended';

  export default {
    name: 'App',
    components: {
      DatePickerExtended,
    },
    data: () => ({
      dates: [moment().format('YYYY-MM-DD'), moment().format('YYYY-MM-DD')],
      menu: false,
    }),

    methods: {
      setDates(dates) {
        this.dates = [...dates].sort(
          (a, b) => moment(a).unix() - moment(b).unix()
        );
      },
    },
  };
</script>
