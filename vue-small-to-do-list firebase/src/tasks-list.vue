<template>
  <div>
    <div class="container">
    <h1>Tasks({{itemsLeft}})</h1>

<div class="container">
<div class="row">
<div class="col-sm-12">
<div class="input-group mb-3">
  <input type="text" class="form-control" placeholder="What needs to be done?"
  v-model="newTaskTitle" @keyup.enter = "addTask">
  <span class="input-group-btn">
   <button class="btn btn-primary" type="button" id="button-addon2" @click="addTask"><span class="glyphicon glyphicon-plus"></span> Add</button>
  </span>
</div>
</div>
</div>
</div>

<div class="container button-group-clear">
<div class="row">
<div class="col-sm-12">
  <div class="tasks__clear button-group pull-right">
    <button type="button" class="btn btn-warning" @click="clearCompleted"><span class="glyphicon glyphicon-ok"></span> Clear completed</button>
    
</div>
</div>
</div>
</div>

<div class="extra-container">
  <div><label><input type="checkbox" class="completed" @change="checkAllTasks" :checked="!anyRemaining"> Check all</label></div>
</div>
<hr>
<div class="col-md-12">
    <app-task-item  v-for="(task, index) in allTasksFiltered" :key="task.id" :task="task" :index="index" @removedTask="removeTask" 
    @finishedEdit="finishedEdit" :checkAll="!anyRemaining">
    </app-task-item>
</div>

<div class="filter-container">
<div>
  <button type="button" class="btn btn-success" :class="{active: filter == 'all'}" @click = "filter = 'all'">All</button>
  <button type="button" class="btn btn-success" :class="{active: filter == 'active'}" @click = "filter = 'active'">Active</button>
  <button type="button" class="btn btn-success" :class="{active: filter == 'completed'}" @click = "filter = 'completed'">Completed</button>
</div>
</div>




</div>
</div>
  
</template>

<script>
import axios from 'axios';
import taskItem from '../src/task-item';
export default {
 
  data () {
    return {
     newTaskTitle: '',
     filter: 'all',
     allTasks: [],
     beforeEditCache: ''
    }
  },
  components: {
   'app-task-item': taskItem
  },
  computed:{
   itemsLeft(){
     return this.allTasks.filter(task => !task.completed).length
   },
   anyRemaining(){
     return this.itemsLeft != 0
   },
   allTasksFiltered(){
     if(this.filter === 'all'){
       return this.allTasks
     } else if (this.filter === 'active'){
       return this.allTasks.filter(task => !task.completed)
     } else if (this.filter === 'completed'){
       return this.allTasks.filter(task => task.completed)
     }
     return this.allTasks
   }
  },
 created(){
 axios.get('https://small-todo-app.firebaseio.com/task.json')
     .then(response => {
      const data = response.data
      for (let key in data){
        const newObject = Object.assign({},data[key],{databaseId: key})
        //console.log('this is a new object', newObject)
        this.allTasks.push(newObject)
      }
     })
     .catch(err => console.log(err))
 },
 methods:{
   addTask(){
     if(this.newTaskTitle.trim().length == 0){
       return
     }
    axios.post('https://small-todo-app.firebaseio.com/task.json', {
     title: this.newTaskTitle,
     completed: false,
     editing: false
   })
   .then ( res => {
     console.log(res.data.name)
      const newTask = {
      title: this.newTaskTitle,
      completed: false,
      editing: false,
      databaseId: res.data.name
      }
   this.allTasks.push(newTask)
    this.newTaskTitle = ""
   })
   .catch (err => console.log(err))
   },
   removeTask(index, databaseId){
     axios.delete(`https://small-todo-app.firebaseio.com/task/${databaseId}.json`,{
     })
     .then (res => {
      this.allTasks.splice(index, 1)
      })
     .catch(err => console.log(err))
   },
  finishedEdit(data){
    this.allTasks.splice(data.index, 1, data.task)
  },
  checkAllTasks() {
      this.allTasks.forEach((task) => { 
      task.completed = event.target.checked
      if(task.completed === false|| task.completed === undefined){
        axios.patch(`https://small-todo-app.firebaseio.com/task/${task.databaseId}.json`,{
         completed: false
       })
       .then(res => {
       console.log(res)
       })
       .catch(err => console.log(err))
     } else{
       axios.patch(`https://small-todo-app.firebaseio.com/task/${task.databaseId}.json`,{
         completed: true
       })
       .then(res => {
       console.log(res)
       })
       .catch(err => console.log(err))
     }
     })
  },
   clearCompleted(){
     console.log( 'from clear completed',this.allTasks)
  this.allTasks = this.allTasks.filter(task => !task.completed)
  // console.log('after clear completed', this.allTasks)
  // this.allTasks.forEach( task => {
  //     console.log('xxxxxxxxxx',task)
  //   if(task.completed === true){
  //     console.log('this comes from if statement')
    
  //     axios.delete(`https://small-todo-app.firebaseio.com/task/${task.databaseId}.json`)
  //     .then(res => console.log(res))
  //     .catch(err => console.log(err))
  //   }
  // })
    },
}
}
</script>

<style>
.task-item{
  margin-bottom: 12px;
  margin-left: 1.8em;
  width: 100%;
  align-items: center;
  justify-content: space-between;
  display: inline-block;
 
}

.task-edit{
  margin-bottom: 3px;
  margin-left: 1px;
  width: 100%;
  padding: 6px;
  align-items: center;
  justify-content: space-between;
  border: 1px solid lightgray;
  border-radius: 5px;
   
}

.task-item-label{
  margin-bottom: 3px;
  margin-left: 1px;
  width: 100%;
  padding: 6px;
  align-items: center;
  justify-content: space-between;
  border: 1px solid lightgray;
  border-radius: 5px;
  
}

.completed{
  text-decoration: line-through;
  color: gray
}


.button-group-clear{
  margin-top: 1em
}

.extra-container{
  margin-left: 1.5em
}

.filter-container{
  margin-left: 3em;
 
}

</style>