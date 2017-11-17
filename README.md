# DotnetNg2

ASP.NET Core WebAPI + Angularプロジェクトを構築する。

## 環境構築

* Node.js
* Angular cli
* [.NET Core](https://www.microsoft.com/net/learn/get-started/macos)

## プロジェクトの作成

### ソリューションの作成

```
$ mkdir SolutionName
$ cd SolutionName
$ dotnet new sln
```

### WebAPIプロジェクトの作成

```
$ dotnet new webapi --name ProjectName
```
プロジェクト名は任意。認証のタイプなども設定できる。詳細はヘルプを参照

プロジェクトをソリューションに追加する

```
$ dotnet sln add ProjectName/ProjectName.csprj
```

### Angularプロジェクトの作成

```
$ ng new ProjectName -dir ProjectDirName -sg
```

* `-dir`オプションで出力先を指定すると、上で作ったWebAPIプロジェクトと同じディレクトリに出力できる。ファイルの競合はない。分けてもいい。
* ソリューション全体をgitで管理したいので`-sg`でgit管理を一旦飛ばす。ただし、`.gitignore`が作られないので自分でで作る必要がある。
* 他のオプションはプロジェクトに合わせて設定する

### 動作確認

プロジェクトディレクトリ内で実施する。

#### WebAPIの確認

```
$ dotnet run
```

プラウザで`http://localhost:5000/api/values`にアクセス。`["value1","value2"]`と表示されればOK

#### Angularの確認

```
$ ng serve 
```

ブラウザで`http://localhost:4200`にアクセス。

## WebAPIとAngularを連携させる

AngularからWebAPIを呼び出すためには、

1. Angularの実行時にproxy configを読み込ませる
2. AngularとWebAPIを同じサーバーで実行する

2.のやり方を解説

### WebAPIの修正

規定のアクセス先として、`Home`Controllerの`Index`Actionを設定する。

`WebClient/Startup.cs`

```C#
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

+   app.UseStaticFiles();
-   app.UseMvc();
+   app.UseMvc(routes =>
+   {
+       routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
+       routes.MapSpaFallbackRoute("spa-fallback", new { controller = "Home", action = "Index" });
+   });
}
```

HomeControllerを実装する。

`WebClient/Controllers/HomeController.cs`を追加する。

```C#
using Microsoft.AspNetCore.Mvc;

namespace Todo.Controllers
{
  public class HomeController: Controller
  {
    public IActionResult Index() => File("/index.html", "text/html");
  }
}
```

これで、他のAPIで定義されていないアクセスについては`index.html`が返るようになる。あとはAngularで適切なルーティングを設定する。

### Angularのビルド

ASP.NET Coreは、`wwwroot`ディレクトリをポームディレクトリとして認識する。（設定で変更可能）Angularのビルド時に、出力先を指定する。

```
$ ng build -op wwwroot
```

ビルドが完了したら

```
$ dotnet run
```

`http://localhost:5000`にアクセスして、Angularアプリが見れればOK。上記コマンドはnpmタスク化しておくといい。

### 課題

`ng serve`などの場合と異なり、Live Reloadが効かない。Angularの修正に対するリアルタイムのビルドは`--watch`オプションをつけてビルドすればいい。.NET Core資産の修正に対してはWacherツールがあるので、導入すればサーバを自動で再起動してくれる。しかし、画面は自動でリフレッシュしてくないので少し不便。

## その他

.NET CoreにはAngularテンプレートがある。`dotnet new`で利用可能なテンプレートを確認できる。

```
dotnet new angular
```

まだ触っていないが、サーバーサイドレンダリングなども最初から入っていて、使ってもいいかもしれない。
