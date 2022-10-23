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
    @toggle-presets="toggleShowDatesPresetes"
    :presets-shown="shownPresets"
  >
    <template #presets>
      <date-picker-presets
        @click-preset-btn="presetsClicked"
        @toggle-presets="toggleShowDatesPresetes"
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
  import { VBtn } from 'vuetify/lib/components/VBtn/';

  import VIcon from 'vuetify/lib/components/VIcon';
  import { convertToUnit } from 'vuetify/lib/util/helpers';
  import DatePickerPresets from '@/components/DatePickerExtended/DatePickerPresets.vue';
  import moment from 'moment';
  import Vue from 'vue';

  const VPickerExtended = VPicker.extend({
    props: {
      presetsShown: Boolean,
    },
    computed: {
      togglerIcon() {
        let icon = '';
        if (this.isXSBreakpoint) {
          icon = this.presetsShown
            ? 'mdi-arrow-collapse-up'
            : 'mdi-arrow-collapse-down';
        }
        return icon;
      },
      isXSBreakpoint() {
        return this.$vuetify.breakpoint.xs;
      },
    },
    methods: {
      genTogglerIcon() {
        return this.$createElement(VIcon, {}, [this.togglerIcon]);
      },
      geenTogglerBtn() {
        return this.$createElement(
          VBtn,
          {
            staticClass: 'v-btn-toggle--presets',
            props: {
              fab: true,
              'x-small': true,
            },
            on: {
              click: () => this.$emit('toggle-presets'),
            },
          },
          [this.genTogglerIcon()]
        );
      },
      genTitle() {
        const btn = this.isXSBreakpoint ? this.geenTogglerBtn() : null;
        return this.$createElement(
          'div',
          this.setBackgroundColor(this.computedTitleColor, {
            staticClass: 'v-picker__title',
            class: {
              'v-picker__title--landscape': this.landscape,
            },
          }),
          [this.$slots.title, btn]
        );
      },
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
    render(h) {
      return h(
        'div',
        {
          staticClass: 'v-picker v-card',
          class: {
            'v-picker--flat': this.flat,
            'v-picker--landscape': this.landscape,
            'v-picker--full-width': this.fullWidth,
            ...this.themeClasses,
            ...this.elevationClasses,
          },
          on: {
            'toggle-presets': () => this.$emit('toggle-presets'),
          },
        },
        [
          this.$slots.title ? this.genTitle() : null,
          this.genBody(),
          this.$slots.actions ? this.genActions() : null,
        ]
      );
    },
  });

  const DatesPickerComponent = Vue.component('dates-picker-component', {
    extends: VDatePicker,
    props: {
      presetsShown: Boolean, // Add
    },
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
              presetsShown: this.presetsShown, // Add
            },
            on: {
              'toggle-presets': () => this.$emit('toggle-presets'),
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
      shownPresets: true,
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
      toggleShowDatesPresetes() {
        this.shownPresets = !this.shownPresets;
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
    position: relative;
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
