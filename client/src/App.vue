
<template>
    <div>
        <navbar @selectNav="selectNav" :isLogin="isLogin" :user="user" @logout="logout"></navbar>
        <div v-if="isLogin === false">
            <login v-if="isRegistered === true" @toRegisterPage="toLoginPage" @login="login" :alert="alert"></login>
            <register v-else-if="isRegistered === false" @toLoginPage="toLoginPage"></register>
        </div>
        <div v-else-if="isLogin === true">
            <sidebar @selectNav="selectNav"></sidebar>
            <contentarea :selectedNav="selectedNav" @editPost="editPost" :selectedArticle="selectedArticle" @save="save" :user="user" @addTag="addTag" @removeTag="removeTag"></contentarea>
        </div>
    </div>
</template>

<script>
import navbar from './components/navbar'
import sidebar from './components/sidebar'
import contentarea from './components/contentarea'
import login from './components/login'
import register from './components/register'
import axios from 'axios'

export default {
    components: {
       navbar,
       sidebar,
       contentarea,
       login,
       register
    },
    data() {
       return {
           selectedNav: 'dashboard',
           articles: [],
           selectedArticle: {
               tags: [],
               image: ''
           },
           tags: [],
           isLogin: false,
           isRegistered: true,
           user: {},
           alert: {},
           server: 'http://localhost:3000',
       }
    },
    methods: {
       selectNav(selectedNav) {
           this.selectedNav = selectedNav
           this.selectedArticle = {}
           this.selectedArticle.tags = []
       },
       editPost(selectedArticle) {
           this.selectedArticle = selectedArticle
           this.selectedNav = 'postdetail'
       },
       save() {
           this.selectedNav = 'posts'
       },
       toLoginPage(alert) {
           this.isRegistered = !this.isRegistered
           this.alert = alert
       },
       login(user) {
           this.isLogin = true
           this.user = user
       },
       logout() {
           this.isLogin = false
           this.alert.status = 'bye'
           this.user = {}
       },
       addTag(tagInput) {
           console.log(this.tags);
           console.log('masuk xxx app', tagInput);
           this.tags.push(tagInput)
           console.log(this.tags);
       },
       removeTag(tag) {
            for (let i = 0; i < this.selectedArticle.tags.length; i++){
                if (this.selectedArticle.tags[i] === tag) {
                    this.selectedArticle.tags.splice(i, 1)
                    break
                }
            }
       },
       checkToken() {

           axios({
                method: `post`,
                url: `${this.server}/users/checkToken`,
                headers: {
                    token: localStorage.getItem('token')
                }
            })
                .then(({ data }) => {
                    console.log('berhasil');
                })
       }
    },
    created: function() {
        if(localStorage.getItem('token')) {
            this.isLogin = true
            // this.checkToken()
        }
        selectedNav: 'dashboard'
    }
}
</script>

<style>

</style>