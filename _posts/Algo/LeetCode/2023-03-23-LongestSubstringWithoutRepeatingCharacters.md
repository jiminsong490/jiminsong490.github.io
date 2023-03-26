---
title: Longest SubString Without Repeating Character
author: jimin
date: 2023-03-23 00:00:00 +0900
categories: [Algo, LeetCode]
tags: [투포인터]
pin: false
---

# 해결

 !["LSWRC"](/assets/img/postpic/Algo/LeetCode/Longest%20Substring%20Without%20Repeating%20Characters.jpeg)

# 코드

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        HashSet<Character> set = new HashSet<>();
        int result = 0;
        int left = 0;

        for (int right = 0; right < s.length(); right++) {
            if (!set.contains(s.charAt(right))) {
                set.add(s.charAt(right));
                result = Math.max(result, right - left + 1);
            } else {
                while (s.charAt(left) != s.charAt(right)) {
                    set.remove(s.charAt(left));
                    left++;
                }
                set.remove(s.charAt(left));
                left++;
                set.add(s.charAt(right));

            }
        }
        return result;
    }
}
```

# 참고

 - 직접구현