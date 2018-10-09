<template>
<div>
 

<div class="task-item">

<div class="checkbox"> 
        <input type="checkbox" v-model="completed" @change="doneEdit">
      </div>
      <div class="input-group mb-3">
      
      <div v-if="!editing" @click="editTask" class="task-item-label" :class="{completed : completed}">{{title}}</div>
      <input type="text" v-model="title" v-else class="task-edit form-control" @blur="doneEdit" @keyup.enter="doneEdit" @keyup.esc="cancelEdit">
       <span class="input-group-btn"><button class="btn btn-danger pull-right" type="button" @click="removeTask(index, databaseId)"> 
         <span class="glyphicon glyphicon-remove"></span></button></span>
    </div>




</div>
</div> 
</template>

<script>
import axios from 'axios';
export default{
data(){
return {
  'title': this.task.title,
  'completed': this.task.completed,
  'editing': this.task.editing,
  'beforeEditCache': '',
  'databaseId': this.task.databaseId
}
},

props: {
  task: {
    type: Object,
    required: true
  },
  index: {
    type: Number,
    required: true
  },
  checkAll:{
    type: Boolean,
    required: true
  },
},
watch: {
 checkAll(){
   if (this.checkAll){
     this.completed = true
   } else{
     this.completed = this.task.completed
   }
 }
},
methods:{
  removeTask(index, databaseId){
  this.$emit('removedTask', this.index, this.databaseId)
   },
   editTask(){
  this.editing = true
  this.beforeEditCache = this.title
   },
   doneEdit(task){
     if(this.title.trim() == ''){
       this.title = this.beforeEditCache
     }
     this.editing = false
     this.$emit('finishedEdit', {
       "index": this.index,
       'task':{
         'title': this.title,
         'completed': this.completed,
         'editing': this.editing,
         'databaseId': this.databaseId,
       }
     })
     axios.patch(`https://small-todo-app.firebaseio.com/task/${this.databaseId}.json`, {
     title: this.title,
     completed: this.completed,
     editing: !this.editing
     })
     .then(res=>console.log(res))
     .catch(err => console.log(err))
   },
   cancelEdit(){
     this.title = this.beforeEditCache
     this.editing = false
   }
}
}
</script>

<style>
.extra-container{
  margin-left: 1.5em
}


</style>