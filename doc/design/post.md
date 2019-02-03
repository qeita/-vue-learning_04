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
---  
  
2. 投稿エリアをラップした形で`[プロジェクト]/components/PostGroup.vue`を作成し、以下のように記述。合わせて、`[プロジェクト]/pages/index.vue`も修正。  
  
**PostGroup.vue**  
```html  
<template>
  <div>
    <Post v-for="(post, i) in posts" :key="i" :post="post" />
  </div>
</template>

<script>
import Post from './Post'

export default {
  components: {
    Post
  },
  props: {
    posts: {
      type: Array,
      default: []
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
    <PostGroup :posts="posts" />
  </v-flex>
</template>

<script>
import firebase from 'firebase'

// component
import PostForm from '~/components/PostForm'
import PostGroup from '~/components/PostGroup'

export default {
  components: {
    PostForm,
    PostGroup
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
</style>
```
  
備考)  
以下のように`[プロジェクト]/components/Post.vue`を修正して見た目をカスタマイズするのも。  
以下サイトからダウンロードした画像3種を`bg_01.png`, `bg_02.png`, `bg_03.png`にリネームして、`[プロジェクト]/static/assets/img/`ディレクトリを作ってそこに格納。    
[http://fukidesign.com/wp/?p=883](http://fukidesign.com/wp/?p=883)  
  
**Post.vue**  
```html  
<template>
  <nuxt-link :class="`link link_${bg}`" :to="`/posts/${post.id}`">
    <v-card-title primary-title>
      <div>
        <h3 class="headline">{{ post.comment }}</h3>
        <p>{{ post.created }}</p>
      </div>
    </v-card-title>
  </nuxt-link>    
</template>

<script>
export default {
  props: {
    post: {
      type: Object,
      default: {}
    }
  },
  data(){
    const v = Math.ceil(Math.random() * 3) 
    return {
      bg: v
    }
  }
}
</script>

<style scoped>
.link{
  text-decoration: none;
  padding: 50px 0;
  text-align: center;
  font-weight: bold;
  color: #999;
  display: block;
  transition: 0.2s all ease-out;
}
.link_1{
  background: url(/assets/img/bg_01.png) no-repeat 50% 50%;
  background-size: contain;
}
.link_2{
  background: url(/assets/img/bg_02.png) no-repeat 50% 50%;
  background-size: contain;
}
.link_3{
  background: url(/assets/img/bg_03.png) no-repeat 50% 50%;
  background-size: contain;
}
.link:hover{
  color: #000;
}
.v-card__title{
  display: flex;
  justify-content: center;
}
</style>
```

