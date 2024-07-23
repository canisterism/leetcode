問題URL：https://leetcode.com/problems/remove-duplicates-from-sorted-list/description/

# Step1

かかった時間：15min


思考ログ：
- valに重複のあるlinked listが得られるので重複を排除したlinked_listを返せば良い。
- 得られるnodeもソート済みなので、今通ったノードの値を覚えておいて、値が同じならスキップして次のノードへ進む、値が異なるならノードを繋いで（？）次のノードへ、みたいな感じか？
- 再帰でも書けそう
- ここで5分筆が止まったので答えを見る
```ruby
def delete_duplicates(head)
    current = head
    sorted = nil
    while current do 
      # ここで書けなかった
      if sorted&.val == current.val
    end
end
```
- あーheadは取っておいてhead.nextに繋いでいかないと駄目か
- 答え見たので全部消して何も見ずに書いた。通った。↓
```ruby
def delete_duplicates(head)
    current = head
    while current && current.next do 
        if current.val == current.next.val
            current.next = current.next.next
        else
            # ここがよく分かってない
            current = current.next
        end
    end

    head
end
```

何がわからなかったか
- 今操作してるlinked listと最終的に返したいlinked listをそれぞれどう表現して最終的な返り値にすればいいのか分からなかった
- 「値が同じなら（ポインタを）次のノードへ移す」操作をどう表現するのか分からなかった
  - というか書いて通ってるけど正直`current = current.next`がなんで正しく動いてるのか説明できない気がする

何を考えて解いていたか
- とりあえずコードの書き方からは離れて日本語で正しく理解してからコードに落とし込むように頑張ってみた...が理解力表現ともに及んでなくて書けなかった。
- 1回答え見てからは日本語で流れをブツブツ言いながら書いてるけど違いがあるのかは知らない

正解してから気づいたこと
- 「覚えておく」方法として単純にlistに詰めていく方法もあったな？と思った。
- head.nextから参照されてるポインタを直接付け替えていくほうがエレガントな気がする

# Step2

思考ログ：
- うーん自分で考えただけだとそんなに問題が見つけられない。なんとなく早期リターンできそうな気はするけど...。
- 過去ログを漁る
  - https://discord.com/channels/1084280443945353267/1261259843671687188/1262997002891821056
  - https://github.com/kenta-kk/arai60/pull/3#discussion_r1680494547
  - 今の実装は割と素直っぽい
- 再帰でチャレンジしてみよう
  - 日本語で整理する
    - 今のノードと次のノードの値を比較して、同じだったら次のノードを「今のノード」として同じ処理を続ける
    - nextを空振った時点で処理が戻ってくる
```ruby
def delete_duplicates(head)
    return head if head.nil? || head.next.nil?

    if head.val == head.next.val
        head = delete_duplicates(head.next)
    else
        head.next = delete_duplicates(head.next)
    end

    head
end
```
- う〜〜〜〜〜〜ん書いたコードが動く理屈はなんとなくわかるけどodaさんのメタファーのイメージがうまく出来ない...
  - https://discord.com/channels/1084280443945353267/1261259843671687188/1262997002891821056
```
前の人から教えて貰う場合は、こうなるでしょう。

1. 前の人から「仕事お願い、ちなみに私の数字は X だった。」と言われる。
2. 自分の輪っかの数字を探す。Y だったとする。X と Y で自分の輪っかを取り除くかを決める。
3. 後ろの人に Y を教えて依頼する。
4. 後ろの人から鎖を渡してもらう。
5. 自分の輪っかを足すか除くかして前の人に鎖を渡す。
```
- 2までは分かる。3から4のイメージが出来ない。
- 例えば...
  - 前の人(head):1
  - 自分:1
  - 後ろの人:2
- 1. 前の人から「（前の人）は1だよ」と言われる
- 2. 自分の数字も1なので自分の輪っかを取り除く必要がありそう
- 3. 後ろの人に「自分は1だよ」と言って仕事を依頼する
- 4. 後ろの人から鎖を渡してもらう←これは何が起こってる...？

- だいぶ詰まってしまったけど、フラットに考えるとこの問題の再帰の解決は手札に入れとく系な気がしたので一旦別のこと考える
- この問題見た時に講師の人と同じ思考回路になれてそうかな？
  - あんまり根拠のない推測だけどたぶんheadのポインタを付け替えていく方式で書きそう
  - 再帰は思いつくけど書かない...それぐらいかな
  - 時間計算量はO(N), 空間計算量はO(1)かな...？
- あとこの辺の思考ログを書いてたら`current = current.next`の理屈を理解した。currentで見るノードのポインタをnextに付け替えてるだけや。
- あ〜while節の条件はcurrentだけにする手段がある？
```ruby
def delete_duplicates(head)
    current = head
    while current do 
        if current.next && current.val == current.next.val
            current.next = current.next.next
        else
            current = current.next
        end
    end

    head
end
```
- 最初のifが若干複雑な気もするけど、その代わり `current.next` が空振っても何も考えずに`current = current.next`してOKになった（次のwhileでcurrent == nilなので）
  - 個人的には`current && current.next` の方が「2つのランプが点灯してますか」「点灯してたらそれらを比較してください」という「確認したら使いたい」単位がwhileの条件とその1行下のif条件節で揃ってるので好きかなぁ...

その他参考にした過去ログ：
- https://github.com/TORUS0818/leetcode/pull/5

# Step3

かかった時間：5min

思考ログ：
- 結局Step1で答えを見た時のコードと同じになった。シュッと書けたのでこんなものか...？

```ruby
def delete_duplicates(head)
    current = head
    while current && current.next do
        if current.val == current.next.val
            current.next = current.next.next
        else
            current = current.next
        end
    end

    head
end
```
