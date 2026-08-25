# 問題タイトル
問題リンク
Meeting Rooms II
https://neetcode.io/problems/meeting-schedule-ii/question

# STEP 1 自力で解く。

```py
import heapq
class Solution:
    def minMeetingRooms(self, intervals: List[Interval]) -> int:
        if not intervals:
            return 0

        sorted_intervals = sorted(intervals, key=lambda i: i.start)
        end_time_heap = [0] # end_time
    
        for interval in sorted_intervals:
            earliest_end = end_time_heap[0]
            if interval.start < earliest_end:
                # 会議室ひとつ追加
                heapq.heappush(end_time_heap, interval.end)
            else:
                # 同じ部屋を利用(時間過ぎたのに部屋にいる人を追い出し、新しい人を入れる)
                heapq.heappop(end_time_heap)
                heapq.heappush(end_time_heap, interval.end)
            
        return len(end_time_heap)

```
- 最初、 `if not intervals:`の考慮が漏れていてWAになった。書き終わったと思ったら、エッジケースの確認をする。
- 解法のイメージ
    - 会議室は時間で予約するが、終わっても次の人が来ない限り使い続けて良い
    - 自分の会議室予約の時間になったとき
        - もし全部屋まだ使用中だったら、部屋が足りないということなので、新たに一つ追加。
        - 終了時間過ぎているが今まだ人が残っている部屋があるなら、予約終了時間が一番早い人を追い出す。
    - すべてのintervalを処理し終えた時、全員が部屋使えたということが保証されていて、かつ、部屋数が一番たくさん必要だった時の数がheapに維持されているので、その数が答え。



# STEP2 読みやすくする＆他の人の解法を見る
- https://github.com/hayashi-ay/leetcode/pull/62/changes#diff-a0ae933995d3a32d66b233c1e96d7f1bbe7ff33e80eb0997d04a4806ba5d2be5R10
    - 一つのmeetingを、開始イベントと終了イベントに分解して、すべてのイベントを時系列に並べて追っていく。

- https://github.com/hayashi-ay/leetcode/pull/62/changes#diff-a0ae933995d3a32d66b233c1e96d7f1bbe7ff33e80eb0997d04a4806ba5d2be5R133
    - 会議室に入る時、すべての会議室を見回って、時間過ぎて残っている人を追い出してから自分が部屋に入る方式

- https://github.com/olsen-blue/Arai60/pull/57/changes#diff-a0ae933995d3a32d66b233c1e96d7f1bbe7ff33e80eb0997d04a4806ba5d2be5R6
    - 上のhayashi-ayさんの最初のと同様に、開始イベントと終了イベントに分解するが、meetingの開始や終了がない時刻も含め、1時間ごとに処理を実行、というロジック
    - 疎な部分を減らすのは座標圧縮と呼ぶとのこと

- https://github.com/shining-ai/leetcode/pull/56/changes#r1573756033
    - 考え方は大体同じ。heappushはどちらの場合もやるので、まとめている。最初にダミーのend_timeを入れない代わりに、end_time_heapが空じゃないか確認している。さらにダミーを入れないで済むことによって、私のstep1で入れていたintervalsが空の時のための特別対応も不要に。


- heapの実装
    - https://github.com/python/cpython/blob/3.14/Lib/heapq.py

- heapは二分木
- 基本的に木は、value,left,rightのプロパティを持つnodeで表すことが多い
- だけど実装はただの配列で表している
- なぜか
    - value,left,right相当の情報をもてれば、nodeでなくても良い
    - 配列中のあるindexの値でvalueを保持し、left, rightは自分のindexから計算できる(最後のもの以外は必ず2つ子を持つように設計するので)から配列で十分
    - nodeで持つより配列でもったほうが、メモリ消費少なく、隣に並ぶのでキャッシュも効きやすい



- 下の図のような順番で配列に格納する(左→右)
    ```
              [0]             ← レベル0（頂点）
            /     \
          [1]      [2]         ← レベル1
        /   \     /   \
      [3]   [4] [5]   [6]     ← レベル2

    インデックス:  0    1   2   3   4   5   6
    配列 (heap): [ A,  B,  C,  D,  E,  F,  G ]
    ```

    - leftとrightは下記のように出せる。従ってparentも出せる
        - left = 2 * i + 1
        - right = 2 * i + 2
        - parent = (i - 1) // 2


    - 仕組み↓

    - 前提:
        - 一人ずつツリーの席に座っていく
        - 自分のindexの値iは、「自分より前に座っている人の数」と一致する
        - 先頭以外は全員双子
        - みな双子を産んで、席につかせて、次の人(まだ子を産んでいないpotential親)のターンへ、と進む。

    - left
        - 「自分より前に座っている人の数」はi人。i人は全員2人産んでる。i回出産があったということなので、自分が産む直前で、2i人の子供が生まれている。追加で最初の始祖の人(誰の子供でもない)がいるので、2i+1人座ってる。つまりleftのindexは2i+1
    - parent
        - left, rightの式から逆算でも良いが、下記のようにも考えられる
            - 今まさにindex=iの子が産まれようとしていて、親のidは何だっけ？という状況
            - 「自分より前に座っている人の数(=i)」から先頭の人を引いて//2すると、「双子の組が今まで何個できたか」がわかる
            - この双子の組の数は、そのまま「(自分の親より前に座っている)親(人)の数」と同じ
            - 「自分より前に座っている人の数」はその人のindexの値になるので、それこそが、自分の親のidになる


- 
    ```py
    class MinHeap:
        def __init__(self):
            self.heap = []

        def push(self, val):
            self.heap.append(val)
            self._up(len(self.heap) - 1)

        def pop(self):
            if not self.heap:
                return None
            if len(self.heap) == 1:
                return self.heap.pop()
            
            root = self.heap[0]
            self.heap[0] = self.heap.pop()  # 末尾の要素をルートに持ってくる
            self._down(0)
            return root

        def peek(self):
            return self.heap[0] if self.heap else None

        def _up(self, i):
            while i > 0:
                parent = (i - 1) // 2
                if self.heap[i] < self.heap[parent]:
                    self.heap[i], self.heap[parent] = self.heap[parent], self.heap[i]
                    i = parent
                else:
                    break

        def _down(self, i):
            n = len(self.heap)
            while 2 * i + 1 < n:
                left = 2 * i + 1
                right = 2 * i + 2
                smallest = left
                
                if right < n and self.heap[right] < self.heap[left]:
                    smallest = right
                    
                if self.heap[i] > self.heap[smallest]:
                    self.heap[i], self.heap[smallest] = self.heap[smallest], self.heap[i]
                    i = smallest
                else:
                    break
    ```

    ```py
    class Solution:
        def minMeetingRooms(self, intervals: List[List[int]]) -> int:
            rooms_end_time = []
            for interval in sorted(intervals, key=lambda i: i.start):
                if rooms_end_time and rooms_end_time[0] <= interval.start:
                    heapq.heappop(rooms_end_time)
                heapq.heappush(rooms_end_time, interval.end)
            return len(rooms_end_time)
    ```

# STEP3　　3回連続でエラーなしで解けるまで解く

- step2と同じ
    ```py
    class Solution:
        def minMeetingRooms(self, intervals: List[List[int]]) -> int:
            rooms_end_time = []
            for interval in sorted(intervals, key=lambda i: i.start):
                if rooms_end_time and rooms_end_time[0] <= interval.start:
                    heapq.heappop(rooms_end_time)
                heapq.heappush(rooms_end_time, interval.end)
            return len(rooms_end_time)
    ```


# STEP4 レビューFB反映