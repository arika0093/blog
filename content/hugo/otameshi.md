+++  
title = 'Hugoを使ってブログを作成してみました'  
date = 2023-11-20T23:25:41+09:00  
tags = ['app','hugo','blog','otameshi']  
categories = ['otameshi']  
draft = false  
+++  
  
この記事が書かれているページがまさにそうなのですが、[Hugo](https://gohugo.io/)を使用してブログを作成してみました。  
以下、落とし穴とか所感とか。  
  
## 作成まで  
### まずは  
  
Hugoをinstallする。    
とりあえずWindows版をInstallした。  
https://gohugo.io/installation/windows/  
  
### Site作成  
```sh  
hugo new site (path)  
```  
で終わり。色々なファイルが吐き出される。  
  
### Content作成  
```sh  
hugo new content posts/test.md  
```  
  
最初`test`とかで良いのかと思ったら、ちゃんとパスや.mdを書かないといけない。  
ダルいと思ったが、このパスがカテゴリ的な扱いになっているのでちゃんと理由があった。`.md`に関しても、HTMLもデプロイできるので仕方ない。  
  
### 適当に書いてみる  
```md  
+++  
title = 'Test'  
date = 2023-11-20T19:04:48+09:00  
tags = ['example','test']  
categories = ['test']  
draft = false  
+++  
  
hogehoge  
ぴよ  
```  
  
テンプレートは`archetypes/_default.md`にあった。  
これを少し改造して、tagsとcategoriesが最初から入っているように改造。  
`draft`はfalseにしないと公開できない。  
逆に言えば下書きをpushしておいて、出先で追記もできる。  
先頭部分のメタ部分に使える情報は多分たくさんあるが、一旦これだけでOK。  
  
## Themeを導入する  
標準のままだと面白くないので、[themes](https://themes.gohugo.io/themes/)から適当に探してみた。    
今回は[dark-theme-editor](https://themes.gohugo.io/themes/dark-theme-editor/)を導入する。  
  
### 導入方法  
```sh  
git clone https://github.com/JingWangTW/dark-theme-editor.git themes/dark-theme-editor  
```  
  
導入したままだとgitで追跡してくれないので`.git`を削除して`git add *`する。  
いじらないならsubmodulesで良いと思うが、微調整をしたくなるので今回はこの方法でInstallした。  
  
（余談だが、こういう人のレポジトリを引っ張ってきてそれを微調整したいときっていい方法ないよね。`patch`を都度あてるのも違うし、今回みたいにパクってupstreamがセットされていないのも気持ち悪いな…）  
  
## 検索機能を導入する  
上記まででいい感じかと思ったのだが、検索機能がない。  
普通にそれはブログとして微妙すぎるので、導入してみる。  
（Google検索でもいい？それはそうなんだけど、UXがちょっとね…）  
  
### 探す  
  
まずは[公式](https://gohugo.io/tools/search/)から。  
公式でのやり方があるわけではなく、~~ぶん投げ~~他のライブラリを使う必要がありそう。  
  
### 導入  
いくつか調べてみて、[これ](https://gist.github.com/Arty2/8b0c43581013753438a3d35c15091a9f)を使ってみることにした。  
  
まずは手順に則る…と思っていくつか試してみたが、これ、`theme`フォルダの方をイジる必要があるらしい。  
わかるわけない。  
  
とりあえず、書いてあるとおりに変更を入れる。  
  
> 1. Add `search.html` file to `layouts/partials/`  
> 2. Add `index.json` file to `layouts/_default/`  
> 3. Add JSON as additional output format in `config.yaml` or `config.toml`  
> 4. Add `fixedsearch.js` and `fuse.js` (downloaded from [fusejs.io](https://fusejs.io/) or directly from [jsdelivr](https://cdn.jsdelivr.net/npm/fuse.js)) to `static/scripts/fixedsearch/`  
> 5. Add the `search.html` partial by iincluding `{{ partialCached "search" . }}` in your theme’s `layouts/_default/baseof.html` before the `</body>` tag.  
> 6. Preview or deploy as usual. You can inspect the JSON index by visiting `localhost:1313/index.json`  
  
……うーん、長い。長いだけならいいのだけど、themeの本体に傷を入れないといけないのはちょっと。    
まあ他にいい方法もなさそうなので、諦めてこれを導入。  
  
### UI微調整  
導入してOKかと思ったら、これのCSSが特定のライブラリを元にしたものでフォントサイズとかが全然合わなくて困った。    
ので修正。    
ちなみにこれぐらい変更した。    
https://github.com/arika0093/blog/commit/b75c18e20125ea40fc8530968edf38b0e0708c9f  
  
  
## GithubPageへdeployする  
  
自分でビルドするのもだるかったので、GitHub Actionsを使ってみた。  
とはいえ、[actions-hugo](https://github.com/peaceiris/actions-hugo)の内容をほぼ丸パクリするだけ。  
  
成果物はこちらになります。  
https://github.com/arika0093/blog/blob/master/.github/workflows/gh-pages.yaml  
  
  
……と調べながら今気づいたのだけど、これは旧来のやり方で、officialのやり方が好ましいぽい。まあいいか。    
https://zenn.dev/zozotech/articles/f2509a21b768ed  
  
## baseURL対応をする  
  
github pagesは`https://hoge.github.io/blog`となり、root directoryが`/blog`になる。これのせいで大変だるかった。  
  
まずパス指定だが、これは`baseURL`をgithub-actionsで変更することで対応した。  
```yaml  
- name: Build  
	run: hugo --minify -b /blog/  
```  
  
relativeURLでも良さそうに見えるが、後述の問題があったのでこちらを採用した。  
  
そして、前述のコンテンツ検索スクリプトが`/`にしか対応してなかったので、書き直した。具体的には`relURL`を使って現在のパスを~~無理やり~~取得する方法で対応した。  
  
何回もミスったが、とりあえず対応完了。  
  
現に、このサイトで `CTRL`+`/`を押せばちゃんと検索ダイアログが開くはず。  
  
## 所感  
導入&この記事を実際に書いてみての所感。  
  
* 良いとこ  
  * ちょっとしたコードメモには良い。  
    * ファイルの実体が全てgit上にあるので、追いやすそう。  
      * zennとかqiitaに書くのは、tracability的にどうなのという気持ちがあった。  
        * とはいえzennは吐き出しできたので、そういう意味じゃこちらのアドはないか。  
  * スタイルを自由に弄れるのは良い  
    * まあこれはそのまま欠点でもあるのだけど。  
    * 世の中のmarkdownブログはどれもデザインというか行間がちょっと気に入らないので、そういう意味では自前は良い。  
  * 殴り書きには良い。  
    * これはmarkdownなら全部そうだろうけど。こうやって階層構造で適当に色々書くのは性に合っている  
* 悪いところ   
  * 長文は微妙。  
    * 書くのがつらいのもそうだけど、画像を手軽に貼れないのがちょっと。  
      * これに関してはvscodeの拡張とかなんかありそうではある。  
      * imgurにアップロードするやつありそう。  
        * imgurかはわからないけど、[一応あった](https://marketplace.visualstudio.com/items?itemName=hancel.markdown-image)。  
  * 改行を自動反映してくれない。  
    * ちゃんと末尾にスペースを2個いれないといけない。  
      * だるすぎるので、vscodeの方で対応した。  
        * `markdown.preview.breaks: true`の設定をONにすれば良い。  
          * と思ったら、これはプレビューの見た目だけだった。残念  
        * 拡張機能(Prettier)で対応。保存時に勝手に整形してくれる。 
          * これも思ったのと違う。
        * 結局、`s/\n/  \n/g` で対応した。力技。
  * 左の項目名が勝手に複数形になる。  
    * `Post`->`posts`とかならいいけど、`Hugoes`ってなんだよ。  
  
  
所感とは違うけど、普段文字を入力しなすぎて20分間タイピングしてるだけでつらい。タイピング筋を鍛えないといけない（？）  
  
  
