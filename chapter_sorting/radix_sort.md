---
comments: true
---

# 11.8. &nbsp; 基数排序

上节介绍的计数排序适用于数据量 $n$ 大但数据范围 $m$ 不大的情况。假设需要排序 $n = 10^6$ 个学号数据，学号是 $8$ 位数字，那么数据范围 $m = 10^8$ 很大，使用计数排序则需要开辟巨大的内存空间，而基数排序则可以避免这种情况。

「基数排序 Radix Sort」主体思路与计数排序一致，也通过统计出现次数实现排序，**并在此基础上利用位与位之间的递进关系，依次对每一位执行排序**，从而获得排序结果。

## 11.8.1. &nbsp; 算法流程

以上述的学号数据为例，设数字最低位为第 $1$ 位、最高位为第 $8$ 位，基数排序的流程为：

1. 初始化位数 $k = 1$ ；
2. 对学号的第 $k$ 位执行「计数排序」，完成后，数据即按照第 $k$ 位从小到大排序；
3. 将 $k$ 自增 $1$ ，并返回第 `2.` 步继续迭代，直至排序完所有位后结束；

![基数排序算法流程](radix_sort.assets/radix_sort_overview.png)

<p align="center"> Fig. 基数排序算法流程 </p>

下面来剖析代码实现。对于一个 $d$ 进制的数字 $x$ ，其第 $k$ 位 $x_k$ 的计算公式为

$$
x_k = \lfloor\frac{x}{d^{k-1}}\rfloor \mod d
$$

其中 $\lfloor a \rfloor$ 代表对浮点数 $a$ 执行向下取整，$\mod d$ 代表对 $d$ 取余。学号数据的 $d = 10$ , $k \in [1, 8]$ 。

此外，我们需要小幅改动计数排序代码，使之可以根据数字第 $k$ 位执行排序。

=== "Java"

    ```java title="radix_sort.java"
    /* 获取元素 num 的第 k 位，其中 exp = 10^(k-1) */
    int digit(int num, int exp) {
        // 传入 exp 而非 k 可以避免在此重复执行昂贵的次方计算
        return (num / exp) % 10;
    }

    /* 计数排序（根据 nums 第 k 位排序） */
    void countingSortDigit(int[] nums, int exp) {
        // 十进制的位范围为 0~9 ，因此需要长度为 10 的桶
        int[] counter = new int[10];
        int n = nums.length;
        // 统计 0~9 各数字的出现次数
        for (int i = 0; i < n; i++) {
            int d = digit(nums[i], exp); // 获取 nums[i] 第 k 位，记为 d
            counter[d]++;                // 统计数字 d 的出现次数
        }
        // 求前缀和，将“出现个数”转换为“数组索引”
        for (int i = 1; i < 10; i++) {
            counter[i] += counter[i - 1];
        }
        // 倒序遍历，根据桶内统计结果，将各元素填入 res
        int[] res = new int[n];
        for (int i = n - 1; i >= 0; i--) {
            int d = digit(nums[i], exp);
            int j = counter[d] - 1; // 获取 d 在数组中的索引 j
            res[j] = nums[i];       // 将当前元素填入索引 j
            counter[d]--;           // 将 d 的数量减 1
        }
        // 使用结果覆盖原数组 nums
        for (int i = 0; i < n; i++)
            nums[i] = res[i];
    }

    /* 基数排序 */
    void radixSort(int[] nums) {
        // 获取数组的最大元素，用于判断最大位数
        int m = Integer.MIN_VALUE;
        for (int num : nums)
            if (num > m) m = num;
        // 按照从低位到高位的顺序遍历
        for (int exp = 1; exp <= m; exp *= 10)
            // 对数组元素的第 k 位执行计数排序
            // k = 1 -> exp = 1
            // k = 2 -> exp = 10
            // 即 exp = 10^(k-1)
            countingSortDigit(nums, exp);
    }
    ```

