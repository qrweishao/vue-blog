<template>
    <div class="login-container">
        <header class="login-head">
            <span>伟少-博客系统</span>
            <img class="logo" src="../assets/image/logo.gif" >
        </header>
        <section class="form">
            <span class="slogan">
                登~登~登~ 登录
                <span>/ Login</span>
            </span>
            <input name="user" v-validate="'required|min:5|max:16|alpha_dash'" type="text" id="user" placeholder="请填写用户名" v-model="LoginForm.user">
            <input name="password"  type="password" id="password" placeholder="请输入密码" v-model="LoginForm.password" @keydown.enter="login">
            <button id="login" @click="login">登录</button>
        </section>
        <footer>Always.</footer>
        <!-- <span>{{errors.first('user')}}</span> -->
        <notifications group="user"/>
        <notifications group="admin"/>
    </div>
</template>

<script>
// 设置验证的中文提示消息
import { Validator } from 'vee-validate'
// 引入设置cookie的方法
import { setToken } from '@/utils/auth'
const dict = {
    custom: {
        user: {
            required:"您的用户名不能为空"
        },
        password: {
            required:()=>"您的密码不能为空"
        }
    }
    
}
Validator.localize('en',dict)
// 引入发请求的模块
import request from '@/utils/request'

export default {
    name:"Login",
    data(){
        return {
            LoginForm:{
                user:'',
                password:''
            }
        }
    },
    methods:{
        // 登录方法
        login() {
        // 首先拿到验证成功失败的结果 如果成功了 在进行登录 如果失败了 则进行消息提示
            if (this.errors.items.length === 0) {
               request({
                   url:'/login',
                   method:'post',
                   data:this.LoginForm
               }).then(res=>{
                //    console.log(res);
                   if (res.success) {
                    // 正确后 要先得到token值 将token值存到cookie里面去
                    //跳转博客的首页 就是/list页面
                    let token = res.token;
                    setToken(token) //在浏览器cookie中保存token 每次请求带着它
                    this.$store.commit('SET_TOKEN',token) //保存在全局的状态下
                    this.$router.push('/list') //跳转到list页面
                   }else{
                    // 如果用户名 密码不正确 要给出提示
                       this.$notify({
                        type:'warn',
                        group:'user',
                        title:'登录失败',
                        text:res.message
                       })
                   }

               }).catch(err=>{
                // 如果发请求有错误 把错误扔给控制台
               })               
            }else{
                // console.log(this.errors.items);
                this.$notify({
                    type:'error',
                    group:'admin',
                    title:'验证失败',
                    text:this.errors.items[0].msg
                })
            }
            
        }
    },
    // 钩子函数 当组件加载完毕后自动执行
    mounted() {
       
    },
}
</script>

<style type="text/scss" lang="scss" scoped>
   //记得引入全局变量的样式
    @import '../assets/style/variable';
    .login-container {
        height: 100%;
        @include flex($flow: column wrap);
        font-family: "KaiShu", "Dosis", "Source Sans Pro", "Helvetica Neue", Arial, sans-serif;
        .login-head {
            @include flex;
            width: 100%;
            height: 3em;
            font-size: 2rem;
    }

    .form {
        @include flex($flow: column wrap);
        width: 400px;
        height: 250px;
        border-top: 20px solid $base;
        border-radius: 5px;
        // box-shadow: 4px 4px 5px -3px rgba(0, 0, 0, .3);
        box-shadow: 0 3px 10px rgba(0, 0, 0, 0.6);
        .slogan {
            font-size: 1.6rem;
            margin-bottom: 1em;
            //color: $quote;
            span {
                color: $base;
            }
        }
        input {
            font: {
                size: 1rem;
                family: Dosis, Arial, sans-serif;
            }
            margin-bottom: 2em;
            width: 20em;
            height: 2.5em;
            outline-color: $base;
            border-radius: 3px;
            border: 1px solid #ccc;
        }
        button {
            color: $white;
            font: {
                size: 1.6rem;
                family: KaiShu, Arial, sans-serif;
            }
            width: 6em;
            height: 2em;
            border: none;
            background: $base;
            outline: none;
            cursor: pointer;
        }
    }

    footer {
        height: 4em;
        @include flex;
        justify-self: flex-end;
    }
}
</style>
