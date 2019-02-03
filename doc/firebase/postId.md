# Vue.js × Server site  
  
## Firebase Realtime Database
  
### Post Detail Page 
以降の作業を行う前に、TOPページ(投稿一覧)が準備できていること。  
  
---  
1. `[プロジェクト]/pages/posts/_postId.vue`を作成、以下のようにコードを記述。`asyncData`メソッドでコンテキストよりルーティングのパラメータを取得できるので、その値を`postId`に設定。    
  
**_postId.vue**  
```html
<template>
  <div>
    <p>{{ postId }}</p>
    <nuxt-link to="/">トップへ戻る</nuxt-link>
  </div>
</template>

<script>
export default {
  asyncData({params}){
    console.log(params)
    return {
      postId: params.postId
    }
  }   
}
</script>

<style>
</style>
```  
---  
  
2. `[プロジェクト]/pages/posts/_postId.vue`を更新。firebaseをインポートして、`fetchPost`メソッドを追加。コメント部分をtextareタグの`v-model`で受け取れるよう設定。  
  
**_postId.vue**  
```html  
<template>
  <div>
    <h2>投稿詳細</h2>
    <div>
      <textarea v-model="comment"></textarea>
      <button>更新</button>
    </div>
    <nuxt-link to="/">トップへ戻る</nuxt-link>
  </div>
</template>

<script>
import firebase from 'firebase'

export default {
  asyncData({params}){
    return {
      postId: params.postId,
      comment: ''
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
          console.log(result.val())
          this.comment = result.val().comment
        })
    }
  }
}
</script>

<style>
</style>
```  
---    

3. `[プロジェクト]/pages/posts/_postId.vue`に`checkLogin`を追加して、自身の投稿のみ(ユーザIDが一致している投稿のみ)編集可能にする。また、更新ボタンに`update`メソッドを追加して、当該投稿に対するコメント・更新日時を上書きするよう設定。    
  
**_postId.vue**  
```html
<template>
  <div>
    <h2>投稿詳細</h2>
    <div>
      <textarea v-model="post.comment" :disabled="uid !== post.uid"></textarea>
      <button @click="update" v-if="uid === post.uid">更新</button>
    </div>
    <nuxt-link to="/">トップへ戻る</nuxt-link>
  </div>
</template>

<script>
import firebase from 'firebase'

export default {
  asyncData({params}){
    return {
      uid: '',
      postId: params.postId,
      post: ''
    }
  },
  created(){
    this.db = firebase.database().ref(`posts/${ this.postId }`)
    this.checkLogin()
    this.fetchPost()
  },
  methods: {
    checkLogin(){
      // Google認証でログインしているかチェック
      firebase.auth().onAuthStateChanged(user => {
        if(user){
          this.uid = user.uid
          console.log(this.uid)
        }
      })
    },
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