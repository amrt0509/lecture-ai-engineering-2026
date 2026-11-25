# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：glossary_search_glossary（および実装宿題で追加した glossary_make_quiz。app/agent.py 側は search_materials の1個）
- STEP 2 の [tool call]：glossary_search_glossary({'query': '3レベル'})
- STEP 3 の [tool call]：fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- STEP 4 の [tool call]：aws_aws___search_documentation({'topics': ['general'], 'search_phrase': 'Amazon S3 versioning overview'})
- 気づき・疑問：`app/mcp_servers.json` に fetch と aws を追加する前は、外部サイトや AWS ドキュメントを検索するツールが存在しないためモデル応答1回・ツール呼び出しなしで回答していたが、設定追記後は `fetch_fetch` や `aws_aws___search_documentation` が呼ばれるようになり、外部情報に基づく回答へ変化した。また、STEP 3 では `fetch_fetch` の後に `glossary_search_glossary({'query': 'MCP'})` も併用されており、外部取得と講座用語集を組み合わせて回答を組み立てていた。

## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：実装
- やったこと：`app/tools.py` に講義回（`lecture: int`）を引数とする `make_quiz` 関数を追加し、`lecture < 1` または対象講義の要点ファイル（`data/lectures/{lecture:02d}.md`）が存在しない場合に、範囲外である旨と利用可能な講義回一覧を含む分かりやすいメッセージを返す処理を実装した。また、`app/mcp_server.py` に `server.tool()(tools.make_quiz)` を追加して MCP サーバーから公開した。
- 結果：`uv run python -m app.mcp_agent --list-tools` で `glossary_make_quiz`（引数 `lecture: integer`）が公開されていることを確認した。続いて `uv run python -m app.mcp_agent --trace "第99回のクイズを作って"` を実行したところ、`[tool call] glossary_make_quiz({'lecture': 99})` が呼ばれ、ツールからの範囲外案内メッセージ（`01, 02, 03, 04, 06`）をもとに、モデルが第99回が存在しない旨と指定可能な回を分かりやすく提示した（入力 864 tokens / 出力 117 tokens / モデル応答 2 回）。
- 気づき・疑問：ツール側で「何が範囲外か」と「指定可能な候補」を明示したメッセージを返すことで、モデルが勝手に第99回の内容を捏造（ハルシネーション）せず、正確な案内へ誘導できた。一方で、利用可能な講義回（`01, 02, 03, 04, 06`）に `05` が含まれていないこともツール出力から把握できたため、データ側の用意状況も合わせて確認できる点が有用だと感じた。