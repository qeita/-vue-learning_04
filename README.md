# Vue.js × Server site  
  
## Firebase  
  
## System Architecture  
サーバ周りで必要となる「データの永続化(データベース)」と「ユーザ認証」をFirebaseが捌き、フロントエンドをNuxt.jsで対応。    
  
- Nuxt.js(v2.4.0)  
- Firebase / リアルタイムデータベース(RTDB) & 認証(Auth)    
- Vuetify  
  
## Product  
Twitterライクな投稿系アプリケーション  
  
必要ページは、
- TOPページ(投稿一覧)  
`pages/index.vue`  

- ログインページ  
`pages/login.vue`  
  
- 投稿詳細ページ  
`pages/posts/_postId.vue`  
  
- マイページ  
`pages/mypage.vue`  
  
## How to make  
### Installation for Nuxt.js 
`create-nuxt-app`でNuxt環境をインストール。  
  
1. Nuxtプロジェクトを以下コマンドで作成(`npx` or `yarn`)。 
その際、UIフレームワークに[vuetify]を指定。
```  
# npx:  
npx create-nuxt-app [project name]  
  
# yarn:  
yarn create nuxt-app [project name]  
```  
  
2. プロジェクトに移動して、ローカルサーバを起動。  
```  
cd [project name]  
  
npm run dev  
```  
  
### Create Project in Firebase  
Firebaseのターミナルに移動、プロジェクトを作成します。  
  

### Create Firebase plugin and use it 
1. firebaseのパッケージをインストール  
```  
npm install firebase --save  
```  
2. `[プロジェクト]/plugins/`配下に firebase.js を作成。  
その際、作成したFirebaseのプロジェクトから発行したAPIキー他を使用。  
  
**firebase.js**  
```js  
import firebase from 'firebase'

if (!firebase.apps.length) {
  // ここから - Firebaseより発行したconfig情報及びそれを使ったイニシャライズ部分をコピー
  var config = {
    apiKey: "-----",
    authDomain: "-----",
    databaseURL: "-----",
    projectId: "-----",
    storageBucket: "-----",
    messagingSenderId: "-----"
  };
  firebase.initializeApp(config);
  // ここまで
}
export default firebase  
```
  
3. プロジェクト直下の nuxt.config.js を編集。`plugins`キーに先ほど作成したfirebase.jsのパスを追加。  
  
**nuxt.config.js**  
```js  
module.exports = {
  /*
  ** Plugins to load before mounting the App
  */
  plugins: [
    '@/plugins/vuetify',
    '@/plugins/firebase'
  ],
}
```
  

### 実装: ページ詳細    
1. ログインページは、[login.md](./doc/firebase/login.md) を参照。  
  
2. TOPページ(投稿一覧)は、[index.md](./doc/firebase/index.md)を参照。  
  
3. 投稿詳細ページは、[postId.md](./doc/firebase/postId.md)を参照。  
  
4. マイページは、[mypage.md](./doc/firebase/mypage.md)を参照。 
  
### デザイン: Vuetifyによるレイアウト・UI調整    
1. レイアウトは、[layout.md](./doc/design/layout.md) を参照。  
  
2. ヘッダーは、[header.md](./doc/design/header.md) を参照。  
  
3. 投稿フォーム・投稿エリアは、[post.md](./doc/design/post.md) を参照。  


参考)  
[https://qiita.com/redshoga/items/da5c0e247e0df314a257](https://qiita.com/redshoga/items/da5c0e247e0df314a257)  
  
[https://www.non-standardworld.co.jp/nuxt-js-firebase/20548/]
(https://www.non-standardworld.co.jp/nuxt-js-firebase/20548/)  