=== "C++"

    ```cpp title="radix_sort.cpp"
    /* 获取元素 num 的第 k 位，其中 exp = 10^(k-1) */
    int digit(int num, int exp) {
        // 传入 exp 而非 k 可以避免在此重复执行昂贵的次方计算
        return (num / exp) % 10;
    }

    /* 计数排序（根据 nums 第 k 位排序） */
    void countingSortDigit(vector<int>& nums, int exp) {
        // 十进制的位范围为 0~9 ，因此需要长度为 10 的桶
        vector<int> counter(10, 0);
        int n = nums.size();
        // 统计 0~9 各数字的出现次数
        for (int i = 0; i < n; i++) {
            int d = digit(nums[i], exp); // 获取 nums[i] 第 k 位，记为 d
            counter[d]++;                // 统计数字 d 的出现次数
        }
        // 求前缀和，将“出现个数”转换为“数组索引”
        for (int i = 1; i < 10; i++) {
            counter[i] += counter[i - 1];
        }
        // 倒序遍历，根据桶内统计结果，将各元素填入 res
        vector<int> res(n, 0);
        for (int i = n - 1; i >= 0; i--) {
            int d = digit(nums[i], exp);
            int j = counter[d] - 1; // 获取 d 在数组中的索引 j
            res[j] = nums[i];       // 将当前元素填入索引 j
            counter[d]--;           // 将 d 的数量减 1
        }
        // 使用结果覆盖原数组 nums
        for (int i = 0; i < n; i++)
            nums[i] = res[i];
    }

    /* 基数排序 */
    void radixSort(vector<int>& nums) {
        // 获取数组的最大元素，用于判断最大位数
        int m = *max_element(nums.begin(), nums.end());
        // 按照从低位到高位的顺序遍历
        for (int exp = 1; exp <= m; exp *= 10)
            // 对数组元素的第 k 位执行计数排序
            // k = 1 -> exp = 1
            // k = 2 -> exp = 10
            // 即 exp = 10^(k-1)
            countingSortDigit(nums, exp);
    }
    ```

=== "Python"

    ```python title="radix_sort.py"
    def digit(num: int, exp: int) -> int:
        """ 获取元素 num 的第 k 位，其中 exp = 10^(k-1) """
        # 传入 exp 而非 k 可以避免在此重复执行昂贵的次方计算
        return (num // exp) % 10

    def counting_sort_digit(nums: list[int], exp: int) -> None:
        """ 计数排序（根据 nums 第 k 位排序） """
        # 十进制的位范围为 0~9 ，因此需要长度为 10 的桶
        counter = [0] * 10
        n = len(nums)
        # 统计 0~9 各数字的出现次数
        for i in range(n):
            d = digit(nums[i], exp)  # 获取 nums[i] 第 k 位，记为 d
            counter[d] += 1          # 统计数字 d 的出现次数
        # 求前缀和，将“出现个数”转换为“数组索引”
        for i in range(1, 10):
            counter[i] += counter[i - 1]
        # 倒序遍历，根据桶内统计结果，将各元素填入 res
        res = [0] * n
        for i in range(n - 1, -1, -1):
            d = digit(nums[i], exp)
            j = counter[d] - 1  # 获取 d 在数组中的索引 j
            res[j] = nums[i]    # 将当前元素填入索引 j
            counter[d] -= 1     # 将 d 的数量减 1
        # 使用结果覆盖原数组 nums
        for i in range(n):
            nums[i] = res[i]

    def radix_sort(nums: list[int]) -> None:
        """ 基数排序 """
        # 获取数组的最大元素，用于判断最大位数
        m = max(nums)
        # 按照从低位到高位的顺序遍历
        exp = 1
        while exp <= m:
            # 对数组元素的第 k 位执行计数排序
            # k = 1 -> exp = 1
            # k = 2 -> exp = 10
            # 即 exp = 10^(k-1)
            counting_sort_digit(nums, exp)
            exp *= 10
    ```

