# JRA指定席ネット予約スケジュール(jra-ticket-schedule)

JRA(日本中央競馬会)公式で掲載されている[指定席予約スケジュール](https://www.jra.go.jp/card/about/schedule.html)があまりにも分かりづらいので、競馬開催日を指定することで具体的な予約開始日を確認できるようにするVue3製のSPA

## 使用技術
- Vue3
- TypeScript
- Tailwind CSS
- Flatpickr
- Vite
- etc...

## 公開URL
https://jra-ticket-schedule.s3.ap-northeast-1.amazonaws.com/index.html

## 利用API
以下のAWS Lambda(Python)によるAPIを利用しています。
https://github.com/k-negishi/jra-ticket-schedule-api

## Project Setup

Node.js 20.X がインストールされていることを前提としています。

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```

### Lint with [ESLint](https://eslint.org/)

```sh
npm run lint
```

## CI/CD
GitHub ActionsでVueをビルドし、S3にpushすることでデプロイする
