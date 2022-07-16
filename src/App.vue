<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/> -->
<main>
  <div>
    <h1>TODO</h1>
  </div>
  <div>
    <ul>
      <li v-for="todo in List" :key="todo.id" :class="{check:todo.check, editing:editedTask===todo}">
        <div class="tasks">
          <input type="checkbox" v-model="todo.check">
          <label @dblclick="editTask2(todo)">{{todo.content}}</label>
          <button @click="delTask">Delete</button>
        </div>
        <div class="editbar">
        <input v-model="todo.content" @keyup.enter="endEdit()"
          @blur="cancelEdit(todo)" v-todo-focus="todo===editedTodo">
        <button @click="endEdit()">Finish</button>
        </div>

      </li>
    </ul>
    
  </div>
  <div class="addTodo">
    <input autofocus placeholder="Input your TODOs" ref="userInput" @keyup.enter="addTask">
    <button @click="addTask">Add</button>
  </div>
  <div class="count">
    <span>Tasks Unfinished: {{calProcTask}}</span>
  </div>
  <div>
    <p>操作说明：在Input框输入TODO名称，按Enter键或Add按钮提交。
      <br>单击TODO右侧的Delete可删除TODO，单击左侧的复选框可完成TODO。
      <br>剩余未完成TODO数目会在Input框下方计算。
      <br>双击文字编辑TODO，编辑完毕按Enter键或Finish按钮提交更改，或单击编辑框外取消更改。
      <br>TODO列表会自动保存，下次打开时自动读取。
    </p>
  </div>
</main>
</template>

<script>
import { reactive, toRefs } from '@vue/reactivity';
import { computed } from '@vue/runtime-core';
import { watchEffect } from 'vue';
//var List=[];
const todoStorge = {
  fetch () {
    let List = JSON.parse(localStorage.getItem('vue3-List') || '[]')
    List.forEach((todo, index) => {
      todo.id = index + 1
    })
    return List
  },
  save (List) {
    localStorage.setItem("vue3-List", JSON.stringify(List))
  }
}
export default{
  setup(){
    const state = reactive({
      userInput:'',
      List: todoStorge.fetch(),
      oriTask:{},
      editedTask:{},

    })
    // 添加TODO
    function addTask(){
      if(state.userInput.value=='')return;
      state.List.push({
        content:state.userInput.value,
        check:false,
        id:state.List.length});
      console.log(state.userInput.value)
      state.userInput.value = "";
      //console.log(state.List[state.List.length-1].content);
    }
    // 删除TODO
    function delTask(target){
      state.List.splice(state.List.indexOf(target),1);
    }
    // 计算剩余未完成任务
    const calProcTask=computed(()=>{
      var count=0;
      for(var i=0;i<state.List.length;i++){
        if(state.List[i].check==false){
          count++;
        }
      }
      return count;
    })

    // 编辑TODO
    function editTask2(target){
      state.oriTask.content=target.content;
      state.editedTask=target;
    }


    // 取消编辑
    function cancelEdit(target){
      if(state.editedTask!=null)
      {
      target.content=state.oriTask.content;
      state.editedTask=null;
      }
    }


    // 结束编辑（保存修改）
    function endEdit(){
      state.editedTask=null;
    }

    watchEffect(() => {
      todoStorge.save(state.List)
    })

    return{
      ...toRefs(state),
      addTask,
      delTask,
      calProcTask,
      cancelEdit,
      endEdit,
      editTask2,
    }
  },
  directives: {
    "todo-focus":(el,{ value},)=>{
      if(value) el.focus();
    }
  }

}

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;


}
.check label{
   text-decoration: line-through;
}
.editbar,
.editing .tasks{
  display: none;
}
.tasks,
.editing .editbar {
  display: block;
}
</style>