=== "Go"

    ```go title="radix_sort.go"
    /* 获取元素 num 的第 k 位，其中 exp = 10^(k-1) */
    func digit(num, exp int) int {
        // 传入 exp 而非 k 可以避免在此重复执行昂贵的次方计算
        return (num / exp) % 10
    }

    /* 计数排序（根据 nums 第 k 位排序） */
    func countingSortDigit(nums []int, exp int) {
        // 十进制的位范围为 0~9 ，因此需要长度为 10 的桶
        counter := make([]int, 10)
        n := len(nums)
        // 统计 0~9 各数字的出现次数
        for i := 0; i < n; i++ {
            d := digit(nums[i], exp) // 获取 nums[i] 第 k 位，记为 d
            counter[d]++             // 统计数字 d 的出现次数
        }
        // 求前缀和，将“出现个数”转换为“数组索引”
        for i := 1; i < 10; i++ {
            counter[i] += counter[i-1]
        }
        // 倒序遍历，根据桶内统计结果，将各元素填入 res
        res := make([]int, n)
        for i := n - 1; i >= 0; i-- {
            d := digit(nums[i], exp)
            j := counter[d] - 1 // 获取 d 在数组中的索引 j
            res[j] = nums[i]    // 将当前元素填入索引 j
            counter[d]--        // 将 d 的数量减 1
        }
        // 使用结果覆盖原数组 nums
        for i := 0; i < n; i++ {
            nums[i] = res[i]
        }
    }

    /* 基数排序 */
    func radixSort(nums []int) {
        // 获取数组的最大元素，用于判断最大位数
        max := math.MinInt
        for _, num := range nums {
            if num > max {
                max = num
            }
        }
        // 按照从低位到高位的顺序遍历
        for exp := 1; max >= exp; exp *= 10 {
            // 对数组元素的第 k 位执行计数排序
            // k = 1 -> exp = 1
            // k = 2 -> exp = 10
            // 即 exp = 10^(k-1)
            countingSortDigit(nums, exp)
        }
    }
    ```

=== "JavaScript"

    ```javascript title="radix_sort.js"
    [class]{}-[func]{digit}

    [class]{}-[func]{countingSortDigit}

    [class]{}-[func]{radixSort}
    ```

=== "TypeScript"

    ```typescript title="radix_sort.ts"
    [class]{}-[func]{digit}

    [class]{}-[func]{countingSortDigit}

    [class]{}-[func]{radixSort}
    ```

=== "C"

    ```c title="radix_sort.c"
    [class]{}-[func]{digit}

    [class]{}-[func]{countingSortDigit}

    [class]{}-[func]{radixSort}
    ```

=== "C#"

    ```csharp title="radix_sort.cs"
    [class]{radix_sort}-[func]{digit}

    [class]{radix_sort}-[func]{countingSortDigit}

    [class]{radix_sort}-[func]{radixSort}
    ```

=== "Swift"

    ```swift title="radix_sort.swift"
    /* 获取元素 num 的第 k 位，其中 exp = 10^(k-1) */
    func digit(num: Int, exp: Int) -> Int {
        // 传入 exp 而非 k 可以避免在此重复执行昂贵的次方计算
        (num / exp) % 10
    }

    /* 计数排序（根据 nums 第 k 位排序） */
    func countingSortDigit(nums: inout [Int], exp: Int) {
        // 十进制的位范围为 0~9 ，因此需要长度为 10 的桶
        var counter = Array(repeating: 0, count: 10)
        let n = nums.count
        // 统计 0~9 各数字的出现次数
        for i in nums.indices {
            let d = digit(num: nums[i], exp: exp) // 获取 nums[i] 第 k 位，记为 d
            counter[d] += 1 // 统计数字 d 的出现次数
        }
        // 求前缀和，将“出现个数”转换为“数组索引”
        for i in 1 ..< 10 {
            counter[i] += counter[i - 1]
        }
        // 倒序遍历，根据桶内统计结果，将各元素填入 res
        var res = Array(repeating: 0, count: n)
        for i in stride(from: n - 1, through: 0, by: -1) {
            let d = digit(num: nums[i], exp: exp)
            let j = counter[d] - 1 // 获取 d 在数组中的索引 j
            res[j] = nums[i] // 将当前元素填入索引 j
            counter[d] -= 1 // 将 d 的数量减 1
        }
        // 使用结果覆盖原数组 nums
        for i in nums.indices {
            nums[i] = res[i]
        }
    }

    /* 基数排序 */
    func radixSort(nums: inout [Int]) {
        // 获取数组的最大元素，用于判断最大位数
        var m = Int.min
        for num in nums {
            if num > m {
                m = num
            }
        }
        // 按照从低位到高位的顺序遍历
        for exp in sequence(first: 1, next: { m >= ($0 * 10) ? $0 * 10 : nil }) {
            // 对数组元素的第 k 位执行计数排序
            // k = 1 -> exp = 1
            // k = 2 -> exp = 10
            // 即 exp = 10^(k-1)
            countingSortDigit(nums: &nums, exp: exp)
        }
    }
    ```

