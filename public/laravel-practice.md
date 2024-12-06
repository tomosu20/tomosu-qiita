---
title: Laravelにおけるクエリ改善
tags:
  - Laravel
private: true
updated_at: '2024-12-06T20:58:53+09:00'
id: 040874d3de2dc037a340
organization_url_name: null
slide: false
ignorePublish: true
---
# はじめに

本記事は、Laravel の Eloquent を使ったクエリ改善を手を動かしながら学習した記録であり、Laravel 初学者向けの記事になります。

# 本記事で扱うモデル

今回のクエリ改善で扱うモデルのイメージです

- ユーザーがブログ記事を投稿する
- 投稿された記事に対してコメントがある

users

| id  | name    | mail                |
| --- | ------- | ------------------- |
| 1   | hoge  | fuga@xxxx  |

posts

| id  | content | user_id（投稿者） |
| --- | ------- | ------- |
| 1   | 記事の内容    | 1       |

comments

| id  | comment | post_id |
| --- | ------- | ------- |
| 1   | コメントの内容    | 1       |

## 記事一覧を表示する

例えば、記事一覧を表示する際、投稿者の名前も一緒に表示します。

### 実装

まずはモデルでリレーションを定義します。

```php: app/Models/Post.php
class Post extends Model
{
    use HasFactory;

    protected $fillable = [ 'title', 'body' ];

    public function user(): BelongsTo {
        return $this->belongsTo(User::class);
    }
```

Controller で記事一覧を取得し、blade へ値を渡します。

```php: app/Http/Controllers/PostController.php
class PostController extends Controller
{
    public function index()
    {
        $posts = Post::all();
        return view('posts.index', [
            'posts' => $posts
        ]);
    }
```

先ほど Model で定義したリレーションを、blade 側で呼び出し、著者名を取得して表示します。

```php: resources/views/posts/index.blade.php
<table>
 <thead class="text-xs bg-gray-500">
  <tr>
   <th scope="col">ID</th>
   <th scope="col">Title</th>
   <th scope="col">Created At</th>
   <th scope="col">User Name</th>
  </tr>
 </thead>
 <tbody>
  @foreach ($posts as $post)
   <tr class="bg-white border-b  dark:border-gray-700">
    <td class="px-6 py-4">{{ $post->id }}</td>
    <td class="px-6 py-4">{{ $post->title }}</td>
    <td class="px-6 py-4">{{ $post->created_at->diffForHumans() }}</td>
    <td class="px-6 py-4">{{ $post->user->name }}</td>
   </tr>
  @endforeach
 </tbody>
</table>
```

### 結果

記事の一覧と、著者名が表示されました。

<!-- 画像 -->

### Laravel Debugbar

laravel でのデバッグが簡単にできる Laravel Debugbar を導入します。

```sh
composer require --dev barryvdh/laravel-debugbar
```

こちらを導入することで、実際に発行されているクエリの内容を確認することもできます。

先程の実装で、発行されたクエリを見てみると、、

<!-- 画像 -->

上図のように、たくさんのクエリが発行されてしまっています。。

これは、記事の数だけクエリが発行されており、このままでは記事が増えるたびにクエリ数が多くなっていき、表示速度に影響してしまいます。

### Eager Loading

この N+1 問題を改善するために、Laravel 公式で Eager Loading の機能があります。

