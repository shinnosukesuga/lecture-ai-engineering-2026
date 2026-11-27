# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：glossary_search_glossary
- STEP 2 の [tool call]：glossary_search_glossary({'query': '3レベル'})
- STEP 3 の [tool call]：fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- STEP 4 の [tool call]：aws_aws___search_documentation({'search_phrase': 'Amazon S3 バージョニング 概要'})、aws_aws___search_documentation({'search_phrase': '"Versioning" Amazon S3 overview what is it'})
- 気づき・疑問：
  - エージェントのコードは変えずに、設定ファイル（app/mcp_servers.json）に書き足すだけで、自作のサーバー・公開サーバー（fetch）・外部のサーバー（AWS）の道具を同じ形で使えた。ツールは「サーバー名_ツール名」の名前で、名前・説明文・引数の形でモデルに渡り、どこで動いているかはモデルからは見えない。
  - エージェントは「AI（判断役）＋道具＋指示文」を組み合わせた仕組み全体で、AIは道具を使う指示を返すだけ。実際に道具を動かすのはプログラム側だと分かった。--list-tools はダミーのモデルで一覧を表示するので、APIキーなしで動く。
  - fetch の説明文には「インターネットにアクセスできるようになった」とモデルの振る舞いを変える文が入っていた。説明文はそのままモデルに渡るので、外部サーバーを足すことは他人が書いた指示文を渡すことでもあり、提供元の確認が必要だと感じた。
  - STEP 4 では、1回目の日本語の検索で的外れな結果が返り、モデルが英語の検索語に変えて再検索していた（モデル応答3回）。道具が7個に増えて入力トークンも約11,000まで増えたため、道具を足しすぎるとコストや選び間違いが増えそうだと思った。また、AWS のサーバーには質問内容が送られるので、社外秘の情報は渡さないよう注意が必要。

## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：基礎
- やったこと：app/mcp_server.py の TODO の下に server.tool()(tools.get_lecture_summary) を1行書き足し、get_lecture_summary を MCP のツールとして公開した。エージェント側（app/mcp_agent.py と app/mcp_servers.json）は変えていない。
- 結果：
  - --list-tools でツールが7個から8個に増え、glossary_get_lecture_summary が表示された。
  - 「第2回では何を学びますか？」と質問すると、[tool call] glossary_get_lecture_summary({'lecture': 2}) が呼ばれ、第2回（ファインチューニングとLoRA）の要点をもとに回答した（モデル応答2回）。
  - uv run pytest -q は 21 passed。
- 気づき・疑問：
  - エージェントは起動するたびにサーバーへツールの一覧を問い合わせるので、サーバー側に1行足すだけで、エージェント側を変えずに新しいツールが使えた。ツールを使う側と作る側を分けられるのが MCP の利点だと実感した。
  - モデルは「第2回」という言葉から、用語検索（search_glossary）ではなく get_lecture_summary を選び、引数に整数の 2 を入れていた。関数の docstring（「回の内容を聞かれたときに使う」）と型ヒント（int）が、そのまま説明文と引数の形になってモデルの判断材料になっている。
  - 質問は短いのに入力トークンが約6,000あった。8個のツールの説明文が毎回モデルに渡るためで、ツールを増やすほど1回あたりのコストが上がる点は、実務で使うときに意識したい。