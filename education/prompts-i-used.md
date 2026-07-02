# 使ったプロンプトと評価

演習中に Codex CLI に投げた主要なプロンプトを記録する。
`EXERCISES.html` に記載の機能実装や、修正提案に関するプロンプトを記述する。

## 書き方

- どのフェーズで使ったか (例: 投稿一覧、いいね機能、リファクタ)
- 実際に投げたプロンプト本文 (省略せずコピペ)
- 結果: 効いた / 部分的に効いた / 効かなかった
- 振り返り: 次に同じ状況が来たらどう変えるか

---

## プロンプト 1(サンプル)

**フェーズ**: サンプルデータ(シーダー)の追加

**プロンプト本文**:

```
src\main\resources\db\migration\V1__init.sql に基づくH2用のシーダーを作成し、実行する。テストデータとして10個程度のシードデータを作成。H2で起動時に自動投入する。
```

**結果**: 効いた

```
• 提案どおり、H2 プロファイル起動時だけ seed が自動適用される構成にしました。

  差分:

  - src/main/resources/application-h2.yml:20: H2 の Flyway location に classpath:db/seed/h2 を追加。
  - src/main/resources/db/seed/h2/R__seed_posts_h2.sql:6: posts に 10 件投入する H2 専用 repeatable migration を追加。posts_seq は次
    ID が 11 になるよう更新。

  - src/test/resources/application-h2.yml:17: テストは空 DB 前提を維持するため seed location を外す設定を追加。
  - src/test/java/com/example/tsubuyaki/db/H2SeedMigrationTest.java:18: seed 適用で 10 件入ることと、次 sequence 値が 11 になることを
    検証。

  検証:

  - .\mvnw -B -Dmaven.repo.local=C:/home/codex/.m2/repository -Dtest=H2SeedMigrationTest test 成功
  - .\mvnw -B -Dmaven.repo.local=C:/home/codex/.m2/repository -Ph2 spring-boot:run "-Dspring-boot.run.arguments=--spring.main.web-
    application-type=none" 成功。ログで Migrating schema "PUBLIC" with repeatable migration "seed posts h2" を確認。

  - .\mvnw -B -Dmaven.repo.local=C:/home/codex/.m2/repository -Ph2 verify 成功。警告: Rule violated for bundle tsubuyaki-board: lines
    covered ratio is 0.48, but expected minimum is 0.60。日本語要約: 現在のカバレッジは 48% で 60% 未満ですが、このプロファイルでは
    fail 設定ではないためビルドは成功しています。
```

**振り返り**:

シーダーは追加できた。
検証方法を指定しなかったためハーネスであらかじめ定義された検証方法を自動的に起動した。
CI検証は成功しているが、実際に画面表示の確認はできていない(未実装のため)。

---

## プロンプト 2

**フェーズ**: サンプルデータ(シーダー)を100件に変更

**プロンプト本文**

```
 src\main\resources\db\migration\V1__init.sql に基づくH2用のシーダーを作成し、実行する。テストデータとして100個程度のシードデータを作成。H2で起動時に自動投入する。
```

