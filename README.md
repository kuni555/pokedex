# アーキテクチャ設計

## 概要

Next.js を採用し、以下のようなディレクトリ構成・命名規則・コーディング規約をもとに開発を行います。共通変数の管理は Redux を利用し、ディレクトリごとに役割を明確化することで、保守性と可読性の向上を目指します。

## ディレクトリ構成

```

src/
├── app/                    # ⭐︎appRouterを使う
│   ├── feature1/           # ⭐︎app配下にはappRouterに関するファイルのみ配置する
│   │   ├── page.tsx
│   │   └── layout.tsx
│   ├── feature2/
│   │    ├── page.tsx
│   │    └── layout.tsx
│   ├── page.tsx
│   └── layout.tsx
│
├── feature/                # ⭐︎業務ごとにディレクトリを切る 機能は全てfeature配下にまとめる(ログイン等)
│   ├── feature1/
│   │   ├── pages/          # appRouterを使い、page.tsxを配置
│   │   │   └── somePage.tsx
│   │   ├── presentation/   # Stateの制御やデータの加工、pagesから呼び出される　UIの決定 componentの塊 template
│   │   ├── component/      # 業務別のコンポーネントを定義
│   │   ├── hooks/          # 業務別のカスタムフック
│   │   ├── types/          # 各コンポーネントで受け渡すプロップスの型定義
│   │   ├── api/            # 業務ごとのapi　RestApiの関数定義
│   │   └── redux/          # apiデータの受け渡し　RestApiを呼ぶ関数をスライスに定義　レスポンスの加工もする。
│   │       └── slices/
│   │           ├── feature1Slice.ts    # userに関するSlice
│   │           └── featureSettingSlice.ts # 設定に関するSlice
│   └── feature2/
│       ├── pages/
│       ├── presentation/
│       ├── component/
│       ├── hooks/
│       ├── types/
│       ├── api/
│       └── redux/
│           └── slices/
│                ├── feature2Slice.ts
│                └── feature2SettingSlice.ts
│
├── utils/                  # ⭐︎共通的な関数
│   └── someUtilityFunction.ts
├── components/             # ⭐︎全ページで使うぐらいの共通的なコンポーネントを定義 atom
│   ├── Header.tsx
│   ├── Footer.tsx
│   └── Button.tsx
├── hooks/                  # ⭐︎全ページで使うぐらいのカスタムフックを定義
│   ├── useAuth.ts
│   └── useFetchData.ts
└── redux/                  # 共通変数をReduxで定義
    ├── store.ts            # Reduxのstore
    └── rootReducer.ts      # rootReducer


```

## 各ディレクトリの役割

### 1. api

- **役割**  
  サーバーサイドで動作し、外部 API との情報の受け渡しを行うディレクトリ。業務ごとにディレクトリを作成し、必要に応じて以下を配置します。

- **構成要素**
  - `hooks/`: API レイヤーに特化したカスタムフックを定義します（基本的にデータの受け渡しがメイン）。
  - `interface/`: リクエスト・レスポンスの型定義を行います。
  - `index.ts` または `service.ts`: 外部 API 通信を行う関数群を配置します。

### 2. feature

- **役割**  
  業務ごとにディレクトリを作成。ページやプレゼンテーション層、UI などを一つのまとまりとして管理します。

