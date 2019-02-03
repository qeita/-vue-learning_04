# Vue.js × Server site  
  
## Firebase Auth
  
### Add auth with Firebase Authentication  
作成したFirebaseのプロジェクト画面より、左メニューから[Authentication]を選択。[Sign-in method]の中からGoogleを選択し、Googleアカウントによるログイン認証を許可(Enable)にする。  
  

### Login page  
---  
1. `[プロジェクト]/pages/login.vue`を作成、以下のようにコードを記述。  
すると、[Google認証でログイン]ボタン押下後「ログイン中」が表示される。  
  
**login.vue**  
```html
<template>
  <div>
    <div v-if="isLogin">
      <p>{{ user.displayName }}</p>
      <p>ログイン中です</p>
    </div>
    <button @click="googleSignIn">Google認証でログイン</button>
  </div>    
</template>

<script>
import firebase from 'firebase'

export default {
  asyncData(){
    return {
      user: null,
      isLogin: false
    }
  },
  created(){
    // Google認証でログインしているかチェック
    firebase.auth().onAuthStateChanged(user => {
      console.log(user);
      if(user){
        this.user = user
        this.isLogin = true
      }
    })
  },
  methods: {
    // Google認証によるログイン ※初回は許可するか確認画面が表示
    googleSignIn(){
      firebase.auth().signInWithRedirect( new firebase.auth.GoogleAuthProvider() )
    }
  }    
}
</script>

<style>
</style>
```  
---    

2. `[プロジェクト]/pages/login.vue`にログアウトボタンの設置及びメソッドを追加。合わせて`isLogin`の切り替えに合わせてボタンを表示・非表示に。  
  
**login.vue**  
```html
<template>
  <div>
    <div v-if="isLogin">
      <p>{{ user.displayName }}</p>
      <p>ログイン中です</p>
    </div>
    <button @click="googleSignIn">Google認証でログイン</button>
  </div>    
</template>

<script>
import firebase from 'firebase'

export default {
  asyncData(){
    return {
      user: null,
      isLogin: false
    }
  },
  created(){
    // Google認証でログインしているかチェック
    firebase.auth().onAuthStateChanged(user => {
      console.log(user);
      if(user){
        this.user = user
        this.isLogin = true
      }
    })
  },
  methods: {
    // Google認証によるログイン ※初回は許可するか確認画面が表示
    googleSignIn(){
      firebase.auth().signInWithRedirect( new firebase.auth.GoogleAuthProvider() )
    },
    // 認証解除してリロード
    logOut(){
      firebase.auth().signOut()
      window.location.reload()
    }
  }    
}
</script>

<style>
</style>
```  