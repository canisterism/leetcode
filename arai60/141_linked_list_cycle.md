https://leetcode.com/problems/linked-list-cycle/

# Step1

かかった時間：40min

```ruby
# Definition for singly-linked list.
# class ListNode
#     attr_accessor :val, :next
#     def initialize(val)
#         @val = val
#         @next = nil
#     end
# end

# @param {ListNode} head
# @return {Boolean}
def hasCycle(head)
    return false if head.nil? || head.next.nil?
    slow, fast = head, head

    while !fast.nil? && !fast.next.nil?
        slow = slow.next
        fast = fast.next.next
        return true if slow.val == fast.val
    end
    false
end
```
思考ログ：
- 解説見たことあったので解法はなんとなく頭に入ってるつもり
  - slowとfastのポインタを用意してfastが追い越したらcycle

何がわからなかったか
  - `@param {ListNode} head` に入ってくるデータの構造がよくわかってなかった
  - こんな感じのコードを書いて`fast`がnilになったりして困っていた
```ruby
def hasCycle(head)
    slow = head
    fast = head.next

    loop do 
       if slow.val == fast.val
            return true
       elsif fast.next.nil?
            return false
       end
       slow = slow.next
       fast = fast.next.next
    end
end
```
- >`@param {ListNode} head` に入ってくるデータの構造がよくわかってなかった
  - これはデータ構造が分かってないんじゃなくて、ノードの長さが1だけのときのケースについてケアできてないだけだった
- 答えを見てから書いたコード
```ruby
def hasCycle(head)
    return false if head.nil? || head.val.nil?
    slow, fast = head, head

    while !fast.nil? && !fast.next.nil? # <- fastがnilで落ちる
      slow = slow.next
      fast = fast.next.next

      return true if slow.val == fast.val
    end
    false
end
```
- パスしなかったのでもう一回消して答えを見てから書いた（一番上のコード）。パスしたのでStep1終わり

何を考えて解いていたか
- アルゴリズムの仕組みとしては分かってたもののコードでどう書いたらその通りに動くのかが掴めなくて時間を使った
  - そもそも事前に解説を見たことがあるという半分ズルなので初見だとまず解き方を思いついてなかったと思う

正解してから気づいたこと
- 頭からnilのチェックをするケースを書き始めると混乱するのでやりたいメジャーケース（今回だとwhileの中）から書き始めて、あとから細かいケースについて詰めていくと自分の考えてることと書いてるコードが一致してて書きやすいかもと思った
- off topic気味だけどleetcodeのエディタにダイレクトで書いてるのはいいのかな（IDE使わないようにしてればOKだと思うが）

# Step2

参考にした過去ログなど：
- https://github.com/sendahuang14/leetcode/pull/1/files
- https://github.com/kagetora0924/leetcode-grind/pull/1/files

思考ログ：
- Step1はwhileの条件が読みにくいかも？改善ポイントってこういうのでいいのかな？
- わからなかったので他の人のPRを探す
  - https://github.com/sendahuang14/leetcode/pull/1/files
  - HashMapでたどったノードをメモする解法。チラッと考えたけど前に見たことある解法で解くイメージが先行して試してなかった。
  - 辞書に書き込むのと配列にpushするのだと辞書に書き込んだ方がO(1)？でアクセスできるので良さそう？
- そういえば計算量について全然意識できてなかったことに気づく
  - 今のアルゴリズムだとどうなるんだ...？
  - 時間計算量は最大でO(N)？
  - 空間計算量はO(1)でいいのかな
- 「より自然な」コードだとたどったノードをメモっていくのが直感的な感じがするので辞書にたどったノードを書いてみる
  - と思ったけど辞書のキーがnode.valだったとして、キーに対応する値に何をいれるべきなのか迷った
    - 値にboolean入れる想像をしたけど、結局アクセスする時に返ってくる値がややこしくなるのでやめた
    - のでとりあえず配列に詰めることにした
- 配列に重複があることをどう判定する？
  - `Array#uniq` は重複を排除した配列を返すので、`visited_nodes.length == visited_nodes.uniq.length`で判定はできるけど冗長な感じがする
- RubyにSetがあったことを思い出す
  - https://docs.ruby-lang.org/ja/latest/class/Set.html#I_--3C--3C
  - >Set#add? は、集合に要素が追加された場合には self を、変化がなかった場合には nil を返します。
  - これの返り値で判定できそう
- とりあえずパスするコードは書けた
```ruby
def hasCycle(head)
    return false if head.nil? || head.next.nil?

    visited_nodes = Set.new
    visited_nodes.add(head.val)
    current = head.next

    while !current.nil? do
        has_visited = visited_nodes.add?(current.val).nil?
        return true if has_visited
        
        current = current.next
    end
    false
end
```
- セルフツッコミしてみる
  - もうちょっと最初の5行ぐらいをシュッと書けるかも
    - 先にhead.valをaddするんじゃなくて、そのまま全部whileの中に入れられないかな
  - `has_visited = visited_nodes.add?(current.val).nil?`が読みづらい気がする
    - `Set#add?`が要素を追加できなかったらnilを返すことを知ってなきゃシュッと読めない

# Step3

かかった時間：15min

```ruby
def hasCycle(head)
    visited_nodes = Set.new
    current = head

    while !current.nil? do
        has_visited = visited_nodes.add?(current.val).nil?
        return true if has_visited

        current = current.next
    end
    false
end
```
思考ログ：
- 書きながらどうやったら直感的になるか考えた
  - 何も考えずに最初から「headをcurrentとして、valを詰める→ポインタが示す先に移動する→ノードがあるか確認する」を繰り返すだけで行けるかも
  - 書けた。だいぶ直感的になった気がする。
  - このぐらいシュッとしてるなら`visited_nodes.add?(current.val).nil?`の部分については許容できるレベルだと思った
  - 変数名については振り返ってなかったけど、見直してもあんまり改善ポイントが浮かばなかった
  - 1発で3回連続パスした！終わり。
