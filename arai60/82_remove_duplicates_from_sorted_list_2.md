問題URL：https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/description/

# Step1

かかった時間：20min


思考ログ：
- 元のリストから重複したノード全てを取り除いてください、かな？
- 83と違うのは重複しても1つは残すけど、今回はそうじゃない
- 基本的には83同じように自身のノードを見ながらポインタの付替えで行けそうかな？
- ポインタの付替えの処理が頭に入ってなくてこんなコードを書いて全然パスしないなど
```ruby
def delete_duplicates(head)
    current = head
    while current && current.next do
        if current.val == current.next.val
            current = current.next.next
        else
            current = current.next
        end
    end
    current
end
```
- 全然答えに辿り着けそうになかったので答え見た
    - あー、配列で返ればいいので乱暴に配列に詰めてからfilterする手段があるか...
    - 想定解ではないっぽいけどRubyらしいとも言えるかも
```ruby
def delete_duplicates(head)
    array = []
    while head do
        array.push(head.val)
        head = head.next
    end

    array.filter { |n| array.count(n) < 2 }
end
```
- とりあえず通ったけどめっちゃ遅そう
    - leetcodeによると時間計算量:O(N^2)、空間計算量O(N)。ですよね。
    - filterとcountの操作で2重ループになってN^2かな
- leetcodeの他のSolution見ても疲れてるのか全くピンと来なくてこれしか書けなかった
- まぁ計算量にだんだんあたりが付いてきて「これは乱暴なアルゴリズム」と認識して書けたので成長してるとしよう
- ポインタの概念、まだ慣れないなぁ...

# Step2

かかった時間：40min

- 計算量がマシな回答を思いつかないので他の人の回答を眺めてみる
- https://github.com/olsen-blue/Arai60/pull/4
    - iPadで手書きしながら10分ぐらい考えてようやくコードの意味がわかった
    - 被ってないListNodeだけを集めた、解答になるListNodeを作る、なるほど
    - 1つのノードだけで操作すると大変だから別のノードを用意するの、完全に正しい
        - 日本語でどうしたいかが思いつければコード書くのはそんな難しくない気がしてきた
        - 逆に手を動かすのが先に来ちゃうと混乱して終わる
        - まぁギョームでもこれは当たり前にやってるか...
    - step1の自分の解答、よく考えるとvalだけ集めた配列でも解答として認められてるが、たぶん問題の意図としてはLinkedListを扱えるかなのであんまり良くない解答な気がしてきたな

```ruby
def delete_duplicates(head)
    dummy = ListNode.new(nil, head)
    duplicates_removed_list = dummy
    node = head

    while node && node.next do
        if node.val == node.next.val
            while node.val == node.next&.val do
                node = node.next
            end
    
            duplicates_removed_list.next = node.next
        else
            duplicates_removed_list = duplicates_removed_list.next
        end
        node = node.next
    end

    dummy.next
end
```

- とりあえずこんな感じで正解を書けはした

# Step3

かかった時間：20分

思考ログ：
- 最後は3回連続でパスしたけど、最初`else`節でdup_list側のポインタ動かし忘れたりして何回かfailした

```ruby
def delete_duplicates(head)
    dummy = ListNode.new(nil, head)
    duplicates_removed_list = dummy
    node = head

    while node && node.next do
        if node.val == node.next.val
            while node.val == node.next&.val do
                node = node.next
            end
            duplicates_removed_list.next = node.next
        else
            duplicates_removed_list = duplicates_removed_list.next
        end
        node = node.next
    end

    dummy.next
end
```

- 前回3問だけやってから、半年ぐらい空いてまた再開したのもあり非常に時間がかかるなぁ...

# Step4

- >動かすのはリストではなくノードというのが直感的だと思うので、変数名は工夫の余地があるのではないかと感じました。(olsen-blue)
  - 完全に正しい。最初の自分の回答でarrayとして書いてたままの発想になってた。
  - うーんやはり`last_fixed_node`あたりの名付けになりそう
- >この問題を、ループを回す仕事の引き継ぎだと思ったときに、どのような状態で引き継いでいるか、について考えてみてください。while を回っているときに、duplicates_removed_list は出力することが確定したものの最後ですね。duplicates_removed_list.next はどうなっていますか。(oda)
  - この引き継ぎの例、分かりそうでわからないので改めて考える
  - https://github.com/atomina1/Arai60_review/pull/2#discussion_r1889800366
  - ↑のスレッド眺めててようやく分かった気がする
  - node, last_fixed_nodeは「今日はここから始めてください」という申し送りか
  - なので、ループが終わった時点で次の人が何も考えずに指示をこなせばOKの状態を（前のループの終了時点で）作っておく必要がある
    - last_fixed_nodeは返したいノードの末尾なので、else節では「今見てるノードは被りがなく正しいノードなので次のノードに移動しておく」操作を忘れるということは次の人への引き継ぎが済んでない状態で帰っちゃったことを意味してる
    - この発想ならポインタの動かし忘れが起きない気がする。なるほど〜〜

- 参考：整理のためにコメント付けて書き下したコード
```ruby
def delete_duplicates(head)
    dummy = ListNode.new(nil, head)
    # 引き継ぎ1：正解であることが確定しているノードの現在地
    last_fixed_node = dummy
    # 引き継ぎ2：重複を排除しながら走査したいノードの現在地
    node = head

    # 出勤する
    # 今の輪に次のがつながってれば
    while node && node.next do
        # 今の輪と次の輪が同じ値なら（外す対象）
        if node.val == node.next.val
            while node.val == node.next&.val do
                # 同じ値の輪の最後まで移動する
                node = node.next
            end
            # その次の輪をfixedの輪に繋ぐ
            last_fixed_node.next = node.next
        else
            # fixedの輪は次の輪に移動する
            last_fixed_node = last_fixed_node.next
        end
        # 今の輪の次の輪に移動する
        node = node.next
    end

    dummy.next
end
```
