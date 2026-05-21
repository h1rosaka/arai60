# 問題タイトル
Split BST
https://www.lintcode.com/problem/847/

注意　leetcode版と少し問題が違うようです。




# STEP 1 自力で解く。

- 思いつけず、他の人の回答を見た
    - https://github.com/naoto-iwase/leetcode/pull/48/changes#diff-bf4c07f093e9b7197d01187d81e45a9d076a1028227be3ed6ba9df6745f3950bR147
    ```py

    """
    ref: https://github.com/naoto-iwase/leetcode/pull/48/changes#diff-bf4c07f093e9b7197d01187d81e45a9d076a1028227be3ed6ba9df6745f3950bR147
    ゴール：v以下のグループ(low)でできたBST、 yより大きいグループ(high)でできたBSTに分ける

    vとの大小関係によって、右足側か、左足側、どちらかは余計なものが混ざっていて、改造が必要な可能性あり

    例）自分がv以下のグループの場合
    「改造が必要になりうる足」は右足(左足は全員自分以下でありv以下を最初から満たす)
    右足を魔法の関数に入れれば、中にいたものたちを「v以下の木」「vより大の木」の2つに分けてくれる
    右足を一度切り離して魔法の関数に入れ、返してくれた「v以下」の方だけ新しい足として付け直せばOK


    終了条件は一番シンプルなベースケースを考えると、書ける
    """
    class Solution:
        def split_b_s_t(self, root: TreeNode, v: int) -> TreeNode:
            def split_bst_helper(node):
                if node is None:
                    return None, None
                
                if node.val <= v:
                    root_lower, root_higher = split_bst_helper(node.right)
                    node.right = root_lower
                    return node, root_higher
                else:
                    root_lower, root_higher = split_bst_helper(node.left)
                    node.left = root_higher
                    return root_lower, node
            
            def count_nodes(node):
                if node is None:
                    return 0
                return 1 + count_nodes(node.left) + count_nodes(node.right)
            
            root_lower, root_higher = split_bst_helper(root)
            size_of_lower = count_nodes(root_lower)
            size_of_higher = count_nodes(root_higher)

            if size_of_lower > size_of_higher:
                return root_lower
            
            return root_higher

    ```
- 補足：サイズが同じだった時の判定をしていないが、それは必ず root_higher.val のほうが大きくなるから




# STEP2 読みやすくする＆他の人の解法を見る
- 再帰関数で書いたので、ループで書き直す

- >再帰とループの中間を念頭において、対応関係から相互に変換できるようにしておくといいでしょう。一例として、中間に来るものは、このような感じです。
    - https://github.com/Ryotaro25/leetcode_first60/pull/50/changes#r1912058276
    - 今まで再帰←→ループ変換は何度かやっていたが、中間があるという理解はできていなかった。
    - 末端からの戻りがけに処理する再帰　←→　行きがけだけに処理する再帰(末端ついたら終了)　←→　ループ(実質行きがけと同じ)

    - スタックを使えば、どんな再帰処理もループで表現できるが、シンプルに書けるとは限らない
        - 行きがけ/ 帰りがけフラグの管理
            - TODO: なぜ必要？　
            - ref: https://github.com/shining-ai/leetcode/pull/47/changes#diff-7f8fd222af7e05ca124f5adb49981c45e9157c2e597bd268955e757cde510c2aR1
        - 計算途中の値を保持するテーブル（辞書など）の維持
        - etc
    
    - トップダウン再帰なら、コードがシンプルになる


