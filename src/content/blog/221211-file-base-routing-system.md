---
title: "舊文-Next.js pages router"
description: "描述Next.js的file-based routing system"
pubDate: "2022-12-11"
heroImage: "/nextjs.avif"
---

## 引言

這篇文章會提到Multi-Page Application(MPA)與Single-Page Application(SPA)，有興趣的朋友可以[到這裡](https://www.youtube.com/watch?v=F_BYg2QGsC0&ab_channel=Academind)，讓我React學習路上的明燈來解釋給你聽：）

## 為什麼要叫 file-based？

React開發者透過[Browser API](https://ncoughlin.com/posts/react-navigation-without-react-router/)或使用像是React Router這類型的library來達成URL routing，以React Router為例，你會在程式裡看到這樣的結構：

```javascript
<Router>
  <div>
    <Switch>
      <Route exact path="/">
        <Home />
      </Route>
      <Route path="/about">
        <About />
      </Route>
      <Route path="/dashboard">
        <Dashboard />
      </Route>
    </Switch>
  </div>
</Router>
```

開發者會在程式中使用`<Router>` `<Switch>` `<Route>`這些React Router提供的component組成router，這種讓library透過URL決定畫面呈現，而不是發出request，讓server決定要呈現畫面的行為，[React Router官方文件稱為**client side routing**](https://reactrouter.com/en/main/start/overview)。

不論使用哪種方式，開發者都要寫更多的code，這讓開發者要花更多精力去管理code，而不是專注在畫面內容的呈現，Next.js提供了我們不一樣的做法。

## 靜態路由

### 基本形式

在建立好全新的Next專案時，你會看到這樣的檔案結構：

![](https://i.imgur.com/eufcrXQ.png)

而我們今天的重點會放在`pages`，我們的路由就是在這完成的。

![](https://i.imgur.com/ELnUCaS.png)

點開`pages`請先看到`_app.js`與`index.js`，`_app.js`會是頁面的進入點，包裹著所有分頁的內容，可以放置一些跨頁面元件如`<NavBar>`。

而`index.js`則是我們的首頁，與`_app.js`一樣位於`pages`的最外層，如果我們使用`npm run dev`啟動Next.js內建的開發環境，我們一般來說會在Next.js預設的port，`http://localhost:3000`看到我們的首頁。

接著看到`first-page.js`與`second-page`這個資料夾，在Next.js的系統裡我們有兩種新增分頁的方式，第一種是直接新增檔案在你期望的階層，舉例來說，當我在與`index.js`的相同檔案夾新增了名為`first-page.js`的檔案時，如果我輸入`http://localhost:3000/first-page`，就會看到`first-page.js`裡面的內容。

第二種方式則是建立檔案夾，並且在檔案夾裡面新增一個`index.js`檔案，就像範例中我建立了一個名為`second-page`的檔案夾與其中的`index.js`，如果我在網址列輸入`http://localhost:3000/second-page`，Next.js會幫我們找到`second-page`檔案夾，並且將檔案夾中的`index.js`做為預設內容。

整理起來我們可以這樣理解：`pages`檔案夾代表了我們的domain（例如範例中的`localhost:3000`），當我們沒有輸入後續路徑時，Next.js會使用位於檔案夾中的`index.js`做為預設渲染的對象，如果我們接著輸入後續的路徑，例如`localhost:3000/XXXX-path`，Next.js便會在`pages`檔案夾裡面搜尋名稱符合`XXXX-path`的檔案或檔案夾，如果是檔案則渲染檔案內容，如果是檔案夾，則渲染檔案夾中的`index.js`檔案。

### 巢狀結構

![](https://i.imgur.com/aeWfYCg.png)

接續著前一段的內容，當我們使用檔案夾作為路徑時，除了預設的`index.js`，我們可以繼續在檔案夾內新增其他檔案或檔案夾，例如如果我們在稍早建立的`second-page`中新增名為`inside-page.js`的檔案，我們可以在路徑`localhost:3000/second-page/inside-page`看到該頁內容，

因此根據相同邏輯，我們可以按照我們的需求，像這樣，

`domain/folder1/folder2/folder3...`

不斷在父層檔案夾中建立子檔案夾，來增加我們的路徑深度。

## 動態路由

某些時候我們會使用相同的結構去渲染不同頁面，例如電商的商品頁，頁面之間的差異只在於文字或圖片不同，並且有可能數量龐大，這時候我們不會想要為每個商品建立獨立頁面，而是讓同一個頁面根據不同資料呈現不同商品，這時候我們就可以使用動態路由。

### 固定結構動態路由

![](https://i.imgur.com/59Eya9J.png)

在剛才的`second-page`檔案夾中出現了一個長相怪異的資料夾，名稱被包在`[]`中括號裡面，這是Next.js所使用的辨識語法，被中括號包住的名稱會成為動態路由。

讓我們回顧一下，當我們輸入`http://localhost:3000/second-page`我們會看到檔案夾`second-page`裡面`index.js`的內容，在我們加入`[dynamic-param]`這個檔案夾後，如果我輸入`domain/second-page/some-value`，Next.js一樣會先在`second-page`檔案夾中搜尋是否有符合`some-value`的檔案或檔案夾，如果沒有，而在資料夾內我們有建立動態路由，則`some-value`會成為動態路由，如果像上圖的範例，`some-value`就會成為名為`dynamic-param`的參數，我們的網頁就可以根據`some-value`去取得對應資料以顯示內容。

舉例來說，我們的`second-page`可能是某種user列表，他的路徑會是`domain/user/`，當使用者想看userId為1的內容時，我們會把使用者導向`domain/user/1`的頁面，這時候因為`user`資料夾下沒有名為`1`的檔案或檔案夾，而我們有設定`[id]`這個動態路由，`1`就會成為我們動態路由讓我們取得`id=1`的參數，在根據舉得的參數去請求userId為1的資料，渲染內容，而不必為每筆user資料建立獨立頁面。

![](https://i.imgur.com/Q0hJ08m.png)

接著我們來看如何取得動態路徑。

想要取得動態路徑，我們可以使用Next.js提供的`useRouter`hook來取得動態路徑：

```javascript
// domain/second-page/[dynamic-param]/index.js

import { useRouter } from "next/router";

export default function DynamicPage() {
  const router = useRouter();

  return <></>;
}
```

在Next.js的框架下，我們可以透過`useRouter`或`withRouter`的方法，取得名為`router`的object，來操作路徑或取得路徑，要用哪個取決於你是用function component或是class component，這邊的範例是使用function component，因此使用`useRouter`這個hook來取得`router`。

```javascript
// domain/user/[id]/index.js
import { useRouter } from next/router;

export default function UserPage {
    const router = useRouter();
    const { query } = router;

    console.log(query); // { id: 1 }

    ...
}
```

在取得`router`後，我們可以透`router.query`這個屬性取得我們的動態路徑，如果我們輸入的路徑是`domain/user/1`，因為`user`下並沒有`1`這個檔案或檔案夾，因此`1`就會被認定是動態路徑，因此當我們使用`query`取得動態路徑時，我們會得到`query = { id: 1 }`這個結果，然後再依照結果取得資料，渲染畫面。

另外，動態路由也支援巢狀結構，因此我們可以根據需要建立像這樣`domain/[param1]/[param2]...`的結構，而我們的`query`就會取得下面範例的結構。

```javascript
{
    param1: any,
    param2: any,
    ...
}
```

### 不固定結構動態路由

某些狀況例如我可能想看某年的資料，然後繼續搜尋特定月份、特定天的資料，這種時候我們可以使用像是`pages/data/[year]/[month]/[day]`這樣的檔案夾結構去完成這項功能，但我們也可以使用另一種方式：

![](https://i.imgur.com/dhziYYl.png)

在這樣的狀況我們可以使用`[...param]`一次取得所有參數，當我們使用`[...param]`時，我們取得的`router.query`會變成array，回到剛才的例子，我們把結構改變成`pages/data/[...time]`，如果我們傳入`domain/data/2011`，`query`會回傳`{ time: ['2011'] }`，我們傳入`domain/data/2011/07/04`，會得到`{ time: ['2011', '07', '04'] }`，這讓我們在處理動態路由時更有彈性，根據不同狀況決定路由的樣式。

## 連結

### Link

Next.js的核心是React，也就是說他依然是SPA，提供的routing本質上與React Router一樣是種偽裝，因此雖然我們已經可以模擬出MPA的行為，透過不同路徑產生不同內容，但如果我們使用傳統的`<a>`tag做頁面切換，`<a>`tag會向server發出請求造成整個頁面reload，在這過程我們會丟失React所管理的state，要解決這個問題我們要使用Next.js提供的`Link`component：

```javascript
<Link href="/">HomePage</Link>
```

他的用法類似`<a>`tag，但他實際上不會造成整個頁面reload，從而避免state丟失，如果要使用動態路由，我們有兩種寫法，例如我們有一包user的資料，當我們要寫特定user的連結時我們可以這樣寫：

```javascript
<Link href={`/user/${user.id}`}>{user.name}</Link>
```

或者也可以這樣：

```javascript
<Link
  href={{
    pathname: "/user/[id]",
    query: { id: user.id },
  }}
>
  {user.name}
</Link>
```

這裡的`pathname`就是我們的路徑，而`query`就是我們想傳入動態路由的內容，這兩種方法沒有好壞，純看個人習慣。

## 參考

[Next官方文件](https://nextjs.org/docs/routing/introduction)
