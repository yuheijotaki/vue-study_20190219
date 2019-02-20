# Vue.js / JSON から情報を引っ張ってくる その6

## やること

- [ポートフォリオページ](https://works.yuheijotaki.com) のコンテンツのエンドポイントを WP REST API を用いて作成
- カテゴリー絞り込みを実装する

### WordPress側（functions.php）でのAPIへのフィールド追加

 `display` という自前のデータを `v-show:` を使って判定させるため下記をfunctions.phpに追加。

```php
// ----------------------------------------
// `customData.display` のAPI登録
// ----------------------------------------
function register_my_custom_data() {
	register_rest_field(
		'post',
		'customData',
		array(
			'get_callback'    => 'get_my_custom_data',
			'update_callback' => null,
			'schema'          => null,
		)
	);
}
add_action( 'rest_api_init', 'register_my_custom_data' );

function get_my_custom_data() {
	return array(
		'display'  => true
	);
}
```

これでJSON側には

```json
"customData":{
  "display":true
}
```

のように出力される。

### 静的なクラスと動的なクラス

```html
<a :class="['naviLink' , { 'is-selected': category.selected }]">...</a>
```

とすると、 `.naviLink` というクラスは付きつつも、`.is-selected` は `selected` は `true` の場合だけつくようになる。

## App.vue

```javascript
<template>
  <div id="app">
    <header>
      <h1>works.yuheijotaki.com</h1>
      <nav>
        <ul>
          <li v-for="(category,index) in categories" :key="index">
            <a href="javascript:void(0);" @click="filterCategory" :data-category="category.name" :class="['naviLink' , { 'is-selected': category.selected }]">{{category.name}}</a>
          </li>
        </ul>
      </nav>
    </header>
    <main>
      <ul>
        <li v-for="(post,index) in posts" :key="index" v-show="post.customData.display">
          <a :href="post.acf.post_url" target="_blank">
            <figure><img :src="post.images.full" :alt="post.title.rendered"></figure>
            <div class="wrap" :style="{ color: post.acf.post_color_letter, background: post.acf.post_color_bg }">
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
      categories: [
        {
          name: 'All',
          selected: true
        },
        {
          name: 'Front-end',
          selected: false
        },
        {
          name: 'WordPress',
          selected: false
        },
        {
          name: 'Web Design',
          selected: false
        },
        {
          name: 'Tumblr',
          selected: false
        }
      ],
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
        // console.log(this.posts);
      })
      .catch( error => {
        console.log(error);
      });
    },
    filterCategory: function(event) { // カテゴリーがクリックされたとき用のメソッド
      // 全体のナビゲーションのクラス削除
      var targetElements = document.getElementsByClassName('naviLink');
      [].forEach.call(targetElements, function(elem) {
        elem.classList.remove('is-selected');
      });
      // 選択したナビゲーションのクラス付与
      event.currentTarget.classList.add('is-selected');
      // 投稿の取得
      const posts = this.posts;
      const selectedCategory = event.currentTarget.getAttribute('data-category'); // クリックしたカテゴリーの取得
      if ( selectedCategory !== 'All' ) {
        // `All` 以外を選択した場合
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
      } else {
        // `All` を選択した場合
        for (var i = 0; i < posts.length; i++) { // 投稿ごとのループ
          posts[i].customData.display = true; // すべての投稿の `display` を `true` に
        }
      }
    }
  }
};
</script>
```

## まとめ

[**Github**](https://github.com/yuheijotaki/vue-study_20190219)

- 基本的な構文への慣れはだいぶできた気がする。理解して使いこなせているかというとまだまだですが。。
- やっぱりドキュメントは公式のが一番いいですね。ベースは公式を参考して発展的な使い方はググるのがいまのところ近道な気がします。
- カテゴリーのクラス付与/削除はいまクラスの `.add` と  `.remove` でやってしまってますが、本来は `categories.selected` の値をいじるほうがよさそう。
- URLが変更しないので変更できるようにしたい。

そのほか参考リンク

- [クラスとスタイルのバインディング — Vue\.js](https://jp.vuejs.org/v2/guide/class-and-style.html)
- [Vue\.js で要素をフィルタリングするシンプルなUIを作ってみた \| WEBMAN](https://webman-japan.com/playground/vue-simple-filiter/)
- [Wordpress REST API の出力結果を整形して絞り込む方法 \- Qiita](https://qiita.com/kinshist/items/c131a1ec9cedb34f54ec)