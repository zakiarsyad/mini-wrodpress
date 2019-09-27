<template>
    <div class="container p-20" id="editPost">
        <div v-if="alert.status === 'error'">
            <div class="absolute mt-16 mr-6 top-0 right-0 bg-gray-800 bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded">
                <strong class="font-bold">Holy smokes!</strong>
                <span class="block sm:inline">{{ alert.msg }}</span>
            </div>
        </div>
        <pulseloader v-if="loading === true" class="flex justify-center"></pulseloader>
        <div v-else-if="loading === false">
            <form enctype="multipart/form-data" @submit.prevent="savearticle">
                <input
                    type="text"
                    class="text-3xl mb-3 bg-transparent focus:outline-none"
                    placeholder="Yout title here"
                    v-model="selectedArticle.title">
                <wysiwyg v-model="selectedArticle.content"></wysiwyg>
                <div class="flex mt-2">
                    <div 
                        v-for="(tag, index) in articleTags" 
                        :key="index"
                        class="bg-white rounded mr-2 px-2">
                        {{ tag }}
                        <button @click.prevent="removeTag(index)">
                            <i class="fas fa-times ml-2"></i>
                        </button>
                    </div>
                </div>
                <input
                    type="text"
                    class="text-base my-3 mr-1 p-1 bg-white focus:outline-none rounded"
                    placeholder="Add more tag here"
                    v-model="tagInput">
                <button @click.prevent="addTag">
                    <i class="fas fa-plus"></i>
                </button>
                <br>
                <input
                    type="file"
                    ref="image"
                    accept="image/*"
                    v-on:change="handleimage">
                <div class="flex">
                    <button
                        type="submit"
                        class="mt-3 bg-white hover:bg-gray-100 py-1 px-4 rounded shadow ml-auto">
                        save article
                    </button>
                </div>
            </form>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
import pulseloader from '../pulseloader'

export default {
    components: {
        pulseloader
    },
    props: ['selectedArticle'],
    data() {
        return {
            // server: 'http://34.87.72.235',
            server: 'http://localhost:3000',
            token: localStorage.getItem('token'),
            loading: false,
            alert: {
                status: '',
                msg: []
            },
            tagInput: '',
            image: null,
            articleTags: [],
            editedImage: ''
        }
    },
    methods: {
        savearticle() {
            this.loading = true

            // this.selectedArticle.tags = []

            let formData = new FormData()

            this.articleTags.forEach(el => {
                formData.append("tags", el)
            })

            formData.append("title", this.selectedArticle.title)
            formData.append("content", this.selectedArticle.content)
            console.log(this.editedImage);
            if (this.editedImage) formData.append("image", this.editedImage)

            let method, url

            if(this.selectedArticle._id) {
                method = `patch`
                url = `${this.server}/articles/${this.selectedArticle._id}`
            } else {
                method = `post`
                url = `${this.server}/articles`
            }

            axios({
                method,
                url,
                headers: {
                    token: this.token
                },
                data: formData
            })
                .then(({ data }) => {
                    console.log('from postdetail', data);
                    this.loading = false
                    this.alert.status = 'success'
                    this.alert.msgs = [`success save an article`]
                    this.$emit('save', this.alert)
                })
                .catch(err => {
                    this.loading = false

                    this.loading = false
                    this.alert.status = 'error'

                    this.alert.msg = err.response.data.message

                    // if (this.alert.msg === "Cannot read property 'cloudStoragePublicUrl' of undefined") this.alert.msg = "Please add image"
                })
        },
        addTag() {
            this.articleTags.push(this.tagInput)
            console.log(this.articleTags);
            this.tagInput = ''
        },
        removeTag(index) {
            this.articleTags.splice(index, 1)
        },
        handleimage(){
            let reader = new FileReader()
            reader.readAsDataURL(this.$refs.image.files[0])
            this.editedImage = this.$refs.image.files[0]
        }
    },
    created: function () {
        if (this.selectedArticle.tags) this.articleTags = this.selectedArticle.tags
        else this.articleTags = []
    }
}
</script>

<style>

</style>