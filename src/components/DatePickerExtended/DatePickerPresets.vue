<template>
  <div class="datepicker__presets-wrapper">
    <v-btn-toggle
      v-model="btnModel"
      group
      dense
      class="d-flex flex-column justify-space-between h-100 rounded-0"
    >
      <v-btn
        v-for="btn in buttons"
        :key="btn.value"
        :value="btn.value"
        v-text="btn.text"
        small
        text
        color="primary"
        class="my-1"
      ></v-btn>
    </v-btn-toggle>
    <!-- :fab="isXSBreakpoint" -->
    <v-btn
      v-if="isSmAndUpBreakpoint"
      ref="toggler"
      light
      x-small
      icon
      @click.stop="$emit('toggle-presets')"
      class="v-btn-toggle--presets"
    >
      <v-icon dark v-text="togglerIcon"></v-icon>
    </v-btn>
  </div>
</template>

<script>
  import moment from 'moment';

  const DATE_PRESETS_LIST = {
    today: {
      text: 'today',
      value: 'today',
      dates: [moment(), moment()],
    },
    week: {
      text: 'this week',
      value: 'week',
      dates: [moment().startOf('week'), moment()],
    },
    '7days': {
      text: '7 days',
      value: '7days',
      dates: [moment().subtract(6, 'days'), moment()],
    },
    '14days': {
      text: '14 days',
      value: '14days',
      dates: [moment().subtract(13, 'days'), moment()],
    },
    '30days': {
      text: '30 days',
      value: '30days',
      dates: [moment().subtract(29, 'days'), moment()],
    },
    month: {
      text: 'this month',
      value: 'month',
      dates: [moment().startOf('month'), moment()],
    },
  };

  export default {
    name: 'datepicker-presets',
    props: {
      collapsed: Boolean,
    },
    data: () => ({
      btnModel: null,
      buttons: DATE_PRESETS_LIST,
      heightTitle: 0,
      heightBtn: 0,
    }),
    computed: {
      isXSBreakpoint() {
        return this.$vuetify.breakpoint.xs;
      },
      isSmAndUpBreakpoint() {
        return this.$vuetify.breakpoint.smAndUp;
      },
      togglerIcon() {
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
    },
    watch: {
      btnModel(newVal) {
        let dates = DATE_PRESETS_LIST[newVal].dates.map((d) =>
          d.format('YYYY-MM-DD')
        );
        this.$emit('click-preset-btn', dates);
      },
    },
  };
</script>

<style>
  .v-picker--dates-extended.breakpoint-sm-and-up .datepicker__presets-wrapper {
    transition: all 0.3s linear;
    border-right: 1px solid rgba(0, 0, 0, 0.12);
    padding-inline: 12px 26px;
    position: relative;
    width: 180px !important;
    min-width: 180px !important; /* fix */
    height: 280px;
  }

  .v-picker--dates-extended.breakpoint-xs .v-btn-toggle--presets {
    position: absolute !important;
    right: 12px;
    top: 56px;
    z-index: 1;
  }

  .v-picker--dates-extended.breakpoint-sm-and-up .v-btn-toggle--presets {
    position: absolute;
    right: -18px;
    top: 0;
    bottom: 0;
    margin: auto 0;
  }
  .v-picker--dates-extended.breakpoint-xs
    .datepicker__presets-wrapper
    .v-btn-toggle {
    height: 290px !important;
    max-width: 80%;
    margin: 0 auto;
  }

  .v-picker--dates-extended.breakpoint-xs.collapsed
    .datepicker__presets-wrapper {
    margin-top: 0px;
  }

  .v-picker--dates-extended.breakpoint-xs .datepicker__presets-wrapper {
    opacity: 1;
    transition: all 0.3s linear;
  }
</style>
