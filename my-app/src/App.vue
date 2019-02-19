<template>
  <div id="app">
    <header>
      <h1>works.yuheijotaki.com</h1>
      <nav>
        <ul>
          <li><a href="javascript:void(0);" v-on:click="filterCategory" data-category="All">All</a></li>
          <li><a href="javascript:void(0);" v-on:click="filterCategory" data-category="Front-end">Front-end</a></li>
          <li><a href="javascript:void(0);" v-on:click="filterCategory" data-category="WordPress">WordPress</a></li>
          <li><a href="javascript:void(0);" v-on:click="filterCategory" data-category="Web Design">Web Design</a></li>
          <li><a href="javascript:void(0);" v-on:click="filterCategory" data-category="Tumblr">Tumblr</a></li>
        </ul>
      </nav>
    </header>
    <main>
      <ul>
        <li v-for="(post,index) in posts" :key="index" v-show="post.customData.display">
          <a v-bind:href="post.acf.post_url" target="_blank">
            <figure><img v-bind:src="post.images.full" v-bind:alt="post.title.rendered"></figure>
            <div class="wrap" v-bind:style="{ color: post.acf.post_color_letter, background: post.acf.post_color_bg }">
              <div class="inner">
                <h2>{{post.title.rendered}}</h2>
                <p>{{post.category_name}}</p>
              </div>
            </div>
          </a>
        </li>
      </ul>
    </main>
  </div>
</template>

<script>
// normalize.css を読み込む
import "normalize.css";
// Ajax通信ライブラリ
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      posts: []
    }
  },
  created: function(){
    this.request();
  },
  methods: {
    request: function(){
      axios.get( 'https://works.yuheijotaki.com/wp-json/wp/v2/posts?per_page=100' )
      .then( response => {
        this.posts = response.data;
        console.log(this.posts);
      })
      .catch( error => {
        console.log(error);
      });
    },
    filterCategory: function(event) { // カテゴリーがクリックされたとき用のメソッド
      const posts = this.posts;
      const selectedCategory = event.currentTarget.getAttribute('data-category'); // クリックしたカテゴリーの取得
      for (var i = 0; i < posts.length; i++) { // 投稿ごとのループ
        const categories = posts[i].category_name; // 投稿のカテゴリーを取得
        const categoriesArray = categories.split(' ,'); // 取得したカテゴリーを配列に変換
        for (var j = 0; j < categoriesArray.length; j++) { // 投稿内のカテゴリーごとのループ
          if ( categoriesArray.indexOf(selectedCategory) >= 0 ) { // 投稿に属するカテゴリーが含まれる場合
            posts[i].customData.display = true;
            break;
          } else { // マッチしない場合
            posts[i].customData.display = false;
          }
        }
      }
    }
  }
};
</script>

<style lang="scss" scoped>
html,* {
  margin: 0;
  padding: 0
}
#app {
  font-family: Helvetica Neue, Helvetica, Arial, 'Hiragino Kaku Gothic ProN', Meiryo, sans-serif;
  -webkit-text-size-adjust: 100%;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
  font-feature-settings : "palt";
  max-width: 800px;
  margin: 40px auto;
  letter-spacing: .01em;
  header {
    display: flex;
    align-items: flex-end;
    h1 {
      color: #222;
      font-size: 20px;
      line-height: 1.2;
    }
    nav {
      margin-left: auto;
      ul {
        display: flex;
        list-style: none;
        li {
          font-size: 13px;
          line-height: 1.2;
          margin-right: 20px;
          &:last-child {
            margin-right: 0;
          }
          a {
            color: #222;
            text-decoration: none;
          }
        }
      }
    }
  }
  main {
    margin-top: 40px;
    ul {
      list-style: none;
      display: flex;
      flex-wrap: wrap;
      li {
        width: 25%;
        a {
          display: block;
          text-decoration: none;
          position: relative;
          figure {
            font-size: 0;
            line-height: 0;
            img {
              width: 100%;
              height: auto;
            }
          }
          .wrap {
            display: none;
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            .inner {
              position: absolute;
              top: 50%;
              left: 50%;
              transform: translate(-50%, -50%);
              width: 100%;
              padding: 0 10px;
              box-sizing: border-box;
              text-align: center;
              letter-spacing: 0.025em;
              h2 {
                font-size: 13px;
                line-height: 1.2;
              }
              p {
                margin-top: .2em;
                font-size: 10px;
                line-height: 1.4;
              }
            }
          }
          &:hover {
            .wrap {
              display: block;
            }
          }
        }
      }
    }
  }
}
</style>
