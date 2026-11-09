# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：第1回の素のLLMは用語集を参照できないため、「lecNN-start タグ」のような講座固有の言葉は推測や一般論で答えるしかない。エージェントは search_glossary({'query': 'lecNN-start タグ'}) で用語集を引き、用語集の定義どおりに「各回の演習開始時点のコードに付けたGitタグで、前の回が終わっていなくてもこのタグからブランチを作って始められる」と答えた（モデル：gemini-3.5-flash-lite）
- `--trace` で見えたツール呼び出し（ツール名と引数）：「LoRAとQLoRAの違いは？」では search_glossary({'query': 'LoRA'}) と search_glossary({'query': 'QLoRA'}) がまとめて呼ばれた。「lecNN-start タグ」の質問では search_glossary({'query': 'lecNN-start タグ'}) が1回呼ばれた
- 1つの質問での LLM 呼び出し回数（`requests`）：どちらの質問も requests=2。ツールの結果を受け取って回答を作るために、素のLLM（1回）より1回多くモデルが呼ばれる
- 追加したツールと、docstring に書いた説明：app/tools.py に make_quiz(lecture: int) を追加し、app/agent.py で登録した（pytest は 11 passed）。用語集から指定した回の用語を1つ選び、「この用語の意味は？」という問題と答え（用語集の定義）を返す。docstring には「理解確認の問題を求められたときに使う。まず問題だけを出題し、答えは学習者が答えたあとに示す」と書いた。実際、「第2回の内容から確認問題を1問出して」では make_quiz({'lecture': 2}) が呼ばれ、「事前学習」の問題だけが出題され、答えは示されなかった（requests=2）

## 宿題

- 取り組んだレベル：基礎
- やったこと：app/agent.py の INSTRUCTIONS から「用語や講座に固有のことを答える前に、必ず search_glossary で用語集を確認する。」の1行だけを消し（他の行は変更なし）、同じ質問「LoRAとQLoRAの違いは？」を --trace 付きで変更前・変更後に実行して比べた。実験後に指示は元に戻した
- 結果：変更前は search_glossary が2回（LoRA、QLoRA）、requests=2、出力195トークン。変更後も search_glossary は同じ2回呼ばれ、requests=2 のまま、出力219トークンで、回答の内容と最後の「関係する回：第2回」の表示もほぼ変わらなかった
- 気づき・疑問：INSTRUCTIONS の1行を消しただけでは挙動は変わらなかった。理由として、search_glossary の docstring 自体にも「答える前に使うこと」と書かれており、指示が INSTRUCTIONS とツールの説明文の両方に重ねて書かれているためだと考えられる。エージェントの挙動を変えるには、指示がどこに・いくつ書かれているかを確認したうえで変える必要がある。また、trace を見ないと「用語集に基づいて答えたか」は回答の見た目では区別できないことも確認できた
