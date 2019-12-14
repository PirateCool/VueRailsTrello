<template>
  <draggable v-model="lists" :options="{group: 'lists'}" class="board dragArea" @end="listMoved">
   <list v-for="(list, index) in lists" :list="list"></list>
    

    <div class="list">
       <a v-if="!editing" v-on:click="startEditing" class="btn" style="margin-left: 2px;">＋Add a list</a>
      <textarea v-if="editing" v-model="message"
        ref="message" 
        @keydown.enter.exact.prevent="createList"
        class="form-control"
        placeholder="Write title for your list ↵"
        style="margin-bottom: 6px;"
        ></textarea>
      <button v-if="editing" v-on:click="createList" class="btn btn-light w-50" style="margin-left: 2px; margin-right: 3px;"><b>+ Add list</b></button>
      <a v-if="editing" v-on:click="editing=false" class="btn btn-transparent"><b>✕</b></a>
      </div>
  </draggable>
</template>

<script>
import draggable from 'vuedraggable'
import list from 'components/list'

export default {
  components: { draggable, list },
  data: function() {
    return {
     
      editing: false,
      message: "",
    }
  },

  computed:  {
    lists() {
      return this.$store.state.lists;
    }
  },

  methods: {
    listMoved: function(event) {
      var data = new FormData
      data.append("list[position]", event.newIndex + 1)
      Rails.ajax({
        url: `/lists/${this.lists[event.newIndex].id}/move`,
        type: "PATCH",
        data: data,
        dataType: "json",
      })
    },
    startEditing: function() {
      this.editing = true
      this.$nextTick(() => {this.$refs.message.focus() })
    },

    createList: function() {
      var data = new FormData
      data.append("list[name]", this.message)
      Rails.ajax({
        url: "/lists",
        type: "POST",
        data: data,
        dataType: "json",
        success: (data) => {
          // this.$store.commit('addList', data)
          this.message = ""
          this.editing = false
        }
      })
    },


 }
}
</script>

<style scoped>
.dragArea {
  min-height: 10px;
}

::placeholder {
  color: lightgrey;
}

.board {
  white-space: nowrap;
  overflow-x: auto;
  padding-bottom: 50px;

}

.list {
  display: inline-block;
  width: 270px; 
  vertical-align: top;
  margin-right: 12px;
  background: #E2E4E6;
  padding-top: 20px;
  padding-bottom: 20px;
  padding-right: 6px;
  padding-left: 6px;

  border-radius: 3px;
}





</style>