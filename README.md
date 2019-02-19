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

## まとめ

[**Github**](https://github.com/yuheijotaki/vue-study_20190219)

そのほか参考リンク

- [Vue\.js で要素をフィルタリングするシンプルなUIを作ってみた \| WEBMAN](https://webman-japan.com/playground/vue-simple-filiter/)

- [クラスとスタイルのバインディング — Vue\.js](https://jp.vuejs.org/v2/guide/class-and-style.html)

- [Wordpress REST API の出力結果を整形して絞り込む方法 \- Qiita](https://qiita.com/kinshist/items/c131a1ec9cedb34f54ec)