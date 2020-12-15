---
layout:     post
title:      算法专题01 - 双指针
subtitle:    "\"two point sulution\""
date:       2019-10-01
author:     Lin
header-img: img/post-bg-coding0813.jpeg
catalog: true
tags:
    - algorithm
---

本文罗列下双指针技巧 及常规的题目:

* 快慢指针: 判断链表是否有环的问题(141题)，查找重复数字(287题目)；
* 左右指针: 二分查找；
* 滑动窗口算法;
* k-sum 问题

## 题目实战

### 快慢指针相关

1. 判断链表中是否有环

用两个指针，一个跑得快，一个跑得慢。如果不含有环，跑得快的那个指针最终会遇到 null，说明链表不含环；如果含有环，快指针最终会超慢指针一圈，和慢指针相遇，说明链表含有环。

```java
class Solution {
    boolean hasCycle(ListNode head) {
        ListNode fast, slow;
        fast = slow = head;
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;

            if (fast == slow) return true;
        }
        return false;
    }
}
```

2. 已知链表中含有环，返回这个环的起始位置

当快慢指针相遇时，让其中任一个指针指向头节点，然后让它俩以相同速度前进，再次相遇时所在的节点位置就是环开始的位置。

为什么? 第一次相遇时，假设慢指针 slow 走了 k 步，那么快指针 fast 一定走了 2k 步，也就是说比 slow 多走了 k 步（也就是环的长度）。设相遇点距环的起点的距离为 m，那么环的起点距头结点 head 的距离为 k - m，也就是说如果从 head 前进 k - m 步就能到达环起点。就是说 如果从相遇点继续前进 k - m 步，也恰好到达环起点。

```java
class Solution {
    ListNode detectCycle(ListNode head) {
        ListNode fast, slow;
        fast = slow = head;
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
            if (fast == slow) break;
        }
        // 上面的代码类似 hasCycle 函数
        slow = head;
        while (slow != fast) {
            fast = fast.next;
            slow = slow.next;
        }
        return slow;
    }
}
```

3. 寻找链表的中点

让快指针一次前进两步，慢指针一次前进一步，当快指针到达链表尽头时，慢指针就处于链表的中间位置。

```java
class Solution {
    ListNode main(ListNode head) {
        ListNode fast, slow;
        fast = slow = head;
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        // slow 就在中间位置
        return slow;
    }
}
```

4. 寻找链表的倒数第 k 个元素

使用快慢指针，让快指针先走 k 步，然后快慢指针开始同速前进。这样当快指针走到链表末尾 null 时，慢指针所在的位置就是倒数第 k 个链表节点。

```java
class Solution {
    ListNode main(ListNode head) {
        ListNode slow, fast;
        slow = fast = head;
        while (k-- > 0)
            fast = fast.next;

        while (fast != null) {
            slow = slow.next;
            fast = fast.next;
        }
        return slow;
    }
}
```

### 滑动窗口

这个算法技巧的思路非常简单，就是维护一个窗口，不断滑动，然后更新答案么。

大致逻辑是这样:

```text
int left = 0, right = 0;

while (right < s.size()) {
    // 增大窗口
    window.add(s[right]);
    right++;

    while (window needs shrink) {
        // 缩小窗口
        window.remove(s[left]);
        left++;
    }
}
```

代码模板:

```java
/* 滑动窗口算法框架 */
class tmp {
    void slidingWindow(string s, string t) {
        unordered_map<char, int> need, window;
        for (char c : t) need[c]++;

        int left = 0, right = 0;
        int valid = 0;
        while (right < s.size()) {
            // c 是将移入窗口的字符
            char c = s[right];
            // 右移窗口
            right++;
            // 进行窗口内数据的一系列更新
            //...

            /*** debug 输出的位置 ***/
            printf("window: [%d, %d)\n", left, right);
            /********************/

            // 判断左侧窗口是否要收缩
            while (window needs shrink) {
                // d 是将移出窗口的字符
                char d = s[left];
                // 左移窗口
                left++;
                // 进行窗口内数据的一系列更新
                //...
            }
        }
    }
}
```

