# Vue.js × Server site  
  
## Firebase Auth
  
### User Profile Page 
以降の作業を行う前に、TOPページ(投稿一覧)が準備できていること。  
  
---  
1. `[プロジェクト]/pages/mypage.vue`を作成、以下のようにコードを記述。  
  
**_userId.vue**  
```html
<template>
  <div>
    <h2>ユーザ詳細</h2>
    <div>
      <p>ユーザ名</p>
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
---  
  
2. `[プロジェクト]/pages/users/mypage.vue`を更新。firebaseをインポートして、`fetchUser`メソッドを追加。  
  
**mypage.vue**  
```html  
<template>
  <div>
    <h2>ユーザ詳細</h2>
    <div>
      <p>{{ displayName }}</p>
    </div>
    <nuxt-link to="/">トップへ戻る</nuxt-link>
  </div>
</template>

<script>
import firebase from 'firebase'

export default {
  asyncData(){
    return {
      displayName: ''
    }
  },
  created(){
    this.fetchUser()
  },
  methods: {
    fetchUser(){
      firebase.auth().onAuthStateChanged(user => {
        if(user){
          this.displayName = user.displayName
        }
      })
    }
  }
}
</script>

<style>
</style>
```  
