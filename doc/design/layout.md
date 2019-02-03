# Vue.js × Server site  
  
## Design  
  
1. `[プロジェクト]/layouts/default.vue`を以下のように修正。  
  
**default.vue**  
```html
<template>
  <v-app>
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
export default {
}
</script>
```  
