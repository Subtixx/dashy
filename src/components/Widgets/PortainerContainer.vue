<template>
  <div class="portainer-container-wrapper">
    <div v-for="container in containers" :key="container.Id" class="item-wrapper wrap-size-medium span-2">
      <div class="item size-medium">
        <div class="tile-title container-name">
          <span class="text">{{ container.Name }}</span>
          <p class="description">
            <span v-if="isRunning(container)">
              {{ container.Uptime | secondsToHuman }}
            </span>
          </p>
        </div>
        <div class="bounce item-icon wrapper-medium">
          <i class="far " :class="container.Icon" aria-hidden="true"></i>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import WidgetMixin from "@/mixins/WidgetMixin";

const STATE_RUNNING = "running";
const STATE_EXITED = "exited";

export default {
  mixins: [WidgetMixin],
  data() {
    return {
      containers: [],
    };
  },
  computed: {
    endpoint() {
      if (!this.options.endpoint) {
        return "";
      }
      if (this.options.endpoint.endsWith("/")) {
        return this.options.endpoint.slice(0, -1);
      }
      return `${this.options.endpoint}`;
    },
    endpointId() {
      if (!this.options.endpointId) {
        return "";
      }
      return `${this.options.endpointId}`;
    },
    authToken() {
      if (!this.options.authToken) {
        return "";
      }
      return `${this.options.authToken}`;
    },
  },
  methods: {
    fetchData() {
      this.makeRequest(
        `${this.endpoint}/api/endpoints/${this.endpointId}/docker/containers/json?all=1`,
        {
          "X-Api-Key": this.authToken
        }
      ).then(this.processData);
    },
    processData(data) {
      // Do processing any here, and set component data
      this.results = data;

      for (let i = 0; i < data.length; i++) {
        let name = data[i].Names[0];
        if (name.startsWith("/")) {
          name = name.slice(1);
        }

        this.containers.push({
          Id: data[i].Id,
          Name: name,
          State: data[i].State,
          Status: data[i].Status,
          Uptime: this.statusToSeconds(data[i].Status),
          Icon: this.stateIcon(data[i].State),
        });
      }
    },
    isRunning(container) {
      return container.State === STATE_RUNNING;
    },
    stateIcon(state) {
      if (state === STATE_RUNNING) {
        return "fa-play-circle";
      } else if (state === STATE_EXITED) {
        return "fa-stop-circle";
      } else {
        return "fa-question-circle";
      }
    },
    statusToSeconds(status) {
      let regex = /Up ([0-9]+) (weeks|days|hours|minutes|seconds)/;
      let match = regex.exec(status);
      if (match) {
        let value = parseInt(match[1]);
        let unit = match[2];
        if (unit === "weeks") {
          return value * 7 * 24 * 60 * 60;
        } else if (unit === "days") {
          return value * 24 * 60 * 60;
        } else if (unit === "hours") {
          return value * 60 * 60;
        } else if (unit === "minutes") {
          return value * 60;
        } else if (unit === "seconds") {
          return value;
        }
      }
    },
  },
  filters: {
    secondsToHuman(seconds) {
      let weeks = Math.floor(seconds / (7 * 24 * 60 * 60));
      seconds -= weeks * 7 * 24 * 60 * 60;
      let days = Math.floor(seconds / (24 * 60 * 60));
      seconds -= days * 24 * 60 * 60;
      let hours = Math.floor(seconds / (60 * 60));
      seconds -= hours * 60 * 60;
      let minutes = Math.floor(seconds / 60);
      seconds -= minutes * 60;

      let result = "";
      if (weeks > 0) {
        result += `${weeks}w `;
      }
      if (days > 0) {
        result += `${days}d `;
      }
      if (hours > 0) {
        result += `${hours}h `;
      }
      if (minutes > 0) {
        result += `${minutes}m `;
      }
      if (seconds > 0) {
        result += `${seconds}s `;
      }
      return result;
    },
  },
};
</script>

<style scoped lang="scss">
.portainer-container-wrapper {
  height: 100%;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap;
}

.container-name {
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100px !important;
}
</style>
