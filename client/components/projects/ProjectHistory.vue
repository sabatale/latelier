<template>
  <div class="project-history">
    <v-dialog v-model="showDialog" :fullscreen="$vuetify.breakpoint.xsOnly" max-width="70%">
      <v-toolbar dark color="primary">
        <v-btn icon flat @click="close()" v-shortkey="['esc']" @shortkey="close()">
          <v-icon>close</v-icon>
        </v-btn>
        <v-toolbar-title>
          Historique
        </v-toolbar-title>
      </v-toolbar>       
      <v-card>
        <v-card-text>
          <v-timeline :dense="$vuetify.breakpoint.xsOnly" clipped> 
            <v-timeline-item 
              v-for="item in history"
              :key="item._id"
              color="indigo"
              :small="$vuetify.breakpoint.xsOnly"
            >
             <span slot="opposite">{{ formatDateDuration(item.createdAt) }} par {{ formatUser(item.userId) }}</span>
              <v-card class="elevation-2">
                <v-card-text>
                  <div>{{ item.properties.task.name }}</div>
                  <div class="grey--text">{{ item.type }}</div>
                </v-card-text>
              </v-card>

            </v-timeline-item>

          </v-timeline>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn flat @click="close()">Fermer</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>    
</template>

<script>
import { Meteor } from 'meteor/meteor'
import DatesMixin from "/imports/ui/mixins/DatesMixin.js";

export default {
  mixins: [DatesMixin],
  props: {
    projectId: String
  },
  data () {
    return {
      showDialog: false,
      history: []
    }
  },
  methods: {
    open () {
      this.refresh();
      this.showDialog = true;
    },

    close () {
      this.showDialog = false;
    },

    refresh () {
      Meteor.call('projects.getHistory', this.projectId, (error, result) => {
        if (error) {
          console.log(error);
          return;
        }
        this.history = result.data;
        console.log (this.history);
      })
    },

    formatUser (userId) {
      if (!userId) {
        return '';
      }
      var user = Meteor.users.findOne({_id: userId});
      return user.emails[0].address;
    },

  }
}
</script>

<style scoped>
.content {
  overflow-y: auto;
  max-height: 500px;
}

</style>