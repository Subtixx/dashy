<template>
  <div v-if="this.services" class="uptime-kuma-wrapper">
    <div class="item-wrapper wrap-size-medium span-2"
         v-for="service in this.services" v-bind:key="service.name">
        <div class="item size-medium">
        <div class="tile-title">
          <span class="text">{{ service.name }}</span>
        </div>
        <div class="bounce item-icon wrapper-medium">
          <i v-if="service.status === '1'"
             class="fas fa-check-circle fa-2x" style="color: var(--success);"></i>
          <i v-else class="fas fa-times-circle fa-2x" style="color: var(--danger);"></i>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import WidgetMixin from '@/mixins/WidgetMixin';

export default {
  mixins: [WidgetMixin],
  data() {
    return {
      services: {},
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

      const apiKey = `:${this.options.apiKey}`;
      return btoa(apiKey);
    },
  },
  methods: {
    fetchData() {
      this.makeRequest(
        `${this.endpoint}/metrics`,
        {
          Authorization: `Basic ${this.apiKey}`,
        },
      ).then(this.processData);
    },
    processData(data) {
      this.services = {};
      const lines = data.split('\n');

      for (let i = 0; i < lines.length; i += 1) {
        if (lines[i].startsWith('monitor_response_time')) {
          const responseTime = this.parseResponseTime(lines[i]);

          if (!this.services[responseTime.name]) {
            this.services[responseTime.name] = responseTime;
          } else {
            // in ms
            this.services[responseTime.name].responseTime = responseTime.value;
          }
        } else if (lines[i].startsWith('monitor_status')) {
          const status = this.parseStatus(lines[i]);

          if (!this.services[status.name]) {
            this.services[status.name] = status;
          } else {
            // 1 = up, 0 = down
            this.services[status.name].status = status.value;
          }
        }
      }
    },
    parseStatus(statusLine) {
      const monitorStatusRegex = /monitor_status{monitor_name="(.*)",monitor_type="(.*)",monitor_url="(.*)",monitor_hostname="(.*)",monitor_port="(.*)"} (.*)/;
      const match = monitorStatusRegex.exec(statusLine);
      if (match) {
        const name = match[1];
        const type = match[2];
        const url = match[3];
        const hostname = match[4];
        const port = match[5];
        const value = match[6];
        return {
          name,
          type,
          url,
          hostname,
          port,
          value,
        };
      }

      return null;
    },
    parseResponseTime(responseTimeLine) {
      const monitorResponseTimeRegex = /monitor_response_time{monitor_name="(.*)",monitor_type="(.*)",monitor_url="(.*)",monitor_hostname="(.*)",monitor_port="(.*)"} (.*)/;
      const match = monitorResponseTimeRegex.exec(responseTimeLine);
      if (match) {
        const name = match[1];
        const type = match[2];
        const url = match[3];
        const hostname = match[4];
        const port = match[5];
        const value = match[6];
        return {
          name,
          type,
          url,
          hostname,
          port,
          value,
        };
      }

      return null;
    },
  },
};
</script>

<style scoped lang="scss">
.uptime-kuma-wrapper {
  height: 100%;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap;
}
</style>
