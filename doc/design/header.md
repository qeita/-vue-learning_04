# Vue.js × Server site  
  
## Design - Header  
  
1. `[プロジェクト]/components/Header.vue`を作成、以下のように記述。 `[プロジェクト]/layouts/default.vue`でインポートして使う。  
  
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

      <v-btn flat>
        <nuxt-link class="link" to="/mypage">
          My Page
        </nuxt-link>
      </v-btn>

      <v-btn flat>
        <nuxt-link class="link" to="/login">
          Login
        </nuxt-link>
      </v-btn>

      <v-btn flat @click="logOut">Logout</v-btn>
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
  
**default.vue**  
```html  
<template>
  <v-app>
    <Header />

    <v-content>
      <v-container fluid>
        <nuxt />
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import Header from '~/components/Header'

export default {
  components: {
    Header
  }
}
</script>
```