=== "Zig"

    ```zig title="radix_sort.zig"
    // 获取元素 num 的第 k 位，其中 exp = 10^(k-1)
    fn digit(num: i32, exp: i32) i32 {
        // 传入 exp 而非 k 可以避免在此重复执行昂贵的次方计算
        return @mod(@divFloor(num, exp), 10);
    }

    // 计数排序（根据 nums 第 k 位排序）
    fn countingSortDigit(nums: []i32, exp: i32) !void {
        // 十进制的位范围为 0~9 ，因此需要长度为 10 的桶
        var mem_arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
        // defer mem_arena.deinit();
        const mem_allocator = mem_arena.allocator();
        var counter = try mem_allocator.alloc(usize, 10);
        std.mem.set(usize, counter, 0);
        var n = nums.len;
        // 统计 0~9 各数字的出现次数
        for (nums) |num| {
            var d = @bitCast(u32, digit(num, exp)); // 获取 nums[i] 第 k 位，记为 d
            counter[d] += 1; // 统计数字 d 的出现次数
        }
        // 求前缀和，将“出现个数”转换为“数组索引”
        var i: usize = 1;
        while (i < 10) : (i += 1) {
            counter[i] += counter[i - 1];
        }
        // 倒序遍历，根据桶内统计结果，将各元素填入 res
        var res = try mem_allocator.alloc(i32, n);
        i = n - 1;
        while (i >= 0) : (i -= 1) {
            var d = @bitCast(u32, digit(nums[i], exp));
            var j = counter[d] - 1; // 获取 d 在数组中的索引 j
            res[j] = nums[i];       // 将当前元素填入索引 j
            counter[d] -= 1;        // 将 d 的数量减 1
            if (i == 0) break;
        }
        // 使用结果覆盖原数组 nums
        i = 0;
        while (i < n) : (i += 1) {
            nums[i] = res[i];
        }
    }

    // 基数排序
    fn radixSort(nums: []i32) !void {
        // 获取数组的最大元素，用于判断最大位数
        var m: i32 = std.math.minInt(i32);
        for (nums) |num| {
            if (num > m) m = num;
        }
        // 按照从低位到高位的顺序遍历
        var exp: i32 = 1;
        while (exp <= m) : (exp *= 10) {
            // 对数组元素的第 k 位执行计数排序
            // k = 1 -> exp = 1
            // k = 2 -> exp = 10
            // 即 exp = 10^(k-1)
            try countingSortDigit(nums, exp);    
        }
    } 
    ```

!!! question "为什么从最低位开始排序？"

    对于先后两轮排序，第二轮排序可能会覆盖第一轮排序的结果，比如第一轮认为 $a < b$ ，而第二轮认为 $a > b$ ，则第二轮会取代第一轮的结果。由于数字高位比低位的优先级更高，所以要先排序低位再排序高位。

## 11.8.2. &nbsp; 算法特性

**时间复杂度 $O(n k)$** ：设数据量为 $n$ 、数据为 $d$ 进制、最大为 $k$ 位，则对某一位执行计数排序使用 $O(n + d)$ 时间，排序 $k$ 位使用 $O((n + d)k)$ 时间；一般情况下 $d$ 和 $k$ 都比较小，此时时间复杂度近似为 $O(n)$ 。

**空间复杂度 $O(n + d)$** ：与计数排序一样，借助了长度分别为 $n$ , $d$ 的数组 `res` 和 `counter` ，因此是“非原地排序”。

与计数排序一致，基数排序也是稳定排序。相比于计数排序，基数排序可适用于数值范围较大的情况，**但前提是数据必须可以被表示为固定位数的格式，且位数不能太大**。比如浮点数就不适合使用基数排序，因为其位数 $k$ 太大，可能时间复杂度 $O(nk) \gg O(n^2)$ 。