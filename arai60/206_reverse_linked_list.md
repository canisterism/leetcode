問題URL：https://leetcode.com/problems/reverse-linked-list/

# Step1


かかった時間：6min

思考ログ：
- スタックに詰めて取り出せばOK
  - nextの付替えでやるか新しいノードを作るかどっちがいいかな〜
  - とりあえずざっくり書いてみる
```ruby
# @param {ListNode} head
# @return {ListNode}
def reverse_list(head)
  stack = []

  node = head
  while node do
    stack << node.val
    node = node.next
  end

  sentinel = ListNode.new
  node = sentinel
  while !stack.empty? do
    node.next = ListNode.new(stack.pop)
    node = node.next
  end

  sentinel.next
end
```
- とりあえずパスした
- stackに値を詰めるブロックと取り出すブロックが同じような形になっててわりかし読みやすいんではなかろうか
- nodeが再代入されてるのがイケてない
- なんかこれポインタの付替えの方が良さそうな気もするな
  - ただどう書けば良いのか分からん
- 計算量は時間空間ともにO(N)
- 一旦パスしたのでStep1終わり

# Step2


思考ログ：
- 一旦何も見ずにポインタを逆方向に付け替える方法ができないか考えてみる
  - 方法としてはひたすらnextを掘って、最終ノードにたどり着いたら逆順に辿っていく
  - うおー再起ぽいが苦手すぎて分からん
  - こんな感じのコードを書いて唸っている
```ruby
def reverse_list(head)
  dig(head)
end

def dig(node)
  return unless node
  res = dig(node.next)
  res&.next = node
  node
end
```
- 分からんので答えを見る
- leetcodeの解答。複数見たけどこれが割とメジャーっぽい
```ruby
def reverse_list(head)
    current = head
    prev = nil
    while current
        tmp = current.next
        current.next = prev
        prev = current
        current = tmp
    end
    prev
end
```
- ややこしくないか？全然分からんのだが...
  - あ〜〜前後と今のノードで考えてひたすらポインタ付け替えてるのか。まぁこっちの方が模範解答っぽい
    - tmpみたいな名前の変数、何してるかちゃんと読まないとわからなくてイマイチ好きになれない
    - originalNextみたいな名前をつけてる人がいた。こっちなら嬉しい
  - 他の人のPR読んでて思ったけど、Step1の自分の書き方だとメモリをO(N)で使っちゃうので、この解答で書いた方がO(1)で嬉しいか...
    - ListNodeごとスタックに入れてMemory Limit Exceededになっちゃってる人を観測した

- 双方向リストを作ってる人がいた。面白いけど面接で一発目に出したら理由を求められそう
```ruby
class ReversedListNode < ListNode
    attr_accessor :prev

    def initialize(val, _next = nil, _prev = nil)
        @val = val
        @next = _next
        @prev = _prev
    end

    def next
        self.prev
    end

    def self.from_left(node, _prev=nil)
        ReversedListNode.new(node.val, node.next, _prev)
    end
end

# @param {ListNode} head
# @return {ListNode}
def reverse_list(head)
    left = head
    right = nil
    while left
        right = ReversedListNode.from_left(left, right)
        left = left.next
    end
    right
end
```
- 再起のコードを発見した。末尾ノードが最終的に先端にならなきゃいけないのがムズい
  - まぁでもブロックごとに分けて考えたら意外と難しくない...？
```ruby
def reverse_list(head)
  if (head == nil || head.next == nil)
      return head
  end

  new_head = reverse_list(head.next)
  # 前後のノードでポインタを逆方向に付け替える
  head_next = head.next
  head_next.next = head
  head.next = nil
  # 末尾ノードが先端ノードになるので常に末尾（だった）ノードを返す
  new_head
end
```

参考にした過去ログなど：
- https://github.com/SanakoMeine/leetcode/pull/8
- https://github.com/goto-untrapped/Arai60/pull/27/

# Step3

かかった時間：5.5min

思考ログ：
- とりあえず3回連続ノーミスでパスできたけど自分の中でしっくり来てはいない.........
- この一連の操作が脈絡無い感じどうにも好きになれない
  - node.nextを取っておく
  - nodeのnextポインタを逆側に付け替える
  - nodeをprevに入れる
  - 取っておいた.nextを次に参照するためにnodeに入れる
```ruby
def reverse_list(head)
  prev = nil
  node = head

  while node do
    original_next = node.next
    node.next = prev
    prev = node
    node = original_next
  end

  prev
end
```
- 想定・許容されるデータの分量によってはスタック使っちゃうかもな〜〜
  - >The number of nodes in the list is the range [0, 5000].
  - >-5000 <= Node.val <= 5000
  - 5000か〜〜〜 intだけ入れて最大5000の前提守れるならなら全然スタック使うかも？
