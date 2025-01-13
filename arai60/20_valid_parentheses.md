問題URL：https://leetcode.com/problems/valid-parentheses/

# Step1

かかった時間：20min


思考ログ：
- うーんパッと思いつかない
- スタックみたいなデータ構造で、開く括弧を詰めてから閉じ括弧が来たらpopする、みたいな感じで書けないかな
```ruby
OPEN_BRACKETS = ['(', '{', '[']
CLOSE_BRACKETS = [')', '}', ']']
BRACKETS_PAIRS = {
  ')' => '(',
  '}' => '{',
  ']' => '[',
}
# @param {String} s
# @return {Boolean}
def is_valid(s)
  stack = []
  brackets = s.chars
  brackets.each do |bracket|
    if OPEN_BRACKETS.include?(bracket)
      stack.push(bracket)
    elsif CLOSE_BRACKETS.include?(bracket)
      open_bracket = BRACKETS_PAIRS.dig(bracket)
      if stack.last == open_bracket
        stack.pop
        next
      else
        return false
      end
    else
      raise "unexpected string #{bracket}"
    end
  end

  true
end
```
- もうちょっと簡潔に書けそうだな〜〜〜〜で迷いながらで15分ぐらいかかったけどとりあえず書けた
- 入ってきた閉じ括弧の取り回しがどうにも煩雑になってしまう
  - `stack.last == open_bracket`で`.last`を確認してから`.pop`するの、まぁ正しいんだが2度手間感がある
  - ネストが深くて可読性が低いかなぁ...微妙な感じだ
  - 根本的な考え方は間違ってない気がする
- submitしたらパスしなかった
  - 閉じ括弧が無い`s ="["`パターンを考慮できてない
  - 正しい組み合わせの時、stackは空になってるはずなので最後は`stack.empty?`であるべきか。
    - これでパスした
- 計算量は空間・時間ともにO(N)かな？

# Step2

- leetcodeのsolution
  - https://leetcode.com/problems/valid-parentheses/solutions/491448/simple-ruby-solution/
```ruby
# @param {String} s
# @return {Boolean}
def is_valid(s)
    return true if s.empty?
    
    stack = []
    s.each_char do |c|
        case c
        when '(', '{', '['
            stack.push(c)
        when ')'
            return false if stack.pop() != '('
        when '}'
            return false if stack.pop() != '{'
        when ']'
            return false if stack.pop() != '['
        end
    end
    return stack.empty?
end
```

- 自分がやりたいことはこれだった
- OPEN, CLOSE_BRACKETともに定数定義の必要がない
- `pop`して正しい括弧だったらループを続ければ良いし、ダメだったらその場でfalseを返せばいいので`.last`でチェックする工程も要らなかった
- たかだか括弧のパターンは3つなので、そのままwhenで並べた方が見たまま理解できるコードになるという発想ができなかった
  - プロダクションコードでも拡張性を意識しすぎて却って読みにくくなっちゃうことがままある
    - 愚直に書いた方が読みやすいかも、という手札は意識しておきたい
- 「stackで解決できそう」という方向性が合ってそうなのは良かった
- `String#each_char`が思い出せなかった
  - https://gist.github.com/jodosha/42fae9b05c1a819323f3
  - パフォーマンス的にも`each_char`の方が速いらしい
- >stack.pop が条件式の中にあるのは少し怖くて、これ、char に )}] でないものが来ると、pop されないわけですね。(https://github.com/olsen-blue/Arai60/pull/6/files#r1904043914)
  - それはそう
  - 「プロダクションで使うならもうちょっと丁寧にケアするけどleetcodeなのでまぁ〜」みたいなケース、毎回扱いに悩んじゃう
  - まぁ書く/書かないを意識的に選んでおくべきで、実装するかは自分の中の根拠として持っておけばレビューなり面接の時に答えられればOKぐらいかな
- >という構造になっているんですが、わりと読みにくいです。A を読んだら、まず、F があるかを私は確認しました。また、最後の else { if () {}} は else if () {} でもいいでしょうか。私はこれは E があることを期待します。(https://github.com/hajimeito1108/arai60/pull/4)
  - 自分がStep1で書いたコードについて考える
    - 簡潔かどうかという観点よりも、読み手に対して認知負荷がデカい
    - `when` で書いてるsolutionのコードは上から読み下した時に同じリズムで読めるので読みやすい
    - 一方自分の書いてるコードはしっかり読まないと何してるか分からん
      - まぁStep1はとりあえず動くコードを優先してるので向き合い方は合ってるんだけども
    - 普段コード書いてて自分のコードが漠然と読みにくいと感じる時の取っ掛かりとして、「読み手の期待に沿っているか」はもうちょっと意識すると良いかもと思った
- という訳でこれがやはりしっくり来るのでStep3へ
```ruby
# @param {String} s
# @return {Boolean}
def is_valid(s)
  stack = []

  s.each_char do |c|
    case c
    when '(', '[', '{'
      stack << c 
    when ')'
      return false if stack.pop != '('
    when ']'
      return false if stack.pop != '['
    when '}'
      return false if stack.pop != '{'
    else
      raise "unexpected character: #{c}"
    end
  end

  stack.empty?
end
```

# Step3

かかった時間：6min （2min × 3）

思考ログ：
- ↑のコードでスッと3回通ったので終わり
