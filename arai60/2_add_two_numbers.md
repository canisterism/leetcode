問題URL：https://leetcode.com/problems/add-two-numbers/description/

# Step1

かかった時間：10min

思考ログ：
- LinkedListで逆順に数字が与えられるので、合計してから再び逆順で返せば良い、かな？
- うーん考え方としてはシンプルにl1,l2をそれぞれ走査して並び替え、合計した数値を逆側にNodeに詰めてけばいい？
  - もうちょっとキレイに書けそうな気がするがやってみる
  - とりあえず↓でパスはした
```ruby
def add_two_numbers(l1, l2)
  number_l1 = get_number_from_node(l1)
  number_l2 = get_number_from_node(l2)

  result = (number_l1 + number_l2).to_s
  dummy = ListNode.new(nil, nil)
  node = dummy
  result.reverse.each_char do |number|
    node.next = ListNode.new(number, nil)
    node = node.next
  end
  
  dummy.next
end

def get_number_from_node(node)
  raw_number = ''
  while node do
    raw_number += node.val.to_s
    node = node.next
  end
  raw_number.reverse.to_i
end
```
- 割と素直に書けたけど、rubyの`String#reverse`が便利なだけな気がするな.........
  - これ封じられた時の会が全然思いつかなかった
- `get_number_from_node`は`get_number`で良かった
- 計算量は空間時間ともにO(N)かな
  - これ以上悪くなりようもない気がするが
- ちなみにシュッと5分ぐらいで書けた

# Step2

- 他の人のコードを読む
  - https://github.com/katsukii/leetcode/pull/4/files
  - ループが1回で済むようにl1,l2を走査してて驚いた。確かに下の位から操作することになるからこれで良いのか
    - 繰り上がりの処理めんどくさそうだが...？あなるほどcarryって繰り上がりの数字か
    - 全然発想できなかった。ざっと書いてみた。
```ruby
def add_two_numbers(l1, l2)
  sentinel = ListNode.new(nil)
  current = sentinel
  carry = 0

  while l1 || l2 || carry != 0 do
    sum = 0
    sum += carry
    if !l1.nil?
      sum += l1.val
      l1 = l1.next
    end

    if !l2.nil?
      sum += l2.val
      l2 = l2.next
    end

    carry, val = sum.divmod(10)
    current.next = ListNode.new(val)
    current = current.next
  end

  sentinel.next
end
```

- divmodの返り値ははcarry, digitのほうが自然っぽい。
- sentinelの末尾ポインタなのでcurrentよりもtailのが自然かな？
- sentinelを使わない書き方、分かるけど結局初期化処理がループの中に入って特別に処理しなきゃいけないのでつらそうに見える

参考にした過去ログなど：
- https://github.com/t0hsumi/leetcode/pull/5/files
- https://github.com/konnysh/arai60/pull/5/files

# Step3

かかった時間：8min（2:55,2:37,2:18）

思考ログ：
- 最初から3回連続でパスできた。慣れるとこっちのが書きやすいかも？
- 書きながら`sum`は`carry`で初期化すれば良い（`sum=0`は不要）な事に気づいた

```ruby
def add_two_numbers(l1, l2)
  sentinel = ListNode.new
  tail = sentinel
  carry = 0

  while l1 || l2 || carry != 0 do
    sum = carry

    if !l1.nil?
      sum += l1.val
      l1 = l1.next
    end

    if !l2.nil?
      sum += l2.val
      l2 = l2.next
    end

    carry, digit = sum.divmod(10)
    tail.next = ListNode.new(digit)
    tail = tail.next
  end

  sentinel.next
end
```

# Step4

- `if !l1.nil?` が冗長なので`if l1`に書き直し
- ポインタの移動はループの末尾で動かす方が分かりやすいので、if節やめてsumの計算とポインタの移動を別々に行う
- 「carryがnilでなければ」の条件は`Numeric#nonzero?`でキレイに書ける（0だったらnilを返す）

```ruby
def add_two_numbers(l1, l2)
  sentinel = ListNode.new
  tail = sentinel
  carry = 0

  while l1 || l2 || carry.nonzero? do
    sum = carry
    sum += l1.val if l1
    sum += l2.val if l2

    carry, digit = sum.divmod(10)
    tail.next = ListNode.new(digit)

    l1 = l1&.next
    l2 = l2&.next
    tail = tail.next
  end

  sentinel.next
end
```
