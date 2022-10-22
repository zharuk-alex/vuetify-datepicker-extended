<template>
  <dates-picker-component
    v-bind="$attrs"
    v-on="$listeners"
    v-model="localDates"
    range
  >
    <template #preselect>
      <preselect-buttons></preselect-buttons>
    </template>
  </dates-picker-component>
</template>

<script>
  import { VDatePicker } from 'vuetify/lib';
  import moment from 'moment';
  import Vue from 'vue';
  const MOMENT_TODAY = moment();

  const DATE_PRESELECTED_LIST = {
    today: {
      text: 'today',
      value: 'today',
      dates: [MOMENT_TODAY, MOMENT_TODAY],
    },
    week: {
      text: 'this week',
      value: 'week',
      dates: [MOMENT_TODAY.startOf('week'), MOMENT_TODAY],
    },
    '7days': {
      text: '7 days',
      value: '7days',
      dates: [MOMENT_TODAY.subtract(6, 'days'), MOMENT_TODAY],
    },
    '14days': {
      text: '14 days',
      value: '14days',
      dates: [MOMENT_TODAY.subtract(13, 'days'), MOMENT_TODAY],
    },
    '30days': {
      text: '30 days',
      value: '30days',
      dates: [MOMENT_TODAY.subtract(29, 'days'), MOMENT_TODAY],
    },
    month: {
      text: 'this month',
      value: 'month',
      dates: [MOMENT_TODAY.startOf('month'), MOMENT_TODAY],
    },
  };

  const PreselectButtons = Vue.component('preselect-buttons', {
    name: 'datepicker-preselect',

    template: `
	<div class="datepicker__preselected-wrapper">
		<v-btn-toggle 
			v-model="btnModel" 
			class="d-flex flex-column justify-space-between h-100 rounded-0 " 
			>
			<v-btn 
				v-for="btn in buttons" 
				:key="btn.value" 
				:value="btn.value" 
				v-text="btn.text"
				small
				color="primary"
				class="my-1 rounded elevation-1"
				></v-btn>
		</v-btn-toggle>
		<v-btn
			v-if="isSmAndUpBreakpoint"
			light
			x-small
			icon
			@click.stop="clickCollapse"
			class="v-btn-toggle--preselect"
		>
			<v-icon dark v-text="collapseIcon"></v-icon>
		</v-btn>
	</div>`,

    data: () => ({
      btnModel: null,
      buttons: DATE_PRESELECTED_LIST,
    }),
    computed: {
      isXSBreakpoint() {
        return this.$vuetify.breakpoint.xs;
      },
      isSmAndUpBreakpoint() {
        return this.$vuetify.breakpoint.smAndUp;
      },
      collapseIcon() {
        let icon = '';
        if (this.isXSBreakpoint) {
          icon = this.collapsed
            ? 'mdi-arrow-collapse-up'
            : 'mdi-arrow-collapse-down';
        } else if (this.isSmAndUpBreakpoint) {
          icon = this.collapsed
            ? 'mdi-arrow-collapse-right'
            : 'mdi-arrow-collapse-left';
        }
        return icon;
      },
      preselectedReset() {
        return this.$parent?.$data?.preselectedReset;
      },
      collapsed() {
        return this.$parent?.$data?.collapsed;
      },
    },
    methods: {
      clickCollapse() {
        this.$emit('click-collapse', !this.collapsed);
      },
    },
    watch: {
      preselectedReset() {
        this.btnModel = null;
      },
      btnModel(newVal) {
        this.$emit('click-preset-btn', newVal);
      },
    },
  });

  const DatesPickerComponent = Vue.component('dates-picker-component', {
    extends: VDatePicker,
    render() {
      return this.genPicker('v-picker--date v-picker--date-extended');
    },
    methods: {
      genPickerBody() {
        const children =
          this.internalActivePicker === 'YEAR'
            ? [this.genYears()]
            : [
                this.$slots.preselect,
                this.genTableHeader(),
                this.internalActivePicker === 'DATE'
                  ? this.genDateTable()
                  : this.genMonthTable(),
              ];
        return this.$createElement(
          'div',
          {
            key: this.internalActivePicker,
          },
          children
        );
      },
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
      PreselectButtons,
    },
    data: () => ({
      moment,
      localDates: [],
    }),
    created() {
      this.localDates = [...this.dates];
    },
  };
</script>