[Laravel 10.x Eloquent リレーション | Eagerロード](https://readouble.com/laravel/10.x/ja/eloquent-relationships.html#eager-loading)

### 実装の改善

Controller で実装していた記事一覧の取得で、`with()`を使います。

```php: app/Http/Controllers/PostController.php
$posts = Post::all();
↓
$posts = Post::with('user')->get();
```

<!-- 画像 -->

上図のように、クエリが減って N+1 問題が改善しております。

## 記事のコメント数を表示する

次に、記事のコメント数を表示していきます。

### 実装

まずはモデルにリレーションを定義していきます。

```php: app/Models/Post.php
class Post extends Model
{
  ...
  public function comments(): HasMany {
      return $this->hasMany(Comment::class);
  }
```

Controller にて先程学んだ Eager Loading を使って comments も先にロードしておきます。

```php: app/Http/Controllers/PostController.php
class PostController extends Controller
{
    public function index()
    {
        $posts = Post::with('user', 'comments')->get();
        return view('posts.index', [
            'posts' => $posts
        ]);
    }
```

blade にて、コメントの数を表示する実装を加えます。

```php: resources/views/posts/index.blade.php
<table>
 <thead class="text-xs bg-gray-500">
  <tr>
   <th scope="col">ID</th>
   <th scope="col">Title</th>
   <th scope="col">Created At</th>
   <th scope="col">User Name</th>
   <th scope="col">Comment Count</th> <!-- 追加 -->
  </tr>
 </thead>
 <tbody>
  @foreach ($posts as $post)
   <tr class="bg-white border-b  dark:border-gray-700">
    <td class="px-6 py-4">{{ $post->id }}</td>
    <td class="px-6 py-4">{{ $post->title }}</td>
    <td class="px-6 py-4">{{ $post->created_at->diffForHumans() }}</td>
    <td class="px-6 py-4">{{ $post->user->name }}</td>
    <td class="px-6 py-4">{{ $post->comments->count() }}</td> <!-- 追加 -->
   </tr>
  @endforeach
 </tbody>
</table>
```

### 結果

<!--画像 -->

無事、コメント数が表示され、N+1 問題は発生しませんでした。

### もう 1 つ改善方法がある

実はまだ改善できます。

集計に関する実装は、`withCount()`を使うことで更にクエリを減らすことができます。

```php: app/Http/Controllers/PostController.php
$posts = Post::with('user', 'comments')->get();
↓
$posts = Post::with('user')->withCount('comments')->get();
```

blade 側のコメント数の取得を下記のように変更。

```php: resources/views/posts/index.blade.php
<td class="px-6 py-4">{{ $post->comments->count() }}</td>
↓
<td class="px-6 py-4">{{ $post->comments_count }}</td>
```

<!-- 結果 -->

クエリを 1 つ減らすことができました。

違いとしては、comments テーブルの情報の取得方法に違いがあります。
改善前は、`select * from comments`で comments テーブルの全てのカラム情報を取得していたのに対し、
改善後は、`select posts.*, (select count(*) from comments ....)`と、副問合せの形で 1 クエリにまとまっています。

また、画面には**コメントの数**のみを表示するので、全てのカラム情報は必要無く、`count(*)`だけで効率よくクエリを発行していますね。

最終的にコメント数は、`...) as comments_count`となっているため、blade 側の実装も変更が必要だったわけですね。

### その他の効果

debugbar にて、利用しているモデルの数が「Models」のタブに表示されています。
先程の`withCount()`にて改善する前と後の Models の内容を比較してみましょう。

改善前
<!-- 画像 -->

改善後
<!-- 画像 -->

コメント数だけを取得したことで、コメントのモデルは取得されておらず、メモリの節約にもなっています。

クエリ数やロードするモデルの数を意識して実装することで、効率よくデータを取得でき、表示速度を改善することができます。

# 感想

Laravel で実装していて、Eager Loading がややこしなと思っていたので、1 から整理し記事にしてみました。
学習を通して、理解度が少し上がったかなと思ます氏、集計の際に使える withCount()は、今回の学習を通して初めて知ったので学び直して良かったなと思います。

今回の記事では触れていませんが、ネストした Eager Loading など、複雑なモデルの場合は特に難しく、実装していてかなり混乱してしまいます。
まだまだ理解が浅い部分があるので、引き続き Laravel の学習に励んで行きます。

個人的に動画教材がインプットしやすいので、学習の際に以下の方の YouTube を参考にしていました。
Laravel 学習中の方や興味が有る方はぜひ

[https://www.youtube.com/@LaravelDaily](https://www.youtube.com/@LaravelDaily)
