# 問題タイトル
Meeting Rooms I
https://neetcode.io/problems/meeting-schedule/question




# STEP 1 自力で解く。
- _
    ```py
    class Solution:
        def canAttendMeetings(self, intervals: List[Interval]) -> bool:
            tuple_interevals = []
            for intereval in intervals:
                tuple_interevals.append((intereval.start, intereval.end))
            sorted_intervals = sorted(tuple_interevals)
            latest_end = 0
            for intereval in sorted_intervals:
                start, end = intereval
                if start < latest_end:
                    return False
                latest_end = end
            return True
    ```
- begin/end の方がしっくりくるが、Intervalの定義に合わせて、start, end
- start, endといった別の変数に一回入れずとも、`if interval.start < latest_end:`などとそのままでも良かった

# STEP2 読みやすくする＆他の人の解法を見る
- https://neetcode.io/problems/meeting-schedule/solution
    - `intervals.sort(key=lambda i: i.start)`でソートしている。
    - >key には 1 引数関数を指定します。これは iterable の各要素から比較キーを展開するのに使われます 
        - https://docs.python.org/ja/3.13/library/functions.html#sorted
    - これが思いつけなかったので、STEP1では半ば強引にtupleにしてからソートしていた。(Interval同士の並び替えは定義されていない)



- https://github.com/hayashi-ay/leetcode/pull/59/changes#diff-ab6cfa3e835a34ed5eb3a6822101cce7a548f12d24be2a329849b326fdc792b6R18
    - heapでやれば、途中でインターバルが追加されてもソートし直す必要なし。
    - ```py
        import heapq
        class Solution:
            def canAttendMeetings(self, intervals: List[Interval]) -> bool:
                tuple_interevals = []
                for intereval in intervals:
                    tuple_interevals.append((intereval.start, intereval.end))

                heapq.heapify(tuple_interevals)
                latest_end = 0
                while tuple_interevals:
                    start, end = heapq.heappop(tuple_interevals)
                    if start < latest_end:
                        return False
                    latest_end = end
                return True
        ```


# STEP3　　3回連続でエラーなしで解けるまで解く
- intervalsのままソート
    ```py
        class Solution:
            def canAttendMeetings(self, intervals: List[Interval]) -> bool:
                sorted_intervals = sorted(intervals, key=lambda i: i.start)
                latest_end = 0
                for interval in sorted_intervals:
                    if interval.start < latest_end:
                        return False
                    latest_end = interval.end
                return True
    ```


# STEP4 レビューFB反映