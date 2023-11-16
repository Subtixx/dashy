<template>
  <div>
    <table style="width: 100%;" class="table" v-if="issues.length > 0">
      <thead>
      <tr>
        <th>Summary</th>
        <th style="width: 8rem;">Reporter</th>
        <th style="width: 8rem;">Created</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="issue in issues" style="color: #ffffff; cursor: pointer;"
          v-bind:key="issue.id" v-on:click="openLink(issue.id)">
        <td>{{ issue.summary }}</td>
        <td>{{ issue.reporter.login }}</td>
        <td :title="fromDate(issue.created, dateTimeFormat)">{{ issue.created | dateToAgo }}</td>
      </tr>
      </tbody>
    </table>
    <div v-else>
      <p>No issues found</p>
    </div>
  </div>
</template>

<script>
import WidgetMixin from '@/mixins/WidgetMixin';

export default {
  mixins: [WidgetMixin],
  data() {
    return {
      issues: [],
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
    accessToken() {
      if (!this.options.accessToken) {
        return '';
      }

      return this.options.accessToken;
    },
    dateTimeFormat() {
      if (!this.options.dateTimeFormat) {
        return 'DD.MM.YYYY HH:mm';
      }

      return this.options.dateTimeFormat;
    },
    filter() {
      if (!this.options.filter) {
        return '';
      }

      return encodeURIComponent(this.options.filter);
    },
    maxIssues() {
      if (!this.options.maxIssues) {
        return 10;
      }

      return this.options.maxIssues;
    },
  },
  methods: {
    openLink(id) {
      window.open(`${this.endpoint}/issue/${id}`, '_blank');
    },
    fetchData() {
      let filter = '';
      if (this.filter) {
        filter = `&filter=${this.filter}`;
      }

      this.makeRequest(
        `${this.endpoint}/api/issues?$top=${this.maxIssues}&fields=id,summary,description,created,reporter(login)&customFields=assignee&customFields=priority${filter}`,
        {
          Authorization: `Bearer ${this.accessToken}`,
        },
      ).then(this.processData);
    },
    processData(data) {
      // Order by created
      this.issues = data.sort((a, b) => {
        if (a.created < b.created) {
          return 1;
        }
        if (a.created > b.created) {
          return -1;
        }
        return 0;
      });
    },
    fromDate(value, dateTimeFormat) {
      const format = (date, pattern) => pattern.replace(/\b(\w)(\1)*\b/g, (match) => {
        switch (match[0]) {
          default:
            return match;
          case 'Y':
            return date.getFullYear();
          case 'M':
            return `${date.getMonth() + 1}`.padStart(match.length, '0');
          case 'D':
            return `${date.getDate()}`.padStart(match.length, '0');
          case 'h':
            return `${date.getHours() % 12}`.padStart(match.length, '0');
          case 'H':
            return `${date.getHours()}`.padStart(match.length, '0');
          case 'm':
            return `${date.getMinutes()}`.padStart(match.length, '0');
          case 's':
            return `${date.getSeconds()}`.padStart(match.length, '0');
          case 'a':
            return date.getHours() < 12 ? 'am' : 'pm';
        }
      });
      const date = new Date(value);
      return format(date, dateTimeFormat);
    },
  },
  filters: {
    dateToAgo(value) {
      const date = new Date(value);
      const seconds = Math.floor((new Date() - date) / 1000);

      let interval = Math.floor(seconds / 31536000);

      if (interval > 1) {
        return `${interval} years ago`;
      }
      interval = Math.floor(seconds / 2592000);
      if (interval > 1) {
        return `${interval} months ago`;
      }
      interval = Math.floor(seconds / 86400);
      if (interval > 1) {
        return `${interval} days ago`;
      }
      interval = Math.floor(seconds / 3600);
      if (interval > 1) {
        return `${interval} hours ago`;
      }
      interval = Math.floor(seconds / 60);
      if (interval > 1) {
        return `${interval} minutes ago`;
      }
      return `${Math.floor(seconds)} seconds ago`;
    },
  },
};
</script>

<style scoped lang="scss">
.table {
  border-collapse: collapse;
  border-spacing: 0;
  width: 100%;
  border: 1px solid var(--medium-grey);
}

.table tr {
  border: 1px solid var(--medium-grey);
  background-color: var(--item-background);
}

.table tr:hover {
  background-color: var(--item-background-hover);
}

.table td {
  padding: 8px;
}

.table th {
  padding: 8px;
  text-align: left;
  background-color: #4caf50;
  color: white;
}
</style>
