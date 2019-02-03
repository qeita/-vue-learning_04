# Vue.js × Server site  
  
## Vuex - Store  
  
1. ローカルサーバを停止して、`[プロジェクト]/store/index.js`を作成。ログイン・ユーザ名をストアで管理する。   
**index.js**
```js  
import firebase from 'firebase'

export const state = () => ({
  isLogin: false,
  userName: '',
  uid: ''
})


export const getters = {
  isLogin(state){
    return state.isLogin
  },
  getUserName(state){
    return state.userName
  },
  getUserId(state){
    return state.uid
  }
}

export const mutations = {
  setLogin(state, obj){
    state.isLogin = obj? true: false
    state.userName = obj.displayName
    state.uid = obj.uid
  }
}

export const actions = {
  setLoginState(context){
    firebase.auth().onAuthStateChanged(user => {
      if(user){
        context.commit('setLogin', user)
      }
    })
  }
}
```  
---  
  
2. `[プロジェクト]/layouts/default.vue`でユーザログイン確認を行うaction`setLoginState`を実行。    
  
**default.vue**  
```html  
<template>
  <v-app>
    <Header />

    <v-content>
      <v-container>
        <v-layout justify-center>
          <nuxt />
        </v-layout>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import Header from '~/components/Header'

export default {
  components: {
    Header
  },
  created(){
    this.$store.dispatch('setLoginState')
  }
}
</script>
```  
---  

3. ログイン状態を参照して切り替える以下ファイルを修正。  
  
- `[プロジェクト]/components/Header.vue`  
- `[プロジェクト]/pages/index.vue`  
- `[プロジェクト]/pages/posts/_postId.vue`
- `[プロジェクト]/pages/mypage.vue`  
- `[プロジェクト]/pages/login.vue`  
  
**Header.vue**  
```html  
<template>
  <v-toolbar dark app>
    <v-toolbar-title>FUKIDASU</v-toolbar-title>
    <v-spacer></v-spacer>
    <v-toolbar-items class="hidden-sm-and-down">    
      <v-btn flat>
        <nuxt-link class="link" to="/">
          Home
        </nuxt-link>
      </v-btn>

      <v-btn
        flat
        v-if="$store.getters.isLogin"
      >
        <nuxt-link class="link" to="/mypage">
          My Page
        </nuxt-link>
      </v-btn>

      <v-btn
        flat
        v-if="!$store.getters.isLogin"
      >
        <nuxt-link class="link" to="/login">
          Login
        </nuxt-link>
      </v-btn>

      <v-btn
        flat
        @click="logOut"
        v-if="$store.getters.isLogin"
      >Logout</v-btn>
    </v-toolbar-items>
  </v-toolbar>
</template>
<script>
import firebase from 'firebase'

export default {
  name: 'Header',
  methods: {
    // 認証解除してリロード
    logOut(){
      firebase.auth().signOut()
      window.location.reload()
    }
  }
}
</script>
<style scoped>
.link{
  color: #fff;
  text-decoration: none;
}
</style>
```
  
**index.vue**  
```html  
<template>
  <v-flex>
    <PostForm v-if="$store.getters.isLogin" />
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
    }
  }
}
</script>

<style>
</style>
```  
  
**_postId.vue**
```html
<template>
  <v-flex>
    <h2>投稿詳細</h2>
    <div class="mb-4">
      <v-text-field
        label="Comment"
        box
        v-model="post.comment"
        :disabled="$store.getters.getUserId !== post.uid"
      ></v-text-field>

      <v-btn
        depressed
        large
        color="primary"
        @click="update"
        v-if="$store.getters.getUserId === post.uid"
      >
        更新
      </v-btn>
    </div>
    <nuxt-link class="mt-4" to="/">トップへ戻る</nuxt-link>
  </v-flex>
</template>

<script>
import firebase from 'firebase'

export default {
  asyncData({params}){
    return {
      postId: params.postId,
      post: ''
    }
  },
  created(){
    this.db = firebase.database().ref(`posts/${ this.postId }`)
    this.fetchPost()
  },
  methods: {
    fetchPost(){
      this.db
        .once('value')
        .then(result => {
          this.post = result.val()
        })
    },
    getCurrentDate(){
      const date = new Date()
      return `${date.getFullYear()}/${date.getMonth()+1}/${date.getDate()} ${date.getHours()}:${date.getMinutes()}:${date.getSeconds()}`
    },
    update(){
      if(!this.post.comment) return
      // 投稿情報をコピー、そのうちコメントと更新日時のみ更新
      const post = Object.assign({}, this.post)
      post.comment = this.post.comment
      post.updated = this.getCurrentDate()
      this.db.set(post)
      this.$router.push('/')
    }
  }
}
</script>

<style>
</style>
```
  
**mypage.vue**  
```html  
<template>
  <div>
    <h2>ユーザ詳細</h2>
    <div>
      <p>{{ $store.getters.getUserName }}</p>
    </div>
    <nuxt-link to="/">トップへ戻る</nuxt-link>
  </div>
</template>

<script>
export default {
}
</script>

<style>
</style>
```  
  
**login.vue**  
```html  
<template>
  <div>
    <div v-if="$store.getters.isLogin">
      <p>{{ $store.getters.getUserName }}</p>
      <p>ログイン中です</p>
    </div>
    <v-btn depressed large color="info" @click="googleSignIn" v-if="!$store.getters.isLogin">Google認証でログイン</v-btn>
    <v-btn depressed large color="info" @click="logOut" v-else>ログアウト</v-btn>
  </div>    
</template>

<script>
export default {
  methods: {
    // Google認証によるログイン ※初回は許可するか確認画面が表示
    googleSignIn(){
      firebase.auth().signInWithRedirect( new firebase.auth.GoogleAuthProvider() )
    },
    // 認証解除してリロード
    logOut(){
      firebase.auth().signOut()
      this.$router.push('/')
    }
  }    
}
</script>

<style>
</style>  
```
