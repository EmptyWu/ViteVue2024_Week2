<script lang="ts" setup>
import {ref}from 'vue'
import axios,{ AxiosResponse } from 'axios'
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

const sign_up= async () :Promise<void> =>{
    try{
        const response:AxiosReponse<signResponse>= await axios.post(`${site}users/sign_up`,{
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
interface sign_inResponse{
    status:boolean;
    exp:number;
    token:string;
}
const sign_in =async():Promise<void>=>{
    try{
        const response:AxiosReponse<sign_inResponse>= await axios.post(`${site}users/sign_in`,{
        email: emailsignin.value,
        password: passwordsignin.value       
        });
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
        const response:AxiosReponse<signResponse>= await axios.get(`${site}users/checkout`,{},{
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
        const response:AxiosReponse<signResponse>=await axios.post(`${site}users/sign_out`,{},{
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

</script>
<template>
    <h2>註冊</h2>
    <input type="text" v-model="email" placeholder="Email" />
    <input type="text" v-model="password" placeholder="password" />
    <input type="text" v-model="nickname" placeholder="別名" />
    <button type="button" v-on:click="sign_up" >送出</button>
    <p>{{ signmsg }}</p>

    <h2>登入</h2>
    <input type="text" v-model="emailsignin" placeholder="Email" />
    <input type="text" v-model="passwordsignin" placeholder="password" />
    <button type="button" v-on:click="sign_in" >登入</button>
    <p>{{msgsignin}}</p>
    
    <h2>驗證</h2>
    <input type="text" v-model="tokencheck" placeholder="toekn" />
    <button type="button" v-on:click="checkOut">驗證</button>
    <p>{{ messageCheckOut }}</p> 

    <h2>登出</h2>
    <input type="text" v-model="tokensign_out" placeholder="toekn" />
    <button type="button" v-on:click="checksign_out">登出</button>
    <p>{{ messagetokensign_out }}</p>

</template> 