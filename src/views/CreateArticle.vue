<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8 offset-md-2">
                <div class="card my-5">
                    <div class="card-body">
                        <picture-input 
                        ref="pictureInput" 
                        accept="image/jpeg,image/png" 
                        size="5" 
                        button-class="btn btn-danger"
                        @change="onChange" class="my-3"/>
                        <select name="" id="" class="form-control mb-3" v-model="category">
                            <option selected>Select a Category</option>
                            <option  :value="category.id" :key="category.id" v-for="category in categories"> {{category.name}} </option>
                        </select>
                        <input type="text" v-model="title" placeholder="Title" class="form-control mb-3">
                        <wysiwyg v-model="content" />
                        <button :disabled="loading" @click="createArticle()" class="btn btn-success btn-lg mt-3">
                            <i class="fas fa-spin fa-spinner" v-if="loading"></i> 
                            {{ loading ? '' : 'Create Article' }}
                            </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import PictureInput from 'vue-picture-input'
    import Axios from 'axios';
    import config from '@/config';

    export default {
        beforeRouteEnter(to, from, next){
            if(!localStorage.getItem('auth')){
                return next({ path: '/login'});
            }
            next();
        },
        mounted(){
            this.getCategories();
        },
        components: {
            PictureInput
        },
        data() {
            return {
                title: '',
                content: '',
                image: null,
                categories: [],
                category:'',
                loading: false
            }
        },
        methods: {
            onChange(image){
                this.image = image;
            },
            createArticle(){

                if (!this.title || !this.image || !this.category || !this.content) {
                    this.$noty.error('Please fill in all fields.');
                    return;
                }

                this.loading = true;
                const form = new FormData();
                form.append('file', this.image);
                form.append('upload_preset', process.env.VUE_APP_CLOUDINARY_PRESET);
                form.append('api_key',process.env.VUE_APP_CLOUDINARY_API_KEY);

                Axios.post(process.env.VUE_APP_CLOUDINARY_URL, form).then(res => 
                    Axios.post(`${config.apiUrl}/articles`,{
                        title: this.title,
                        content: this.content,
                        category_id: this.category,
                        imageUrl: res.data.secure_url
                    },{
                        headers: {
                            Authorization: `Bearer ${this.$root.auth.token}`
                        }
                    }).then(()=>{
                        this.loading = false;
                        this.$noty.success("Article created successfully.");
                        this.$router.push('/');
                    }).catch(()=>{
                        this.loading = false;
                        this.$noty.error('Oops! Something went wrong!');
                    })
                ).catch(()=>{
                        this.loading = false;
                        this.$noty.error('Oops! Something went wrong!');
                    });
            },
            getCategories(){
                const categories = localStorage.getItem('categories');
                if (categories) {
                    this.categories = JSON.parse(categories);
                    return;
                }
                Axios.get(`${config.apiUrl}/categories`).then(res => {
                    this.categories = res.data.categories;
                    localStorage.setItem('categories', JSON.stringify(this.categories));
                });
            }
        }
    };
</script>
