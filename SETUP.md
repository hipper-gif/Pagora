# Pagora 開発環境セットアップ手順書

## 前提条件

- Windows PC（Windows 10 バージョン2004以降 / Windows 11）
- 管理者権限があること
- インターネット接続があること

---

## Step 1: WSL2のインストール

### 1-1. PowerShellを管理者で起動

スタートメニューで「PowerShell」を検索し、**「管理者として実行」**をクリック。

### 1-2. WSL2をインストール

```powershell
wsl --install
```

インストール完了後、**PCを再起動**する。

### 1-3. Ubuntuの初期設定

再起動後、Ubuntuが自動起動する。  
ユーザー名とパスワードを設定する（Linuxのユーザー名・パスワードなので任意でOK）。

### 1-4. WSL2のバージョン確認

```powershell
wsl --version
```

`WSL version: 2.x.x` と表示されればOK。

---

## Step 2: Docker Desktopのインストール

### 2-1. Docker Desktopをダウンロード

以下のURLからインストーラーをダウンロードする。

```
https://www.docker.com/products/docker-desktop/
```

### 2-2. インストール

ダウンロードしたインストーラーを実行。  
インストールオプションは以下を確認する。

- ✅ Use WSL 2 instead of Hyper-V（チェックを入れる）

インストール完了後、**PCを再起動**する。

### 2-3. Docker Desktopの起動・確認

Docker Desktopを起動し、画面左下のステータスが **「Engine running」** になればOK。

### 2-4. 動作確認

PowerShellで以下を実行する。

```powershell
docker --version
docker compose version
```

バージョンが表示されればOK。

---

## Step 3: Gitのインストール

### 3-1. Gitをダウンロード・インストール

```
https://git-scm.com/download/win
```

インストールオプションはすべてデフォルトでOK。

### 3-2. 初期設定

PowerShellで以下を実行する（名前・メールは自分のGitHubアカウントに合わせる）。

```powershell
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

---

## Step 4: Node.jsのインストール

Claude Codeの実行に必要。

### 4-1. nvmのインストール（バージョン管理ツール）

```
https://github.com/coreybutler/nvm-windows/releases
```

`nvm-setup.exe` をダウンロードして実行。

### 4-2. Node.jsのインストール

```powershell
nvm install 20
nvm use 20
```

### 4-3. 確認

```powershell
node --version   # v20.x.x と表示されればOK
npm --version
```

---

## Step 5: リポジトリのクローン

```powershell
cd C:\
mkdir projects
cd projects
git clone https://github.com/hipper-gif/Pagora.git
cd Pagora
```

---

## Step 6: Pagoraプロジェクトの初期構築

### 6-1. Next.jsプロジェクトの作成

```powershell
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
```

各オプションの選択は以下の通り。

| 質問 | 回答 |
|------|------|
| Would you like to use TypeScript? | Yes |
| Would you like to use ESLint? | Yes |
| Would you like to use Tailwind CSS? | Yes |
| Would you like to use `src/` directory? | Yes |
| Would you like to use App Router? | Yes |
| Would you like to customize the import alias? | No |

### 6-2. 必要なパッケージのインストール

```powershell
npm install @tiptap/react @tiptap/starter-kit @tiptap/extension-placeholder
npm install @prisma/client prisma
npm install next-auth
npm install @auth/prisma-adapter
```

### 6-3. Prismaの初期化

```powershell
npx prisma init
```

`prisma/schema.prisma` と `.env` ファイルが生成される。

---

## Step 7: Docker Composeの設定

### 7-1. docker-compose.yml の作成

プロジェクトルートに `docker-compose.yml` を作成する。

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://pagora:pagora@db:5432/pagora
      - NEXTAUTH_SECRET=your-secret-here
      - NEXTAUTH_URL=http://localhost:3000
    volumes:
      - ./:/app
      - /mnt/nas/uploads:/app/uploads
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: pagora
      POSTGRES_PASSWORD: pagora
      POSTGRES_DB: pagora
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

### 7-2. Dockerfile の作成

プロジェクトルートに `Dockerfile` を作成する。

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

### 7-3. .env の設定

`prisma/schema.prisma` を編集し、datasourceを設定する。

```
DATABASE_URL="postgresql://pagora:pagora@localhost:5432/pagora"
NEXTAUTH_SECRET="任意のランダム文字列"
NEXTAUTH_URL="http://localhost:3000"
```

---

## Step 8: バッファローNASのマウント設定

### 8-1. NASの共有フォルダを確認

NASの管理画面で共有フォルダ（例：`pagora-uploads`）を作成しておく。

### 8-2. Windowsのネットワークドライブとしてマウント

エクスプローラーを開き、「PC」→「ネットワークドライブの割り当て」から  
NASの共有フォルダをドライブとして割り当てる（例：`Z:\pagora-uploads`）。

### 8-3. 起動時に自動マウントされるよう設定

「ログイン時に再接続する」にチェックを入れる。

---

## Step 9: 起動確認

### 9-1. Docker Composeで起動

```powershell
docker compose up -d
```

### 9-2. DBマイグレーションの実行

```powershell
npx prisma migrate dev --name init
```

### 9-3. ブラウザで確認

```
http://localhost:3000
```

Pagoraのトップ画面が表示されればセットアップ完了。

---

## Step 10: Claude Codeのインストール

開発はClaude Codeで行うため、インストールしておく。

```powershell
npm install -g @anthropic-ai/claude-code
```

プロジェクトフォルダで起動する。

```powershell
cd C:\projects\Pagora
claude
```

---

## フォルダ構成（初期）

```
Pagora/
├── src/
│   ├── app/              # Next.js App Router
│   │   ├── api/          # APIルート
│   │   ├── (auth)/       # 認証関連ページ
│   │   └── (main)/       # メインページ
│   ├── components/       # Reactコンポーネント
│   │   ├── editor/       # Tiptapエディタ関連
│   │   ├── sidebar/      # サイドバー
│   │   └── ui/           # 共通UIコンポーネント
│   └── lib/              # ユーティリティ・設定
├── prisma/
│   └── schema.prisma     # DBスキーマ
├── public/               # 静的ファイル
├── docker-compose.yml
├── Dockerfile
├── SPEC.md               # 仕様書
├── SETUP.md              # 本ファイル
└── .env
```

---

## トラブルシューティング

### Docker Desktopが起動しない

WSL2が正しくインストールされているか確認する。

```powershell
wsl --status
```

### ポート3000が使用中

```powershell
netstat -ano | findstr :3000
```

該当のプロセスIDをタスクマネージャーで終了する。

### NASに接続できない

NASとWindowsPCが同じLANに接続されているか確認する。  
NASのIPアドレスにpingが通るか確認する。

```powershell
ping 192.168.x.x
```

---

*最終更新：2026-02-27*
