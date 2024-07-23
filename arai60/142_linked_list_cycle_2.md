問題URL：https://leetcode.com/problems/linked-list-cycle-ii/

# Step1

かかった時間：10min

思考ログ：
- ん？これ最初の問題と何が違うんだ...？
  - >Do not modify the linked list.
    - これかな？
  - >If there is no cycle, return null.
    - これもか
  - >return the node where the cycle begins
    - あああ全く読めてない
    - サイクルが始まったノードを返せと言ってるのか
- フロイドのアルゴリズムだとダメそう
- 訪れたノードを覚えておけば大丈夫かな？
- 考え方
  - 今のノードに来たことがある？
    - あったら終わり
  - ないなら今のノードを覚える
  - 次のノードはある？
    - 無いならそこで終わり
    - 進めるなら進む
- ん〜〜〜〜〜〜〜〜〜なんか自然じゃないけどとりあえず何も見ずに書けたは書けた
  - タイマー動かすタイミング忘れてて10分ぐらいかかってしまった
```ruby
def detectCycle(head)
    current = head
    visited_nodes = Set.new

    loop do
        return current if visited_nodes.include?(current)

        visited_nodes.add(current)

        return nil if current.next.nil?
        current = current.next
    end
end
```

# Step2

思考ログ：
- シンプルに読みにくい気がするので時間気にせずにまずリファクタする
- 考え方から整理する
  - 今いるノードが存在する？
    - 存在してなかったら循環してないので終わり
  - 存在してたら今まで辿ったノードの中に今のノードがあるかチェックする
    - 今まで辿ったことがあったら循環してるので終わり
  - 辿ってなかったら次のノードへ
- というわけで書き直してみた
```ruby
def detectCycle(head)
    current = head
    visited_nodes = Set.new

    while current do
        return current if visited_currents.include?(current)

        visited_currents.add(current)
       
        current = current.next
    end

    nil
end
```
改善したポイント

- whileで存在チェックするとが「今いるノードが存在する？」をシュッとできて読みやすい
- ノードの存在チェックしてから辿ったノードとの重複チェックするのがちょっと変な気がする
  - シンプルに存在してたらまず記憶したい気もする....？
- currentをnodeに直そうか迷ったけど、まあループの最後でポインタ動かすのであればcurrentで自然だと思ったのでそのまま
  - と思ったらまぁまぁディスられてた コンテキスト的な文脈で使うのではという指摘は確かに...。
    - https://github.com/Mike0121/LeetCode/pull/7#discussion_r1587660995

他の人の回答を参考にする。
- https://github.com/goto-untrapped/Arai60/pull/22/files
  - コード読んでもさっぱり分からなかったけど、解説を書いてくれてた&手元のiPadで絵描いて理解した
  - >出会った後に、slowを最初から戻してslowとfastを1つずつ進めたら出会う場所がループの始まりになる
  - 割とアルゴリズムに対して事前の知識がないと書けない答えなのではと思ったが...
- https://github.com/sendahuang14/leetcode/pull/2#discussion_r1685695038
  - 再帰で書ける...？なるほど？
    - 再帰苦手すぎて全然分からなかったけど、見よう見まねで書いてみた
```ruby
def detectCycle(head)
    visited_nodes = Set.new
    check_node(head, visited_nodes)
end

def check_node(node, visited_nodes)
    return nil if node.nil?
    return node if visited_nodes.include?(node)

    visited_nodes.add(node)
    check_node(node.next, visited_nodes)
end
```
- エレガントだ...
  - 手札に持っておけるとよさそうだけどスタックが深くなりそうな気がする...？
  - でも空間計算量はvisited_nodes引き回してるだけだからO(N)で変わらない...のか？
- 結局素直に問題を目の前にしたらナイーブにSetで訪れたノードをチェックするのが直感的で良さそうだと思った
  - けど、解法としては思いついておきたい

# Step3

かかった時間：5min

```ruby
def detectCycle(head)
    node = head
    visited_nodes = Set.new

    while node do
        return node if visited_nodes.include?(node)

        visited_nodes.add(node)
        node = node.next
    end
    nil
end
```
思考ログ：
- Step2の解き方でシュッと書けたのでこれで終了。
- 変数名だけnodeに変えた。

# Step4

レビューを受けて修正する。
- >僕は単にvisitedとしてしまうのが好みです。
  - 確かに！こっちの方が読みやすい。
```ruby
def detectCycle(head)
    node = head
    visited = Set.new

    while node do
        return node if visited.include?(node)

        visited.add(node)
        node = node.next
    end
    nil
end
```
