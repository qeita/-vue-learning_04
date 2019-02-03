# Vue.js × Server site  
  
## Design - Post / PostForm 
  
1. `[プロジェクト]/components/Post.vue`、`[プロジェクト]/components/PostForm.vue`、`[プロジェクト]/pages/index.vue`を以下のように修正。  
  
**Post.vue**  
```html
<template>
  <v-hover>
    <v-card
      slot-scope="{ hover }"
      class="mt-4"
      :class="`elevation-${hover ? 12 : 2}`"
    >
      <nuxt-link class="link" :to="`/posts/${post.id}`">
        <v-card-title primary-title>
          <div>
            <h3 class="headline">{{ post.comment }}</h3>
            <p>{{ post.created }}</p>
          </div>
        </v-card-title>
      </nuxt-link>    
    </v-card>
  </v-hover>
</template>

<script>
export default {
  props: {
    post: {
      type: Object,
      default: {}
    }
  }    
}
</script>

<style scoped>
.link{
  text-decoration: none;
  color: #999;
  transition: 0.2s all ease-out;
}
.link:hover{
  color: #000;
}
</style>
```  
  
**PostForm.vue**  
```html
<template>
  <div>
    <v-text-field
      label="Comment"
      box
      v-model="comment"
    ></v-text-field>
    
    <v-btn
      depressed
      large
      color="primary"
      @click="addComment"
    >
      投稿
    </v-btn>
  </div>
</template>

<script>
import firebase from 'firebase'

export default {
  props: {
    uid: {
      type: String,
      default: ''
    }
  },
  data(){
    return {
      comment: ''
    }
  },
  created(){
    this.db = firebase.database().ref('posts/')
  },
  methods: {
    getCurrentDate(){
      const date = new Date()
      return `${date.getFullYear()}/${date.getMonth()+1}/${date.getDate()} ${date.getHours()}:${date.getMinutes()}:${date.getSeconds()}`
    },
    addComment(){
      if(!this.comment) return
      this.db.push({
        comment: this.comment,
        uid: this.uid,
        created: this.getCurrentDate(),
        updated: this.getCurrentDate()
      })
      this.comment = ''
    }
  }    
}
</script>

<style>
</style>
```
  
**index.vue**  
```html  
<template>
  <v-flex>
    <PostForm v-if="uid" :uid="uid" />
    <div>
      <Post v-for="(post, i) in posts" :key="i" :post="post" />
    </div>
  </v-flex>
</template>

<script>
import firebase from 'firebase'

// component
import PostForm from '~/components/PostForm'
import Post from '~/components/Post'

export default {
  components: {
    PostForm,
    Post
  },
  asyncData(){
    return {
      uid: '',
      posts: []
    }
  },
  created(){
    this.db = firebase.database().ref('posts/')
    this.checkLogin()
    this.fetchAllPosts()
  },
  methods: {
    checkLogin(){
      // Google認証でログインしているかチェック
      firebase.auth().onAuthStateChanged(user => {
        if(user){
          this.uid = user.uid
        }
      })
    },
    fetchAllPosts(){
      this.db
        .on('child_added', (result) => {
          this.setPosts(result)
        })
    },
    setPosts(postData){
      const obj = postData.val()
      obj.id = postData.key
      this.posts.unshift(obj)
    }
  }
}
</script>

<style>
```

