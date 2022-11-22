<template>
  <div>
    <draggable
      class="list-group text-left mt-4"
      :list="tasks"
      style="cursor:move; min-height: 100px;"
      group="people"
      ghost-class="ghost"
      @change="changed(column_id, $event)"
      :animation="200"
    >
      <b-card
        class="mb-2 text-left task"
        :class="getTaskCustomization"
        :style="getTaskHeight(element)"
        v-for="(element) in tasks"
        @contextmenu.prevent.stop="e => {element['column_id'] = column_id; taskHandle($event, element)}"
        :key="element._id"
      >
        <h4 class="card-title">{{element.title}}</h4>
        <p> {{element.description}} </p>
        <!-- <button> <img v-bind:src="'http://127.0.0.1:3000/photos/' + element.filename" width="100px" height="100px" alt=""> {{element.filePath}} </button> -->
        <button> <a target="_blank" v-bind:href="'http://127.0.0.1:3000/photos/' + element.filename" width="100px" height="100px" > {{element.filePath}} </a> </button>


        <span
          style="position: absolute; top: 0px; cursor:pointer;"
          @click.prevent.stop="e => {element['column_id'] = column_id; taskHandle($event, element)}"
        >
          <i class="fas fa-ellipsis-h"></i>
        </span>
        <p v-if="element.expireAt">
          <i class="fa fa-clock" />
          {{element.expireAt | moment('MMMM D')}}
        </p>
        <span
          :class="`badge badge-${element.labelType}`"
          style="text-transform:capitalize; font-size: 0.635rem"
          v-if="element.labelType"
        >
          <i class="fas fa-circle" />
          {{element.label}}
        </span>
      </b-card>
    </draggable>
  </div>
</template>

<script>
// import Axios from 'axios'
import eventBus from "../../eventBus";
import draggable from "vuedraggable";
export default {
  data() {
    return {
      URL: 'http://127.0.0.1:3000/photos/fileUpload-1669108104946.jpg',
      image: null,
      file: null,
      base64: null,
    }
  },
  props: ["tasks", "column_id", "customization"],
  components: {
    draggable
  },
  computed: {
    getTaskCustomization() {
      if (this.customization.theme.selected === "Dark") {
        return "bg-dark text-white";
      }
      return "";
    }
  },
  methods: {

    viewImage() {
      this.$http.get('/user/viewImage').then(res => {
        console.log(res.data)
        this.image = res.data
        this.fileReader();
      })
    },

    baseToImage() {
      this.file = new FileReader();
      this.file(this.image)
      this.file.onload = function (evt) {
        return this.base64 = evt.target.result
        
      }
    },

    getTaskHeight(element) {
      if (element.date || element.label) {
        return "height: 90px";
      } else {
        return "height: 80px";
      }
    },
    taskHandle(event, item) {
      eventBus.$emit("task-option-handled", { event, item });
    },
    changed: function(column_id, evt) {
      if (evt.added) {
        let task_id = evt.added.element._id;
        this.$http
          .post("/user/addToColumn", { task_id, column_id })
          .then(res => {
            console.log(res.data);
          })
          .catch(err => {
            if (err.response.status === 401) {
              this.$router.push("/login");
            }
          });
      } else if (evt.removed) {
        let task_id = evt.removed.element._id;
        this.$http
          .post("/user/removeFromColumn", { task_id, column_id })
          .then(res => {
            console.log(res.data);
          })
          .catch(err => {
            if (err.response.status === 401) {
              this.$router.push("/login");
            }
          });
      } else {
        console.log("Unknown process.");
      }
    }
  }
};
</script>

<style scoped >
.file-path{
  color: black;
  font: bold;
}
</style>