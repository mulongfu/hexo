---
title: [Leetcode] Binary Search - (Easy)704、(Medium)34
date: 2023-05-06 23:20:56
tags: 
- [c++]
- [leetcode]
categories:
- [Leetcode]
---

## [704. Binary Search](https://leetcode.com/problems/binary-search/)

一個簡單的Binary Search，須注意邊界條件。這邊我使用的是左閉右閉:
**[left, right]**

```cpp
class Solution {
   public:
    int search(vector<int>& nums, int target) {
        // for(auto i : nums) cout << i << ' ';
        int left = 0, right = nums.size() - 1;
        while (left <= right) {
            int mid = left + ((right - left) / 2);
            if (target == nums[mid])
                return mid;
            if (target < nums[mid]) {
                // target在左半邊
                //因為left<=right, 目前的nums[mid]一定不是target, 所以右邊界-1
                right = mid - 1; 
            } else {
                // target在右半邊
                //因為left<=right, 目前的nums[mid]一定不是target, 所以左邊界+1
                left = mid + 1;
            }
        }
        return -1;
    }
};


```


## [34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

這題給定一個vector，vector內的數字有可能重複，若target在vector內，則列出target在vector中，開始和結束的index。若不在，則output[-1, -1]


如：
```
Input: nums = [5,7,7,8,8,10], target = 8
Output: [3,4]
```

```
Input: nums = [5,7,7,8,8,10], target = 6
Output: [-1,-1]
```
直覺想法是，先找出target在不在vector內，如果在的話，則用兩個variable，再找target的起點和終點。

```cpp
class Solution
{
public:
    vector<int> searchRange(vector<int> &nums, int target)
    {
        vector<int> res;
        int left = 0, right = nums.size() - 1;
        bool found = false;
        if(nums.size() == 0){
            res.push_back(-1);
            res.push_back(-1);
            return res;
        }
        
        while (left <= right)
        {
            int mid = left + ((right - left) / 2);
            if (target == nums[mid] && !found)
            {
                found = true;
                int start = mid, end = mid;
                
                while(start >= 0 && nums[start] == target){
                    --start;
                }
                ++start; //多減了,要加回去
                while(end < nums.size() && nums[end] == target){
                    ++end;
                }
                --end; //多加了,要減回去

                res.push_back(start);
                res.push_back(end);
            }
            if (target < nums[mid])
            {
                right = mid - 1;
            }
            else
            {
                left = mid + 1;
            }
        }
        if(!found){
            res.push_back(-1);
            res.push_back(-1);
        }
        return res;
    }
};

```