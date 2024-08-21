<script lang="ts" setup>
import {ref , onMounted}from 'vue';
import axios from 'axios';
import type {AxiosResponse} from 'axios';
const site='https://todolist-api.hexschool.io/';
const email =ref<string>('');
const password=ref<string>('');
const nickname=ref<string>('');
const signmsg=ref<string>('');
//users/sign_up

interface signResponse{
    status:boolean;
    message:string;
    uid:string;
}
interface Todo{
    id:string;
    createTime:number;
    content:string;
    status:boolean;
};
interface todoResponse{
    status:boolean;
    newTodo:Todo;
    };
interface todosResponse{
    status:boolean;
    data:Todo[]
}
interface sign_inResponse{
    status:boolean;
    exp:number;
    token:string;
}
interface deltodoResponse{
    status:boolean;
    message:string;
}

const sign_up= async () :Promise<void> =>{
    try{
        const response:AxiosResponse<signResponse>= await axios.post(`${site}users/sign_up`,{
        email: email.value,
        password: password.value,
        nickname: nickname.value
        });
        signmsg.value=`註冊成功，${response.data.uid}`;
    }
    catch(error:any){
        if(axios.isAxiosError(error)){
            signmsg.value = `註冊失敗，${error.response?.data?.message || error.message}`;
        }else {
            signmsg.value=`註冊失敗，發生未知錯誤:${error.message}`;
        }
    }
};

// /users/sign_in
const emailsignin=ref<string>('');
const passwordsignin=ref<string>('');
const msgsignin=ref<string>('');

const sign_in =async():Promise<void>=>{
    try{
        const response:AxiosResponse<sign_inResponse>= await axios.post(`${site}users/sign_in`,{
        email: emailsignin.value,
        password: passwordsignin.value       
        });
        token.value=response.data.token;
        msgsignin.value=`登入成功，${response.data.token}`;
    }catch(error:any){
        if(axios.isAxiosError(error)){
            msgsignin.value = `登入失敗，${error.response?.data?.message || error.message}`;
        }else {
            msgsignin.value=`登入失敗，發生未知錯誤:${error.message}`;
        }
    }

};

// users/checkout
const tokencheck=ref<string>('');
const messageCheckOut=ref<string>('');

const checkOut =async():Promise<void>=>{
    const usedata=new Date();
    usedata.setDate(usedata.getDate()+1);
    document.cookie = `hexschoolTodo=${tokencheck.value}; expires=${usedata.toUTCString()}`;
    try{
        const response:AxiosResponse<signResponse>= await axios.get(`${site}users/checkout`,{
                headers: {Authorization: tokencheck.value}
        });
        
        messageCheckOut.value=`驗證${response.data.status} UID:${ response.data.uid}`;
    }catch(error:any){
        if(axios.isAxiosError(error)){
            messageCheckOut.value = `驗證失敗，${error.response?.data?.message || error.message}`;
        }else {
            messageCheckOut.value=`驗證失敗，發生未知錯誤:${error.message}`;
        }
    }
};

//users/sign_out
const tokensign_out=ref<string>('');
const messagetokensign_out=ref<string>('');

const checksign_out =async():Promise<void>=>{
    try{
        const response:AxiosResponse<signResponse>=await axios.post(`${site}users/sign_out`,{},{
            headers: {Authorization: tokensign_out.value}
        });
        console.log(response.data);
        messagetokensign_out.value=`登出${response.data.status} ${ response.data.message}`;
    }catch(error:any){
        if(axios.isAxiosError(error)){
            messagetokensign_out.value = `登出失敗，${error.response?.data?.message || error.message}`;
        }else {
            messagetokensign_out.value=`登出失敗，發生未知錯誤:${error.message}`;
        }
    }
};
interface TodoEdit {
  [key: string]: string;  // 允许使用字符串类型的键
}
// todos/
const token=ref<string>('');
const todos=ref<Todo[]>([]);
const content=ref<string>('');
const todomsg=ref<string>('');
const todoEdit = ref<TodoEdit>({});

// 取得所有 todos
const addcontent=async():Promise<void>=>{
    try{
        const response:AxiosResponse<signResponse>=await axios.post(`${site}todos/`,{
            content:content.value
        },{
            headers: {Authorization: token.value}
        });
        console.log(response.data);
        todomsg.value=`新增${response.data.status} ${ response.data.message}`;
        getTodos();
    }catch(error:any){
        if(axios.isAxiosError(error)){
            todomsg.value = `新增失敗，${error.response?.data?.message || error.message}`;
        }else {
            todomsg.value=`新增失敗，發生未知錯誤:${error.message}`;
        }
    }
};
// 新增todos
const getTodos=async():Promise<void>=>{
    try{
        const response:AxiosResponse<todosResponse>=await axios.get(`${site}todos/`,{
            headers: {Authorization: token.value}
        });
        console.log(response.data);
        todos.value=response.data.data;
    }catch(error:any){
        if(axios.isAxiosError(error)){
            todomsg.value = `取得所有代辦事項失敗，${error.response?.data?.message || error.message}`;
        }else {
            todomsg.value=`取得所有代辦事項失敗，發生未知錯誤:${error.message}`;
        }
    }
};

