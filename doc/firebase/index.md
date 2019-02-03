# Vue.js × Server site  
  
## Firebase Realtime Database
  
### Add Realtime Database  
作成したFirebaseのプロジェクト画面より、左メニューから[Database]を選択。[Realtime Database]を選択し、データ参照・書き込みルールをtest modeに設定。  
  

### Post Index page  
---  
1. `[プロジェクト]/pages/index.vue`を作成、以下のようにコードを記述。  
`fetchAllPosts`, `setPosts`メソッドによってRealtime Databaseを参照・描画し、`addComment`メソッドでデータベースに書き込みを行なっている。
  
**index.vue**  
```html
<template>
  <div>
    <div>
      <textarea v-model="comment"></textarea>
      <button @click="addComment">投稿</button>
    </div>
    <div>
      <div v-for="(post, i) in posts" :key="i">
        <p>{{ post.comment }}</p>
        <p>{{ post.created }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import firebase from 'firebase'

export default {
  asyncData(){
    return {
      uid: '',
      comment: '',
      posts: []
    }
  },
  created(){
    this.db = firebase.database().ref('posts/')
    this.fetchAllPosts()
  },
  methods: {
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
    },
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
---  

2. `[プロジェクト]/pages/index.vue`に`checkLogin`メソッドを追加。ログイン中のみ投稿フォームを表示するよう設定。
  
**index.vue**  
```html
<template>
  <div>
    <div v-if="uid">
      <textarea v-model="comment"></textarea>
      <button @click="addComment">投稿</button>
    </div>
    <div>
      <div v-for="(post, i) in posts" :key="i">
        <p>{{ post.comment }}</p>
        <p>{{ post.created }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import firebase from 'firebase'

export default {
  asyncData(){
    return {
      uid: '',
      comment: '',
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
    },
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
---  

3. `[プロジェクト]/pages/index.vue`の各投稿エリアに`<nuxt-link :to="...">`を使って導線を作成
  
**index.vue**  
```html  
<template>
  <!-- 投稿エリアのみ抜粋 -->
  <nuxt-link v-for="(post, i) in posts" :key="i" :to="`/posts/${post.id}`">
    <p>{{ post.comment }}</p>
    <p>{{ post.created }}</p>
  </nuxt-link>
</template>
``` 
---   
  
4. 投稿フォーム・各投稿エリアをコンポーネントとして別ファイルで管理するため、`[プロジェクト]/components/PostForm.vue`、`[プロジェクト]/components/Post.vue`を作成。`[プロジェクト]/pages/index.vue`はそれをインポートする記述に置換。  
  
**PostForm.vue**  
```html  
<template>
  <div>
    <nuxt-link :to="`/users/${uid}`">マイページ</nuxt-link>
    <textarea v-model="comment"></textarea>
    <button @click="addComment">投稿</button>
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
  
**Post.vue**  
```html  
<template>
  <div>
    <nuxt-link :to="`/posts/${post.id}`">
      <p>{{ post.comment }}</p>
      <p>{{ post.created }}</p>
    </nuxt-link>    
  </div>
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

<style>
</style>
```  
  
**index.vue**  
```html  
<template>
  <div>
    <PostForm v-if="uid" :uid="uid" />
    <div>
      <Post v-for="(post, i) in posts" :key="i" :post="post" />
    </div>
  </div>
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
</style>  
```
