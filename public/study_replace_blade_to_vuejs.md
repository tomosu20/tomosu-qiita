---
title: bladeからVue.jsに書き直した話
tags:
  - Laravel
  - Vue.js
private: false
updated_at: '2024-06-26T16:04:33+09:00'
id: f8c9d39d1dbb911b4e06
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

Vue.js での実装方法を一通り学習したが、実際に手を動かして書いてみたいと思い、以前に Udemy の[【Laravel】マルチログイン機能を構築し本格的なECサイトをつくってみよう【Breeze/tailwindcss】](https://www.udemy.com/course/laravel-multi-ec)を受講しながら作成した EC サイトの blade ファイルを Vue.js に書き直してみることにしました。

本記事は、Vue.js に書き換える際に、やったことや学んだことをメモ書きレベルで書き留めているものになります。

## ソースコード

先に対応した結果のリポジトリを貼っておきます。

https://github.com/tomosu20/laravel10-ec-practice

# 各バージョンの更新

Udemy の講座を受講しながら構築した EC サイトは、PHP7.4、Laravel8 にて構築し、フロントエンドのバンドルツールは Laravel Mix を使用しておりました。
Laravel8 にてプロジェクトを作成した際には、デフォルトで Laravel Mix を使用するように構築されるのですが、後述する Vue.js を導入するための Breeze(Inertia)を install すると、バンドルツールは vite を使うようにして install されてしまいました。

vite を laravel mix(webpack)に戻す方法もいくつか散見されましたが、Laravel9.18 以降だと Breeze を install しなくても vite を使用することがデフォルトに変更されてるようでした。
また、今後開発していく上で、上記のバンドルツールの依存性も考慮したうえで、Laravel8 のまま blade を Vue.js に書き換えるケースが少ない気がしており、Laravel のバージョンアップのほうがニーズとしては多いのかなと想定しました。

以上のように自分なりに検討し、vite を使ったほうが今後に活かせそうだなと思い、vite のままで開発を進めていくことにしました。
そのため、Vue.js を導入する前に、各ツールのバージョン更新対応の内容を書き留めております。

## PHP7.4 → 8.2 へ

開発環境は Docker を使用しており、PHP7.4 を入れておりました。
今回 Laravel をアップデートするため、Dockerfile を修正して 10 をサポートしている PHP8.2 に上げました。

```Dockerfile:Dockerfile
FROM php:7.4-apache
↓
FROM php:8.2-apache
```

xdebug の最新バージョンは PHP7.4 で使えなかったので xdebug のバージョン指定していたが、PHP8.2 では最新バージョンが使えるのと、そのままだと build 時にエラーがでるため、バージョン指定を除きました。

```Dockerfile:Dockerfile
pecl install xdebug-3.1.6
↓
pecl install xdebug
```

これで `docker-compose up -d --build` したら環境が立ち上がりました。
下記にて、バージョン PHP 8.2 が入っていることを確認。

```bash
root@622b154614ff:/var/www/html# php -v

Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0
PHP 8.2.20 (cli) (built: Jun 13 2024 06:05:29) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.2.20, Copyright (c) Zend Technologies
    with Xdebug v3.3.2, Copyright (c) 2002-2024, by Derick Rethans
```

## Laravel8 → 9 へ

composer.json にあるバージョン指定を下記のように修正。

```json
"laravel/framework": "^8.40",
↓
"laravel/framework": "^9.0",
```

```json
"nunomaduro/collision": "^5.0",
↓
"nunomaduro/collision": "^6.0", 
```

```json
"facade/ignition": "^2.5",
↓
"spatie/laravel-ignition": "^1.0",
```

修正できたら、`composer update` で更新する。
下記にて、Laravel がアップデートされたことを確認。

```bash
root@622b154614ff:/var/www/html# php artisan -V

Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0
Laravel Framework 9.52.16
```

---

> Deprecated: PHP Startup: Use of mbstring.internal_encoding is deprecated in Unknown on line 0

上記は、php.ini の下記をコメントアウトすることで解決した。
PHP の version を上げたことで、下記の設定値は対応されなくなったらしい。

```ini
mbstring.internal_encoding = "UTF-8"
```

## Laravel9 → 10 へ

composer.json にあるバージョン指定を下記のように修正。

```json
"php": "^7.3|^8.0",
↓
"php": "^8.0",
```

```json
"laravel/framework": "^9.0",
↓
"laravel/framework": "^10.0",
```

```json
"laravel/sanctum": "^2.11",
↓
"laravel/sanctum": "^3.2",
```

```json
"fruitcake/laravel-cors": "^2.0",
↓
削除
```

```json
"spatie/laravel-ignition": "^1.0",
↓
"spatie/laravel-ignition": "^2.1",
```

```json
"phpunit/phpunit": "^9.5.10"
↓
"phpunit/phpunit": "^10.0"
```

`app/Http/Kernel.php`の`$middleware`の配列に定義されている下記を修正。

```php
\Fruitcake\Cors\HandleCors::class,
↓
\Illuminate\Http\Middleware\HandleCors::class,
```

修正できたら、8→9 へのアップデートと同様に`composer update` で更新し、`php artisan -V`で確認。

# 既存の Laravel プロジェクトに Vue.js を導入

Laravel の認証系のスターターキットとして Laravel Breeze があり、すでに構築された認証認可に関する UI やバックエンドのロジック、migration 等が簡単に導入できるようです。
Breeze を install する際、UI は通常 blade にて install されますが、Vue または React を指定して install することもできます。
今回は、Breeze を install する際に一緒に Vue も導入していこうと思います。

## Inertia について

上記の Breeze を install すると一緒に Inertia も使用できるようになり、route 設定や Controller での view 側への値の受け渡し方が、Laravel の使用感と同じように実装ができました。
Inertia は、Laravel(PHP)と Vue.js をつなぐ接着剤のような役割があるようです。

## Breeze (Inertia) のinstall

下記で、Vue を指定して Breeze を install できました。

```bash
php artisan breeze:install vue
```

# 対応内容や学び

## Controller の view の指定を blade から Vue ファイルの指定へ変更

`view()`で blade のファイルを指定していたところを、`Inertia::render()`を使って Vue ファイルを指定し、表示側で使用する値も渡すように修正した。

```php
$hoge = 'xxxxx';

return view('sample.index', compact('hoge'));
↓
return Inertia::render('Sample/Index', ['hoge' => $hoge]);
```

## リダイレクト処理の修正

Laravel9 からの機能であるが、リダイレクトの処理は`to_route()`に簡略化できる。

```php
return redirect()->route('sample.index');
↓
return to_route('sample.index');
```

## Vue ファイルの route 対応

認証周りやヘッダーの href などで指定されている`route()`の指定を、初期状態から`php artisan route:list`で対応しているものに修正した。

## ページネーションの実装

blade だと、`sample->link()` とすることでページネーションの UI を簡単に表示できたが、
Vue だと PHP のメソッドをそのまま実行できないため、独自の Paginate コンポーネントを作成して表示する。

ページネーションに関する情報自体は、`sample.links`で object として値を取得でき、そのデータを使って UI を作成していく。

## select タグ関連の実装

プルダウンメニューの選択肢を作成する際に、blade だと現在の選択状況を if 文を使って判定し selected をつけるように実装していた。

```php
<select name="shop_id" id="shop_id">
  @foreach ($shops as $shop)
    <option value="{{ $shop->id }}" @if ($shop->id === $product->shop_id) selected @endif>
      {{ $shop->name }}
    </option>
  @endforeach
</select>
```

Vue.js の場合は、v-model ディレクティブを使うことで、上記のような if 文の実装は不要で、実装が少し楽であった。

```vue
<select name="shop_id" id="shop_id" v-model="form.shop_id">
  <option v-for="shop in shops" :value="shop.id">
    {{ shop.name }}
  </option>
</select>
```

## 画像アップロードの実装

blade で画像アップロード(複数可)の input タグを実装する場合、下記のように実装していた。

```php
<input type="file" id="image" name="files[][image]" multiple accept="image/png,image/jpeg,image/jpg">
```

Vue.js の場合、イベントハンドラにて、`$event.target.files` で選択したファイル情報が取得できるため、下記のように実装。

```vue
<script setup>
const form = useForm({
  files: null,
})
</script>

<template>
  <input id="image" type="file" multiple accept="image/png,image/jpeg,image/jpg"
  @change="form.files = $event.target.files" />
```

## 金額表示でカンマをつける対応

blade では、`number_format()`を使うことで金額表示のためのカンマをつけることができていた。

```php
{{ number_format($product->price) }}
```

Vue.js では、`toLocaleString()`を使って同じことができる。

```vue
{{ product.price.toLocaleString() }}
```

## Swiper によるスライド表示対応

blade の場合、css を指定して Swiper の機能を割り当てていた。
ボタンやページネーションの表示が欲しければ、それ用の div タグを用意し class を指定していた。

```php
<div class="swiper swiper-container">
    <div class="swiper-wrapper">
        <div class="swiper-slide">
            <img src="xxx">
        </div>
        <div class="swiper-slide">
            <img src="xxx">
        </div>
        <div class="swiper-slide">
            <img src="xxx">
        </div>
        <div class="swiper-slide">
            <img src="xxx">
        </div>
    </div>
    <!-- If we need pagination -->
    <div class="swiper-pagination"></div>

    <!-- If we need navigation buttons -->
    <div class="swiper-button-prev"></div>
    <div class="swiper-button-next"></div>

    <!-- If we need scrollbar -->
    <div class="swiper-scrollbar"></div>
</div>
```

Vue.js の場合は、Component を利用し、ボタンやページネーションの機能必要であれば、props に真偽値を渡すことで足すことができた。

```vue
<script setup>
import { Swiper, SwiperSlide } from 'swiper/vue';

</script>

<template>
<Swiper :pagination="true" :scrollbar="true" :navigation="true" :loop="true">
    <SwiperSlide>
        <img src="xxx">
    </SwiperSlide>
    <SwiperSlide>
        <img src="xxx">
    </SwiperSlide>
    <SwiperSlide>
        <img src="xxx">
    </SwiperSlide>
    <SwiperSlide>
        <img src="xxx">
    </SwiperSlide>
</Swiper>
```

## リレーションに関して

blade の場合は、リレーションとして定義したメソッドを呼び出せる。

```php
{{ $product->image }}
```

Vue.js では、上記のように PHP の関数呼び出しができないので、Controller 側で先に呼び出しておき、props で渡される値に含めておく必要がある。

```php: Controller側
$product = Product::with('image')->get();

return Inertia::render('Product/Show', ['product' => $product]);
```

```vue
{{ product.image }}
```

## 今回の対応はSPAになっているのか？

Inertia, Vue を導入し Link タグを使うことで SPA を実現できているが、
a タグで作成したリンクで画面遷移するものは SPA の挙動にはならない。

# 学習を通しての感想

とにかく Vue.js を書いてみたくて blade から書き換える作業をしてみて、blade よりも記述量が若干少なくなって楽だなと感じたり、ディレクティブの使い方やどういう挙動をするのかなどの理解が深まったかなと感じました。
ただ、元々構築していた EC サイトでは、複雑な UI がなく、リアクティブな実装はほとんど無かったので、`ref`やライフサイクルフックを使う機会が少なくて、少し物足りない感がありました。

リッチな UI は不要で、ほんとに簡易的なサイトであれば、リレーションのメソッドがそのまま使用できる blade で実装するほうが良かったりするのかなとも思いました。
ただ、顧客に提供するサービスでは UI を凝る事が多いとは思うので、設計フェーズで複雑になりすぎないようにして、少しでもリアクティブな実装を減らし、工数削減したり保守性を高めたりすることも検討できそうだなと感じました。

# 参考まとめ

https://imanaka.me/laravel/laravel8to9upgrade

https://zenn.dev/iret/articles/laravel-upgrade-from-9-to-10