// 刪除todos
const deltodo=async(id:string):Promise<void>=>{
    try{
        const response:AxiosResponse<deltodoResponse>=await axios.delete(`${site}todos/${id}`,{
            headers: {Authorization: token.value}
        });
        console.log(response.data);
        getTodos();
    }catch(error:any){
        if(axios.isAxiosError(error)){
            todomsg.value = `刪除代辦事項失敗，${error.response?.data?.message || error.message}`;
        }else {
            todomsg.value=`刪除代辦事項失敗，發生未知錯誤:${error.message}`;
        }
    }
};

//修改
const updatetodo=async(id:string):Promise<void>=>{
    try{
        const todo=todos.value.find((todo)=>todo.id===id);
        if(todo){
            todo.content = todoEdit.value[id];
            const response:AxiosResponse<deltodoResponse>=await axios.put(`${site}todos/${id}`,todo,{
                headers: {Authorization: token.value}
            });
            console.log(response.data);
            getTodos();
            todoEdit.value = {
                ...todoEdit.value,
                [id]: '',
            };
        }
    }catch(error:any){
        if(axios.isAxiosError(error)){
            todomsg.value = `修改代辦事項失敗，${error.response?.data?.message || error.message}`;
        }else {
            todomsg.value=`修改代辦事項失敗，發生未知錯誤:${error.message}`;
        }
    }
};

// todos/{id}/toggle
const toggletodo=async(id:string):Promise<void>=>{
    try{
       
        const response:AxiosResponse<deltodoResponse>=await axios.patch(`${site}todos/${id}/toggle`,{},{
            headers: {Authorization: token.value}
        });
        console.log(response.data);
        getTodos();
       
    }catch(error:any){
        if(axios.isAxiosError(error)){
            todomsg.value = `toggle代辦事項失敗，${error.response?.data?.message || error.message}`;
        }else {
            todomsg.value=`toggle代辦事項失敗，發生未知錯誤:${error.message}`;
        }
    }
};

const updateTodoEdit=(event:Event,id:string):void=>{
    const target=event.target as HTMLInputElement;
    todoEdit.value={
        ...todoEdit.value,
        [id]:target.value
    };
};

//驗證token
const todotoken= document.cookie.split('; ').find((row)=>row.startsWith('hexschoolTodo='))?.split('=')[1];
//
onMounted(()=>{
    if(todotoken){
        token.value=todotoken;
    }
});

</script>
<template>
    <div>
        <h2>註冊</h2>
        <div class="row">
            <div class="col">
                <input type="text" class="form-control" v-model="email" placeholder="email" >
            </div>
            <div class="col">
                <input type="text" class="form-control" v-model="password" placeholder="password" >
            </div>
            <div class="col">
                <input type="text" class="form-control" v-model="nickname" placeholder="nickname" >
            </div>
            <div class="col">
                <button type="button" class="btn btn-primary form-control" v-on:click="sign_up">送出</button>
            </div>
        </div>
        <p>{{ signmsg }}</p>

        <h2>登入</h2>
        <div class="row">
            <div class="col">
                <input type="text" class="form-control" v-model="emailsignin" placeholder="Email" />
            </div>
            <div class="col">
                <input type="text" class="form-control" v-model="passwordsignin" placeholder="password" />
            </div>
            <div class="col">
                <button type="button"  class="btn btn-primary form-control" v-on:click="sign_in">登入</button>
            </div>
        </div>
        <p>{{msgsignin}}</p>
        
        <h2>驗證</h2>
        <div class="row">
            <div class="col">
                <input type="text" class="form-control" v-model="tokencheck" placeholder="toekn" />
            </div>
            <div class="col">
                <button type="button" class="btn btn-primary form-control" v-on:click="checkOut" >驗證</button>
            </div>
        </div>
        <p>{{ messageCheckOut }}</p> 

        <h2>登出</h2>
        <div class="row">
            <div class="col">
                <input type="text"  class="form-control" v-model="tokensign_out" placeholder="toekn" />
            </div>
            <div class="col">
                <button type="button" v-on:click="checksign_out" class="btn btn-primary form-control">登出</button>
            </div>
        </div>
        <p>{{ messagetokensign_out }}</p>
    </div>
    <hr />
    <div>
        <h2>todolist-api</h2>
        <div v-if="token">
            <div class="row">
                <div class="col">
                    <input type="text"  class="form-control" v-model="content" placeholder="內容" />
                </div>
                <div class="col">
                    <button type="button" v-on:click="addcontent" class="btn btn-primary form-control">新增</button>
                </div>
            </div>
            <button type="button" class="btn btn-primary" v-on:click="getTodos">列出</button>
            <ul>
                <li v-for="(todo,index) in todos" :key="index">
                    {{ todo.id }}
                    {{ todo.content }} {{ todo.status?'完成':'未完成' }} | {{ todoEdit[todo.id] }}
                    <input type="text" placeholder="更新值" @change="updateTodoEdit($event,todo.id)" />
                    <button type="button" class="btn btn-primary" @click="deltodo(todo.id)">刪除</button>
                    <button type="button" class="btn btn-primary" @click="updatetodo(todo.id)">更新</button>
                    <button type="button" class="btn btn-primary" @click="toggletodo(todo.id)">更新狀態</button>
                </li>
            </ul>
            {{ todomsg }}
        </div>
       
    </div>
</template> 
<style>

</style>