**結果**: 効いた
差分：
    18      @Test
    19 -    @DisplayName("H2シーダー_起動時_投稿10件を投入し次IDが11になる")
    20 -    void H2シーダー_起動時_投稿10件を投入し次Idが11になる() throws SQLException {
    19 +    @DisplayName("H2シーダー_起動時_投稿100件を投入し次IDが101になる")
    20 +    void H2シーダー_起動時_投稿100件を投入し次Idが101になる() throws SQLException {
    21          String url = "jdbc:h2:mem:seed_" + UUID.randomUUID().toString().replace("-", "")
       ⋮
    31               Statement statement = connection.createStatement()) {
    32 -            assertThat(singleLong(statement, "SELECT COUNT(*) FROM posts")).isEqualTo(10);
    33 -            assertThat(singleLong(statement, "SELECT NEXT VALUE FOR posts_seq")).isEqualTo(11);
    32 +            assertThat(singleLong(statement, "SELECT COUNT(*) FROM posts")).isEqualTo(100);
    33 +            assertThat(singleLong(statement, "SELECT NEXT VALUE FOR posts_seq")).isEqualTo(101);
    34          }

検証

**振り返り**:

---

## プロンプト 3

**フェーズ**:投稿登録画面

**プロンプト本文**:

```
› ■ 投稿登録（POST /posts）実装プロンプト

  あなたはSpring Boot（MVC + Thymeleaf + Bean Validation）の開発者です。
  ■ 機能概要

  画面：posts/form
  一覧画面：posts/list
  ■ バリデーション仕様
  author（投稿者）
  必須
  空白のみは禁止
  ■ 成功時の処理
  HTTPステータス：302
  リダイレクト先：/posts
  DBに投稿を保存する（Service層経由）
  ■ 失敗時の処理（バリデーションエラー）
  HTTPステータス：200
  画面：posts/form を再表示
  入力値は保持する
  エラーメッセージを画面に表示する
  ■ 実装要件
  Controller
  @PostMapping("/posts")
  @Valid @ModelAttribute PostForm form
  BindingResult result を必ず直後に置く
  バリデーション

  以下を使用すること：

  @NotBlank
  @Size(min=1, max=30)
  @Size(min=1, max=280)

  ※ 空白のみ対策は @NotBlank を使用すること

  完了条件は ./mvnw -B -Ph2 test が緑になることです。
```

**結果**: 部分的に効いた
差分
  - src/main/java/com/example/tsubuyaki/controller/PostController.java:34: POST /posts 追加
  - src/main/java/com/example/tsubuyaki/web/dto/PostForm.java:8: @NotBlank + @Size(min=1, ...) に更新
  - src/main/java/com/example/tsubuyaki/service/PostService.java:25: 投稿保存 create 追加
  - src/test/java/com/example/tsubuyaki/controller/PostControllerTest.java:119: 成功・空白のみエラーの Controller テスト追加
  - src/test/java/com/example/tsubuyaki/service/PostServiceTest.java:44: 保存処理の Service テスト追加
検証:
  ./mvnw -B -Ph2 test 成功。Tests run: 16, Failures: 0, Errors: 0, Skipped: 0 です。Surefire の fork JVM 終了待ち警告は出ていますが、最終結果は BUILD SUCCESS / exit code 0 でした。

**振り返り**:
  30文字しか入力できない仕様になっていたため、30文字以上のエラーの確認ができなかった

---

## プロンプト 4

**フェーズ**:投稿登録画面

**プロンプト本文**:

```

■ 投稿登録画面

あなたはSpring Boot（MVC + Thymeleaf + Bean Validation）の開発者です。
現在、HTML側の入力制限（maxlength / required）により、サーバー側バリデーション（@Valid）の動作検証ができない状態です。

そのため、バックエンドのバリデーションを正しくテストできるようにフロントエンドの制約を撤廃してください。

■ 目的
サーバーサイドバリデーション（@Valid）の動作確認を可能にする
HTML側の制限に依存しない設計にする
不正入力をバックエンドで検知できる状態にする


■ バックエンド条件（再確認）

PostFormには以下が存在すること：

@NotBlank
@Size(max=30)
@Size(max=280)

Controllerは以下：

@Valid + BindingResult を使用

■ 期待する動作
成功時
302リダイレクト /posts
失敗時（バリデーションエラー）
200で posts/form 再表示
エラーメッセージ表示
入力値保持

■ 完成状態
フロント：制限なし入力フォーム
バックエンド：Springで厳密チェック
テスト：31文字入力でエラー確認可能


完了条件は ./mvnw -B -Ph2 test が緑になることです。

```

**結果**: 効いた
差分
 • Edited 2 files (+33 -3)
  └ src/main/resources/templates/posts/form.html (+3 -3)
    18              <label for="author">投稿者</label>
    19 -            <input type="text" id="author" th:field="*{author}" maxlength="30" required>
    19 +            <input type="text" id="author" th:field="*{author}">
    20              <p class="error" th:if="${#fields.hasErrors('author')}" th:errors="*{author}">エラー</p>
       ⋮
    22          <div>
    23 -            <label for="body">本文 (280 文字まで)</label>
    24 -            <textarea id="body" th:field="*{body}" maxlength="280" rows="4" required></textarea>
    23 +            <label for="body">本文</label>
    24 +            <textarea id="body" th:field="*{body}" rows="4"></textarea>
    25              <p class="error" th:if="${#fields.hasErrors('body')}" th:errors="*{body}">エラー</p>

  └ src/test/java/com/example/tsubuyaki/controller/PostControllerTest.java (+30 -0)
    131      @Test
    133 +    void newForm_HTML入力制限を出力しない() throws Exception {
    134 +        MvcResult result = mockMvc.perform(get("/posts/new"))
    135 +                .andExpect(status().isOk())
    136 +                .andExpect(view().name("posts/form"))
    137 +                .andReturn();
    139 +        String html = result.getResponse().getContentAsString();
    140 +        assertThat(html).doesNotContain("maxlength=");
    141 +        assertThat(html).doesNotContain(" required");
    142 +    }
    143 +
    144 +    @Test
    145      @DisplayName("投稿登録_空白のみのとき_入力値を保持してフォームを再表示する")
        ⋮
    160
    161 +    @Test
    162 +    @DisplayName("投稿登録_投稿者名31文字のとき_エラーを表示して入力値を保持する")
    163 +    void create_投稿者名31文字のとき_エラーを表示して入力値を保持する() throws Exception {
    164 +        String author = "a".repeat(31);
    165 +
    166 +        mockMvc.perform(MockMvcRequestBuilders.post("/posts")
    167 +                        .param("author", author)
    168 +                        .param("body", "本日の共有です"))
    169 +                .andExpect(status().isOk())
    170 +                .andExpect(view().name("posts/form"))
    171 +                .andExpect(model().attributeHasFieldErrors("postForm", "author"))
    172 +                .andExpect(content().string(containsString("投稿者名は 1 文字以上 30 文字以内で入力してください")))
    173 +                .andExpect(content().string(containsString("value=\"" + author + "\"")));
    174 +
    175 +        verify(postService, never()).create(author, "本日の共有です");
    176 +    }
    177 +
    178      private static Post post(String author, String body, Instant createdAt) {

  根拠:

  - src/main/resources/templates/posts/form.html の maxlength / required を削除
  - 本文 (280 文字まで) の表示も 本文 に変更し、フロント側の制限表現を除去
  - src/test/java/com/example/tsubuyaki/controller/PostControllerTest.java に以下を追加
      - フォームHTMLに maxlength= / required が出ないこと
      - 投稿者名31文字で 200、posts/form 再表示、エラー表示、入力値保持、Service未呼び出しになること

  検証:

  - ./mvnw -B -Ph2 test
  - 結果: Tests run: 18, Failures: 0, Errors: 0, Skipped: 0
  - 最終結果: BUILD SUCCESS

**振り返り**:
  成功: 302 で /posts にリダイレクト
　失敗: 200 で posts/form を再表示しエラー表示　できることを確認した。

---

## プロンプト　5

**フェーズ**: 詳細一覧

**プロンプト本文**:

```
あなたはSpring Boot（MVC + Service層 + Thymeleaf）の開発者です。
以下の仕様に従って「投稿詳細表示機能（M4）」を実装してください。

■ 機能概要
エンドポイント：GET /posts/{id}
目的：指定した投稿IDの詳細を表示する
画面：posts/detail.html
■ 表示仕様
正常系（存在するID）
投稿データを取得する
posts/detail.html に表示する
表示項目：
author（投稿者）
body（本文）
createdAt（投稿日時）
■ 異常系（存在しないID）
該当データが存在しない場合
HTTPステータス：404 Not Found
エラーページまたはSpring標準の404を返す
■ Controller実装要件
@GetMapping("/posts/{id}") を使用
@PathVariable Long id を使用
Service層を経由してデータ取得すること
Optionalやnullチェックを適切に行うこと
■ Service要件
findById(id) のようなメソッドを使用
データが存在しない場合の扱いはControllerに委譲する
■ View（Thymeleaf）
作成必須
templates/posts/detail.html を作成すること
表示内容
投稿者（author）
本文（body）
投稿日時（createdAt）
■ 注意事項
Serviceでnullを返す設計は禁止ではないが、Optional推奨
Controllerで必ず404制御を行うこと
HTMLは新規作成すること（既存流用不可）
■ ゴール
投稿IDを指定して詳細画面を表示し、存在しない場合は正しく404を返すこと
```

**結果**: 効いた

```
  差分:

  - src/main/java/com/example/tsubuyaki/service/PostService.java に Optional<Post> findById(Long id) を追加
  - src/main/java/com/example/tsubuyaki/controller/PostController.java に GET /posts/{id} を追加
      - 存在するID: posts/detail を表示
      - 存在しないID: 404 Not Found

  - src/main/resources/templates/posts/detail.html を新規作成
      - author
      - body
      - createdAt
        を th:text / time で表示

  - src/test/java/com/example/tsubuyaki/controller/PostControllerTest.java に正常系・404系を追加
  - src/test/java/com/example/tsubuyaki/service/PostServiceTest.java に findById の委譲テストを追加

  検証:

  - ./mvnw -B -Ph2 test
  - 結果: Tests run: 21, Failures: 0, Errors: 0, Skipped: 0
  - BUILD SUCCESS

```

**振り返り**:





## プロンプト6

**フェーズ**: S2 キーワード検索

**プロンプト本文**:

```
タスク:  S2: キーワード検索を TDD で実装してください。

順序:
1. まず受入基準を満たすための失敗テストを 1 本だけ書きます (Red)。
2. その失敗テストを通す最小実装を書きます (Green)。
3. 重複・命名・抽象度の観点でリファクタリングします (Refactor)。
4. `./mvnw -B -Ph2 test` で全テストが緑であることを確認します。
5. Conventional Commits でコミットします (`feat(<scope>): <要約>`)。

受入基準:
GET /posts?q=xxx で本文 LIKE 検索
一覧画面を再利用 (検索ボックスを上部に追加)

制約:
- AGENTS.md と .codex/instructions.md を参照すること。
- このタスクのスコープ外のファイルは触らないこと。
- テストを書かずに本体コードを書き始めないこと。
- 失敗テストを通すために `@Disabled` を貼ることは絶対に禁止。
- 完了後にどの 3 ファイル前後を変更したかを要約して報告すること。
```

**結果**: 効いた

```
• Edited 4 files (+22 -1)
  └ src/main/java/com/example/tsubuyaki/controller/PostController.java (+12 -1)
     14  import org.springframework.web.bind.annotation.PostMapping;
     15 +import org.springframework.web.bind.annotation.RequestParam;
     16  import org.springframework.web.server.ResponseStatusException;
        ⋮
     32      @GetMapping({ "/", "/posts", "/posts/" })
     32 -    public String list(Model model) {
     33 +    public String list(@RequestParam(name = "q", required = false) String query, Model model) {
     34 +        if (hasText(query)) {
     35 +            model.addAttribute("posts", postService.searchByBodyContaining(query));
     36 +            model.addAttribute("q", query);
     37 +            return "posts/list";
     38 +        }
     39          model.addAttribute("posts", postService.findLatest50());
     40 +        model.addAttribute("q", "");
     41          return "posts/list";
        ⋮
     95      }
     96 +
     97 +    private static boolean hasText(String value) {
     98 +        return value != null && !value.isBlank();
     99 +    }
    100  }

  └ src/main/java/com/example/tsubuyaki/repository/PostRepository.java (+2 -0)
    10      List<Post> findTop50ByOrderByCreatedAtDesc();
    12 +    List<Post> findTop50ByBodyContainingOrderByCreatedAtDesc(String keyword);
    13  }

  └ src/main/java/com/example/tsubuyaki/service/PostService.java (+4 -0)
    31
    33 +        return repository.findTop50ByBodyContainingOrderByCreatedAtDesc(keyword);
    34 +    }
    35 +
    36      public Optional<Post> findById(Long id) {

  └ src/main/resources/templates/posts/list.html (+4 -0)
    19      </form>
    20 +    <form class="post-list__search" method="get" th:action="@{/posts}">
    21 +        <input type="search" name="q" th:value="${q}" placeholder="本文を検索">
    22 +        <button type="submit">検索</button>
    23 +    </form>

```

**振り返り**:
  検索欄が作成され、条件抽出ができた。
  今回は初めてテンプレートのプロンプトを使用して実装を行った。自作のプロンプトと比べて余計な出力が少なく、細かく効率的に実装を進めることができたため、今後もこの形式を活用していきたい。
  また、実装した機能については問題なく動作することを確認できた。一方で、コミットを細かい単位で行えていなかったことに気付いたため、今後は機能ごとやキリの良いタイミングでコミットすることを意識して作業を進めたい。


## プロンプト 7

**フェーズ**: 投稿者名フィールド拡張

**プロンプト本文**:

```
› タスク:  S3: 投稿者名フィールド拡張を TDD で実装してください。

  順序:
  1. まず受入基準を満たすための失敗テストを 1 本だけ書きます (Red)。
  2. その失敗テストを通す最小実装を書きます (Green)。
  3. 重複・命名・抽象度の観点でリファクタリングします (Refactor)。
  4. `./mvnw -B -Ph2 test` で全テストが緑であることを確認します。
  5. Conventional Commits でコミットします (`feat(<scope>): <要約>`)。

  受入基準:
  投稿者名を必須にし、フォームに任意のアバター色を選択させる (簡易)

  制約:
  - AGENTS.md と .codex/instructions.md を参照すること。
  - このタスクのスコープ外のファイルは触らないこと。
  - テストを書かずに本体コードを書き始めないこと。
  - 失敗テストを通すために `@Disabled` を貼ることは絶対に禁止。
```

**結果**: 部分的に効いた

```

  • Edited 2 files (+29 -0)
  └ src/main/java/com/example/tsubuyaki/web/dto/PostForm.java (+10 -0)
    15
    16 +    private String avatarColor;
    17 +
    18      public PostForm() {
       ⋮
    35      }
    36 +
    37 +    public String getAvatarColor() {
    38 +        return avatarColor;
    39 +    }
    40 +
    41 +    public void setAvatarColor(String avatarColor) {
    42 +        this.avatarColor = avatarColor;
    43 +    }
    44  }

  └ src/main/resources/templates/posts/form.html (+19 -0)
    21          </div>
    22 +        <fieldset>
    24 +            <label>
    25 +                <input type="radio" th:field="*{avatarColor}" value="">
    26 +                指定なし
    27 +            </label>
    28 +            <label>
    30 +                青
    31 +            </label>
    32 +            <label>
    33 +                <input type="radio" th:field="*{avatarColor}" value="green">
    34 +                緑
    35 +            </label>
    36 +            <label>
    37 +                <input type="radio" th:field="*{avatarColor}" value="pink">
    38 +                ピンク
    39 +            </label>
    40 +        </fieldset>
    41          <div>

```

**振り返り**:
  アバターの色は指定できるようになったが、実際にUIは変わらなかった。
  一覧で選択した色で出力されるように依頼分を投げる

## プロンプト8

**フェーズ**: 投稿者名フィールド拡張

**プロンプト本文**:

```
  アバターの色は指定できますが、一覧で選択した色になるようにしてください
  制約:
  - AGENTS.md と .codex/instructions.md を参照すること。
  - このタスクのスコープ外のファイルは触らないこと。
  - テストを書かずに本体コードを書き始めないこと。
  - 失敗テストを通すために `@Disabled` を貼ることは絶対に禁止。
```

**結果**: 効いた

```
• Edited 5 files (+28 -3)
  └ src/main/java/com/example/tsubuyaki/controller/PostController.java (+1 -1)
    73          }
    74 -        postService.create(form.getAuthor(), form.getBody());
    74 +        postService.create(form.getAuthor(), form.getBody(), form.getAvatarColor());
    75          return "redirect:/posts";

  └ src/main/java/com/example/tsubuyaki/domain/Post.java (+12 -0)
    28
    29 +    @Column(name = "avatar_color", length = 20)
    30 +    private String avatarColor;
    31 +
    32      @Column(name = "created_at", nullable = false)
       ⋮
    39      public Post(String author, String body, Instant createdAt) {
    40 +        this(author, body, createdAt, null);
    41 +    }
    42 +
    43 +    public Post(String author, String body, Instant createdAt, String avatarColor) {
    44          this.author = author;
       ⋮
    46          this.createdAt = createdAt;
    47 +        this.avatarColor = avatarColor;
    48      }
       ⋮
    61
    62 +    public String getAvatarColor() {
    63 +        return avatarColor;
    64 +    }
    65 +
    66      public Instant getCreatedAt() {

  └ src/main/java/com/example/tsubuyaki/service/PostService.java (+6 -1)
    58      public void create(String author, String body) {
    59 -        repository.save(new Post(author, body, Instant.now()));
    59 +        create(author, body, null);
    60 +    }
    61 +
    62 +    @Transactional
    63 +    public void create(String author, String body, String avatarColor) {
    64 +        repository.save(new Post(author, body, Instant.now(), avatarColor));
    65      }

  └ src/main/resources/db/migration/V2__add_avatar_color_to_posts.sql (+6 -0)
    1 +-- =========================================================================
    2 +-- 社内つぶやきボード V2: POSTS にアバター色を追加
    3 +-- Oracle XE 21c および H2(MODE=Oracle) の双方で動く DDL
    4 +-- =========================================================================
    5 +
    6 +ALTER TABLE posts ADD avatar_color VARCHAR2(20 CHAR);

  └ src/main/resources/templates/posts/list.html (+3 -1)
    27          <article class="post" th:each="post : ${posts}">
    28 -            <p class="post__author" th:text="${post.author}">投稿者</p>
    28 +            <p class="post__author"
    29 +               th:style="${#strings.isEmpty(post.avatarColor)} ? null : |color: ${post.avatarColor}|"
    30 +               th:text="${post.author}">投稿者</p>
    31              <p class="post__body" th:text="${post.body}">本文</p>

```

**振り返り**:

  一覧で選択した色で出力されるようになった。
  余裕があれば、詳細画面などでも反映したい。


## プロンプト 9

**フェーズ**: C2: 投稿削除

**プロンプト本文**:

```
› タスク:  C2: 投稿削除を TDD で実装してください。

  順序:
  1. まず受入基準を満たすための失敗テストを 1 本だけ書きます (Red)。
  2. その失敗テストを通す最小実装を書きます (Green)。
  3. 重複・命名・抽象度の観点でリファクタリングします (Refactor)。
  4. `./mvnw -B -Ph2 test` で全テストが緑であることを確認します。
  5. Conventional Commits でコミットします (`feat(<scope>): <要約>`)。

  受入基準:
  論理削除 (deleted_at カラム)、論理削除済みは一覧で非表示。
  削除ボタンは詳細画面に作ること

  制約:
  - AGENTS.md と .codex/instructions.md を参照すること。
  - このタスクのスコープ外のファイルは触らないこと。
  - テストを書かずに本体コードを書き始めないこと。
  - 失敗テストを通すために `@Disabled` を貼ることは絶対に禁止。
  - 完了後にどの 3 ファイル前後を変更したかを要約して報告すること。

  完了条件は ./mvnw -B -Ph2 test
  ./mvnw -B -Ph2 -Pcoverage-day3 -Pstrict verify が緑になることです。
```

**結果**: 効いた

```
  • Edited src/main/java/com/example/tsubuyaki/domain/Post.java (+11 -0)
    34
    35 +    @Column(name = "deleted_at")
    36 +    private Instant deletedAt;
    37 +
    38      protected Post() {
       ⋮
    72
    73 +    public Instant getDeletedAt() {
    74 +        return deletedAt;
    75 +    }
    76 +
    77 +    public void markDeleted(Instant deletedAt) {
    78 +        this.deletedAt = deletedAt;
    79 +    }
    80 +
    81      @Override

• Edited src/main/java/com/example/tsubuyaki/repository/PostRepository.java (+5 -2)
     6  import java.util.List;
     7 +import java.util.Optional;
     8
       ⋮
    10
    10 -    List<Post> findTop50ByOrderByCreatedAtDesc();
    11 +    List<Post> findTop50ByDeletedAtIsNullOrderByCreatedAtDesc();
    12
    12 -    List<Post> findTop50ByBodyContainingOrderByCreatedAtDesc(String keyword);
    13 +    List<Post> findTop50ByDeletedAtIsNullAndBodyContainingOrderByCreatedAtDesc(String keyword);
    14 +
    15 +    Optional<Post> findByIdAndDeletedAtIsNull(Long id);
    16  }

• Edited src/main/java/com/example/tsubuyaki/service/PostService.java (+9 -3)
    28      public List<Post> findLatest50() {
    29 -        return repository.findTop50ByOrderByCreatedAtDesc();
    29 +        return repository.findTop50ByDeletedAtIsNullOrderByCreatedAtDesc();
    30      }
       ⋮
    32      public List<Post> searchByBodyContaining(String keyword) {
    33 -        return repository.findTop50ByBodyContainingOrderByCreatedAtDesc(keyword);
    33 +        return repository.findTop50ByDeletedAtIsNullAndBodyContainingOrderByCreatedAtDesc(keyword);
    34      }
       ⋮
    36      public Optional<Post> findById(Long id) {
    37 -        return repository.findById(id);
    37 +        return repository.findByIdAndDeletedAtIsNull(id);
    38      }
       ⋮
    65      }
    66 +
    67 +    @Transactional
    68 +    public void delete(Long id) {
    69 +        repository.findByIdAndDeletedAtIsNull(id)
    70 +                .ifPresent(post -> post.markDeleted(Instant.now()));
    71 +    }
    72  }

• Edited src/main/java/com/example/tsubuyaki/controller/PostController.java (+6 -0)
    68
    69 +    @PostMapping("/posts/{id}/delete")
    70 +    public String delete(@PathVariable Long id) {
    71 +        postService.delete(id);
    72 +        return "redirect:/posts";
    73 +    }
    74 +
    75      @PostMapping("/posts")

• Added src/main/resources/db/migration/V3__add_deleted_at_to_posts.sql (+8 -0)
    1 +-- =========================================================================
    2 +-- 社内つぶやきボード V3: POSTS に論理削除日時を追加
    3 +-- Oracle XE 21c および H2(MODE=Oracle) の双方で動く DDL
    4 +-- =========================================================================
    5 +
    6 +ALTER TABLE posts ADD deleted_at TIMESTAMP(6);
    7 +
    8 +CREATE INDEX posts_deleted_at_created_at_idx ON posts (deleted_at, created_at);

```

**振り返り**:
  削除ボタン押下で、一覧に出力されないようになったが、論理削除されているのか確認が必要




## プロンプト 10

**フェーズ**: 一覧画面

**プロンプト本文**:

```
タスク:  詳細画面遷移ボタンを TDD で実装してください。

  順序:
  1. まず受入基準を満たすための失敗テストを 1 本だけ書きます (Red)。
  2. その失敗テストを通す最小実装を書きます (Green)。
  3. 重複・命名・抽象度の観点でリファクタリングします (Refactor)。
  4. `./mvnw -B -Ph2 test` で全テストが緑であることを確認します。
  5. Conventional Commits でコミットします (`feat(<scope>): <要約>`)。

  受入基準:
  一覧画面から投稿の詳細に遷移できるように変更

  制約:
  - AGENTS.md と .codex/instructions.md を参照すること。
  - このタスクのスコープ外のファイルは触らないこと。
  - テストを書かずに本体コードを書き始めないこと。
  - 失敗テストを通すために `@Disabled` を貼ることは絶対に禁止。
  - 完了後にどの 3 ファイル前後を変更したかを要約して報告すること。

  完了条件は ./mvnw -B -Ph2 test
  ./mvnw -B -Ph2 -Pcoverage-day3 -Pstrict verify が緑になることです。

```

**結果**: 効いた

```
• Edited src/main/resources/templates/posts/list.html (+1 -0)
    34                    th:text="${#temporals.format(post.createdAt, 'yyyy-MM-dd HH:mm')}">2026-05-23 09:00</time>
    35 +            <a th:href="@{/posts/{id}(id=${post.id})}">詳細</a>

```

**振り返り**:
  一覧から詳細画面へ遷移することができた。


## プロンプト11

**フェーズ**:C1:タグ機能

**プロンプト本文**:

```
タスク:  C1：タグ機能を TDD で実装してください。

順序:
1. まず受入基準を満たすための失敗テストを 1 本だけ書きます (Red)。
2. その失敗テストを通す最小実装を書きます (Green)。
3. 重複・命名・抽象度の観点でリファクタリングします (Refactor)。
4. `./mvnw -B -Ph2 test` で全テストが緑であることを確認します。
5. Conventional Commits でコミットします (`feat(<scope>): <要約>`)。

受入基準:
新規登録する際に各投稿にタグを設定できるようにします。
タグを管理するテーブルを作成
以下の Entity 定義に基づいて JPA レイヤを生成してください。

テーブル設計：
正規化を意識し、冗長性のない構成としてください。要件を満たす範囲で適切なテーブル分割・リレーションを設計してください。
- ...

生成するもの:
1. `@Entity` クラス (パッケージ: `com.example.tsubuyaki.domain`)
2. `JpaRepository<EntityName, Long>` インタフェース
3. `@DataJpaTest` のテストクラス (正常系 1 本以上 + 異常系 1 本以上)
4. Flyway マイグレーション `V<n>__<entity>.sql` (Oracle / H2(MODE=Oracle) 両方で動く SQL)

制約:
- ID は `@GeneratedValue(strategy = GenerationType.SEQUENCE)` + `@SequenceGenerator(allocationSize = 1)` で統一。
- 日時カラムは `java.time.Instant` を使う。`java.util.Date` は禁止。
- 文字列カラムには `@Column(length = ..., nullable = ...)` を明示。
- DDL は `NUMBER(19)`, `VARCHAR2(... CHAR)`, `TIMESTAMP(6)` を使い、H2(MODE=Oracle) でも動くように書く。
- リポジトリのメソッド名は派生クエリ (`findTop50ByOrderByCreatedAtDesc`) を優先し、`@Query` を使うときは bind 変数を必ず使う (`?1` ではなく `:name` 推奨)。

本文中の #tag をパースして Tag テーブルに保存、GET /tags/{name} で関連投稿一覧。



制約:
- AGENTS.md と .codex/instructions.md を参照すること。
- このタスクのスコープ外のファイルは触らないこと。
- テストを書かずに本体コードを書き始めないこと。
- 失敗テストを通すために `@Disabled` を貼ることは絶対に禁止。
- 完了後にどの 3 ファイル前後を変更したかを要約して報告すること。

```

**結果**: 部分的に聞いた

```
 Edited src/main/java/com/example/tsubuyaki/controller/PostController.java (+9 -0)
    44
    45 +    @GetMapping("/tags/{name}")
    46 +    public String listByTag(@PathVariable String name, Model model) {
    47 +        String tagName = normalizeQuery(name).toLowerCase();
    48 +        model.addAttribute("posts", postService.findByTagName(tagName));
    49 +        model.addAttribute("q", "");
    50 +        model.addAttribute("tagName", tagName);
    54      @GetMapping("/posts/new")

• Edited src/main/java/com/example/tsubuyaki/controller/PostController.java (+2 -1)
    20  import java.security.NoSuchAlgorithmException;
    21 +import java.util.Locale;
    22  import java.util.HexFormat;
       ⋮
    47      public String listByTag(@PathVariable String name, Model model) {
    47 -        String tagName = normalizeQuery(name).toLowerCase();
    48 +        String tagName = normalizeQuery(name).toLowerCase(Locale.ROOT);
    49          model.addAttribute("posts", postService.findByTagName(tagName));

```

**振り返り**:
テーブルレイアウトからすべてお任せでやってみた。中間テーブルを作成し、多対多になる設計がされていたので簡単なものであればお任せでやってほうが早いと感じた。
しかし、一覧画面が表示されなくなった。


## プロンプト 12

**フェーズ**: C1:タグ機能

**プロンプト本文**:

```
一覧画面が表示されませんエラーを解決、または調査を行ってください
```

**結果**: 効いた

```


```

**振り返り**:
設計の続きだったため短い文章で送ってみた。
テスト検証でループになり
■ Quota exceeded. Check your plan and billing details.
と出力され終了した。


## プロンプト13

**フェーズ**: REST API

**プロンプト本文**:

```
# タスク

**REST API（`GET /api/posts`）で投稿一覧をJSON形式で返却し、簡易OpenAPI（Swagger）ドキュメントを利用できるようにしてください。**

## 実装内容

* `GET /api/posts` を追加する。
* 投稿一覧をJSON形式で返却する。
* OpenAPI（Swagger UI）からAPI仕様を確認・実行できるようにする。

---

## 手順

1. **受入基準を満たすための失敗テストを1本だけ作成する（Red）**

   * `GET /api/posts` が期待するJSONを返すことを検証するテストを追加する。
   * 最初は必ず失敗する状態にする。
   * 本体コードはまだ変更しない。

2. **失敗テストを通すための最小限の実装を行う（Green）**

   * API用Controllerを実装する。
   * 必要に応じてService・DTOなどを追加する。
   * テストが通るために必要最小限の実装とする。

3. **リファクタリングを実施する（Refactor）**

   * 重複コードの排除
   * 命名の改善
   * 責務の整理
   * 可読性の向上
   * 振る舞いは変更しないこと。

4. **OpenAPI（Swagger）を利用できるようにする**

   * 必要なライブラリを追加する。
   * `GET /api/posts` がSwagger UIに表示されることを確認する。

5. **テストを実行する**

   ```bash
   ./mvnw -B -Ph2 test
   ```

   全テストが成功することを確認する。

6. **Conventional Commits形式でコミットする**

   ```
   feat(api): add posts REST API
   ```

---

 受入基準

* `GET /api/posts` にアクセスすると投稿一覧がJSON形式で返却される。
* HTTPステータス200を返却する。
* レスポンスが期待するJSON構造になっている。
* OpenAPI（Swagger UI）でAPI仕様を確認できる。
* Swagger UIからAPIを実行できる。

---

 制約

* 作業開始前に **AGENTS.md** と **.codex/instructions.md** を確認すること。
* このタスクに関係のないファイルは変更しないこと。
* **必ずTDD（Red → Green → Refactor）** の順で実装すること。
* テストを書かずに本体コードを書き始めないこと。
* `@Disabled` を使用してテストを無効化しないこと。
* テストを改ざんして成功させないこと。
* 受入基準を満たすための最小限の変更に留めること。
* 既存の画面機能（一覧・詳細・登録等）の挙動を変更しないこと。

---

 完了後に報告する内容

以下を簡潔にまとめて報告してください。

1. Red・Green・Refactorで行った内容
2. 実行したテスト結果
3. Swagger UIの確認結果
4. 変更したファイル（3ファイル前後）
5. コミットメッセージ
6. 受入基準を満たしていることの確認

```

**結果**: 効いた

```
• Added src/main/java/com/example/tsubuyaki/controller/PostApiController.java (+45 -0)
     1 +package com.example.tsubuyaki.controller;
     2 +
     3 +import com.example.tsubuyaki.domain.Post;
     4 +import com.example.tsubuyaki.service.PostService;
     5 +import org.springframework.web.bind.annotation.GetMapping;
     6 +import org.springframework.web.bind.annotation.RequestMapping;
     7 +import org.springframework.web.bind.annotation.RestController;
     8 +
     9 +import java.time.Instant;
    10 +import java.util.List;
    11 +
    12 +@RestController
    13 +@RequestMapping("/api/posts")
    14 +public class PostApiController {
    15 +
    16 +    private final PostService postService;
    17 +
    18 +    public PostApiController(PostService postService) {
    19 +        this.postService = postService;
    20 +    }
    21 +
    22 +    @GetMapping
    23 +    public List<PostResponse> list() {
    24 +        return postService.findLatest50().stream()
    25 +                .map(PostResponse::from)
    26 +                .toList();
    27 +    }
    28 +
    29 +    public record PostResponse(
    30 +            Long id,
    31 +            String author,
    32 +            String body,
    33 +            String avatarColor,
    34 +            Instant createdAt) {
    35 +
    36 +        private static PostResponse from(Post post) {
    37 +            return new PostResponse(
    38 +                    post.getId(),
    39 +                    post.getAuthor(),
    40 +                    post.getBody(),
    41 +                    post.getAvatarColor(),
    42 +                    post.getCreatedAt());
    43 +        }
    44 +    }
    45 +}

```

**振り返り**:
  http://localhost:8080/api/postsにアクセスし、正常に動作していることを確認。
  何かの機能で使用したい。
```



## プロンプト14

**フェーズ**:　コメント機能追加

**プロンプト本文**:

```
# タスク

**投稿詳細画面の下部にコメント入力フォームを追加し、コメントを投稿・表示できる機能をTDD（Red → Green → Refactor）で実装してください。**

## 実装内容

* 投稿詳細画面の下部にコメント一覧を表示する。
* コメント一覧の下にコメント入力フォームを配置する。
* 「コメントする」ボタンを押すとコメントを登録する。
* 登録後は投稿詳細画面へ戻り、追加したコメントが一覧の最後に表示される。

---

## 手順

1. 受入基準を満たすための失敗テストを1本だけ作成する（Red）。
2. 失敗テストを通すための最小限の実装を行う（Green）。
3. 重複・命名・責務の整理などのリファクタリングを行う（Refactor）。
4. `./mvnw -B -Ph2 test` を実行し、全テストが成功することを確認する。
5. Conventional Commits形式でコミットする。

   ```
   feat(comment): add comment form to post detail
   ```

---

## 受入基準

* 投稿詳細画面にコメント一覧が表示される。
* コメントが0件でも画面は正常に表示される。
* コメント一覧の下部にコメント入力欄（textarea）と「コメントする」ボタンが表示される。
* コメントを入力して送信すると、その投稿に紐付けて保存される。
* 登録後は投稿詳細画面へリダイレクトされる。
* 登録したコメントがコメント一覧に表示される。
* 空文字のコメントは登録できない。

---

## 制約

* 作業開始前に **AGENTS.md** と **.codex/instructions.md** を確認すること。
* このタスク以外のファイルは変更しないこと。
* 必ず **TDD（Red → Green → Refactor）** で実装すること。
* テストを書かずに本体コードを書き始めないこと。
* `@Disabled` を使用しないこと。
* テストを改ざんして成功させないこと。
* 既存の投稿一覧・投稿詳細・投稿登録機能の挙動を変更しないこと。
* コメントは投稿（Post）と紐付けて管理すること。

---

## 完了後に報告する内容

1. Red・Green・Refactorで行った内容
2. 実行したテスト結果
3. 変更したファイル（3～6ファイル程度）
4. コミットメッセージ
5. 受入基準を満たしていることの確認

```

**結果**: 効いた

```
• Edited 6 files (+126 -1)
  └ src/main/java/com/example/tsubuyaki/controller/PostController.java (+7 -0)
    67                      model.addAttribute("likeCount", postService.countLikes(id));
    68 +                    model.addAttribute("comments", postService.findCommentsByPostId(id));
    69                      return "posts/detail";
       ⋮
    73
    74 +    @PostMapping("/posts/{id}/comments")
    75 +    public String createComment(@PathVariable Long id, @RequestParam String body) {
    76 +        postService.createComment(id, body);
    77 +        return "redirect:/posts/" + id;
    78 +    }
    79 +
    80      @PostMapping("/posts/{id}/likes")

  └ src/main/java/com/example/tsubuyaki/domain/Comment.java (+61 -0)
     1 +package com.example.tsubuyaki.domain;
     2 +
     3 +import jakarta.persistence.Column;
     4 +import jakarta.persistence.Entity;
     5 +import jakarta.persistence.FetchType;
     6 +import jakarta.persistence.GeneratedValue;
     7 +import jakarta.persistence.GenerationType;
     8 +import jakarta.persistence.Id;
     9 +import jakarta.persistence.JoinColumn;
    10 +import jakarta.persistence.ManyToOne;
    11 +import jakarta.persistence.SequenceGenerator;
    12 +import jakarta.persistence.Table;
    13 +
    14 +import java.time.Instant;
    15 +import java.util.Objects;
    16 +
    17 +@Entity
    18 +@Table(name = "comments")
    19 +public class Comment {
    20 +
    21 +    @Id
    22 +    @SequenceGenerator(name = "comments_seq_gen", sequenceName = "comments_seq", allocationSize = 1)
    23 +    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "comments_seq_gen")
    24 +    private Long id;
    25 +
    26 +    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    27 +    @JoinColumn(name = "post_id", nullable = false)
    28 +    private Post post;
    29 +
    30 +    @Column(name = "body", length = 280, nullable = false)
    31 +    private String body;
    32 +
    33 +    @Column(name = "created_at", nullable = false)
    34 +    private Instant createdAt;
    35 +
    36 +    protected Comment() {
    37 +        // JPA
    38 +    }
    39 +
    40 +    public Comment(Post post, String body, Instant createdAt) {
    41 +        this.post = Objects.requireNonNull(post);
    42 +        this.body = body;
    43 +        this.createdAt = createdAt;
    44 +    }
    45 +
    46 +    public Long getId() {
    47 +        return id;
    48 +    }
    49 +
    50 +    public Post getPost() {
    51 +        return post;
    52 +    }
    53 +
    54 +    public String getBody() {
    55 +        return body;
    56 +    }
    57 +
    58 +    public Instant getCreatedAt() {
    59 +        return createdAt;
    60 +    }
    61 +}

  └ src/main/java/com/example/tsubuyaki/repository/CommentRepository.java (+11 -0)
     1 +package com.example.tsubuyaki.repository;
     2 +
     3 +import com.example.tsubuyaki.domain.Comment;
     4 +import org.springframework.data.jpa.repository.JpaRepository;
     5 +
     6 +import java.util.List;
     7 +
     8 +public interface CommentRepository extends JpaRepository<Comment, Long> {
     9 +
    10 +    List<Comment> findByPostIdOrderByCreatedAtAsc(Long postId);
    11 +}

  └ src/main/java/com/example/tsubuyaki/service/PostService.java (+16 -1)
      2
      3 +import com.example.tsubuyaki.domain.Comment;
      4  import com.example.tsubuyaki.domain.Post;
      5  import com.example.tsubuyaki.domain.Tag;
      6 +import com.example.tsubuyaki.repository.CommentRepository;
      7  import com.example.tsubuyaki.repository.PostRepository;
        ⋮
     33
     34 +    private final CommentRepository commentRepository;
     35 +
     36      private final Map<Long, Set<String>> likedClientHashesByPostId = new ConcurrentHashMap<>();
     37
     34 -    public PostService(PostRepository repository, TagRepository tagRepository) {
     38 +    public PostService(PostRepository repository, TagRepository tagRepository, CommentRepository co
         mmentRepository) {
     39          this.repository = repository;
     40          this.tagRepository = tagRepository;
     41 +        this.commentRepository = commentRepository;
     42      }
        ⋮
     59
     60 +    public List<Comment> findCommentsByPostId(Long postId) {
     61 +        return commentRepository.findByPostIdOrderByCreatedAtAsc(postId);
     62 +    }
     63 +
     64      public long countLikes(Long postId) {
        ⋮
    105
    106 +    @Transactional
    107 +    public void createComment(Long postId, String body) {
    108 +        repository.findByIdAndDeletedAtIsNull(postId)
    109 +                .ifPresent(post -> commentRepository.save(new Comment(post, body, Instant.now())));
    110 +    }
    111 +
    112      private Tag findOrCreateTag(String name) {

  └ src/main/resources/db/migration/V5__comments.sql (+17 -0)
     1 +-- =========================================================================
     2 +-- 社内つぶやきボード V5: コメントテーブルを追加
     3 +-- Oracle XE 21c および H2(MODE=Oracle) の双方で動く DDL
     4 +-- =========================================================================
     5 +
     6 +CREATE SEQUENCE comments_seq START WITH 1 INCREMENT BY 1 NOCACHE;
     7 +
     8 +CREATE TABLE comments (
     9 +    id          NUMBER(19)          NOT NULL,
    10 +    post_id     NUMBER(19)          NOT NULL,
    11 +    body        VARCHAR2(280 CHAR)  NOT NULL,
    12 +    created_at  TIMESTAMP(6)        NOT NULL,
    13 +    CONSTRAINT comments_pk PRIMARY KEY (id),
    14 +    CONSTRAINT comments_post_fk FOREIGN KEY (post_id) REFERENCES posts (id)
    15 +);
    16 +
    17 +CREATE INDEX comments_post_created_at_idx ON comments (post_id, created_at);

  └ src/main/resources/templates/posts/detail.html (+14 -0)
    36      </form>
    37 +    <section>
    38 +        <h2>コメント</h2>
    39 +        <form method="post" th:action="@{/posts/{id}/comments(id=${postId})}">
    40 +            <textarea name="body"></textarea>
    41 +            <button type="submit">コメントする</button>
    43 +        <p th:if="${#lists.isEmpty(comments)}">まだコメントはありません</p>
    44 +        <article class="comment" th:each="comment : ${comments}">
    45 +            <p class="comment__body" th:text="${comment.body}">コメント本文</p>
    46 +            <time class="comment__created-at"
    47 +                  th:datetime="${comment.createdAt}"
    48 +                  th:text="${#temporals.format(comment.createdAt, 'yyyy-MM-dd HH:mm')}">2026-05-23 0
    49 +        </article>
    50 +    </section>
    51
```

**振り返り**:
  詳細画面でコメント機能を追加することができた。しかし、空白のままコメントを送信するとエラーになった。
  また、現在はフォーム送信後にリダイレクトで画面を再描画する方式だが、
  REST APIやJavaScriptを利用し、リロードなしでコメントが動的に追加されるような実装に改善していきたい。



## プロンプト 15

**フェーズ**:複数タグ

**プロンプト本文**:

```
タスク:  タグ機能を「複数タグ対応」に拡張し、新規投稿・表示・検索がすべて正しく動作するようにTDDで実装してください。

順序:
1. まず受入基準を満たすための失敗テストを 1 本だけ書きます (Red)。
2. その失敗テストを通す最小実装を書きます (Green)。
3. 重複・命名・抽象度の観点でリファクタリングします (Refactor)。
4. `./mvnw -B -Ph2 test` で全テストが緑であることを確認します。
5. Conventional Commits でコミットします (`feat(<scope>): <要約>`)。

	受入基準:
		投稿時に複数タグを登録できる
		タグは投稿と正しく紐づく（多対多）
		投稿詳細画面で複数タグが表示される
		タグ一覧検索（/tags/{name}）が正しく動作する
		タグが0件の場合でもエラーにならない

	変更内容：
		既存のタグ機能を以下のように拡張する：
		1投稿に対して複数タグを登録できるようにする
		新規投稿画面で複数タグを入力できるUIに変更する
		タグは複数個表示できるようにする
		タグ検索（/tags/{name}）が複数タグに対して正しく動作すること

	UI要件（新規投稿画面）：
		「タグ追加」ボタンを設置する
		ボタン押下でタグ入力欄を追加できるようにする
		複数のタグ入力欄を持てるようにする
		それぞれのタグが個別に送信される
	例：
		タグ1 [ java ]
		タグ2 [ spring ]
		タグ3 [ backend ]
		[ + タグ追加 ]


制約:
- AGENTS.md と .codex/instructions.md を参照すること。
- このタスクのスコープ外のファイルは触らないこと。
- テストを書かずに本体コードを書き始めないこと。
- 失敗テストを通すために `@Disabled` を貼ることは絶対に禁止。
- 完了後にどの 3 ファイル前後を変更したかを要約して報告すること。

```

**結果**: 効いた

```
• まず Green に必要な最小差分として、フォーム HTML にタグ入力リスト・追加ボタン・追加用スクリプトを入れます。Controller は複数 tagNames パラメータを空白結合して、既存のタグ正規化ロジックに
  渡します。

• Edited 3 files (+65 -4)
  └ src/main/java/com/example/tsubuyaki/controller/PostController.java (+13 -2)
     21  import java.util.HexFormat;
     22 +import java.util.List;
     23  import java.util.Locale;
        ⋮
     93      @PostMapping("/posts")
     93 -    public String create(@Valid @ModelAttribute PostForm form, BindingResult result) {
     94 +    public String create(
     95 +            @Valid @ModelAttribute PostForm form,
     96 +            BindingResult result,
     97 +            @RequestParam(name = "tagNames", required = false) List<String> tagNames) {
     98          if (result.hasErrors()) {
        ⋮
    100          }
     97 -        postService.create(form.getAuthor(), form.getBody(), form.getAvatarColor(), form.getTagNames());
    101 +        postService.create(form.getAuthor(), form.getBody(), form.getAvatarColor(), toTagNamesText(tagNames));
    102          return "redirect:/posts";
        ⋮
    131      }
    132 +
    133 +    private static String toTagNamesText(List<String> tagNames) {
    134 +        if (tagNames == null) {
    135 +            return "";
    136 +        }
    137 +        return String.join(" ", tagNames);
    138 +    }
    139  }

  └ src/main/resources/static/css/app.css (+21 -0)
    102
    103 +.post-form__tag-list,
    104 +.post-form__tag-field {
    105 +    display: grid;
    106 +    gap: 0.4rem;
    107 +}
    108 +
    109 +.post-form__tag-list {
    110 +    gap: 0.75rem;
    111 +}
    112 +
    113  .post-form__choices {
        ⋮
    128
    129 +.post-form__add-tag {
    130 +    justify-self: start;
    131 +    border: 1px solid var(--color-border);
    132 +    border-radius: 6px;
    133 +    padding: 0.5rem 0.8rem;
    134 +    font: inherit;
    135 +    color: var(--color-accent);
    136 +    background: white;
    137 +    cursor: pointer;
    138 +}
    139 +
    140  .post-form__actions {

  └ src/main/resources/templates/posts/form.html (+31 -2)
    48          <div class="post-form__field">
    49 -            <label class="post-form__label" for="tagNames">タグ</label>
    50 -            <input class="post-form__input" type="text" id="tagNames" th:field="*{tagNames}">
    49 +            <label class="post-form__label" for="tagNames">タグ1</label>
    50 +            <div class="post-form__tag-list" data-tag-list>
    51 +                <input class="post-form__input" type="text" id="tagNames" th:field="*{tagNames}">
    52 +            </div>
    53 +            <button class="post-form__add-tag" type="button" data-tag-add>+ タグ追加</button>
    54          </div>
       ⋮
    58      </form>
    59 +    <script>
    60 +        (() => {
    61 +            const tagList = document.querySelector("[data-tag-list]");
    62 +            const addButton = document.querySelector("[data-tag-add]");
    63 +            if (!tagList || !addButton) {
    64 +                return;
    65 +            }
    66 +            addButton.addEventListener("click", () => {
    67 +                const nextIndex = tagList.querySelectorAll("input[name='tagNames']").length + 1;
    68 +                const fieldId = `tagNames${nextIndex}`;
    69 +                const wrapper = document.createElement("div");
    70 +                wrapper.className = "post-form__tag-field";
    71 +                const label = document.createElement("label");
    72 +                label.className = "post-form__label";
    73 +                label.htmlFor = fieldId;
    74 +                label.textContent = `タグ${nextIndex}`;
    75 +                const input = document.createElement("input");
    76 +                input.className = "post-form__input";
    77 +                input.type = "text";
    78 +                input.id = fieldId;
    79 +                input.name = "tagNames";
    80 +                wrapper.append(label, input);
    81 +                tagList.append(wrapper);
    82 +            });
    83 +        })();
    84 +    </script>
    85  </main>

```

**振り返り**:
  投稿フォームのタグ機能を拡張し、複数タグを登録できるように改善した。
  従来はタグ入力が1件のみで拡張性が低かったため、SNSのように柔軟なタグ付けができるよう改善を行った。
  フロントエンドでは「+タグ追加」ボタンを設置し、JavaScriptにより入力欄を動的に生成することで、ユーザーが必要な数だけタグを追加できるUIに変更した。これにより、固定フォー ムでは実現できなかった入力ができるようになった。
  バックエンドでは @RequestParam List<String> tagNames を用いて複数タグを受け取れるように変更し、フロントで増やされた入力値をそのままリストとして処理できる構造に改善 していた。


## プロンプト 16

**フェーズ**:レイアウト　SNS風

**プロンプト本文**:

```
タスク:
既存の投稿フォームをCSSでSNS（X / Instagram風）デザインに変更してください。


順序:
1. まず受入基準を満たすための失敗テストを 1 本だけ書きます (Red)。
2. その失敗テストを通す最小実装を書きます (Green)。
3. 重複・命名・抽象度の観点でリファクタリングします (Refactor)。
4. `./mvnw -B -Ph2 test` で全テストが緑であることを確認します。
5. Conventional Commits でコミットします (`feat(<scope>): <要約>`)。
Conventional Commitsでコミットする（feat(style): <要約>）

受入基準:

フォーム全体が中央配置されカード風デザインになっている
背景は薄いグレーで統一されている
input / textarea は横幅100%かつ角丸になっている
textareaはSNS投稿のように十分な高さがある
ラジオボタンは横並びまたは折り返し表示になっている
投稿ボタンは青系で強調され右寄せになっている
SNS（X / Instagram）の投稿画面のようなUIになっている

変更内容：
既存の投稿フォームをSNS風UIに改善するため、CSSのみを修正する。

フォーム全体をカードデザイン化する
背景色を薄いグレーに変更する
入力欄のスタイルを統一する（横幅100%、角丸）
textareaを大きめに調整する
ラジオボタンを横並び or 折り返しにする
投稿ボタンを強調色（青系）で右寄せにする

UI要件（新規投稿画面）：

フォーム全体を中央配置しカード風にする
「SNS投稿画面」のようなシンプルな構造にする
視認性と入力しやすさを重視する

制約:
- AGENTS.md と .codex/instructions.md を参照すること。
- このタスクのスコープ外のファイルは触らないこと。
- テストを書かずに本体コードを書き始めないこと。
- 失敗テストを通すために `@Disabled` を貼ることは絶対に禁止。
- 完了後にどの 3 ファイル前後を変更したかを要約して報告すること。

出力:
変更・追加するCSSのみを提示すること
適用ファイル（例：/css/app.css）を明記すること
```

**結果**: 効いた

```
  app.css

  Before:

  :root {
      --color-bg: #fafafa;
      --color-text: #222;
      --color-accent: #1d4ed8;
      --color-border: #e5e7eb;
      --color-muted: #6b7280;
  }

  main {
      max-width: 720px;
      margin: 2rem auto;
      padding: 0 1rem;
  }

  .post {
      padding: 1rem;
      border: 1px solid var(--color-border);
      border-radius: 8px;
      margin-bottom: 0.75rem;
      background: white;
  }

  After:

  :root {
      --color-bg: #f5f7f9;
      --color-surface: #ffffff;
      --color-text: #0f172a;
      --color-accent: #1da1f2;
      --color-accent-hover: #0c8bdc;
      --color-accent-soft: #e8f5fe;
      --color-border: #dbe3ea;
      --color-muted: #64748b;
      --shadow-card: 0 10px 28px rgb(15 23 42 / 0.08);
  }

  main {
      max-width: 640px;
      margin: 0 auto;
      padding: 2rem 1rem 3rem;
  }

  .post {
      margin-bottom: 1rem;
      padding: 1.2rem;
      border: 1px solid var(--color-border);
      border-radius: 14px;
      background: var(--color-surface);
      box-shadow: var(--shadow-card);
  }

  追加された共通UI:

  button {
      border: 0;
      border-radius: 999px;
      padding: 0.65rem 1.1rem;
      color: var(--color-surface);
      background: var(--color-accent);
      font-weight: 700;
      cursor: pointer;
  }

  input,
  textarea {
      width: 100%;
      border: 1px solid var(--color-border);
      border-radius: 12px;
      padding: 0.8rem 0.9rem;
      color: var(--color-text);
      background: #fbfdff;
  }

  .post__tag-link {
      display: inline-flex;
      align-items: center;
      border-radius: 999px;
      color: var(--color-accent);
      background: var(--color-accent-soft);
      font-size: 0.85rem;
      font-weight: 700;
      text-decoration: none;
  }

  .comment {
      margin-bottom: 0.8rem;
      padding: 1rem;
      border: 1px solid var(--color-border);
      border-radius: 14px;
      background: var(--color-surface);
      box-shadow: var(--shadow-card);
  }

  detail.html

  Before:

  <a th:each="tag : ${post.tags}"
     th:href="@{/tags/{name}(name=${tag.name})}"
     th:text="|#${tag.name}|">#tag</a>

  After:

  <a th:each="tag : ${post.tags}"
     class="post__tag-link"
     th:href="@{/tags/{name}(name=${tag.name})}"
     th:text="|#${tag.name}|">#tag</a>

  list.html

  Before:

  <a th:each="tag : ${post.tags}"
     th:href="@{/tags/{name}(name=${tag.name})}"
     th:text="|#${tag.name}|">#tag</a>

  After:

  <a th:each="tag : ${post.tags}"
     class="post__tag-link"
     th:href="@{/tags/{name}(name=${tag.name})}"
     th:text="|#${tag.name}|">#tag</a>

  PostControllerTest.java

  追加されたテスト:

  @Test
  @DisplayName("タグUI_一覧と詳細でタグリンクをバッジ表示できる")
  void tagUi_一覧と詳細でタグリンクをバッジ表示できる() throws Exception {
      Post post = postWithId(10L, "alice", "タグ付き投稿です", BASE_TIME.plusSeconds(1));
      post.addTag(new Tag("java", BASE_TIME));
      given(postService.findLatest50()).willReturn(List.of(post));
      given(postService.findById(10L)).willReturn(Optional.of(post));

      String listHtml = mockMvc.perform(get("/posts"))
              .andExpect(status().isOk())
              .andReturn()
              .getResponse()
              .getContentAsString();
      String detailHtml = mockMvc.perform(get("/posts/10"))
              .andExpect(status().isOk())
              .andReturn()
              .getResponse()
              .getContentAsString();

      assertThat(listHtml).contains("class=\"post__tag-link\"");
      assertThat(detailHtml).contains("class=\"post__tag-link\"");
  }
  src/main/resources/static/css/app.css
  SNS風UI統一のためのCSS変更です。

  - デザイン変数を更新
      - 背景: #f5f7f9
      - アクセント: #1da1f2
      - 白カード用 --color-surface
      - 淡い青 --color-accent-soft
      - 共通シャドウ --shadow-card

  - 全体レイアウトを中央寄せ
      - main を max-width: 640px
      - 余白を広めに調整

  - ボタン共通化
      - 丸型ボタン
      - 青背景
      - hover/focus時の色変更

  - input / textarea 共通化
      - 横幅100%
      - 角丸12px
      - focus時に青枠＋薄いシャドウ

  - 投稿カード .post をカードUI化
      - 白背景
      - 枠線
      - 角丸14px
      - 軽い影
      - .post__tags
      - .post__tag-link
      - #tag を青系バッジ表示
      - hover/focus対応

  - 投稿フォーム .post-form をカード化
  - コメント欄 .comment / .comment__body / .comment__created-at をカードUI化
  - 詳細画面のLike欄・削除ボタン・コメントフォームをCSSだけで整形
  - モバイル向けに検索フォームを縦並び化

  src/main/resources/templates/posts/detail.html
  投稿詳細画面のタグリンクにCSSクラスを追加しています。

  class="post__tag-link"

  これにより、詳細画面のタグがバッジ表示になります。

  src/main/resources/templates/posts/list.html
  投稿一覧画面のタグリンクにも同じCSSクラスを追加しています。

  class="post__tag-link"

  これにより、一覧画面のタグもバッジ表示になります。

  src/test/java/com/example/tsubuyaki/controller/PostControllerTest.java
  タグUI用のテストが1本追加されています。

  - テスト名:
      - タグUI_一覧と詳細でタグリンクをバッジ表示できる

  - 内容:
      - タグ付き投稿を用意
      - /posts と /posts/10 を表示
      - HTML内に class="post__tag-link" が含まれることを検証

```

**振り返り**:
・AIエージェントを利用してSNS風のレイアウトへ変更できた。
・全体的なデザイン変更は効率的に行えたが、細かなレイアウト調整は期待どおりにならない部分もあった。今後は変更箇所やデザインイメージをより具体的に指示し、修正回数を減らせるよう意識したい。

---


---

## プロンプト 17

**フェーズ**:  詳細画面のレイアウト調整

**プロンプト本文**:

```
› タスク: コメント入力フォームのレイアウトを改善してください。

  目的:
  コメントが0件の場合でも、コメント入力欄がすぐ見えるようにレイアウトを調整する。

  受入基準:
  - コメント入力フォームは「コメント」見出しの直下に表示すること
  - コメントが0件でも入力フォームが画面下に離れないこと
  - 「まだコメントはありません」のメッセージは入力フォームの下に表示すること
  - コメントが存在する場合は、「入力フォーム → コメント一覧」の順で表示すること
  - デザインは既存のSNS風UIに合わせること

  変更内容:
  - detail.html のレイアウトを調整する
  - 必要に応じて app.css を修正する
  - コメント入力フォームの余白（margin・padding）を見直す
  - コメント0件時でもフォームが自然な位置に表示されるようにする

  制約:
  - コメント投稿機能の動作は変更しないこと
  - HTML構造は必要最小限の変更に留めること
  - 既存クラスを優先して使用すること
  - デザインの統一感を維持すること

  完了後:
  - 変更したファイルを要約すること
```

**結果**: 効いた

```
• 想定通り、コメント入力の下に記載したコメントが追加されていくようになりました。

  • Edited 2 files (+7 -7)
  └ src/main/resources/static/css/app.css (+3 -3)
    336      display: grid;
    337 -    grid-template-rows: auto minmax(16rem, 1fr) auto;
    337 +    grid-template-rows: auto auto minmax(12rem, 1fr);
    338      gap: 0.8rem;
    339 -    height: min(78vh, 46rem);
    340 -    min-height: 34rem;
    339 +    height: min(72vh, 42rem);
    340 +    min-height: 28rem;
    341      overflow: hidden;

  └ src/main/resources/templates/posts/detail.html (+4 -4)
    39          <h2>コメント</h2>
    40 +        <form class="comment-form" method="post" th:action="@{/posts/{id}/comments(id=${postId})}">
    41 +            <textarea name="body"></textarea>
    42 +            <button type="submit">コメントする</button>
    43 +        </form>
    44          <div class="comment-list">
       ⋮
    52          </div>
    49 -        <form class="comment-form" method="post" th:action="@{/posts/{id}/comments(id=${postId})}">
    50 -            <textarea name="body"></textarea>
    51 -            <button type="submit">コメントする</button>
    52 -        </form>
    53      </section>
```

**振り返り**:

横幅のサイズを変えた際にレイアウトが崩れる。

---


---

## プロンプト 18

**フェーズ**: 削除機能

**プロンプト本文**:

```
 タスク: 投稿詳細画面の削除ボタンをSNS風の三点メニュー（…）に変更してください。

  順序:
  1. まず受入基準を満たすための失敗テストを1本だけ書きます（Red）。
  2. その失敗テストを通す最小実装を書きます（Green）。
  3. 重複・命名・抽象度の観点でリファクタリングします（Refactor）。
  4. `./mvnw -B -Ph2 test` で全テストが緑であることを確認します。
  5. Conventional Commitsでコミットします（`feat(<scope>): <要約>`）。

  受入基準:
  - 投稿カード右上に「…」ボタンを配置する
  - 「…」ボタンをクリックするとドロップダウンメニューが表示される
  - メニュー内に「削除」を表示する
  - 「削除」をクリックすると既存の削除処理（POST）が実行される
  - メニュー外をクリックするとドロップダウンを閉じる
  - デザインはXやThreadsのようなSNS風のシンプルなUIとする
  - 既存の削除機能・URL・動作は変更しない

  変更内容:
  - detail.html に三点メニューを追加する
  - app.css にドロップダウンメニューのスタイルを追加する
  - 必要最小限のJavaScriptでメニューの開閉を実装する
  - Controller・Serviceなどのバックエンド処理は変更しない

  制約:
  - AGENTS.md と .codex/instructions.md を参照すること。
  - このタスクのスコープ外のファイルは触らないこと。
  - テストを書かずに本体コードを書き始めないこと。
  - 失敗テストを通すために `@Disabled` を貼ることは絶対に禁止。
  - 完了後にどの3ファイル前後を変更したかを要約して報告すること。
```

**結果**: 効いた

```
  ・・・メニューから削除できることを確認
• Edited src/main/resources/static/css/app.css (+66 -11)
    142  .post {
    143 +    position: relative;
    144      margin-bottom: 1rem;
        ⋮
    151
    152 +.post-menu {
    153 +    position: absolute;
    154 +    top: 0.7rem;
    155 +    right: 0.7rem;
    156 +    z-index: 2;
    157 +}
    158 +
    159 +.post-menu__toggle {
    160 +    display: inline-flex;
    161 +    align-items: center;
    162 +    justify-content: center;
    163 +    width: 2rem;
    164 +    height: 2rem;
    165 +    padding: 0;
    166 +    color: var(--color-muted);
    167 +    background: transparent;
    168 +    font-size: 1.25rem;
    169 +    line-height: 1;
    170 +}
    171 +
    172 +.post-menu__toggle:hover,
    173 +.post-menu__toggle:focus {
    174 +    color: var(--color-accent);
    175 +    background: var(--color-accent-soft);
    176 +}
    177 +
    178 +.post-menu__dropdown {
    179 +    position: absolute;
    180 +    top: calc(100% + 0.35rem);
    181 +    right: 0;
    182 +    min-width: 8rem;
    183 +    padding: 0.35rem;
    184 +    border: 1px solid var(--color-border);
    185 +    border-radius: 12px;
    186 +    background: var(--color-surface);
    187 +    box-shadow: var(--shadow-card);
    188 +}
    189 +
    190 +.post-menu__dropdown[hidden] {
    191 +    display: none;
    192 +}
    193 +
    194 +.post-menu__dropdown form {
    195 +    margin: 0;
    196 +}
    197 +
    198 +.post-menu__delete {
    199 +    display: flex;
    200 +    justify-content: flex-start;
    201 +    width: 100%;
    202 +    padding: 0.6rem 0.75rem;
    203 +    border-radius: 10px;
    205 +    background: transparent;
    206 +}
    207 +
    208 +.post-menu__delete:hover,
    209 +.post-menu__delete:focus {
    210 +    color: #991b1b;
    211 +    background: #fee2e2;
    212 +}
    213 +
    214  .post__author {
        ⋮
    371
    309 -main > article.post + section form,
    310 -main > article + section + form {
    372 +main > article.post + section form {
    373      display: flex;
        ⋮
    377
    316 -main > article + section + form {
    317 -    margin-bottom: 0.35rem;
    318 -}
    319 -
    320 -main > article.post + section button,
    321 -main > article + section + form button {
    378 +main > article.post + section button {
    379      padding: 0.5rem 0.9rem;
        ⋮
    384  main > article.post + section button:hover,
    328 -main > article.post + section button:focus,
    329 -main > article + section + form button:hover,
    330 -main > article + section + form button:focus {
    385 +main > article.post + section button:focus {
    386      color: var(--color-surface);


• Edited src/main/resources/templates/posts/detail.html (+50 -3)
     16      <article class="post">
     17 +        <div class="post-menu" data-post-menu>
     18 +            <button class="post-menu__toggle"
     19 +                    type="button"
     20 +                    aria-label="投稿メニュー"
     21 +                    aria-expanded="false"
     22 +                    data-post-menu-toggle>…</button>
     23 +            <div class="post-menu__dropdown" hidden data-post-menu-dropdown>
     24 +                <form method="post" th:action="@{/posts/{id}/delete(id=${postId})}">
     25 +                    <button class="post-menu__delete" type="submit">削除</button>
     26 +                </form>
     27 +            </div>
     28 +        </div>
     29          <p class="post__author" th:text="${post.author}">投稿者</p>
        ⋮
     46      </section>
     35 -    <form method="post" th:action="@{/posts/{id}/delete(id=${postId})}">
     36 -        <button type="submit">削除</button>
     37 -    </form>
     47      <section class="comment-panel">
        ⋮
     64  </main>
     65 +<script>
     66 +    (() => {
     67 +        const menu = document.querySelector("[data-post-menu]");
     68 +        if (!menu) {
     69 +            return;
     70 +        }
     71 +
     72 +        const toggle = menu.querySelector("[data-post-menu-toggle]");
     73 +        const dropdown = menu.querySelector("[data-post-menu-dropdown]");
     74 +        if (!toggle || !dropdown) {
     75 +            return;
     76 +        }
     77 +
     78 +        const closeMenu = () => {
     79 +            dropdown.hidden = true;
     80 +            toggle.setAttribute("aria-expanded", "false");
     81 +        };
     82 +
     83 +        toggle.addEventListener("click", (event) => {
     84 +            event.stopPropagation();
     85 +            const willOpen = dropdown.hidden;
     86 +            dropdown.hidden = !willOpen;
     87 +            toggle.setAttribute("aria-expanded", String(willOpen));
     88 +        });
     89 +
     90 +        document.addEventListener("click", (event) => {
     91 +            if (!menu.contains(event.target)) {
     92 +                closeMenu();
     93 +            }
     94 +        });
     95 +
     96 +        document.addEventListener("keydown", (event) => {
     97 +            if (event.key === "Escape") {
     98 +                closeMenu();
     99 +            }
    100 +        });
    101 +    })();
    102 +</script>
    103  </body>
```

**振り返り**:
・投稿詳細画面の削除機能をSNS風の三点メニュー（…）から操作できるように改善した。
・削除完了後は一覧画面へ遷移する一連の操作フローを実装できた。
・今後はメニュー内に「編集」機能も追加し、投稿操作を一箇所にまとめることで、より使いやすいUIにしたい。

---



---

## プロンプト 19

**フェーズ**: 発表手順書

**プロンプト本文**:

```
目的:
研修で3～5分程度の機能紹介を行います。

出力内容:
1. 実装した機能の概要
2. なぜこの機能を作成したか
3. 実装方法（Controller・Service・Repository・HTML・CSSなど）
4. デモで見せる操作手順
5. 工夫した点
6. 苦労した点
7. 今後改善したい点
8. 想定質問と回答（5問程度）

制約:
- 実際のソースコードから読み取れる内容のみ説明すること
- 推測で機能を補完しないこと
- 初学者向けに分かりやすく説明すること




デモ操作の手順も作成してください。

例:
① 一覧画面を表示
② 新規投稿を作成
③ タグを複数追加
④ 詳細画面へ遷移
⑤ コメント追加
⑥ いいね
⑦ 三点メニューから削除
⑧ 一覧へ戻る
```

**結果**: 効いた

```
• 1. 実装した機能の概要

  社内つぶやきボードとして、投稿の作成・一覧表示・詳細表示・検索・並び替え・タグ検索・コメント投稿・いいね・削除ができる画面を実装しています。

  主な機能は以下です。

  - 投稿一覧: /posts
  - 新規投稿: /posts/new
  - 投稿詳細: /posts/{id}
  - 本文検索: /posts?q=...
  - 並び替え: 新しい順 / 古い順
  - タグ検索: /tags/{name}
  - コメント投稿: /posts/{id}/comments
  - いいね: /posts/{id}/likes
  - 削除: /posts/{id}/delete
  - 投稿一覧API: GET /api/posts
  - Swagger UI: springdoc-openapi-starter-webmvc-ui を追加済み

  2. なぜこの機能を作成したか

  投稿をただ一覧表示するだけでなく、SNSのように「投稿する」「タグで探す」「コメントする」「いいねする」「削除する」という一連の操作ができるようにするためです。

  特にタグ、コメント、いいね、三点メニューを追加することで、XやThreadsに近い操作感を持つ投稿アプリとして使えるようにしています。

  3. 実装方法

  Controller は src/main/java/com/example/tsubuyaki/controller/PostController.java で画面遷移を担当しています。

  - /posts で投稿一覧を表示
  - /posts/new で投稿フォームを表示
  - /posts/{id} で詳細画面を表示
  - /tags/{name} でタグ検索
  - POSTで投稿作成、コメント作成、いいね、削除を処理

  Service は src/main/java/com/example/tsubuyaki/service/PostService.java で業務処理を担当しています。

  - 投稿作成時に本文とタグ入力欄からタグを抽出
  - タグ名を小文字化して保存
  - コメントが空白の場合は保存しない
  - 削除は物理削除ではなく deletedAt を入れる論理削除
  - いいねはDBではなくメモリ上の Map<Long, Set<String>> で管理

  Repository は Spring Data JPA で実装しています。

  - src/main/java/com/example/tsubuyaki/repository/PostRepository.java: 投稿一覧、検索、並び替え
  - src/main/java/com/example/tsubuyaki/repository/TagRepository.java: タグ名から投稿検索
  - src/main/java/com/example/tsubuyaki/repository/CommentRepository.java: 投稿IDごとのコメント取得

  Entity は以下です。

  - src/main/java/com/example/tsubuyaki/domain/Post.java: 投稿
  - src/main/java/com/example/tsubuyaki/domain/Tag.java: タグ
  - src/main/java/com/example/tsubuyaki/domain/Comment.java: コメント

  DBは Flyway で管理しています。

  - posts
  - tags
  - post_tags
  - comments

  タグは posts と tags を post_tags でつなぐ多対多構成です。

  HTML は Thymeleaf です。

  - src/main/resources/templates/posts/list.html: 投稿一覧、検索、並び替え、タグ表示
  - src/main/resources/templates/posts/form.html: 新規投稿、複数タグ入力
  - src/main/resources/templates/posts/detail.html: 詳細、コメント、いいね、三点メニュー削除

  CSS は src/main/resources/static/css/app.css でSNS風に統一しています。

  - 薄いグレー背景
  - 白いカードUI
  - 青系アクセント
  - タグバッジ
  - コメント欄
  - 三点メニュー

  4. デモで見せる操作手順

  1. /posts を開く
     投稿一覧画面を表示します。

  2. 「新規投稿」をクリック
     /posts/new に移動します。

  3. 投稿を作成
     投稿者名、アバター色、本文を入力します。
     例: 本文に Spring Bootを勉強しました #java と入力します。

  4. 「+ タグ追加」をクリック
     タグ入力欄を追加します。
     例: spring、backend を追加します。

  5. 「投稿」をクリック
     一覧画面に戻り、作成した投稿が表示されます。

  6. タグバッジを確認
     #java、#spring、#backend がバッジ表示されていることを見せます。

  7. タグをクリック
     /tags/{name} に遷移し、同じタグが付いた投稿だけが表示されることを見せます。

  8. 検索と並び替えを確認
     検索欄にキーワードを入れます。
     並び順を「新しい順」「古い順」で切り替えます。

  9. 「詳細」をクリック
     投稿詳細画面を表示します。

  10. コメントを投稿
     コメント欄に入力して「コメントする」をクリックします。
     投稿後、同じ詳細画面に戻り、コメントが表示されることを見せます。

  11. いいねをクリック
     いいね 0 が いいね 1 に変わることを見せます。
     もう一度押すと戻ることも確認できます。

  12. 三点メニューを開く
     投稿カード右上の「…」をクリックします。

  13. 「削除」をクリック
     既存の削除処理が実行され、一覧画面に戻ります。
     削除した投稿が一覧に表示されないことを確認します。

  14. Swagger UIを確認
     /swagger-ui/index.html を開き、GET /api/posts を実行してJSONが返ることを見せます。

  5. 工夫した点

  - タグは本文中の #tag とタグ入力欄の両方から登録できます。
  - タグは Set で扱い、同じタグが重複しにくい構造にしています。
  - 投稿削除は deleted_at を使った論理削除にしています。
  - コメントが空白の場合は保存せず、そのまま詳細画面へ戻るようにしています。
  - UIはカード、タグバッジ、三点メニューでSNS風に統一しています。
  - 削除ボタンは直接表示せず、三点メニュー内に入れて画面をすっきりさせています。

  6. 苦労した点

  - 複数タグ対応で、投稿とタグを多対多で紐付ける必要がありました。
  - タグ入力欄を画面上で追加しつつ、同じ tagNames パラメータとして送信する必要がありました。
  - コメント欄は、フォームとコメント一覧が被らないようにレイアウト調整が必要でした。
  - 削除ボタンを三点メニュー化する際、既存の削除URLとPOST処理は変えずにHTML構造だけ変更しました。

  7. 今後改善したい点

  - いいねは現在メモリ管理なので、DBに保存できるようにしたいです。
  - コメントにも削除や編集機能を追加したいです。
  - 投稿削除時に確認ダイアログを出したいです。
  - タグ入力欄に削除ボタンを付けたいです。
  - APIのレスポンスにタグやコメント数も含められるようにしたいです。

  8. 想定質問と回答

  Q1. タグはどうやって複数登録していますか？
  A. 新規投稿画面で tagNames という同じ名前の入力欄を複数送信し、Controllerで List<String> として受け取っています。その後Serviceで空白区切りの文字列にしてタグ抽出しています。

  Q2. タグ検索はどう実装していますか？
  A. /tags/{name} にアクセスすると、Controllerが postService.findByTagName() を呼びます。Repositoryでは Post と Tag をJOINして、指定タグを持つ投稿を取得しています。

  Q3. 削除はDBから完全に消していますか？
  A. 消していません。deleted_at に日時を入れる論理削除です。一覧や検索では deleted_at IS NULL の投稿だけを表示しています。

  Q4. 空コメントを投稿した場合はどうなりますか？
  A. PostService#createComment で null または空白のみの場合は何も保存せずに処理を終了します。画面は詳細画面へ戻ります。

  Q5. 三点メニューの開閉はどうしていますか？
  A. detail.html に最小限のJavaScriptを追加しています。「…」クリックで表示、メニュー外クリックまたはEscapeキーで閉じる実装です。削除処理自体は既存のPOSTフォームをそのまま使っています。

```

**振り返り**:
・AIエージェントを活用することで、画面操作手順書などのドキュメント作成を効率的に行えると感じた。
・また、実装内容をもとに想定質問と回答まで作成できたため、発表準備にも活用できると感じた。


---




---

## プロンプト 1(サンプル)

**フェーズ**: サンプルデータ(シーダー)の追加

**プロンプト本文**:

```

```

**結果**: 効いた

```

```

**振り返り**:



---