1. 无重复字符的最长子串(leetcode#3)

右指针前进，当发现窗口内有重复字符时，左指针前进，直到无重复字符；当窗口内没有重复字符时，更新结果；

```java
public class Solution {
    public int lengthOfLongestSubstring(String s) {
        int res, left, right;
        res = left = right = 0;
        Set<Character> set = new HashSet<>();
        while (right < s.length()) {
            Character ch = s.charAt(right);
            while (set.contains(ch)) {
                set.remove(s.charAt(left));
                left++;
            }
            set.add(ch);
            res = Math.max(res, right - left + 1);
            right++;
        }
        return res;
    }
}
```

### 左右指针

1. 盛最多水的容器(leetcode#11)

初始化左右指针为 左指针指向0，右指针指向末尾，逐渐向中间的高度大于右指针时，移动右指针；反之，移动左指针。同时更新结果。

```java
public class Solution {
    public int maxArea(int[] height) {
        int maxArea = 0;
        for (int i = 0, j = height.length - 1; i < j;) {
            int tmpArea = (j - i) * Math.min(height[i], height[j]);
            if (tmpArea > maxArea) {
                maxArea = tmpArea;
            }
            if (height[i] > height[j]) {
                j--;
            } else {
                i++;
            }
        }
        return maxArea;
    }
}
```

### K-SUM问题

> 问题描述: Given an array of integers, return all unique combinations of k numbers such that they add up to a specific target. (Assuming k < number of elements in the given integer array)

这是面试常见题目类型，首先先将数据排序: `Arrays.sort(nums);`。

方法定义为: `List<List<Integer>> kSum(int[] nums, int k, int target)`。

先来看看 `k=1` 和 `k=2`的场景:

#### k=1

```text
if (k == 1) {
  List<List<Integer>> res = new ArrayList<>();
  if (Arrays.binarySearch(nums, target) >= 0) {
    res.add(Collections.singletonList(target));
    return res;
  }
}
```

#### k=2

使用左右指针技巧，定义 `left`指针指向头 `right`指针指向尾，并不断往中间靠拢，计算 `nums[left] + nums[j]` 与 `target`:

* 当 `nums[left] + nums[right] == target`，恭喜 找到一个组合，加入到结果集后，同时移动两个指针 `++left; --right`;
* 当 `nums[left] + nums[right] > target`，移动 `--right`(需 找一个更小的和);
* 当 `nums[left] + nums[right] < target`，移动 `++left`。
* 直至 `left` >= `right`， 结束。

**伪代码:**

```text
List<List<Integer>> res = new ArrayList<>();
int left = 0, right = nums.length - 1;
while (left < right) {
  int sum = nums[left] + nums[right];
  if (sum == target) {
    res.add(new ArrayList<>(Arrays.asList(nums[left], nums[right])));
    # de-dup
    while (left < right && nums[left] == nums[left + 1]) ++left;
    while (left < right && nums[right] == nums[right - 1]) --right;
    ++left;
    --right;
  } else if (sum < target) {
    ++left;
  } else {
    --right;
  }
}
return res;
```

#### k>=3

当 `k=3`时，我们可以降级到 `k=2`问题: 我们可以遍历数组，来求得 `nums[left] + nums[right] = target - nums[index]`。

所以，K-SUM问题可以通过递归（通过求(K-1)问题）求解。

`k-1`函数定义: `private List<List<Integer>> kSumHelper(int[] nums, int k, int target, int beginIdx)`。

那么，3-sum 的伪代码为:

```text
List<List<Integer>> res = new ArrayList<>();
for (int i = 0; i <= nums.length - 3; i++) {
  // Get 2-Sum result for each element
  List<List<Integer>> subResults = kSumHelper(nums, 2, target - nums[i], i + 1);
  for (List<Integer> list : sub) {
    list.add(0, nums[i]);
  }
res.addAll(sub);
return res;
```

最后，完整的代码为:

```java
/**
 * @author lin
 * @version v 0.1 2020/11/27
 **/
public class Ksum {
    /**
     * Find all k elements groups that adding up to given target.
     *
     * @param nums   input array
     * @param k      k
     * @param target target value
     * @return all groups
     */
    public List<List<Integer>> kSum(int[] nums, int k, int target) {
        Arrays.sort(nums);
        if (k <= 1) {
            List<List<Integer>> res = new ArrayList<>();
            if (k == 1 && Arrays.binarySearch(nums, target) >= 0) {
                res.add(Collections.singletonList(target));
            }
            return res;
        }
        return kSum(nums, k, target, 0);
    }

    /**
     * Helper function for K-sum.
     *
     * @param nums     input array
     * @param k        k
     * @param target   target value
     * @param beginIdx begin index
     * @return all groups of k elements from beginIdx that adding up to target.
     */
    private List<List<Integer>> kSum(int[] nums, int k, int target, int beginIdx) {
        int len = nums.length;
        if (k == 2) {
            List<List<Integer>> res = new ArrayList<>();
            int left = beginIdx, right = len - 1;
            while (left < right) {
                int sum = nums[left] + nums[right];
                if (sum == target) {
                    res.add(new ArrayList<>(Arrays.asList(nums[left], nums[right])));
                    while (left < right && nums[left] == nums[left + 1]) {
                        ++left;
                    }
                    while (left < right && nums[right] == nums[right - 1]) {
                        --right;
                    }
                    ++left;
                    --right;
                } else if (sum < target) {
                    ++left;
                } else {
                    --right;
                }
            }
            return res;
        }
        List<List<Integer>> res = new ArrayList<>();
        for (int i = beginIdx; i <= len - k; i++) {
            if (i > beginIdx && nums[i] == nums[i - 1] || nums[i] + (k - 1) * nums[len - 1] < target) {
                continue;
            }
            if (nums[i] + (k - 1) * nums[i + 1] > target) {
                break;
            }
            List<List<Integer>> sub = kSum(nums, k - 1, target - nums[i], i + 1);
            for (List<Integer> list : sub) {
                list.add(0, nums[i]);
            }
            res.addAll(sub);
        }
        return res;
    }
}
```

## leetcode 的双指针专题

* [Two Pointers](https://leetcode.com/tag/two-pointers/)

## 参考资料:

* leetcode
* [labuladong的算法小抄: 双指针技巧总结](https://labuladong.gitbook.io/algo/shu-ju-jie-gou-xi-lie/2.5-shou-ba-shou-shua-shu-zu-ti-mu/shuang-zhi-zhen-ji-qiao)
