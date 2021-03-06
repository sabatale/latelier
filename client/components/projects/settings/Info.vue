<template>
  <div class="project-info">
    <v-subheader>Description</v-subheader>
    <div class="elevation-1">
      <div class="description">
        <div v-show="project.description && project.description.length > 0">
          <div class="tiptap-editor-view" v-html="markDown(project.description)" />
        </div>
        <div v-show="!project.description">
          {{ $t("No description") }}
        </div>
      </div>
    </div>

    <v-subheader>Dates</v-subheader>
    <v-list two-line class="elevation-1">
      <v-list-item>
        <v-list-item-avatar>
          <v-icon>mdi-calendar-today</v-icon>
        </v-list-item-avatar>
        <v-list-item-content>
          <v-list-item-title>{{ $t("Start date") }}</v-list-item-title>
          <v-list-item-subtitle>
            <span v-show="project.startDate">{{
              formatDate(project.startDate)
            }}</span>
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>

      <v-divider />

      <v-list-item>
        <v-list-item-avatar>
          <v-icon>mdi-alarm-check</v-icon>
        </v-list-item-avatar>
        <v-list-item-content>
          <v-list-item-title>{{ $t("End date") }}</v-list-item-title>
          <v-list-item-subtitle>
            <span v-show="project.endDate">{{
              formatDate(project.endDate)
            }}</span>
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-list>

    <v-subheader>Couleur</v-subheader>

    <v-list class="elevation-1">
      <v-list-item>
        <v-list-item-content>
          <div ref="color" class="color" :style="getColor(project)" />
        </v-list-item-content>
      </v-list-item>
    </v-list>

    <v-subheader>
      {{ $t("Categories") }}
    </v-subheader>
    <v-list class="elevation-1">
      <template v-for="group in assignedGroups">
        <v-list-item :key="group._id">
          <v-list-item-avatar>
            <v-icon>mdi-folder</v-icon>
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title>{{ group.name }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </template>
    </v-list>

    <v-subheader>Organisation</v-subheader>
    <v-list v-if="$subReady.organizations" class="elevation-1">
      <v-list-item>
        <v-avatar>
          <v-icon>mdi-folder</v-icon>
        </v-avatar>
        <v-list-item-content>
          <v-list-item-title>{{ organization.name }}</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-list>
  </div>
</template>

<script>
import { ProjectGroups } from "/imports/api/projectGroups/projectGroups.js";

import DatesMixin from "/imports/ui/mixins/DatesMixin.js";
import MarkdownMixin from "/imports/ui/mixins/MarkdownMixin.js";

export default {
  name: "ProjectInfo",
  mixins: [DatesMixin, MarkdownMixin],
  props: {
    project: {
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {};
  },
  meteor: {
    $subscribe: {
      projectGroups: () => []
    },

    assignedGroups: {
      params() {
        return {
          project: this.project
        };
      },
      deep: false,
      update({ project }) {
        if (project) {
          return ProjectGroups.find(
            { projects: project._id },
            { sort: { name: 1 } }
          );
        }
        return null;
      }
    }
  },
  methods: {
    getColor(project) {
      return `background-color: ${project.color}`;
    }
  }
};
</script>

<style scoped>
.groups {
  width: 100%;
}

.description {
  margin-left: 24px;
  margin-right: 24px;
  margin-bottom: 12px;
  padding-top: 12px;
  padding-bottom: 12px;
}

.color {
  height: 32px;
  width: 200px;
}
</style>
