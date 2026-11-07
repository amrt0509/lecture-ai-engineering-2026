# 第3回 解答

## 演習
- 第1回（素のLLM）とエージェントの回答の違い：
  素のLLMでは講座固有の用語定義を正確に参照できず一般論の回答になりがちだったが、エージェントでは `search_glossary` ツールを用いて用語集（`glossary.json`）を検索し、正確な定義とともに末尾に関連講義回（第2回）を付与して回答できた。
- `--trace` で見えたツール呼び出し：
  `search_glossary({'query': 'LoRA'})`
  `search_glossary({'query': 'QLoRA'})`
- 1つの質問でのLLM呼び出し回数（requests）：
  2回（RunUsage requests=2, tool_calls=2）
- 追加したツールと docstring：
  `make_quiz(lecture: int) -> str`
  「指定した回の用語から1つを選び、確認問題と答えを返す。「第2回の確認問題を出して」のように、復習用の問題を求められたときに使う。」

## 宿題
- 取り組んだレベル：基礎
- やったこと：
  `app/agent.py` の `INSTRUCTIONS` から「用語や講座に固有のことを答える前に、必ず search_glossary で用語集を確認する。」の行を削除し、「LoRA と QLoRA の違いは何ですか？」という質問に対して `--trace` 付きでツールの呼ばれ方と回答の変化を検証した。
- 結果（`--trace` の出力や比較）：
  - 【変更前の予想】システムプロンプトの強制指示を外したため、ツール検索を行わずにLLMが直接回答すると予想した。
  - 【実際の実行結果】
    `[tool call] search_glossary({'query': 'LoRA'})`
    `[tool call] search_glossary({'query': 'QLoRA'})`
    強制指示を削除した後でも、モデルは自律的に用語集検索ツールを2回呼び出し、正確な用語集データをもとに回答を生成した（requests=2, tool_calls=2）。
- 気づき・疑問：
  エージェント全体の `INSTRUCTIONS` から強制文を外しても、`tools.py` 側の docstring に「答える前に使うこと」と明記されていることや、`INSTRUCTIONS` 内の「用語集に見つからないことは推測で断定しない」という別のルールが残っている場合、モデルはツールの必要性を自ら推論して選択・実行することが確認できた。エージェントの振る舞いはシステムプロンプト単体だけでなく、ツールの docstring との相互作用によって決まるという講義内容を実機で体感できた。