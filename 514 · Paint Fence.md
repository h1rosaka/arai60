# 問題タイトル
https://www.lintcode.com/problem/514/
(https://leetcode.com/problems/paint-fence/description/)



# STEP 1 自力で解く。
- ※TLEになります
    ```py
    class Solution:
        """
        postsのおしり2個を取得
        2連続の時→違うやつ全パターン
        それ以外→全パターン追加

        n=3, k=2  
        """
        def num_ways(self, n: int, k: int) -> int:
            my_deque = deque([]) # []必要なかったかも？
            num_ways = 0
            for i in range(k):
                my_deque.append([i])
            while my_deque:
                posts = my_deque.popleft()
                if len(posts) == n:
                    num_ways += 1
                    continue
                if len(posts) >= 2 and posts[-2] == posts[-1]:
                    for i in range(k):
                        if i == posts[-1]:
                            continue
                        my_deque.append(posts + [i]) # コピーになるので増えるなぁ再帰な再利用でなんとかしたい
                    continue
            
                for i in range(k):
                    my_deque.append(posts + [i])

            return num_ways

    ```

- TLEになる件
    - 連続のこと考えなかったら、毎回樹形図にk本枝が生えるので、最終的にk^n種類の組み合わせ。これを全て作る。しかもコピーして付け足しなので、毎度O(N)もかかる。
    - 実際には連続していてk-1本しか枝が生えない場合があるが、劇的にk^nから減るわけでもない。
    - 書き始める前に計算量を考えないといけない。

- 再帰な再利用でバックトラックした場合
    - 再帰関数にするかstack使えば、再利用できてコピーしなくて済んだが、それでも結局k^n種類の組み合わせ作るのは重いか。

- deque([]) # []必要なかったかも？　→必要ない。deque()で良い。
    - >If iterable is not specified, the new deque is empty.
    - https://docs.python.org/3/library/collections.html#collections.deque

# STEP2 読みやすくする＆他の人の解法を見る
- https://github.com/hayashi-ay/leetcode/pull/17/changes
    - >n番目の塗り方の組み合わせは（n-1）番目の塗り方の組み合わせに依存するのでDPの問題として解ける。
        - なるほど。
    - >あと、この問題だと「メモ化」という言葉を聞かれるかもしれません

- top down(メモ化再帰), bottom up
    - bottom up
    ```py
        """
        buttom up DP

        樹形図で全部k^n種類の組み合わせを作ると、O(k^n)かかってしまう。
        1個1個実際に作って確認する必要はなく、全体の数だけわかれば良い
        次のpostの塗り方が何通りあるかは、前のpostのうち、2連続で塗ってるものの数と、そうでないものの数がわかれば出せる

        tree diagram(樹形図) を書くイメージ
        iでの塗り方の数 = i-1で2連続になっていた場合からの伸ばす枝の数 + i-1では連続じゃない場合から伸ばす枝の数
        　　　　　　　　=  i-1で2連続になっているノードの数 * k-1    +  i-1では連続じゃないノードの数 * k
        """
        class Solution:

        def num_ways(self, n: int, k: int) -> int:
            two_consective = [0] * (n + 1)
            one_consective = [0] * (n + 1)

            two_consective[1] = 0
            one_consective[1] = k

            for i in range(2, n + 1):
                one_consective[i] =  (two_consective[i - 1] + one_consective[i - 1]) * (k - 1)
                two_consective[i] =  one_consective[i - 1] * 1

            return two_consective[n] + one_consective[n]

    ```
    - 上記は、DPでよくある配列を用意して、先頭から埋めていって、、という書き方をしているが、実は最終結果だけあれば良いので、途中結果はtemporaryな変数にとっておくだけにすることで、必要なメモリの量を減らせる↓

    ```py
        def num_ways(self, n: int, k: int) -> int:
            two_consective = 0
            one_consective = k

            for i in range(2, n + 1):
                previous_two_consective = two_consective
                previous_one_consective = one_consective
                one_consective =  (previous_two_consective + previous_one_consective) * (k - 1)
                two_consective =  previous_one_consective * 1

            return two_consective + one_consective
    ```
    - top down(メモ化再帰)で書き直す
    ```py
        @cache
        def num_ways(self, n: int, k: int) -> int:
             """
            previous_one_consectiveは、樹形図内で2個前の全てのノードから、それぞれk-1本の枝を生やした数になる。
            """
            if n == 1:
                return k
            if n == 2:
                return k**2
            return self.num_ways(n-2, k) * (k-1) + self.num_ways(n-1, k) * (k-1)
    ```
    - @cacheがない場合の時間計算量
        - 再帰の形が、f(n) = f(n-1) + f(n-2)で、フィボナッチと同じ
        - 具体例
            - ```
                f(5)
                ├─ f(3)
                │  ├─ f(1)
                │  └─ f(2)
                └─ f(4)
                    ├─ f(2)
                    └─ f(3)
                        ├─ f(1)
                        └─ f(2)

                ```
            - f(3)等、何度も同じのを計算しているのでキャッシュ/メモしたい

        - これの時間計算量はO(φ^n)　φ= 1.6
            - ざっくり理解
                - 毎回ほぼ2個に分岐するから、2^n弱 (実際は時々被りがあるから、2より小さめの1.6)
                    ```
                    f(n)
                    ├─ f(n-1)
                    └─ f(n-2)
                    ```
            - しっかり理解
                - 時間計算量を T(n) とすると、
                    - T(n) = T(n-1) + T(n-2)
                - 実際に計算すると、前の数x1.618くらいで増えているように見える
                - 「解が指数関数でかける」つまり、T(n) = r^n と仮定
                - 代入して
                    - r^n =  r^(n-1) + r^(n-2)
                - 両辺を r^(n-2) で割る
                    - r^2 =  r + 1
                - 変形して
                    - r = (1 ± √5)/2
                - rは正だとわかっているので、r = φ ≈ 1.618
                - 補足: 上記議論だと、あくまで「指数関数としてモデル化するなら、ベストなパラメータの値はφ」という感じだが、フィボナッチの漸化式は線形漸化式と呼ばれるのもので、これは指数関数の線型結合になることがわかっているよ、という定理が別にあるらしい。

        - キャッシュすると、各nに関して1回だけ計算なので、O(n)になる




- @cache`デコレータの自力実装
    - https://discord.com/channels/1084280443945353267/1201211204547383386/1220666008881336331

    - そもそも@cacheは
        - >簡単で軽量な無制限の関数キャッシュです
            - https://docs.python.org/ja/3/library/functools.html#functools.cache
        - これは無制限なので、ほぼただの辞書。しかしこれだとほぼ使わないものもメモリに乗り続けて効率が悪いので、LRUで大事なものだけ残したい。

    - 実現したいこと
        - 関数の引数をキーにして、すぐにメモしておいた結果を取り出したい→辞書
        - 今使ったものを新しいものを上に持ってきつつ、一番古いものを消したい。→ 辞書内の各要素を順番管理。取り出して前に持ってきたいので配列よりはポインタで連結したい。
            →さらに、単方向での連結だと、「取り出したいものの一つ後ろ」の指定のために全て舐めないといけなくなるので、双方向で連結。
    
    ```py
    from collections import deque
    from functools import cache


    class Node:
        def __init__(self, key=None, value=None, next=None, prev=None):
            self.key= key
            self.value = value
            self.prev = prev
            self.next = next

    class LRU_Cache:
        def __init__(self, capacity):
            self.capacity = capacity
            self.cache = {}  # key: Node
            # センチネル（番兵）の初期化
            self.sentinel = Node()
            self.sentinel.next = self.sentinel
            self.sentinel.prev = self.sentinel

        def get(self, key):
            if key not in self.cache:
                return None
            
            node = self.cache[key]
            # 「使った」ので先頭に移動させる
            self._move_to_front(node)
            return node.value

        def put(self, key, value):
            if key in self.cache:
                # 既存キーの更新
                node = self.cache[key]
                node.value = value
                self._move_to_front(node)
            else:
                # 新規追加
                if len(self.cache) >= self.capacity:
                    # 溢れる場合は一番後ろ（LRU）を削除
                    self._remove_lru()
                
                new_node = Node(key, value)
                self.cache[key] = new_node
                self._add_to_front(new_node)

        # --- 内部ヘルパーメソッド（繋ぎ変えロジック） ---

        def _move_to_front(self, node):
            """既存のノードをリストから切り離し、先頭に持ってくる"""
            self._detach_node(node)
            self._add_to_front(node)

        def _add_to_front(self, node):
            """センチネルの直後にノードを差し込む"""
            old_head = self.sentinel.next
            # step1 新参者からつなぐ
            node.prev = self.sentinel
            node.next = old_head
            # step2 両隣から新参者へ
            self.sentinel.next = node
            old_head.prev = node

        def _remove_lru(self):
            """一番後ろのノードを削除(外した後、キャッシュからも消す)"""
            lru_node = self.sentinel.prev
            self._detach_node(lru_node)
            del self.cache[lru_node.key]

        def _detach_node(self, node):
            """ノードの前後を繋いで、リストから外す"""
            node.prev.next = node.next
            node.next.prev = node.prev





    def _create_lru_wrapper(func, max_size):
        """
        デコレータの実体。LRUキャッシュのインスタンスを保持し、wrapperを返す。
        """
        
        cache = LRU_Cache(max_size)

        def wrapper(*args, **kwargs):
            # args(タプル) と kwargsのアイテム(タプルのリスト) をまとめて1つのキーにする
            key = (args, tuple(sorted(kwargs.items())))

            result = cache.get(key)
            if result is not None:
                return result

            result = func(*args, **kwargs)
            cache.put(key, result)
            
            return result

        return wrapper


    def my_lru_cache(max_size=1):
        """
        デコレータを作るための関数 (Decorator Maker / Factory)。
        ユーザーは @my_lru_cache(max_size=10) のように使う。
        """
        if max_size <= 0:
            max_size = 1

        def decorator(func):
            return _create_lru_wrapper(func, max_size)

        return decorator


    class Solution:
        @my_lru_cache(max_size=1000)
        def numWays(self, n: int, k:int) -> int:
            if n == 1:
                return k
            if n == 2:
                return k * k
            return (k - 1) * (self.numWays(n-1, k) + self.numWays(n-2, k))


    ```

- decoratorはデコレートするfuncだけを引数に取って関数を返すので、他の引数も渡したい時は、decomakerを経由する。
    - decoratorと同じよう@に続く形で書いて、引数も取るようにすると、それはdecomakerとして動いて、decoratorを返し、その返したdecoratorでラップされる 
    - https://peps.python.org/pep-0318/?utm_source=chatgpt.com#:~:text=%E7%8F%BE%E5%9C%A8%E3%81%AE%E6%A7%8B%E6%96%87%E3%81%A7%E3%81%AF%E3%80%81%E3%83%87%E3%82%B3%E3%83%AC%E3%83%BC%E3%82%BF%E5%AE%A3%E8%A8%80%E3%81%A7%E3%83%87%E3%82%B3%E3%83%AC%E3%83%BC%E3%82%BF%E3%82%92%E8%BF%94%E3%81%99%E9%96%A2%E6%95%B0%E3%82%92%E5%91%BC%E3%81%B3%E5%87%BA%E3%81%99%E3%81%93%E3%81%A8%E3%82%82%E5%8F%AF%E8%83%BD%E3%81%A7%E3%81%99%E3%80%82
    - https://mail.python.org/pipermail/python-dev/2004-September/048874.html#:~:text=%3E%20%20%20%20%20%40deco%20%20%20%20%20%20%20%20%20%20%20%20%20%23%20calls%20deco(f)%0A%3E%20%20%20%20%20%40decomaker(arg)%20%20%20%23%20calls%20tmp(f)%20where%20tmp%3Ddecomaker(arg)

- デコレータ: 関数を改造する人
- wrapper: 改造後に実際動く新しい関数

- クロージャ
    - 関数が定義されたときの環境（スコープ内の変数）を、関数が終了した後もずっと保持し続ける仕組み
    - 関数なのだけれど、インスタンス変数的なものをもてる。
    - 関数がメソッド的動きをするので、インスタンス変数もあってメソッドもあるので、実質クラスみたいな感じになる。
    - 今回の例だとcacheはwrapper()が終了した後も保持される。
    - 今回のように、デコレーションするごとにキャッシュを別で持ちたい、みたいな場合意外にも、こんな感じで関数ごとにcountという状態を持てたりして便利↓
        ```py
        def make_counter():
            count = 0
            def counter():
                nonlocal count  # 外側の変数を書き換えるための宣言
                count += 1
                return count
            return counter

        counter_a = make_counter()
        print(counter_a())  # 1
        print(counter_a())  # 2

        # counter_b は別の「環境」を持つ
        counter_b = make_counter()
        print(counter_b())  # 1（aとは独立している）

        ```




# STEP3　　3回連続でエラーなしで解けるまで解く
- 樹形図を上から段ごとに書いていき、各段で、2種類のノードがそれぞれ何個あるかを数えて次へ行く。
    ```py
    class Solution:
        def numWays(self, n:int, k:int) -> int:
            one_consective = k
            two_consective = 0

            for _ in range(n - 1):
                prev_one_consective = one_consective
                prev_two_consective = two_consective
                one_consective = (prev_one_consective + prev_two_consective) * (k - 1)
                two_consective = prev_one_consective

            return one_consective + two_consective
    ```





# STEP4 レビューFB反映