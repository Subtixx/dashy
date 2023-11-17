<template>
  <div class="code-stats-wrapper">
    <!-- User Info -->
    <div class="user-meta" v-if="basicInfo && !hideMeta">
      <div class="user-info-wrap">
        <p class="username">
          {{ basicInfo.username }}
        </p>
      </div>
    </div>
    <!-- Language Breakdown -->
    <div :id="`languages-${chartId}`" class="language-pie-chart"></div>
    <!-- Machines Percentage -->
    <div :id="`machines-${chartId}`" class="machine-percentage-chart"></div>
  </div>
</template>

<script>
import WidgetMixin from '@/mixins/WidgetMixin';
import ChartingMixin from '@/mixins/ChartingMixin';

export default {
  mixins: [WidgetMixin, ChartingMixin],
  data() {
    return {
      basicInfo: null,
    };
  },
  computed: {
    endpoint() {
      if (!this.options.endpoint) {
        return '';
      }
      if (this.options.endpoint.endsWith('/')) {
        return this.options.endpoint.slice(0, -1);
      }
      return `${this.options.endpoint}`;
    },
    apiKey() {
      if (!this.options.apiKey) {
        return '';
      }

      return btoa(this.options.apiKey);
    },
    user() {
      if (!this.options.user) {
        return '';
      }
      return this.options.user;
    },
    hideMeta() {
      return this.options.hideMeta || false;
    },
    hideLanguages() {
      return this.options.hideLanguages || false;
    },
    hideMachines() {
      return this.options.hideMachines || false;
    },
    monthsToShow() {
      return this.options.monthsToShow || 5;
    },
    chartStartDate() {
      const now = new Date();
      return new Date((now.setMonth(now.getMonth() - this.monthsToShow)));
    },
  },
  methods: {
    fetchData() {
      this.makeRequest(`${this.endpoint}/api/compat/wakatime/v1/users/${this.user}/stats/last_7_days`, {
        Authorization: `Basic ${this.apiKey}`,
      })
        .then((response) => {
          this.processData(response.data);
        })
        .catch((dataFetchError) => {
          this.error('Unable to fetch data from CodeStats.net', dataFetchError);
        })
        .finally(() => {
          this.finishLoading();
        });
    },
    processData(data) {
      // Make basic info data
      if (!this.hideMeta) {
        this.basicInfo = {
          username: data.username,
        };
      }
      // Make language breakdown pie chart data
      if (!this.hideLanguages) {
        const langLabels = [];
        const langTimeValues = [];
        Object.keys(data.languages).forEach((lang) => {
          langLabels.push(data.languages[lang].name);
          langTimeValues.push(data.languages[lang].total_seconds);
        });
        const languagesPieData = {
          labels: langLabels,
          datasets: [{values: langTimeValues}],
        };
        this.drawLanguagePieChart(languagesPieData);
      }
      // Make machine proportion percentage chart data
      if (!this.hideMachines) {
        const machinesLabels = [];
        const machinesTimeValues = [];
        Object.keys(data.machines).forEach((machine) => {
          machinesLabels.push(data.machines[machine].name);
          machinesTimeValues.push(data.machines[machine].total_seconds);
        });
        const machinesPercentageData = {
          labels: machinesLabels,
          datasets: [{values: machinesTimeValues}],
        };
        this.drawMachinesPercentageChart(machinesPercentageData);
      }
    },
    drawLanguagePieChart(languagesPieData) {
      return new this.Chart(`#languages-${this.chartId}`, {
        title: 'Languages',
        type: 'donut',
        data: languagesPieData,
        height: 250,
        strokeWidth: 15,
        tooltipOptions: {
          formatTooltipY: d => this.formatTime(d),
        },
      });
    },
    drawMachinesPercentageChart(machineChartData) {
      return new this.Chart(`#machines-${this.chartId}`, {
        title: 'Machines',
        type: 'percentage',
        data: machineChartData,
        height: 180,
        strokeWidth: 15,
        tooltipOptions: {
          formatTooltipY: d => this.formatTime(d),
        },
        colors: ['#f9c80e', '#43bccd', '#ea3546', '#662e9b', '#f86624'],
      });
    },
    formatTime(d){
      const seconds = d % 60;
      const minutes = Math.floor(d / 60) % 60;
      const hours = Math.floor(d / 3600) % 24;
      const days = Math.floor(d / 86400) % 30;
      const months = Math.floor(d / 2592000) % 12;
      let timeString = '';
      if (months > 0) {
        timeString += `${months}M `;
      }
      if (days > 0) {
        timeString += `${days}d `;
      }
      if (hours > 0) {
        timeString += `${hours}h `;
      }
      if (minutes > 0) {
        timeString += `${minutes}m `;
      }
      if (seconds > 0) {
        timeString += `${seconds}s`;
      }
      return timeString;
    }
  },
};
</script>

<style scoped lang="scss">
.code-stats-wrapper {
  p {
    margin: 0;
    font-size: 1rem;
    color: var(--widget-text-color);
  }

  .user-meta {
    display: flex;
    margin: 0.5rem;
    padding: 0.5rem 0;
    border-bottom: 1px dashed var(--widget-text-color);
    justify-content: space-between;

    .user-info-wrap {
      .username {
        font-size: 1.4rem;
        text-transform: capitalize;
      }

      .user-level {
        font-size: 0.8rem;
        text-transform: capitalize;
        opacity: var(--dimming-factor);
        color: var(--widget-text-color);
      }
    }
  }

  .language-pie-chart,
  .machine-percentage-chart {
    &:not(:last-child) {
      border-bottom: 1px dashed var(--widget-text-color);
    }
  }
}
</style>
