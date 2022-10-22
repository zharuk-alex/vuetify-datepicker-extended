<template>
  <dates-picker-component
    v-model="localDates"
    range
    :class="{
      'breakpoint-xs': isXSBreakpoint,
      'breakpoint-sm-and-up': isSmAndUpBreakpoint,
      collapsed: shownPresets,
    }"
    @click:date="presetsKey++"
  >
    <template #presets>
      <date-picker-presets
        @click-preset-btn="presetsClicked"
        @click-collapse="toggleShowDatesPresetes"
        :collapsed="shownPresets"
        :key="presetsKey"
      ></date-picker-presets>
    </template>
    <v-spacer></v-spacer>
    <v-btn text color="success" @click="$emit('set-dates', localDates)" icon>
      <v-icon v-text="'mdi-checkbox-marked-circle-outline'"></v-icon>
    </v-btn>
    <v-btn text color="warning" icon>
      <v-icon v-text="'mdi-close-circle-outline'"></v-icon>
    </v-btn>
  </dates-picker-component>
</template>

<script>
  import { VDatePicker } from 'vuetify/lib';
  import { VPicker } from 'vuetify/lib/components/VPicker/';
  import { convertToUnit } from 'vuetify/lib/util/helpers';
  import DatePickerPresets from '@/components/DatePickerExtended/DatePickerPresets.vue';
  import moment from 'moment';
  import Vue from 'vue';

  const VPickerExtended = VPicker.extend({
    methods: {
      genBody() {
        return this.$createElement(
          'div',
          {
            staticClass: 'v-picker__body',
            class: {
              'v-picker__body--no-title': this.noTitle,
              ...this.themeClasses,
            },
            style: this.fullWidth
              ? undefined
              : {
                  width: convertToUnit(this.width),
                },
          },
          [this.genBodyTransition(), this.$slots.presets]
        );
      },
    },
  });

  const DatesPickerComponent = Vue.component('dates-picker-component', {
    extends: VDatePicker,
    computed: {
      isXSBreakpoint() {
        return this.$vuetify.breakpoint.xs;
      },
      isSmAndUpBreakpoint() {
        return this.$vuetify.breakpoint.smAndUp;
      },
      defaultTitleMultipleDateFormatter() {
        return (dates) => {
          let datesSorted = [...dates].sort(
            (a, b) => moment(a).unix() - moment(b).unix()
          );
          let isSameDate = dates[0] === dates[1] && dates.length > 1;
          let datesFormated = datesSorted.map((date) =>
            moment(date).format('dd, D MMM').toLowerCase()
          );
          return isSameDate ? datesFormated[0] : datesFormated.join(' ~ ');
        };
      },
    },
    methods: {
      genPicker(staticClass) {
        const children = [];
        if (!this.noTitle) {
          const title = this.genPickerTitle();
          title && children.push(title);
        }
        const body = this.genPickerBody();

        children.push(
          this.$createElement(
            'template',
            {
              slot: 'presets',
            },
            [this.$slots.presets]
          )
        );
        body && children.push(body);
        children.push(
          this.$createElement(
            'template',
            {
              slot: 'actions',
            },
            [this.genPickerActionsSlot()]
          )
        );
        return this.$createElement(
          VPickerExtended,
          {
            staticClass,
            props: {
              color: this.headerColor || this.color,
              dark: this.dark,
              elevation: this.elevation,
              flat: this.flat,
              fullWidth: this.fullWidth,
              landscape: this.landscape,
              light: this.light,
              width: this.width,
              noTitle: this.noTitle,
            },
          },
          children
        );
      },
    },
    render() {
      return this.genPicker('v-picker--date v-picker--dates-extended');
    },
  });

  export default {
    props: {
      dates: {
        type: Array,
        required: true,
      },
    },
    components: {
      DatesPickerComponent,
      DatePickerPresets,
    },
    data: () => ({
      moment,
      localDates: [],
      shownPresets: false,
      presetsKey: 0,
    }),

    computed: {
      isXSBreakpoint() {
        return this.$vuetify.breakpoint.xs;
      },
      isSmAndUpBreakpoint() {
        return this.$vuetify.breakpoint.smAndUp;
      },
    },

    created() {
      this.localDates = [...this.dates];
    },
    methods: {
      presetsClicked(dates) {
        if (dates != null) {
          this.localDates = dates;
        }
      },
      toggleShowDatesPresetes(val) {
        this.shownPresets = val;
        this.$nextTick(() => {
          this.SetStateToLocalStorage(this.shownPresets);
        });
      },
      SetStateToLocalStorage(state) {
        window.localStorage.setItem('datepicker-extended-collapsed', state);
      },
      GetStateFromLocalStorage() {
        this.shownPresets = JSON.parse(
          window.localStorage.getItem('datepicker-extended-collapsed')
        );
      },
    },
  };
</script>
<style>
  .h-100 {
    height: 100%;
  }

  .v-picker--dates-extended .v-picker__title {
    margin-bottom: 20px;
  }

  .v-picker--dates-extended .v-date-picker-title__date {
    font-size: 20px;
  }

  .v-picker--dates-extended.breakpoint-xs .v-picker__body {
    height: 290px;
  }

  .v-picker--dates-extended .v-picker__body > :first-child:is(div) {
    width: 290px;
    margin-top: 0;
    opacity: 1;
    transition: all 0.3s linear;
  }

  .v-picker--dates-extended.breakpoint-xs.collapsed
    .v-picker__body
    > :first-child:is(div) {
    margin-top: -290px;
    opacity: 0;
  }

  /* Sm-And-Up */
  .v-picker--dates-extended.breakpoint-sm-and-up {
    overflow: hidden;
  }

  .v-picker--dates-extended.breakpoint-sm-and-up .v-picker__body {
    width: 470px !important;
    display: flex;
    flex-direction: row-reverse;
    margin-left: 0px;
    transition: margin-left 0.3s linear;
  }

  .v-picker--dates-extended.breakpoint-sm-and-up.collapsed .v-picker__body {
    margin-left: -180px;
  }
</style>