- **構成要素**

  1. `pages/`

     - 実際にレンダリングする `page.tsx` の実体を配置。Next.js の `appRouter` を使用し、ルーティングに応じたディレクトリ配下の `page.tsx` を呼び出します。
     - 名前は `{画面名}Page.tsx` の命名規則で作成します。

  2. `presentation/`

     - componentの集合体。
     - クライアントサイドで動作し、State の制御やデータの加工を行う。
     - データの取得やロジックをまとめたカスタムフックやコンポーネントをここで管理し、UI (`ui` ディレクトリ) に依存しない形で実装します。
     - ページ (`pages/`) から呼び出されます。

  3. `ui/`

     - クライアントサイドで動作し、UI を決定するレイヤー。
     - プレゼンテーション層から呼び出されます。

     - **components/**

       - 業務別コンポーネントを配置。以下の命名規則に従います。
         - 例: `{操作名}Form.tsx`, `{項目名}Card.tsx` など
       - `{componentName}.tsx`: コンポーネントの実態。
       - `{componentName}interface.ts`: そのコンポーネントで使用する props などの型定義。
       - `{componentName}handler.ts`: データを加工する static な関数を定義。
       - `{componentName}handlerinterface.ts`: handler の引数や戻り値などの型定義。

     - **hooks/**
       - 業務別のカスタムフックを定義。プレゼンテーションレイヤーのロジックやデータ受け渡しをさらに細分化する場合に使用します。

### 3. middleware

- **役割**  
  共通的な業務ロジックをまとめる場所。Redux を操作する場合、直接コンポーネントから Redux に触らず、この `middleware` ディレクトリ内の関数を経由します。
- **例**
  - `updateUserMiddleware.ts`: ユーザ情報を更新するためのビジネスロジックを記述する。内部で Redux のアクションを dispatch するなどを実装。

### 4. components

- **役割**  
  全ページや複数の feature で使い回される、共通的・汎用的なコンポーネントを配置します。
- **例**
  - `PrimaryButton.tsx`, `Dialog.tsx` など

### 5. hooks

- **役割**  
  全ページや複数の feature で共通的に使用するカスタムフックを定義します。
- **例**
  - `useWindowSize.ts`, `useForm.ts` など

### 6. utils

- **役割**  
  共通的な関数や定数、型定義などを配置します。
- **例**
  - `dateUtil.ts`, `stringUtil.ts` など

### 7. Redux

- **役割**  
  アプリケーション全体で使用する共通の state 管理を行います。
  ここでapiからデータの受け渡しの関数を定義することで、関数を共通化できる。
- **例**
  - `store.ts`
  - `userSlice.ts`
  - `authSlice.ts`

---

## 共通変数と型定義

- **共通変数は Redux で定義**  
  アプリケーション全体で必要なグローバルな state はすべて Redux で管理します。
- **共通的なモジュールの型定義はモジュールのファイル内に定義**  
  例: `dateUtil.ts` 内で使用する型がある場合、同ファイル内または同ディレクトリ内に `{ファイル名}interface.ts` として定義します。

---

## 命名規則

| ファイルの種類            | 命名規則              | 具体例                      |
| :------------------------ | :-------------------- | :-------------------------- |
| ページコンポーネント      | `{画面名}Page.tsx`    | `UserDetailPage.tsx`        |
| フォームコンポーネント    | `{操作名}Form.tsx`    | `LoginForm.tsx`             |
| ダイアログコンポーネント  | `{操作名}Dialog.tsx`  | `DeleteConfirmDialog.tsx`   |
| モーダルコンポーネント    | `{内容}Modal.tsx`     | `UserRegistrationModal.tsx` |
| ボタンコンポーネント      | `{操作}Button.tsx`    | `SubmitButton.tsx`          |
| リストコンポーネント      | `{項目名}List.tsx`    | `UserList.tsx`              |
| カードコンポーネント      | `{項目名}Card.tsx`    | `ProductCard.tsx`           |
| 表示 UI コンポーネント    | `{表示名}View.tsx`    | `ProfileView.tsx`           |
| 共通の汎用コンポーネント  | `{UI名}.tsx`          | `Spinner.tsx`               |
| サービスや API のヘルパー | `{機能名}Service.tsx` | `AuthService.tsx`           |
| ユーティリティ関数        | `{動作名}Util.ts`     | `dateUtil.ts`               |
| スタイルファイル          | `{画面名}.module.css` | `UserDetailPage.module.css` |

---

## 実装フロー例

1. **機能追加や画面を作成するとき**

   1. `feature/<featureName>` ディレクトリを作成。
   2. 画面が必要な場合は `pages/` ディレクトリ下に `{画面名}Page.tsx` を用意。
   3. ビジネスロジックは `presentation/` 層に、UI は `ui/` 層に整理して配置。
   4. API 通信が必要であれば `api/<serviceName>` ディレクトリを作成し、`hooks/` や `interface/`、必要なら `service.ts` を定義。
   5. Redux で管理する state があれば `redux/` で slice を作成し、`store.ts` に登録。
   6. Redux を呼び出す際は可能な限り `middleware` ディレクトリに処理を集約し、そこから dispatch する。

2. **共通的なコンポーネントの作成や修正**
   1. 全ページで使用されるようなコンポーネントは `components/` へ配置。
   2. 汎用的なカスタムフックは `hooks/` へ配置。
   3. ネーミングルールに沿ってファイル名・フォルダ名を決定。

---

## まとめ

- **Redux で共通変数を管理**し、UI レイヤーからのビジネスロジックへのアクセスは可能な限り `middleware` を経由します。
- **ディレクトリ構成・命名規則を明確化** することで、チーム開発や保守を容易にし、コードの再利用性を向上させます。
- **型定義は極力分離しすぎず**、関連するファイル内または同階層に `{componentName}interface.ts` や `{componentName}handlerinterface.ts` として配置し、可読性を維持します。

この構成および命名規則に従うことで、Next.js + Redux アプリケーションを見通しよく開発・保守することが期待できます。