- 行きがけだけに処理する再帰
    ```py
    class Solution:
        def split_b_s_t(self, root: TreeNode, v: int) -> TreeNode:
            
            # tail_low / tail_high は「次にノードを繋ぐべき親ノード」
            # side_low / side_high は「親のどっち側（'left' か 'right'）に繋ぐか」
            def split_bst_helper(# 今注目するnode
                                node: TreeNode,
                                # 再構築してる二つの木(low/high)の状態。(現状の末端と、次左右どちらに繋げるか)
                                tail_low: TreeNode, side_low: str, 
                                tail_high: TreeNode, side_high: str):
                if node is None:
                    # 探索終了。それぞれのグループの「最後の末端」を None で閉じる
                    setattr(tail_low, side_low, None)
                    setattr(tail_high, side_high, None)
                    return

                if node.val <= v:
                    # 現在のノードを「lowグループ」に繋ぐ
                    setattr(tail_low, side_low, node)
                    next_node = node.right  # 次に探索するのは「右の子供」（右部分木を見に行く。左部分木はもうそのままで良いので）
                    
                    # 次の注目nodeと、lowグループの情報を更新：
                    # 「次の親は今繋いだnode。BSTのルール上、次は node の 'right' 側に繋ぐ」
                        # これ以降は今回のnodeの右部分木を見に行くので、もし今回繋いだlowの方に来るなら、確実にrightへの接続
                    split_bst_helper(next_node, node, 'right', tail_high, side_high)
                else:
                    # 現在のノードを「highグループ」に繋ぐ
                    setattr(tail_high, side_high, node)
                    next_node = node.left   # 次に探索するのは「左の子供」
                    
                    # 次の注目nodeと、highグループの情報を更新：
                    # 「次の親は今繋いだnode。BSTのルール上、次は node の 'left' 側に繋ぐ」
                    split_bst_helper(next_node, tail_low, side_low, node, 'left')

            # 2. ノード数を数えるヘルパー
            def count_nodes(node):
                if node is None:
                    return 0
                return 1 + count_nodes(node.left) + count_nodes(node.right)
            
            # ダミーノードを用意
            dummy_low = TreeNode(0)
            dummy_high = TreeNode(0)
            
            # 最初は、dummy_low の 'left'、dummy_high の 'right' に繋ぐようにスタートする
            split_bst_helper(root, dummy_low, 'left', dummy_high, 'right')
            
            root_low = dummy_low.left
            root_high = dummy_high.right
            
            # サイズを比較して、大きい方のルートを返す
            size_of_low = count_nodes(root_low)
            size_of_high = count_nodes(root_high)

            if size_of_low > size_of_high:
                return root_low
            
            return root_high

    ```
    - 


- ループ
    ```py
    class Solution:
        def split_b_s_t(self, root: TreeNode, v: int) -> TreeNode:        
            dummy_low = TreeNode(0)
            dummy_high = TreeNode(0)
            
            nodes = [(root,dummy_low, 'left', dummy_high, 'right')]
            while nodes:
                node, parent_low, side_low, parent_high, side_high = nodes.pop()
                if node is None:
                    setattr(parent_low, side_low, None)
                    setattr(parent_high, side_high, None)
                    break
                
                if node.val <= v:
                    setattr(parent_low, side_low, node)
                    next_node = node.right 
                    nodes.append((next_node, node, 'right', parent_high, side_high))
                else:
                    setattr(parent_high, side_high, node)
                    next_node = node.left
                    nodes.append((next_node, parent_low, side_low, node, 'left'))

            root_low = dummy_low.left
            root_high = dummy_high.right

            def count_nodes(node):
                count = 0
                nodes = [node]
                while nodes:
                    node = nodes.pop()
                    count += 1
                    if node.right is not None:
                        nodes.append(node.right)
                    if node.left is not None:
                        nodes.append(node.left)
                return count

            
            # サイズを比較して、大きい方のルートを返す
            size_of_low = count_nodes(root_low)
            size_of_high = count_nodes(root_high)

            if size_of_low > size_of_high:
                return root_low
            
            return root_high

    ```

- 再帰は、「すでに魔法の関数があって、それを使って部下に何を返してもらうか」と考えていたが、これは正確には帰りがけに処理する再帰のことだったのだと学んだ。

- 帰りがけにやりつつ、再帰関数を使わない方法
    - https://github.com/shining-ai/leetcode/pull/47/changes#diff-7f8fd222af7e05ca124f5adb49981c45e9157c2e597bd268955e757cde510c2aR2




# STEP3　　3回連続でエラーなしで解けるまで解く
- step1と大体同じ。
    ```py
    class Solution:
        def split_b_s_t(self, root: TreeNode, v: int) -> TreeNode:

            def split_bst_helper(node):
                if node is None:
                    return None, None
                if node.val <= v:
                    root_low, root_high = split_bst_helper(node.right)
                    node.right = root_low
                    return node, root_high
                else:
                    root_low, root_high = split_bst_helper(node.left)
                    node.left = root_high
                    return root_low, node

            def count_size(node):
                if node is None:
                    return 0
                return 1 + count_size(node.left) + count_size(node.right)

            root_low, root_high = split_bst_helper(root)
            size_low = count_size(root_low)
            size_high = count_size(root_high)

            if size_low <= size_high:
                return root_high
            else:
                return root_low
    ```



# STEP4 レビューFB反映