# nextjs-https
windowsとnext.jsにてhttps環境構築の流れ


## Chocolatey のインストール
brew 同様のコマンドを使うために chocolateyのインストール（要管理者権限）   
PowerShellを管理者権限で開き、以下のコマンドでインストール   

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

## mkcert のインストール
これを使うと証明書の取得を一気にやってくれて便利です   
下記の mkcer の内容よりWindows向けの処理を行う   
管理者権限で開き、以下のコマンドでインストール    
https://github.com/FiloSottile/mkcert

```
choco install mkcert
mkcert -install
```


## mkcert にて証明書の発行
certificates のディレクトリに localhost 用の証明書を発行します。   
そうすると「localhost+1-key.pem」「localhost+1.pem」のファイルが作成されます。   

```
cd ./certificates
mkcert localhost 127.0.0.1
```

また、.gitignore に 証明書が含まれない様に追記します。

```
# localhost 証明書を無視する
/certificates/*.pem
```


## HTTPS で 開発環境が起動するように修正

※以下のサイトを参考にしました。   
https://medium.com/responsetap-engineering/nextjs-https-for-a-local-dev-server-98bb441eabd7   

プロジェクトのルートフォルダに「server.js」を生成し   
下記の内容を記述します。

```
const { createServer } = require("https");
const { parse } = require("url");
const next = require("next");
const fs = require("fs");
const dev = process.env.NODE_ENV !== "production";
const app = next({ dev });
const handle = app.getRequestHandler();
const httpsOptions = {
  key: fs.readFileSync('./certificates/localhost+1-key.pem'),
  cert: fs.readFileSync('./certificates/localhost+1.pem'),
};
app.prepare().then(() => {
  createServer(httpsOptions, (req, res) => {
    const parsedUrl = parse(req.url, true);
    handle(req, res, parsedUrl);
  }).listen(3000, (err) => {
    if (err) throw err;
    console.log("> Server started on https://localhost:3000");
  });
});

```


## package.json に 開発環境の起動設定を行う

```
  "scripts": {
    "dev": "next dev",
```
↓
```
  "scripts": {
    "dev": "node server.js",
```

## 開発環境を起動し接続する
開発環境を立ち上げ https://localhost:3000 にアクセスする   

```
npm run dev
```


## 後は通常の流れで開発を行う


This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `pages/index.js`. The page auto-updates as you edit the file.

[API routes](https://nextjs.org/docs/api-routes/introduction) can be accessed on [http://localhost:3000/api/hello](http://localhost:3000/api/hello). This endpoint can be edited in `pages/api/hello.js`.

The `pages/api` directory is mapped to `/api/*`. Files in this directory are treated as [API routes](https://nextjs.org/docs/api-routes/introduction) instead of React pages.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js/) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/deployment) for more details.
