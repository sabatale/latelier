<template>
  <div
    ref="kanban"
    v-dragscroll="scrollEnabled"
    class="kanban noselect"
    @click="(e) => hideProperties(e)"
    @mousemove="onMouseMove"
  >
    <div
      v-for="list in lists"
      :key="list._id"
      class="kanban-flex dragscroll"
      :data-id="list._id"
    >
      <list
        :ref="list._id"
        :list="list"
        class="kanban-list-item dragscroll"
        :data-id="list._id"
      />
    </div>
    <div class="swimlane dragscroll new">
      <h2 @click="newListInline">
        {{ $t("New list") }}
      </h2>
    </div>
  </div>
</template>

<script>
import { Lists } from "/imports/api/lists/lists.js";
import { dragscroll } from "vue-dragscroll";
import Sortable from "sortablejs";

export default {
  directives: {
    dragscroll: dragscroll
  },
  props: {
    projectId: {
      type: String,
      default: null
    },
    addMargin: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      scrollEnabled: false,
      sortable: null
    };
  },
  mounted() {
    const options = {
      delayOnTouchOnly: true,
      delay: 250,
      animation: 150,
      handle: ".list-header",
      group: "lists",
      onUpdate: (event) => {
        this.handleMove(event);
      },
      onAdd: (event) => {
        this.handleMove(event);
      }
    };
    this.sortable = Sortable.create(this.$refs.kanban, options);
  },
  beforeDestroy() {
    this.sortable.destroy();
  },
  meteor: {
    lists() {
      return Lists.find({ projectId: this.projectId }, { sort: { order: 1 } });
    }
  },
  methods: {
    hideProperties() {
      this.$store.dispatch("showTaskDetail", false);
    },

    newListInline() {
      Meteor.call(
        "lists.insert",
        this.projectId,
        this.$t("New list"),
        (error, createdList) => {
          if (error) {
            return;
          }
          this.$events.fire("edit-list", createdList._id);
        }
      );
    },

    onMouseMove(e) {
      if (e && e.target && e.target.classList.contains("dragscroll")) {
        this.scrollEnabled = true;
      } else {
        this.scrollEnabled = false;
      }
    },

    handleMove(event) {
      const listId = event.item.dataset.id;
      const index = event.newIndex;
      let newOrder = 0;

      if (index >= this.lists.length) {
        const lastList = this.lists[this.lists.length - 1];
        newOrder = lastList.order + 1;
      } else {
        const { oldIndex } = event;
        const nextList = this.lists[index];
        let inc;
        if (oldIndex < index) {
          inc = 1;
        } else {
          inc = -1;
        }
        newOrder = nextList.order + inc;
      }

      Meteor.call("lists.move", this.projectId, listId, newOrder);
    }
  }
};
</script>

<style scoped>
.kanban-flex {
  display: flex;
  flex-direction: column;
  cursor: grab;
}

.swimlane.new h2 {
  font-weight: normal;
  font-size: 14px;
  padding: 5px;
  padding-top: 10px;
  margin-bottom: 0px;
  border: 1px dashed #2d6293;
  padding-bottom: 7px;
  color: #777;
  cursor: pointer;
}

@media (min-width: 601px) {
  .swimlane.new {
    flex: 0 0 auto;
    width: 272px;
    display: inline-block;
    margin-right: 8px;
  }
  .swimlane.task-detail-margin {
    flex: 0 0 auto;
    width: 592px;
    display: inline-block;
    margin-right: 8px;
  }
}

.swimlane.new h2:hover {
  color: rgb(48, 48, 48);
  cursor: pointer;
}

.list {
  overflow-y: scroll;
}
@media (min-width: 601px) {
  .kanban {
    overflow-y: hidden !important;
  }

  .kanban-flex {
    border-right: 1px solid rgba(0, 0, 0, 0.12);
    margin-right: 10px;
  }
}
</style>
