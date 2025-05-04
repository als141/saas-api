# SEO Article Generation API

このプロジェクトは、指定されたキーワードやペルソナに基づいてSEO記事を生成するためのFastAPIバックエンドAPIです。内部ではOpenAI Agents SDKを利用し、複数のAIエージェント（テーマ提案、リサーチ、アウトライン作成、執筆、編集）が連携して記事を作成します。

記事生成の進捗状況と最終的な結果は、Server-Sent Events (SSE) を介してリアルタイムでクライアントにストリーミングされます。

## 機能

* キーワード、ペルソナ、目標文字数などの入力に基づいてSEO記事を生成
* OpenAI Agents SDKを利用したマルチエージェントシステム
    * テーマ提案エージェント
    * リサーチプランナーエージェント
    * ディープリサーチャーエージェント (詳細な情報と出典URLを収集)
    * リサーチシンセサイザーエージェント (詳細なレポートを作成)
    * アウトライン作成エージェント
    * セクション執筆エージェント (リサーチ結果に基づき、必要に応じて出典リンクを生成)
    * 編集エージェント (リサーチとの整合性、リンクの適切性をチェック)
* Server-Sent Events (SSE) によるリアルタイムな進捗更新と結果ストリーミング
* FastAPIによる非同期処理
* 設定管理 (.envファイル)
* カスタム例外処理

## セットアップ

1.  **リポジトリのクローン:**
    ```bash
    git clone <repository_url>
    cd seo_article_api
    ```

2.  **uv のインストール (まだの場合):**
    ```bash
    # macOS / Linux
    curl -LsSf [https://astral.sh/uv/install.sh](https://astral.sh/uv/install.sh) | sh
    # Windows
    powershell -c "irm [https://astral.sh/uv/install.ps1](https://astral.sh/uv/install.ps1) | iex"
    ```
    詳細は [uv 公式ドキュメント](https://github.com/astral-sh/uv) を参照してください。

3.  **uv を使用した仮想環境の作成と有効化:**
    ```bash
    uv venv  # .venv ディレクトリに仮想環境を作成
    source .venv/bin/activate  # Linux/macOS
    # .venv\Scripts\activate  # Windows
    ```

4.  **依存ライブラリのインストール (uv を使用):**
    ```bash
    uv pip install -r requirements.txt
    ```

5.  **環境変数の設定:**
    * `.env.example` ファイルをコピーして `.env` ファイルを作成します。
    * `.env` ファイルを開き、`OPENAI_API_KEY` にあなたのOpenAI APIキーを設定します。
    * 必要に応じて、他のAPIキーやモデル名を環境変数で設定します。

## APIの実行 (開発用)

以下のコマンドでFastAPIサーバーを起動します。
```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```--reload` オプションにより、コード変更時にサーバーが自動的に再起動します。

APIサーバーは `http://localhost:8000` で利用可能になります。

## APIドキュメント

サーバー起動後、以下のURLにアクセスすると、Swagger UIによるインタラクティブなAPIドキュメントが表示されます。
* Swagger UI: `http://localhost:8000/docs`
* ReDoc: `http://localhost:8000/redoc`

## エンドポイント

* **`POST /articles/generate-stream`**:
    * 記事生成プロセスを開始し、SSEストリームを返します。
    * リクエストボディには `schemas.request.GenerateArticleRequest` で定義されたパラメータを指定します。
    * レスポンスは `text/event-stream` 形式で、`schemas.response.AnySSEventData` に基づくJSONデータがストリーミングされます。

## SSEイベントの例

クライアントは以下のようなイベントを受信します（`data:`フィールドの値はJSON文字列です）。

data: {"event_type": "status_update", "step": "start", "message": "Starting step: start"}data: {"event_type": "theme_proposed", "themes": [{"title": "...", "description": "...", "keywords": [...]}, ...], "selected_theme_index": 0}data: {"event_type": "research_plan_generated", "plan": {"topic": "...", "queries": [{"query": "...", "focus": "..."}, ...]}}data: {"event_type": "research_progress", "query_index": 0, "total_queries": 5, "query": "..."}data: {"event_type": "section_generated", "section_index": 0, "heading": "...", "html_content_chunk": "これは生成されたHTMLの一部です...", "is_complete": false}data: {"event_type": "section_generated", "section_index": 0, "heading": "...", "html_content_chunk": "", "is_complete": true}data: {"event_type": "final_result", "title": "最終的な記事タイトル", "final_html_content": "..."}data: {"event_type": "error", "step": "writing_sections", "error_message": "..."}
## 今後の拡張

* **状態管理の強化:** 長時間実行されるタスクのために、Redisやデータベースを用いたタスク状態の永続化。
* **インタラクティブ性の向上:** WebSocketなどを利用して、生成プロセス中のユーザーによる選択（テーマ選択など）を可能にする。
* **認証・認可:** APIエンドポイントへのアクセス制御。
* **非同期タスクキュー:** CeleryやARQを導入し、記事生成プロセスをバックグラウンドで実行する。
* **ツールの拡充:** ダミーのツールを実際のAPI呼び出しやデータベースアクセスに置き換える。
* **モデル選択の柔軟性:** APIリクエストでモデルを指定できるようにする。
* **テスト:** 単体テスト、結合テスト、E2Eテストを追加する。
* **デプロイ:** Docker化し、クラウドプラットフォーム（AWS, GCP, Azureなど）へのデプロイ設定を行う。

