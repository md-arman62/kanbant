<template>
  <div>
    <draggable class="list-group text-left mt-14 container" :list="tasks" style="cursor:pointer; min-height: 100px;" group="people"
      ghost-class="ghost" @change="changed(column_id, $event)" :animation="200">
      <b-card class="mb-2 text-left task" :class="getTaskCustomization" :style="getTaskHeight(element)"
        v-for="(element) in tasks"
        @contextmenu.prevent.stop="e => { element['column_id'] = column_id; taskHandle($event, element) }"
        :key="element._id">
        <!-- card description start -->

        <div class="title">
          <h4 class="card-title">{{ element.title }}</h4>
        </div>
        <div class="description">
          <p > {{ element.description }} </p>
        </div>

        <br>
        <br>

        <div class="list">
          <ul >
            <img src="../icon/icons8-attach-50.png" alt="" class="icon">
          <li> <a v-bind:href="'http://127.0.0.1:3000/multipleFiles/' + filePath"
              v-for="(filePath, index) in element.multipleFilePath" :key="index" target="_black" class="file-path"
              style="background-image:url('http://127.0.0.1:3000/multipleFiles/' + filePath[0][0])"
              > {{( index + 1 )}} . {{ filePath[0].slice(8) }} </a> 
            </li>
        </ul>
        </div>

        <!-- <div v-for="(filePath, index) in element.multipleFilePath" :key="index" >
          <ul>
            <li>
              <a target="_blank" v-bind:href="('http://127.0.0.1:3000/multipleFiles/' + filePath)"> a </a>
            </li>
          </ul>
        </div> -->

        <!-- card description end -->

        <!-- test git -->

   

        <span style="position: absolute; top: 0px; cursor:pointer;"
          @click.prevent.stop="e => { element['column_id'] = column_id; taskHandle($event, element) }">
          <i class="fas fa-ellipsis-h"></i>
        </span>
        <p v-if="element.expireAt">
          <i class="fa fa-clock" />
          {{ element.expireAt | moment('MMMM D') }}
        </p>

        <span :class="`badge badge-${element.labelType}`" style="text-transform:capitalize; font-size: 0.635rem"
          v-if="element.labelType">
          <i class="fas fa-circle" />
          {{ element.label }}
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
    changed: function (column_id, evt) {
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

/* *{
  padding: 0;
  margin: 0;
  box-sizing: border-box;
} */
/* .description {
  color: black;
  font: bold;
  margin-top: 10px;
} */

/* .singleFile {
  background-color: #fff;
  margin-top: 30px;
  margin-left: -21px;
  width: 167px;
} */
/* .full-card{
  background-color: green;
  min-height: 100px;
 } */


.file-path {
  display: inline-block;
  color: rgb(255, 255, 255);
  width: 100px;
  /* margin-top: 10px; */

}

.list {
  background-color: #7f7b7b;
  height: 100px;
  margin: 43px 0px 20px -21px;
  width: 184px;
  overflow: scroll;
}

.list > ul {
  list-style-type: none;
}

.icon{
    margin-top: 10px;
    width: 100px;
    height: 20px;
    font: white;
}
/* .container{
  background-color: red;
} */

</style>