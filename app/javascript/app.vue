<template>
  <draggable v-model="lists" :options="{group: 'lists'}" class="board dragArea" @end="listMoved">
    <div v-for="(list, index) in lists" class="list">
      <h6 style="padding-left: 10px">{{ list.name }}</h6>
      <hr />
      
      <draggable v-model="list.cards" :options="{group: 'cards'}" class="dragArea" @change="cardMoved">
      <div v-for="(card, index) in list.cards" class="list-card card card-body mb-3">
        {{ card.name }}
      </div>
      </draggable>

      <div class="card card-body">
       <textarea v-model="messages[list.id]" 
        @keydown.enter.exact.prevent="submitMessages(list.id)"
        class="form-control"
        placeholder="Add card (press ↵)"
        ></textarea>
        <!-- <button v-on:click="submitMessages(list.id)" class="btn btn-light w-100"><b>+</b></button> -->
      
      </div>
    </div>
  </draggable>
</template>

<script>
import draggable from 'vuedraggable'
export default {
  components: { draggable },
  props: ["original_lists"],
  data: function() {
    return {
      messages: {},
      lists: this.original_lists,
    }
  },
  methods: {
    cardMoved: function(event) {
      const evt = event.added || event.moved
      if (evt == undefined) { return }

      const element = evt.element 

      const list_index = this.lists.findIndex((list) => {
        return list.cards.find((card) => {
          return card.id == element.id
        })
      }) 

      var data = new FormData 
      data.append("card[list_id", this.lists[list_index].id)
      data.append("card[position]", evt.newIndex + 1)

      Rails.ajax({
        url: `/cards/${element.id}/move`,
        type: "PATCH",
        data: data, 
        dataType: "json"
      })

    },

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

    submitMessages: function(list_id) {
      var data = new FormData
      data.append("card[list_id]", list_id)
      data.append("card[name]", this.messages[list_id])
      Rails.ajax({
        url: "/cards",
        type: "POST",
        data: data,
        dataType: "json",
        success: (data) => {
          const index = this.lists.findIndex(item => item.id == list_id)
          this.lists[index].cards.push(data)
          this.messages[list_id] = undefined
        }
      })
    }
  }, 

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

.list-card{
  margin-bottom: 7px!important;
  padding: 9px;
}


</style>