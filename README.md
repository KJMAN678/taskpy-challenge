## 環境構築

### 方法 A: VSCode Dev Container を使用する場合（推奨）

1. VSCode で「Dev Containers」拡張機能をインストール
2. `.envrc.example` を `.envrc` にコピーし、環境変数を設定
3. VSCode でプロジェクトを開き、「Reopen in Container」を選択

Dev Container 起動時に以下が自動実行されます：
1. `uv sync --extra dev` - uv で開発依存関係をインストール
2. `uv run task lock_dev` - taskipy で requirements-dev.txt を生成
3. `uv run task install_dev` - pip で requirements-dev.txt からインストール

```sh
# コンテナ内でアプリケーションを起動
$ python manage.py migrate
$ python manage.py runserver 0.0.0.0:8000
```

### 方法 B: Docker Compose を直接使用する場合

#### 1. 環境変数の設定

```sh
# 環境変数用のファイル作成
$ cp .envrc.example .envrc
$ direnv allow
```

ローカル環境で direnv を使用する場合:
```sh
$ brew install direnv  # macOS
```

#### 2. Docker コンテナの起動

```sh
$ docker compose up -d

# コンテナの作り直しが必要な場合
$ source ./remake_container.sh
```

### 3. 開発依存関係の管理（taskipy）

このプロジェクトでは taskipy を使用して開発依存関係を管理しています。

#### ロックファイルの生成

pyproject.toml を編集した後、以下のコマンドで requirements-dev.txt を再生成します:

```sh
$ docker compose run --rm backend uv run task lock_dev
```

このコマンドは `uv pip compile` を実行し、pyproject.toml の `[project.optional-dependencies] dev` セクションから requirements-dev.txt を生成します。

#### 開発依存関係のインストール

```sh
$ docker compose run --rm backend uv run task install_dev
```

#### 依存関係の追加方法

1. pyproject.toml の `[project.optional-dependencies] dev` セクションにパッケージを追加
2. `task lock_dev` を実行して requirements-dev.txt を再生成
3. `uv lock` を実行して uv.lock を更新

```sh
# 例: 新しいパッケージを追加した後
$ docker compose run --rm backend uv lock
$ docker compose run --rm backend uv run task lock_dev
```

### 4. アプリケーションの起動

```sh
$ docker compose up -d
```

アプリケーションURL: http://localhost:8000/web/

---

### Command

```sh
# app 追加
$ mkdir web
$ docker compose run --rm backend uv run django-admin startapp web web
$ docker compose run --rm backend uv run ruff check . --fix
$ docker compose run --rm backend uv run ruff format .
# djlint によるフォーマット
$ docker compose run --rm backend uv run djlint templates/*/*.html --extension=html.j2 --reformat

http://127.0.0.1:8000/web/

# セキュリティチェック
$ docker compose run --rm backend uv tool run djcheckup http://host.docker.internal:8000/web/
```

### Devin

- [Devin's Machine](https://app.devin.ai/workspace) でリポジトリ追加

#### 1.Git Pull
- そのまま

#### 2.Configure Secrets
```sh
# 環境変数用のファイル作成
$ touch .envrc
$ cp .envrc.example .envrc
$ direnv allow
```

- ローカル用
```sh
$ brew install direnv
```
#### 4.Maintain Dependencies
```sh
$ docker compose up -d

# コンテナ作り直し
$ source ./remake_container.sh
```

#### 5.SetUp Lint
```sh
$ docker compose run --rm backend uv run ruff check .

# 参考 フォーマット
$ docker compose run --rm backend uv run ruff format .

# mypy による型ヒントチェック
$ docker compose run --rm backend uv run mypy .

$ docker compose run --rm backend uv run djlint templates/*/*.html --extension=html.j2 --lint
```

#### 6.SetUp Tests
- no tests ran in 0.00s だと Devin の Verify が通らないっぽい
```sh
$ docker compose run --rm backend uv run pytest
```

### 7.Setup Local App

```sh
$ http://localhost:8000/ がアプリケーションのURL
```

#### 8.Additional Notes
- 必ず日本語で回答してください
- Python, Django を利用する
- データベースは Postgres
- テストは pytest を利用する
を入力

