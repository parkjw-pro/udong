<template>
  <div id="app" class="pb-5">  
    <div v-if="login"><Navbar /></div>
    <div>
      <router-view ></router-view>
    </div>   
  </div>
</template>

<script>
import Navbar from '@/components/app/Navbar'
import 'vue-select/dist/vue-select.css';

export default {
  name: 'App',
  components: {
    Navbar,
  },
  data: function () {
    return {
      login: false,
    }
  },
  methods: {
  },
  mounted() {
    if (localStorage.getItem('auth-token') != undefined) {
        this.login = true
    }
  // router.push( "/account");
  }
}
</script>

<style>
/* 1번 폰트 - Jeju Gothic */
@import url(//fonts.googleapis.com/earlyaccess/jejugothic.css);
/* 2번 폰트 - 나눔펜 스크립트 */
@import url('https://fonts.googleapis.com/css2?family=Nanum+Pen+Script&display=swap');
/* 3번 폰트 - Hanna */
@import url(//fonts.googleapis.com/earlyaccess/hanna.css);

@import url('https://fonts.googleapis.com/css2?family=Jua&display=swap');

#app {
  font-family: 'Jeju Gothic', sans-serif;
  /* font-family: 'Nanum Pen Script', cursive; */
  /* font-family: 'Hanna', sans-serif; */
  /* 기본설정 */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}


/* --------------------------------------------------하위 컴포넌트에서 사용-------------------------------------------------- */

/* 페이지를 담는 박스 */
#box {
  display: block;
  width: 50%;
  position: absolute;
  left: 25%;
  margin-top: 0%;
  padding-bottom: 7%;
}

#top_title {
  color: #695549;
  font-family: 'Nanum Pen Script', cursive;
}

</style>
