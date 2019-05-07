---
title: leetCode-1-两数之和
date: 2018-06-12 10:54:21
type: "tags"
tags:
- leetCode
---

给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。

你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。

示例:

给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
<!-- more -->
用哈希表，存进去，然后检测存不存在，存在就输出对应key，放到数组里，得到答案，

JavaScript 中其实每个 object 对象就是一个简单的哈希表，具体可到stackoverflow看这个问题：[How is a JavaScript hash map implemented?
](https://stackoverflow.com/questions/8877666/how-is-a-javascript-hash-map-implemented)

时间复杂度和空间复杂度均为O(n);

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    var map = new Map();
    for (var i = 0; i < nums.length; i++) {
        var item = target - nums[i];
        if (map.has(item)){
            var array =[map.get(item), i];
            return array;
        }
        map.set(nums[i], i);
    }
};

console.log(twoSum([1,3,4,5], 9));

```
