# 链表

## 链表相交

例题 [链表相交](https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/)

给你两个单链表的头节点 headA 和 headB ，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 null 。



判断链表是否相交的思路是

1. 首先将两个链表尾部对齐 
   1. 计算两个链表的长度
   2. 将较长的链表的指针curA移动lenA-lenB步
2. 然后将两个链表的当前指针放在距尾部相同的位置（1.2步）
3. 开始往后遍历，直到curA==curB

## 环形链表

参考链接：https://www.programmercarl.com/0142.%E7%8E%AF%E5%BD%A2%E9%93%BE%E8%A1%A8II.html#%E6%80%9D%E8%B7%AF

**判断一个链表是否存在环的方法：**双指针法。定义一个慢指针slow一个快指针fast，slow移动步长为1，fast移动步长为2。如果存在slow==fast，那么就说明存在环。

**确定环的入口：**若已确定存在环，那么可以通过上面的方法得到相遇节点的位置。**在fast与slow相遇节点 index1 和初始节点index2同时出发，每次移动相同步长的两个指针，最终index1与index2相遇的位置就是环的入口。**

代码实现：

```c++
// 问题代码
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if(head == NULL || head->next == NULL){
            return NULL;
        }

        ListNode* slow = head;
        ListNode* fast = head->next;

        while(slow != fast){
            slow = slow->next;
            fast = fast->next->next;
        }

        ListNode* index1 = slow;
        ListNode* index2 = head;
        while(index1 != index2){
            index2 = index2->next;
            index1 = index1->next;
        }

        return index1;
        
    }
};
```

上面这个代码逻辑上貌似很清晰，就是根据前面两条定理来写的。但是在初始化的时候，slow和fast并没有初始化为同一指针，导致相遇的节点不满足第二条定理，导致代码运行结果为《超出时间限制》。

这里初始化的时候，初始化成了slow在前一个，fast在后一个，那么其实相遇的时候，不满足`x = z`，而是满足`x = z - 1?存疑`。

所以其中一种解决办法是保存一个 z-1；（测试不可以）

另一种解决办法就是fast slow初始化同一位置，但是不能以`fast ！=slow` 作为循环的判断条件了。一种办法是：

```c++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if(head == NULL || head->next == NULL){
            return NULL;
        }

        ListNode* slow = head;
        ListNode* fast = head;
        
        while(fast != NULL && fast->next != NULL){
            slow = slow->next;
            fast = fast->next->next;
            if(slow == fast){
                ListNode* index1 = slow;
                ListNode* index2 = head;
                while(index1 != index2){
                    index2 = index2->next;
                    index1 = index1->next;
                }
                return index1;
            }
        }
        return NULL;
    }
};
```

# 哈希表

## cpp基础用法

|                      | 是否有序 | 是否可以重复 | 复杂度                  |
| -------------------- | -------- | ------------ | ----------------------- |
| std::set()           | 是       | 否           | 查询O(logn) 增删O(logn) |
| std::multiset()      | 是       | 是           | 查询O(logn) 增删O(logn) |
| std::unordered_set() | 否       | 否           | 查询O(1) 增删O(1)       |
| std::map()           | 是       | 否           | 查询O(logn) 增删O(logn) |
| std::multimap()      | 是       | 是           | 查询O(logn) 增删O(logn) |
| std::unordered_map() | 否       | 否           | 查询O(1) 增删O(1)       |

`.find(x)`在哈希表内查找`key == x`，如果查找到，则返回对应的迭代器`iter`，**`iter->first`是key, `iter->second`是value；**如果没有查到，则返回`.end()`的结果。 `.end()`返回的是**最后一个元素的下一个迭代器**。

## 例题

### 例题 [两数之和](https://leetcode.cn/problems/two-sum/)

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        std::unordered_map<int, int> umap;
        int i = 0;
        for(; i < nums.size(); i++){
            auto iter = umap.find(target - nums[i]);
            if(iter != umap.end()){
                return vector<int>{iter->second, i}; // {iter->second, i} 也可以
            }
            
            umap.insert(pair<int, int>{nums[i], i}); // 注意 pair
        }
        return {};
    }
};
```



# 字符串 string

## cpp基础用法

| 函数                                                       | 用法                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| `string& insert (size_t pos, const string& str);`          | 插入字符串。pos 表示要插入的位置，也就是下标，是标量；str 表示要插入的字符串，它可以是 string 字符串，也可以是C风格的字符串。 |
| `string& erase (size_t pos = 0, size_t len = npos);`       | 删除字符串某元素。pos 表示要删除的子字符串的起始下标，len 表示要删除子字符串的长度。如果不指明 len 的话，那么直接删除从 pos 到字符串结束处的所有字符（此时 len = str.length - pos） |
| `string substr (size_t pos = 0, size_t len = npos) const;` | 提取子字符串。pos 为要提取的子字符串的起始下标，len 为要提取的子字符串的长度。 |

## KMP算法

### KMP的核心

KMP算法是用来解决 `判断字符串a中是否存在字符串b`的方法。KMP中有一个重要的概念即`前缀表`，前缀表表示的是：**字符串b中的 索引 [0, i] 对应的字符串中，最长相同前缀后缀的长度。**

**前缀是指不包含最后一个字符的所有以第一个字符开头的连续子串**。

**后缀是指不包含第一个字符的所有以最后一个字符结尾的连续子串**。

判断前后缀是否相同要注意：前后缀的顺序还是沿着从前到后的顺序。而不是前缀沿着从前到后的顺序，后缀沿着从后到前的顺序。

前缀表的作用就是*避免字符匹配的过程中，遇到不相同的字符，重新开始从头匹配的行为，从而提高效率。*

### next数组的构造

给一个字符串a，b。假设字符串a是我们待搜索的字符串（长字符串）；字符串b是我们要去匹配的字符串（短字符串）。KMP算法为了避免遇到不匹配的字符后 重新匹配的问题，于是就提出next数组，该next数组用来记录在字符串b的[0~i]位的字符串中最长相同前缀后缀的长度。那么在匹配过程中，当遇到a的下一位字符与b的下一位字符不一致时，我可以查询next数组，使得指针跳转到字符串b中next数组指定的位置，从而继续匹配。直到找到字符串或得出字符串不存在的结果。

因此，next数组的构造是最关键的一步。构造的思路为：

1. 初始化

   我们用 i 代表后缀的最后一位；j 代表前缀的最后一位。next[i] 代表的是 字符串b的 [0~i] 的子字符串中 最大前缀和后缀完全相等的位数（**也就是前缀的最后一位的索引值+1**）。

   由于一开始，字符串b的 [0~i] 就只有一个字符，也就没有所谓的前缀后缀，因此初始化：

   ```c++
   j = -1; next[0] = j+1; j++;
   ```

2. 处理 b[j] != b[i] 的情况

   ```c++
   while(j > 0 && b[j] != b[i]){
   	j=next[j-1];
   }
   ```

3. 处理 `b[j] == b[i] `的情况，并给next[i]赋值。

   跳出步骤2的循环之后，

   要么就是`b[j] == b[i]`，**这里我们把视角移到对于i的这一层循环上**，我们想，经历了步骤2中的while循环，那么在上一轮对i的循环时（i-1的时候），这个j要么是0，要么是落在j-1的位置上，并且`b[j-1] == b[i-1]`。这里就有一点动态规划的意思了，因为可以保证在当前循环中，要么 i 的前一位和 j 的前一位是相等的，要么 `j==0`，所以当`b[i]==b[j]`的时候，我们直接j++即可找到 最大前缀和后缀完全相等的位数，后面赋值next[i]=j。

   要么就是 j==0：b[j] == b[i]时，和前文描述一样，j++然后赋值；b[j] != b[i]时这就说明前后缀没有完全相同的，就给next[i]=0，也同样是next[i]=j。

   ```c++
   if(b[j] == b[i]){
   	j++;
   }
   next[i]=j;
   ```

完整代码：

```c++
void getNext(int *next, string b){
    j = -1; next[0] = j+1; j++; // 初始化
    for(int i = 1; i < b.size(); i++){
        //
        while(j > 0 && b[j] != b[i]){
            j=next[j-1];
        }
        if(b[j] == b[i]){
            j++;
        }
        next[i]=j;
    }
}
```



### 例题

#### LC-28 [字符串匹配](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)

```c++
class Solution {
public:
    void getNext(int* next, string needle){
        int j = -1;
        next[0] = j+1;
        j++;
        for(int i = 1; i < needle.size(); i++){
            while(j > 0 && needle[i] != needle[j]){
                j = next[j-1];
            }
            if(needle[j] == needle[i]){                
                j++;
            }
            next[i] = j;
        }
    }
    int strStr(string haystack, string needle) {
        int length_h = haystack.size();
        int length_n = needle.size();
        int next[length_n];
        getNext(next, needle);
        int i = 0, j = 0;
        
        while(i < length_h){
            while(j > 0 && haystack[i] != needle[j]){
                j = next[j-1];
            }
            if(haystack[i] == needle[j]){
                j++;
            }
            i++; // 指针先后移，返回结果就不用+1
            if(j == length_n){
                return i - length_n;
            }
            
        }
        return -1;
    }
};
```

#### LC-459 [重复的子字符串](https://leetcode.cn/problems/repeated-substring-pattern/)

```c++
class Solution {
public:
    void getNext(int* next, string s){
        int j = 0;
        next[0] = j;
        for(int i = 1; i < s.size(); i++){
            while(j > 0 && s[j] != s[i]){
                j = next[j-1];
            }
            if(s[i] == s[j]){
                j++;
            }
            next[i] = j;
        }
    }

    bool repeatedSubstringPattern(string s) {
        int length = s.size();
        if(length == 1){
            return false;
        }
        int next[length];
        getNext(next, s);
        // next最后一位不为0 并且 总长度为基准字符串长度的倍数
        if(next[length-1] != 0 && length%(length-next[length-1]) == 0){
            return true;
        }
        return false;
        
    }
};
```



# 双指针

## 例题

### 例题1 [移除元素](https://leetcode.cn/problems/remove-element/description/)

给你一个数组 `nums` 和一个值 `val`，你需要 **[原地](https://baike.baidu.com/item/原地算法)** 移除所有数值等于 `val` 的元素，并返回移除后数组的新长度。

相向指针的思路是 **左指针一直指向等于val的元素，右指针一直指向不等于val的元素。交换/移动元素之后呢，就将左右指针按照各自方向前进。**

```c++
// 相向指针，该方法会改变数组的相对位置
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int length = nums.size();
        if(length == 0){
            return 0;
        }
        int i = 0, j = length - 1;

         while(j >= 0 && nums[j] == val){
             j--;
         }
         while(i <= j){
             while(i <= j && nums[j] == val){
                 j--;
             }
             while(i <= j && nums[i] != val){
                 i++;
             }
             if(i < j){
                 nums[i] = nums[j];
                 j--;i++;
             }
         }
         return i;
	}
};
```

快慢指针的思路是 **快指针一直前进，只有在快指针满足条件的时候（对应元素不等于val）才会进行移除/交换操作，同时更新慢指针。**

```c++
// 同向快慢指针，该方法不会改变数组的相对位置
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int length = nums.size();
        if(length == 0){
            return 0;
        }
        int i_slow = 0, i_fast = 0;

        while(i_fast <= length-1){
            if(nums[i_fast] != val){
                nums[i_slow] = nums[i_fast];
                i_slow++;
            }            
            i_fast++;            
        }        
        return i_slow;
    }
};
```

### 例题2 [删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)

给你一个 **升序排列** 的数组 `nums` ，请你**[ 原地](http://baike.baidu.com/item/原地算法)** 删除重复出现的元素，使每个元素 **只出现一次** ，返回删除后数组的新长度。元素的 **相对顺序** 应该保持 **一致** 。

```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int i_slow = 0, i_fast = 0;
        while(i_fast < nums.size() - 1){
            
            //要注意在这里多一步的判断条件
            while(i_fast < nums.size() - 1 && nums[i_fast] == nums[i_fast + 1]){
                i_fast++;
            }
            i_fast++;        
            if(i_fast < nums.size()){
                i_slow++;
                nums[i_slow] = nums[i_fast];
            }
            
        }
        return i_slow + 1;

    }
};
```

### 例题3 [移动零](https://leetcode.cn/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

```c++
// 题目要求保持相对顺序，所以使用快慢指针的方法，同时注意 是移动0，不是覆盖0
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int i_slow = 0, i_fast = 0;
        int temp;
        while(i_fast < nums.size()){
            if(nums[i_fast] != 0){
                temp = nums[i_slow];
                nums[i_slow] = nums[i_fast];
                nums[i_fast] = temp;
                i_slow++;
            }
            i_fast++;
        }
    }
};
```

### 例题4 [比较含退格的字符串](https://leetcode.cn/problems/backspace-string-compare/)

给定 `s` 和 `t` 两个字符串，当它们分别被输入到空白的文本编辑器后，如果两者相等，返回 `true` 。`#` 代表退格字符。

```c++
// 使用双指针，可能会减少内存，但是感觉这个题更适合使用栈

```



```c++
//使用栈
class Solution {
public:
    void getSimplexStr(string s, stack<char>& S){
        for(char ch:s){
            if(ch == '#'){
                if(S.size() == 0){
                    continue;
                }
                else{
                    S.pop();
                }                
            }
            else{
                S.push(ch);
            }
            
        }
    }

    bool backspaceCompare(string s, string t) {
        stack<char> S;
        stack<char> T;
        
        getSimplexStr(s, S);
        getSimplexStr(t, T);
        
        
        if(S.size() != T.size()){
            return false;
        }


        char temp1,temp2;
        while(! S.empty()){
            temp1 = S.top();S.pop();
            
            temp2 = T.top();T.pop();
            
            if(temp1 != temp2){
                return false;
            }
        }
        
        return true;
    }
};
```

### 例题5 [有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/description/)

```c++
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        int left = 0;
        int right = nums.size() - 1;
        int index = right;
        vector<int> result(nums.size(), 0);
        while(left < right){
            if(nums[left] * nums[left] >= nums[right] * nums[right]){
                result[index] = nums[left] * nums[left];
                left++;
                index--;
            }
            else{
                result[index] = nums[right] * nums[right];
                right--;
                index--;
            }
        }
        result[index] = nums[left] * nums[left];
        return result;
    }
};
```

### 例题 6 [三数之和](https://leetcode.cn/problems/3sum/description/)

返回的是数值

```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> res;
        sort(nums.begin(), nums.end());
        for(int i = 0; i < nums.size()-2; i++){
            
            if(nums[i] > 0){
                return res; // 是return , break也可以
            }
            if(i > 0 && nums[i] == nums[i-1]){
                continue;
            }
            int left = i+1, right = nums.size()-1;
            while(left < right){
                if(nums[i] + nums[left] + nums[right] > 0){
                    right--;
                }
                else if(nums[i] + nums[left] + nums[right] < 0){
                    left++;
                }
                else{
                    res.push_back(vector<int>{nums[i], nums[left],nums[right]});
                    while(left < right && nums[left] == nums[left + 1]){
                        left++;
                    }
                    while(left < right && nums[right] == nums[right - 1]){
                        right--;
                    }
                    left++;
                    right--;
                }
            }
        }
        return res;
    }
};
```

### 例题7 [四数之和](https://leetcode.cn/problems/4sum/)

返回的是值，因此可以进行sort

你可以按 **任意顺序** 返回答案 。

难点在于剪枝，因为target并不一定是0，除了判断nums[i] > target，在只有 nums[i] > 0 同时满足的情况下才会剪枝。

```c++
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        if(nums.size() < 4){
            return res;
        }
        for(int i = 0; i < nums.size()-3;i++){

            // 注意剪枝条件 不仅仅是 nums[i] > target
            if(nums[i] > target && nums[i] > 0){
                break;
            }
            if(i > 0 && nums[i]==nums[i-1]){
                continue;
            }
            
            for(int j = i+1; j < nums.size()-2;j++){
                if(nums[i] + nums[j] > target && nums[i] + nums[j] > 0){
                    break;
                }
                if(j > i+1 && nums[j]==nums[j-1]){
                    continue;
                }
                int left = j+1, right = nums.size()-1;
                //cout << i << j << left << right << endl;
                while(left < right){
                    if((long)nums[i] + nums[j] + nums[left] + nums[right] > target){
                        right--;
                    }else if((long)nums[i] + nums[j] + nums[left] + nums[right] < target){
                        left++;
                    }else{
                        
                        res.push_back({nums[i],nums[j],nums[left],nums[right]});
                        while(left < right && nums[right]==nums[right-1]){
                            right--;
                        }
                        while(left < right && nums[left]==nums[left+1]){
                            left++;
                        }
                        left++;right--;
                    }
                }
            }
        }
        return res;
    }
};
```

### 例题8 [无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/description/)

利用unordered_set定义一个队列，left指针指向队列头，如果遇到重复字符（利用.find函数来判断），则重复删除队列头，直到不存在重复字符。

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_set<int> window;
        if(s.size() == 0){
            return 0;
        }
        int left = 0;
        window.insert(s[left]);
        int maxLen = 1;
        for(int i = 1;i < s.size();i++){            
            while(window.find(s[i]) != window.end()){
                window.erase(s[left]);
                left++;
            }
            window.insert(s[i]);
            maxLen = max(maxLen, i-left+1);
        }
        return maxLen;
    }
};
```



# 栈与队列（stack & queue）

## 基础知识

1. C++中stack 是容器么？

   并不是，C++的STL中提供了七种容器。分别是三个序列式容器：向量（vector）、双端队列（deque）、列表（list），此外你也可以把 string 和 array 当做一种序列式容器。四个关联式容器：集合（set）、多重集合（multiset）、映射（map）和多重映射（multimap）。所以STL中栈往往不被归类为容器，而被归类为container adapter（容器适配器）。

2. 我们使用的stack是属于哪个版本的STL？

3. 我们使用的STL中stack是如何实现的？

   栈的内部结构，栈的底层实现可以是vector，deque，list 都是可以的， 主要就是数组和链表的底层实现。stack和queue都是默认使用deque来实现的。

4. stack 提供迭代器来遍历stack空间么？

   不提供，栈提供push 和 pop 等等接口，所有元素必须符合先进后出规则，**所以栈不提供走访功能，也不提供迭代器(iterator)**。 不像是set 或者map 提供迭代器iterator来遍历所有元素。队列也不允许有遍历行为，不提供迭代器。

   栈是先进后出，队列是先进先出，但并不是两端队列。

5. 基础用法

   | stack的函数 | 用法                     |
   | ----------- | ------------------------ |
   | push()      | 入栈                     |
   | pop()       | 弹出栈顶；但是不返回数值 |
   | top()       | 返回栈顶数值；但是不弹出 |
   | empty()     | 判断是否为空             |

   | queue的函数 | 用法         |
   | ----------- | ------------ |
   | push()      | 入队列       |
   | pop()       | 出队列       |
   | front()     | 返回队列头   |
   | back()      | 返回队列尾   |
   | empty()     | 判断是否为空 |
   | size()      | 返回大小     |

## 例题 [有效括号](https://leetcode.cn/problems/valid-parentheses/description/)

解题思路：用栈是肯定的，那么为了方便匹配的判断，采用一个哈希表，用反括号作为key，用正括号作为value。然后遍历字符串，在没有遇到反括号时，就将遇到的正括号入栈；遇到反括号之后就依次弹出栈并判断是否栈顶元素是反括号对应的正括号，不满足则返回false，满足则正常出栈。最后判断栈是否为空，完全匹配则栈为空，否则不为空。

```c++
class Solution {
public:
    bool isValid(string s) {
        unordered_map<char,char> umap;
        umap.insert(pair<char,char>(')','('));
        umap.insert(pair<char,char>(']','['));
        umap.insert(pair<char,char>('}','{'));

        stack<char> S; 
        for(char ch:s){
            if(umap.count(ch)){
                if(S.empty() || S.top() != umap[ch]){
                    return false;
                }
                else{
                    S.pop();
                }
            }
            else{
                S.push(ch);
            }
        }
        return S.empty();

    }
};
```

## 例题 [删除字符串中的所有相邻重复项](https://leetcode.cn/problems/remove-all-adjacent-duplicates-in-string/description/)

```c++
class Solution {
public:
    string removeDuplicates(string s) {
        stack<char> S;

        for(char ch:s){            
            if(S.size() >= 1 && S.top() == ch){
                S.pop();
            }else{
                S.push(ch);
            }            
        }

        string res;
        while(!S.empty()){
            res += S.top();
            S.pop();
        }

        reverse(res.begin(), res.end());
        return res;

    }
};
```

## 例题 [滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/) （单调队列）

这里需要自己实现一种单调队列。

```c++
class MyQueue{ // 这是一个单调队列
public:
    deque<int> que;
    void pop(int x_front){
        if(!que.empty() && x_front == que.front()){
            que.pop_front();
        }
    }
    void push(int x){
    // push进来的新元素如果比之前队列的back，那就一股脑把之前的元素都从back端pop掉
        // 注意不是从front进行pop
        while(!que.empty() && que.back() < x){
            que.pop_back();
        }        
        que.push_back(x);
    }
    int getmaxvalue(){
    // 要维护队列内的最大值一直在front位置
        return que.front();
    }
};
```

全部代码

```c++

class Solution {
private:
class MyQueue{ // 这是一个单调队列
public:
    deque<int> que;
    void pop(int x_front){
        if(!que.empty() && x_front == que.front()){
            que.pop_front();
        }
    }
    void push(int x){
    // push进来的新元素如果比之前队列的back，那就一股脑把之前的元素都从back端pop掉
        while(!que.empty() && que.back() < x){
            que.pop_back();
        }        
        que.push_back(x);
    }
    int getmaxvalue(){
    // 要维护队列内的最大值一直在front位置
        return que.front();
    }
};

public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        MyQueue que;
        vector<int> res;
        for(int i = 0; i < k;i++){
            que.push(nums[i]);
        }
        res.push_back(que.getmaxvalue());
        for(int i = k; i < nums.size(); i++){
            que.pop(nums[i-k]);
            que.push(nums[i]);
            res.push_back(que.getmaxvalue());
        }
        
        return res;

    }
};
```

## 例题 [前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/) （优先级队列，大顶堆小顶堆）

大顶堆：父亲总是要比孩子大；小顶堆：父亲总是要比孩子小。该类数据结构很适合在一个大数据集中寻找前k个高频/低频的元素。

C++中的优先级队列即为堆的数据结构。

```c++
class Solution {
public:
    class myComparison{
    public:
        bool operator()(const pair<int, int>& lhs, const pair<int, int>& rhs) {
            return lhs.second > rhs.second;
        }
    };

    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int,int> umap;
        for(auto num:nums){
            umap[num]++;
        }
        
        priority_queue<pair<int,int>, vector<pair<int,int>>, myComparison> pri_que;

        for(unordered_map<int,int>::iterator it = umap.begin(); it != umap.end(); it++){
            pri_que.push(*it);
            if(pri_que.size() > k){
                pri_que.pop();
            }
        }
        vector<int> res;
        for(int i = k-1; i >= 0;i--){
            res.push_back(pri_que.top().first);
            pri_que.pop();
        }
        return res;

    }
};
```



### 填坑：C++中的优先级队列priority_queue的应用与原理

**优先级队列其实是一个堆，堆就是一棵完全二叉树，同时保证父子节点的顺序关系。**





# 二叉树

## 基础知识

### 二叉树的分类

满二叉树：二叉树中的每个结点的度为0或者2，并且度为0的结点在同一层上，则为满二叉树。也可以说深度为k，有2^k-1个节点的二叉树。

完全二叉树：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，*并且最下面一层的节点都集中在**该层最左边的**若干位置。*若最底层为第 h 层，则该层包含 1~ 2^(h-1)  个节点。

二叉搜索树(BST)：若左子树不为空，则**左子树上所有节点均小于根节点**；若右子树不为空，则右子树上所有节点均大于根节点；

平衡二叉搜索树：又被称为AVL（Adelson-Velsky and Landis）树，且具有以下性质：它是**一棵空树**或它的**左右两个子树的高度差的绝对值不超过1**，**并且左右两个子树都是一棵平衡二叉树**。

### 二叉树的存储

链式存储：每一个节点存储节点值、左指针、右指针。

```c++
struct TreeNode{
	int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x): val(x), left(NULL), right(NULL) {};
}
```

顺序存储：用数组存储二叉树的节点的值（层序遍历的顺序），如果父节点的下标为 i，则左孩子的下标为 `2i+1`，右孩子的下标为`2i+2`。

### 二叉树的遍历

遍历二叉树节点的过程中，每一次打印节点都是将其作为中间节点（父节点）打印的。就是说根据下面遍历的顺序，在中的位置打印节点。

前序遍历：中左右

```c++
class Solution {
public:
    void preorderTraversal(TreeNode* cur, vector<int> & result){
        if(cur == NULL){
            return;
        }
        result.push_back(cur->val);
        preorderTraversal(cur->left, result);
        preorderTraversal(cur->right, result);
        
    }
    
    vector<int> Traversal(TreeNode* root) {
        vector<int> result;
        preorderTraversal(root, result);
        return result;
    }
};
```

中序遍历：左中右

```c++
class Solution {
public:
    void midorderTraversal(TreeNode* cur, vector<int> & result){
        if(cur == NULL){
            return;
        }
        midorderTraversal(cur->left, result);
        result.push_back(cur->val);        
        midorderTraversal(cur->right, result);
        
    }
    
    vector<int> Traversal(TreeNode* root) {
        vector<int> result;
        midorderTraversal(root, result);
        return result;
    }
};
```

后序遍历：左右中

```c++
class Solution {
public:
    void postorderTraversal(TreeNode* cur, vector<int> & result){
        if(cur == NULL){
            return;
        }
        postorderTraversal(cur->left, result);   
        postorderTraversal(cur->right, result);
        result.push_back(cur->val);     
        
    }
    
    vector<int> Traversal(TreeNode* root) {
        vector<int> result;
        postorderTraversal(root, result);
        return result;
    }
};
```

二叉树的层序遍历

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        queue<TreeNode*> que;
        vector<vector<int>> res;
        if(root == nullptr){
            return res;
        }

        que.push(root);
        while(!que.empty()){
            vector<int> level;
            int size = que.size(); //每一层的节点个数
            for(int i = 0;i < size;i++){
                TreeNode* node = que.front();
                level.push_back(node->val);
                que.pop();
                if(node->left) que.push(node->left);
                if(node->right) que.push(node->right);
            }
            res.push_back(level);
        }
        return res;
    }
};
```

- [x] 102.[二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)
- [x] 107.[二叉树的层次遍历II](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/)
- [x] 199.[二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)
- [x] 637.[二叉树的层平均值](https://leetcode.cn/problems/average-of-levels-in-binary-tree/)
- [x] 429.[N叉树的层序遍历](https://leetcode.cn/problems/n-ary-tree-level-order-traversal/)
- [x] 515.[在每个树行中找最大值](https://leetcode.cn/problems/find-largest-value-in-each-tree-row/)
- [x] 116.[填充每个节点的下一个右侧节点指针](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node/)
- [x] 117.[填充每个节点的下一个右侧节点指针II](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node-ii/)
- [x] 104.[二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/description/)
- [x] 111.[二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/description/) （只需要多判断一步是不是叶子节点，如果是的话直接return即可）

## 二叉树的深度/高度

二叉树的深度是从根节点到该节点的最长简单路径上的节点数目或边的条数。

二叉树的高度是从该节点到叶子节点的最长简单路径上的节点数目或边的条数。

求二叉树的最大深度，我们可以用层序遍历的方法来进行depth++，这是迭代的方法

```c++
class Solution {
public:
    int maxDepth(TreeNode* root) {
        queue<TreeNode*> que;
        if(root == nullptr){
            return 0;
        }
        int depth = 0;
        que.push(root);
        while(!que.empty()){
            int size = que.size();
            depth++;
            for(int i = 0;i < size;i++){
                TreeNode* node = que.front();
                que.pop();
                if(node->left) que.push(node->left);
                if(node->right) que.push(node->right);
            }
        }
        return depth;
    }
};
```

也可以用递归的方法计算最大深度（后序遍历）

```c++
class Solution {
public:
    int travel(TreeNode* root){
        if(root == nullptr){
            return 0;
        }
        int leftDepth = travel(root->left);
        int rightDepth = travel(root->right);
        return 1+max(leftDepth, rightDepth);
    }
    int maxDepth(TreeNode* root) {
        if(root == nullptr){
            return 0;
        }
        
        return travel(root);
        
    }
};
```

二叉树的最小深度呢？**根节点到叶子节点**的最短简单路径上的节点数目或边的条数。

我们可以层序遍历，在找到第一个叶子结点的时候返回深度。

```c++
class Solution {
public:
    int minDepth(TreeNode* root) {
        queue<TreeNode*> que;
        if(root == nullptr){
            return 0;
        }
        int depth = 0;
        que.push(root);
        while(!que.empty()){
            int size = que.size();
            depth++;
            for(int i = 0;i < size;i++){
                TreeNode* node = que.front();
                que.pop();
                if(node->left) que.push(node->left);
                if(node->right) que.push(node->right);
                if(node->left == nullptr && node->right == nullptr){
                    return depth;
                }
            }
        }
        return depth;
    }
};
```

也可以利用递归，但是需要注意的是，并不是简单的` 1+min(leftDepth, rightDepth)`，需要考虑到的是，当节点 N 只有一侧有孩子时，并不能直接` 1+min(leftDepth, rightDepth)`，这时这个 节点 N 并不是最小深度的叶子节点。所以代码如下：

```c++
class Solution {
public:
    int getDepth(TreeNode* node) {
        if (node == NULL) return 0;
        int leftDepth = getDepth(node->left);           // 左
        int rightDepth = getDepth(node->right);         // 右
                                                        // 中
        // 当一个左子树为空，右不为空，这时并不是最低点
        if (node->left == NULL && node->right != NULL) { 
            return 1 + rightDepth;
        }   
        // 当一个右子树为空，左不为空，这时并不是最低点
        if (node->left != NULL && node->right == NULL) { 
            return 1 + leftDepth;
        }
        int result = 1 + min(leftDepth, rightDepth);
        return result;
    }

    int minDepth(TreeNode* root) {
        return getDepth(root);
    }
};
```



## 完全二叉树的节点个数

当然可以通过层序遍历或者后序遍历的方式去计算二叉树的节点个数，但是考虑到完全二叉树的特殊性质（除了最底层节点可能没填满外，其余每层节点数都达到最大值，*并且最下面一层的节点都集中在**该层最左边的**若干位置*）我们可以进一步优化。

```c++
class Solution {
public:
    // 确定返回量和传入参数
    int travel(TreeNode* cur){
        // 确定递归的终止条件
        if(cur == nullptr){
            return 0;
        }
        int leftDepth = 1, rightDepth = 1;
        TreeNode* left = cur->left;
        TreeNode* right = cur->right;
        // 计算靠左靠右遍历的树深度，判断是不是满二叉树
        while(left){
            left = left->left;
            leftDepth++;
        }
        while(right){
            right = right->right;
            rightDepth++;
        }
        
        if(leftDepth == rightDepth){
            cout << cur->val << ' ' << leftDepth << ' ' << rightDepth << endl;
            // 返回的是这个子树的节点，所以指数部分不是(leftDepth)
            return (2 << (leftDepth-1)) - 1;
        }
        // 单层的递归逻辑
        int leftCount = travel(cur->left);
        int rightCount = travel(cur->right);
        return 1+leftCount+rightCount;
    }
    
    int countNodes(TreeNode* root) {
		if(root == nullptr){
            return 0;
        }
        return travel(root);
    }
};
```

## 平衡二叉树 （双递归）

判断平衡二叉树，我这里自己写的双递归的解法，虽然AC但是运行效率较低。

```c++
class Solution {
public:
    int getHeight(TreeNode* cur){
        if(cur == nullptr){
            return 0;
        }

        int leftHeight = getHeight(cur->left);
        int rightHeight = getHeight(cur->right);
        return 1+max(leftHeight,rightHeight);
    }
    bool isBalancedFunction(TreeNode* cur){
        if(cur == nullptr){
            return true;
        }

        int leftHeight = getHeight(cur->left);        
        int rightHeight = getHeight(cur->right);
        bool leftflag = isBalancedFunction(cur->left);
        bool rightflag = isBalancedFunction(cur->right);
        return abs(leftHeight-rightHeight) <= 1 && leftflag && rightflag;
        
    }

    bool isBalanced(TreeNode* root) {
        return isBalancedFunction(root);

    }
};
```

将两个递归联合在一起，用-1作为一个标志位，运行效率有所提高。

```c++
class Solution {
public:
    int getHeight(TreeNode* cur){
        if(cur == nullptr){
            return 0;
        }

        int leftHeight = getHeight(cur->left);
        if(leftHeight == -1) return -1;
        int rightHeight = getHeight(cur->right);
        if(rightHeight == -1) return -1;

        if(abs(leftHeight-rightHeight)>1){
            return -1;
        }
        else{
            return 1+max(leftHeight,rightHeight);
        }
        
    }

    bool isBalanced(TreeNode* root) {
        return getHeight(root) == -1 ? false : true ;

    }
};
```

## 二叉树的所有路径 （回溯+递归）

```c++
class Solution {
public:
//前序遍历
    void dfs(TreeNode* cur, vector<int>& path, vector<string>& res){
        path.push_back(cur->val);

        if(cur->left == nullptr && cur->right == nullptr){
            string sPath;
            for(int i = 0; i < path.size() - 1; i++){
                sPath += to_string(path[i]);
                sPath += "->";
            }
            sPath += to_string(path[path.size() - 1]);
            res.push_back(sPath);
        }
        
        if(cur->left){            
            dfs(cur->left, path, res);
            path.pop_back();
        }
        if(cur->right){
            dfs(cur->right, path, res);
            path.pop_back();
        }
    }
    vector<string> binaryTreePaths(TreeNode* root) {
        vector<string> res;
        vector<int> path;
        if(root == nullptr){
            return res;
        }
        dfs(root, path, res);
        return res;
    }
};
```

## 左叶子之和

```c++
class Solution {
public:
// 先收集左叶子
    void travel(TreeNode* cur, vector<int>& leftLeaves){
        if(cur == nullptr){
            return;
        }

        travel(cur->left, leftLeaves);
        travel(cur->right, leftLeaves);
        if(cur->left != nullptr && cur->left->left == nullptr && cur->left->right == nullptr){
            leftLeaves.push_back(cur->left->val);
        }

    }
    int sumOfLeftLeaves(TreeNode* root) {
        vector<int> leftLeaves;
        int sum = 0;
        if(root == nullptr){
            return 0;
        }
        travel(root, leftLeaves);
        for(auto val:leftLeaves){
            sum += val;
        }
        return sum;
    }
};

#############进一步优化###################
class Solution {
public:
    //用一个变量直接去求和
    void travel(TreeNode* cur, int* sum){
        if(cur == nullptr){
            return;
        }

        travel(cur->left, sum);
        travel(cur->right, sum);
        if(cur->left != nullptr && cur->left->left == nullptr && cur->left->right == nullptr){
            *sum += cur->left->val;
        }

    }
    int sumOfLeftLeaves(TreeNode* root) {
        vector<int> leftLeaves;
        int sum = 0;
        if(root == nullptr){
            return 0;
        }
        travel(root, &sum);
       
        return sum;
    }
};
```

## [找树左下角的值](https://leetcode.cn/problems/find-bottom-left-tree-value/)

层序遍历找到左视图，然后返回左视图的最后一个元素。

vector中的pop_back()只是删除最后一个元素，并没有返回值。

```c++
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        queue<TreeNode*> que;
        
        que.push(root);
        vector<int> leftValue;
        while(!que.empty()){
            int size = que.size();
            for(int i = 0;i < size;i++){
                TreeNode* node = que.front();
                que.pop();
                if(node->left) que.push(node->left);
                if(node->right) que.push(node->right);
                if(i == 0){
                    leftValue.push_back(node->val);
                }
            }
        }
        return leftValue[leftValue.size() - 1];
    }
};
```

## [路径总和](https://leetcode.cn/problems/path-sum/)

```c++
class Solution {
public:
    int sumVector(vector<int> path){
        int summ = 0;
        for(auto p:path){
            summ += p;
        }
        return summ;
    }
    void dfs(TreeNode* cur, vector<int>& path, vector<int>& pathSum){
        path.push_back(cur->val);
        if(cur->left == nullptr && cur->right == nullptr){ //
            //cout <<  sumVector(path) << endl;
            pathSum.push_back(sumVector(path));
            return;
        }
        
      
        if(cur->left){
            dfs(cur->left, path, pathSum);
            path.pop_back();
        }
        if(cur->right){
            dfs(cur->right, path, pathSum);
            path.pop_back();
        }

        

    }

    bool hasPathSum(TreeNode* root, int targetSum) {
        vector<int> path;
        vector<int> pathSum;
        if(root == nullptr){
            return false;
        }
        dfs(root, path, pathSum);
       
        bool res = find(pathSum.begin(), pathSum.end(), targetSum) != pathSum.end() ? true:false;
        return res;
    }
};
```

## [路径总和 II](https://leetcode.cn/problems/path-sum-ii/)

```c++
class Solution {
public:
    int sumVector(vector<int> path){
        int summ = 0;
        for(auto p:path){
            summ += p;
        }
        return summ;
    }
    void dfs(TreeNode* cur, vector<vector<int>>& pathSet, vector<int>& path){
        path.push_back(cur->val);
        if(cur->left == nullptr && cur->right == nullptr){
            pathSet.push_back(path);
            return;
        }

        if(cur->left){
            dfs(cur->left, pathSet, path);
            path.pop_back();
        }
        if(cur->right){
            dfs(cur->right, pathSet, path);
            path.pop_back();
        }
    }
    vector<vector<int>> pathSum(TreeNode* root, int targetSum) {
        vector<vector<int>> pathSet;
        vector<int> path;
        vector<vector<int>> res;
        if(root == nullptr){
            return res;
        }
        dfs(root, pathSet, path);
        for(auto p:pathSet){
            if(sumVector(p) == targetSum){
                res.push_back(p);
            }
        }
        return res;

    }
};
```

## [从中序与后序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

**中序数组大小一定是和后序数组的大小相同的（这是必然），这也是用以切割后序数组方法。**

```c++
class Solution {
public:
    TreeNode* build(vector<int>& inorder, vector<int>& postorder){
    	//空节点
        if(postorder.size() == 0){
            return nullptr;
        }
        //叶子节点
        TreeNode* root = new TreeNode(postorder[postorder.size() - 1]);
        if(postorder.size() == 1){
            return root;
        }
		//分割中序
        int index_temp = 0; 
        while(inorder[index_temp] != root->val){
            index_temp++;
        } 
        vector<int> leftInorder(inorder.begin(), inorder.begin() + index_temp);
        vector<int> rightInorder(inorder.begin() + index_temp + 1, inorder.end());
		//分割后序
        vector<int> leftPosetorder(postorder.begin(), postorder.begin() + leftInorder.size());
        vector<int> rightPosetorder(postorder.begin() + leftInorder.size(), postorder.end() - 1);// 左闭右开

        root->left = build(leftInorder, leftPosetorder);
        root->right = build(rightInorder, rightPosetorder);

        return root;
        
    }
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        return build(inorder, postorder);
    }
};
```

## [从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

```c++
class Solution {
public:
        TreeNode* build(vector<int>& preorder, vector<int>& inorder) {
        if(preorder.size()==0){
            return nullptr;
        }
        int rootVal = preorder[0];
        TreeNode* root = new TreeNode(rootVal);
        if(preorder.size() == 1){
            return root;
        }

        int index = 0;
        while(inorder[index] != rootVal){
            index++;
        }
        vector<int> leftInorder(inorder.begin(), inorder.begin() + index);
        vector<int> rightInorder(inorder.begin() + index + 1, inorder.end());

        vector<int> leftPreorder(preorder.begin() + 1, preorder.begin() + 1 + leftInorder.size());
        vector<int> rightPreorder(preorder.begin() + 1 + leftInorder.size(), preorder.end());

        root->left = buildTree(leftPreorder, leftInorder);
        root->right = buildTree(rightPreorder, rightInorder);

        return root;
    }
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        return build(preorder, inorder);
    }
};
```

## [最大二叉树](https://leetcode.cn/problems/maximum-binary-tree/)

```c++
class Solution {
public:
    int max(vector<int>& nums){
        int res = INT_MIN;
        for(auto num:nums){
            if(res < num){
                res = num;
            }
        }
        return res;
    }
    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
		//结束条件
        //  如果空数组，返回空指针
        if(nums.size() == 0){
            return nullptr;
        }
        int max_val = max(nums);
        //  如果只有一个树，返回该节点
        TreeNode* root = new TreeNode(max_val);
        if(nums.size() == 1){
            return root;
        }
        
        //单层循环的逻辑
        //	寻找最大值索引
        int max_index = 0;
        while(nums[max_index] != max_val){
            max_index++;
        }
        //	拆分左右数组
        vector<int> leftNums(nums.begin(), nums.begin()+max_index);
        vector<int> rightNums(nums.begin()+max_index+1, nums.end());
        //递归
        root->left = constructMaximumBinaryTree(leftNums);
        root->right = constructMaximumBinaryTree(rightNums);

        return root;

    }
};
```

## [合并二叉树](https://leetcode.cn/problems/merge-two-binary-trees/)

```c++
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
    	//结束条件这么写其实没问题，可以画一下真值表确定
        if(root1 == nullptr) return root2;
        if(root2 == nullptr) return root1;

        TreeNode* root = new TreeNode(root1->val + root2->val);

        root->left = mergeTrees(root1->left, root2->left);
        root->right = mergeTrees(root1->right, root2->right);
        return root;

    }
};
```

## [二叉搜索树中的搜索](https://leetcode.cn/problems/search-in-a-binary-search-tree/)

```c++
class Solution {
public:
    TreeNode* travel(TreeNode* cur, int val){
        if(cur == nullptr){
            return nullptr;
        }
        if(cur->val == val){
            return cur;
        }
        TreeNode* res = nullptr;
        if(cur->val > val){
            res = travel(cur->left, val);
        }
        if(cur->val < val){
            res = travel(cur->right, val);
        }
        return res;
    }
    TreeNode* searchBST(TreeNode* root, int val) {
        return travel(root,val);

    }
};
```

## [剑指 Offer II 053. 二叉搜索树中的中序后继](https://leetcode.cn/problems/P5rCT8/)

先中序遍历，记录节点，然后返回下一个节点

```c++
class Solution {
public:
    void dfs(TreeNode* cur, vector<TreeNode*>& midTravel){
        
        if(cur == nullptr){
            return ;
        }
        dfs(cur->left, midTravel);
        midTravel.push_back(cur);        
        dfs(cur->right,midTravel);
    }

    TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {
        vector<TreeNode*> midTravel;
        dfs(root, midTravel);  
        for(int i = 0; i < midTravel.size();i++){
            if(midTravel[i] == p){
                return i == midTravel.size() - 1? nullptr : midTravel[i+1];
            }
        }
        return nullptr;      
    }
};
```

**如何通过中序遍历递归，并仅仅来维护中序遍历的前一个节点呢？**没能写好。

## [验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/)

首先的想法是通过递归来做，但是递归结束条件不好写。不确定应该在什么时候返回：如果在叶子节点返回，那返回值是ture还是false？对一个叶子没办法判断吧；如果在倒数第二层返回，那么如何判断倒数第二层呢？

最终的方法还是利用性质 **二叉搜索树的中序遍历单调递增，且没有重复**

## [二叉搜索树中的众数](https://leetcode.cn/problems/find-mode-in-binary-search-tree/)

最简单的方法是遍历一遍，然后用哈希表记录每一个元素出现的次数，得到出现频次最大的数。

注意：按照map的value排序，可以先转化成vector<pair<int,int>>，再调用sort排序。**sort()函数的第三个参数自定义排序规则，要将其命名为静态成员变量**

```c++
class Solution {
public:
    // 要注意，sort()函数的第三个参数自定义排序规则，要将其命名为静态成员变量
    static bool Mycmp (pair<int, int> p1, pair<int, int> p2){
        return p1.second > p2.second;
    }
    void dfs(TreeNode* cur, unordered_map<int,int>& umap){
        if(cur == nullptr){
            return;
        }
        
        if(umap.find(cur->val) == umap.end()){
            umap.insert(pair<int,int>{cur->val, 1});
        }else{
            umap[cur->val]++;
        }
        dfs(cur->left, umap);
        dfs(cur->right, umap);
    }
    vector<int> findMode(TreeNode* root) {        
        unordered_map<int,int> umap;
        dfs(root, umap);
        // 按照map的value排序，可以先转化成vector<pair<int,int>>，再调用sort排序。
        vector<pair<int,int>> vec_pair;
        for(unordered_map<int,int>::iterator it = umap.begin(); it != umap.end(); it++){
            vec_pair.push_back(pair<int,int>{it->first, it->second});
        }
        sort(vec_pair.begin(), vec_pair.end(),Mycmp);
        int max_count = vec_pair[0].second;
        vector<int> res;
        for(auto p:vec_pair){
            if(p.second == max_count){
                res.push_back(p.first);
            }
        }
        return res;
    }
};
```

## [二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)

二叉树回溯的话，只能用后序遍历。

寻找方法就是 在左子树里找 p或q，找到了就返回 p或q；在右子树里找 p或q，找到了就返回 p或q，没有的话就返回NULL；**如果，左右子树返回值都不是 NULL，就说明这个节点就是最近的公共祖先了，就继续向上返回 该节点。**

后序遍历，前面提到的 找到了就返回 p或q，也就是返回 cur；找到最近公共祖先就返回公共祖先，也是返回cur。(cur也可以是递归的root)

那么代码就这么写

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
		if(root == p || root == q || root == NULL){
            return root;
        }        
        TreeNode* left =  lowestCommonAncestor(root->left, p,q);
        TreeNode* right =  lowestCommonAncestor(root->right, p,q);
        // 这部分就是向上返回的过程
        if(left == NULL && right != NULL){
            return right;
        }
        else if(left != NULL && right == NULL){
            return left;
        }
        else if(left == NULL && right == NULL){
            return NULL;
        }
        else{
            return root;        
        }
        
    }
};
```

## [二叉搜索树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/)

又来到了二叉搜索树，那就需要考虑应用BST的性质：左小；右大；没重复；中序遍历有序。

有用的性质应该是左小右大，来减少搜索的次数。

所以代码这么写

```c++
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
		if(root == p || root == q || root == NULL){
            return root;
        }
        TreeNode* left = NULL;
        TreeNode* right = NULL;
        if(root->val > p->val && root->val > q->val){
            left =  lowestCommonAncestor(root->left, p,q);
            
        }else if(root->val < p->val && root->val < q->val){
            right =  lowestCommonAncestor(root->right, p,q);
            left = NULL;
        }else{
            left =  lowestCommonAncestor(root->left, p,q);
            right =  lowestCommonAncestor(root->right, p,q);
        }
        
        // 这部分就是向上返回的过程
        if(left == NULL && right != NULL){
            return right;
        }
        else if(left != NULL && right == NULL){
            return left;
        }
        else if(left == NULL && right == NULL){
            return NULL;
        }
        else{
            return root;        
        }
        
    }
};
```

## [二叉搜索树中的插入操作](https://leetcode.cn/problems/insert-into-a-binary-search-tree/)

首先的思路是直接用val一个个比较，直到找到合适的叶子节点，把该值插入。写完代码发现时间超出限制。

```c++
//时间超出限制
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if(root == nullptr){
            return root;
        }
        TreeNode* cur;
        cur = root;
        while(cur->left != nullptr || cur->right != nullptr){
            if(cur->left && cur->val > val){
                cur = cur->left;
            }else if(cur->right && cur->val < val){
                cur = cur->right;
            }
        }
        if(cur->val > val){
            cur->left = new TreeNode(val);
        }else{
            cur->right = new TreeNode(val);
        }
        return root;
    }
};
```

上面的代码问题是**没有考虑到val该插入的地方是根节点**的情况。更改代码如下：

```c++
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if(root == nullptr){
            TreeNode* root = new TreeNode(val);
            return root;
        }
        TreeNode* cur;
        cur = root;
        if(root->val > val && root->left == nullptr){
            TreeNode* temp = root;
            TreeNode* root = new TreeNode(val);
            root->right = temp;
            return root;
        }
        if(root->val < val && root->right == nullptr){
            TreeNode* temp = root;
            TreeNode* root = new TreeNode(val);
            root->left = temp;
            return root;
        }
        while(cur->left != nullptr || cur->right != nullptr){
            if(cur->left && cur->val > val){
                cur = cur->left;
            }else if(cur->right && cur->val < val){
                cur = cur->right;
            }
        }
        if(cur->val > val){
            cur->left = new TreeNode(val);
        }else{
            cur->right = new TreeNode(val);
        }
        return root;
    }
};
```

结果还是超出时间限制，决定查看题解。发现是 二叉搜索树遍历的问题。

```c++
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if(root == nullptr){
            TreeNode* root = new TreeNode(val);
            return root;
        }
        TreeNode* cur;
        cur = root;
        if(root->val > val && root->left == nullptr){
            TreeNode* temp = root;
            TreeNode* root = new TreeNode(val);
            root->right = temp;
            return root;
        }
        if(root->val < val && root->right == nullptr){
            TreeNode* temp = root;
            TreeNode* root = new TreeNode(val);
            root->left = temp;
            return root;
        }
        TreeNode* pre = nullptr;
        while(cur != nullptr){
            pre = cur;
            if(cur->val > val){
                cur = cur->left;
            }else if(cur->val < val){
                cur = cur->right;
            }
        }
        if(pre->val > val){
            pre->left = new TreeNode(val);
        }else{
            pre->right = new TreeNode(val);
        }
        return root;
    }
};
```

## [删除二叉搜索树中的节点](https://leetcode.cn/problems/delete-node-in-a-bst/)

想法是先找到要删除的节点cur，然后**把删除后的节点的左孩子插入到右孩子里**，返回一个新的node，然后按照cur与其父节点pre的关系，将node插入到pre的孩子。

```c++
class Solution {
public:
//cur的左子树一定是插入到cur的右子树中，最左侧的节点
    TreeNode* insertIntoBST(TreeNode* rootLeft, TreeNode* rootRight) {
        TreeNode* cur = rootRight;
        TreeNode* pre = nullptr;
        while(cur != nullptr){
            pre = cur;
            //cout << cur->val << endl;
            cur = cur->left;
        }
        pre->left = rootLeft;
        return rootRight;
    }

    TreeNode* deleteNode(TreeNode* root, int key) {
        if(root == nullptr){
            return root;
        }
        TreeNode* pre = nullptr;
        TreeNode* cur = root;
        while(cur != nullptr && cur->val != key){
            pre = cur;
            if(cur->val > key){
                cur = cur->left;
            }else{
                cur = cur->right;
            }
        }
        // 后面相当于把cur->right 的子树 插入到cur->left 的子树上
        TreeNode* temp = nullptr;
        if(cur != nullptr){
            //cout << cur->val;
            if(cur->left != nullptr && cur->right != nullptr){
                //cout << cur->left->val << ' '<< cur->right->val << endl;
                temp = insertIntoBST(cur->left, cur->right);
                
            }else if(cur->left != nullptr && cur->right == nullptr){
                temp = cur->left;
            }else if(cur->left == nullptr && cur->right != nullptr){
                temp = cur->right;
            }else{
                temp = nullptr;
            }
        }
        if(pre == nullptr){
            root = temp;
        }else{
            if(pre->left == cur){
                pre->left = temp;
            }else{
                pre->right = temp;
            }
        }
        return root;
    }
};
```



## [修剪二叉搜索树](https://leetcode.cn/problems/trim-a-binary-search-tree/)

首先的想法是找到**下边界**的节点，然后将**该节点的左侧部分**都删除。然后找到**上边界**的节点，然后将**该节点的右侧部分**都删除。

迭代的方法分三步：

1. 先把root节点定位到[low, high]区间内。
2. 再修建左子树，
3. 再修减右子树。

之所以可以分别进行2.3步，就是因为root已经在low, high区间内了，也就是说low在root的左子树，high在root的右子树。

```c++
class Solution {
public:
    TreeNode* trimBST(TreeNode* root, int low, int high) {
        if(root == nullptr){
            return root;
        }
		// 先把root节点定位到[low, high]区间内。
        while(root != nullptr && (root->val < low || root->val > high)){
            if(root->val < low){
                root = root->right;
            }else{
                root = root->left;
            }
        }
		// 修左子树
        TreeNode* cur = root;
        while(cur != nullptr){
            while(cur->left && cur->left->val < low){
                cur->left = cur->left->right;
            }
            cur = cur->left;
        }
		// 修左子树
        cur = root;
        while(cur != nullptr){
            while(cur->right && cur->right->val > high){
                cur->right = cur->right->left;
            }
            cur = cur->right;
        }
        return root;
    }
};

```

## [将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)

该问题即为通过二叉搜索树的中序遍历构造一个二叉搜索树，且要求这棵树是一个**高度平衡的二叉树**。思路就是先找中间节点，分割左右，递归构造。

```c++
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        if(nums.size() == 0){
            return nullptr;
        }

        int index = nums.size() >> 1;
        TreeNode* root = new TreeNode(nums[index]);
        vector<int> numsLeft(nums.begin(), nums.begin() + index);
        vector<int> numsRight(nums.begin() + index + 1, nums.end());
        root->left = sortedArrayToBST(numsLeft);
        root->right = sortedArrayToBST(numsRight);

        return root;

    }
};
```

[有序链表转换二叉搜索树](https://leetcode.cn/problems/convert-sorted-list-to-binary-search-tree/)

这道题的思路就是利用快慢指针找到链表的中间节点，其他思路类似。

```c++
class Solution {
public:
    //快慢指针找到链表的中间节点
    ListNode* getMedian(ListNode* left, ListNode* right) {
        ListNode* fast = left;
        ListNode* slow = left;
        while (fast != right && fast->next != right) {
            fast = fast->next;
            fast = fast->next;
            slow = slow->next;
        }
        return slow;
    }

    TreeNode* buildTree(ListNode* left, ListNode* right) {
        if (left == right) {
            return nullptr;
        }
        ListNode* mid = getMedian(left, right);
        TreeNode* root = new TreeNode(mid->val);
        root->left = buildTree(left, mid);
        root->right = buildTree(mid->next, right);
        return root;
    }

    TreeNode* sortedListToBST(ListNode* head) {
        return buildTree(head, nullptr);
    }
};
```

# 回溯算法

## 基础知识

递归逻辑的下面一般是回溯的逻辑。

回溯的效率：**因为回溯的本质是穷举，穷举所有可能，然后选出我们想要的答案**，如果想让回溯法高效一些，可以加一些剪枝的操作，但也改不了回溯法就是穷举的本质。

回溯法，一般可以解决如下几种问题：

- 组合问题：N个数里面按一定规则找出k个数的集合
- 切割问题：一个字符串按一定规则有几种切割方式
- 子集问题：一个N个数的集合里有多少符合条件的子集
- 排列问题：N个数按一定规则全排列，有几种排列方式
- 棋盘问题：N皇后，解数独等等

**回溯法解决的问题都可以抽象为树形结构**

##  回溯法模板

- 回溯函数模板返回值以及参数

  回溯法的参数一般不容易一次性确定下来，所以需要先写逻辑然后确定参数。

- 回溯函数终止条件

  ```
  if (终止条件) {
      存放结果;
      return;
  }
  ```

- 回溯搜索的遍历过程

  for循环：横向遍历

  递归：纵向遍历

  ```
  for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
      处理节点/元素;
      backtracking(路径，选择列表); // 递归
      回溯，撤销处理结果
  }
  ```

```
void backtracking(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        处理节点;
        backtracking(路径，选择列表); // 递归
        回溯，撤销处理结果
    }
}

```

## 组合问题

### [组合](https://leetcode.cn/problems/combinations/)

给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。

回溯的宽度 就是 n，回溯的深度 是 k，当path的size为k时，就返回。注意`startIndex`的作用。

```c++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> path;
    void backTrack(int n, int k, int startIndex){
        if(path.size() == k){
            res.push_back(path);
            return;
        }

        for(int i = startIndex; i < n;i++){
            path.push_back(i+1);
            backTrack(n, k, i+1);
            path.pop_back();
        }
    }
    vector<vector<int>> combine(int n, int k) {
        backTrack(n, k, 0);
        return res;
    }
};
```

进一步优化的剪枝思想是如果从startIndex开始遍历的剩余节点已经不够结果要求的长度了，那么就不要再进行后续的宽度搜索了。

剩余的结果要求的长度是`k-path.size()`；那么遍历就要满足`i < or <= n-(k-path.size())`，举个例子，目前的情况是`path.size()=0, k=3, n=4`，`n-(k-path.size()) = 1`，而此时 i 可以等于 1，从索引 1 开始遍历的后三个数，是可以满足条件的。所以 `i  <= n-(k-path.size())`。

改进后：

```c++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> path;
    void backTrack(int n, int k, int startIndex){
        if(path.size() == k){
            res.push_back(path);
            return;
        }

        for(int i = startIndex; i < n-(k-path.size()) +1;i++){
            path.push_back(i+1);
            backTrack(n, k, i+1);
            path.pop_back();
        }
    }
    vector<vector<int>> combine(int n, int k) {
        backTrack(n, k, 0);
        return res;
    }
};
```

### [组合总和](https://leetcode.cn/problems/combination-sum/)

这一道题自己的想法有两点错误：

1. 没有判断 `*sum > target`的时候剪枝，这个时候会溢出报错，如果少了这个条件话，那么如果出现大于target的情况之后就不会返回，并且以后也不会返回，那么就会死循环，溢出。
2. 没有引入 startIndex 这个参数，因为我想的是可以重复出现，就可以每次都从i=0开始，但是如果是这样的话，结果会重复输出同一个target的不同的排列情况。与题意不符。

```c++
class Solution {
public:
    vector<int> path;
    vector<vector<int>> res;
    void backTrack(vector<int>& candidates, int target, int* summ, int startIndex){
        if(*summ > target){
            return;
        }
        
        if(target == *summ){
            res.push_back(path);
            return;
        }
		
        for(int i = startIndex; i < candidates.size(); i++){
            path.push_back(candidates[i]);
            (*summ) = (*summ) + candidates[i];
            backTrack(candidates, target, summ, i);
            path.pop_back();
            (*summ) = (*summ) - candidates[i];
        }
    }

    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        int summ = 0;
        backTrack(candidates, target, &summ, 0);
        return res;
    }
};
```

### [ 组合总和 II](https://leetcode.cn/problems/combination-sum-ii/)

需要去重？去重方法：横向搜索的时候不能有重复。思考了一下可不可以在单层递归逻辑那里实现去重，但发现这跟二叉树的层序遍历还有一些区别，没办法完全区分开横向搜索和纵向搜索。

**所以我们要去重的是同一树层上的“使用过”，同一树枝上的都是一个组合里的元素，不用去重**。如何判断同一树层上元素（相同的元素）是否使用过了呢。

首先将数组进行排序，之后通过构造used数组来判断，

**如果`candidates[i] == candidates[i - 1]` 并且 `used[i - 1] == false`，就说明：前一个树枝，使用了candidates[i - 1]，也就是说同一树层使用过candidates[i - 1]**。

**如果`candidates[i] == candidates[i - 1]` 并且 `used[i - 1] == true`，就说明：当前树枝，使用了candidates[i - 1]**

为什么可以这么表示呢？看代码注释

```c++
class Solution {
public:
    vector<int> path;
    vector<vector<int>> res;
    
    void backTrack(vector<int>& candidates, int target, int* summ, int startIndex, vector<bool>& used){
    if(target < *summ){
        return;
    }
    
    if(target == *summ){
        res.push_back(path);
        return;
    }
    
    for(int i = startIndex; i < candidates.size(); i++){
        if(i > 0 && candidates[i-1] == candidates[i] && used[i-1] == false){
            continue;
        }
        
        path.push_back(candidates[i]);
        used[i] = true;//因为在这里回溯，把下面递归中前一位的标志位设为了true
        *summ += candidates[i];
        backTrack(candidates, target, summ, i+1, used);
        path.pop_back();
        *summ -= candidates[i];
        used[i] = false; //因为在这里回溯，把前一位的标志位设为了false
    }
} 

    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        int summ = 0;
        sort(candidates.begin(), candidates.end());
        vector<bool> used(candidates.size(), false);
        backTrack(candidates, target, &summ, 0, used);
        return res;
    }
};
```

### [组合总和 III](https://leetcode.cn/problems/combination-sum-iii/)

```c++
class Solution {
public:
    vector<int> path;
    vector<vector<int>> res;
    void backTrack(vector<int>& candidates, int k, int n, int satrtIndex, int* summ){
        // 结束条件1
        if(*summ > n || path.size() > k){
            return;
        }
        // 结束条件2
        if(*summ == n && path.size() == k){
            res.push_back(path);
            return;
        }
        //						剪枝
        for(int i = satrtIndex; i <= candidates.size() - (k-path.size()); i++){
            path.push_back(candidates[i]);
            *summ += candidates[i];
            backTrack(candidates, k, n, i+1, summ);
            *summ -= candidates[i];
            path.pop_back();
        }
    }

    vector<vector<int>> combinationSum3(int k, int n) {
        vector<int> candidates;
        for(int i = 1; i <= 9;i++){
            candidates.push_back(i);
        }        
        int summ = 0;
        backTrack(candidates, k, n, 0, &summ);
        return res;
    }
};
```

### [组合总和 Ⅳ](https://leetcode.cn/problems/combination-sum-iv/)

这个题会考虑到不同的排列组合（可以重复使用，令`startIndex=0`即可），**如果用回溯的话会超出时间限制**。回溯代码如下：

```c++
class Solution {
public:
    int res = 0;
    void backTrack(vector<int>& nums, int target, int* summ, int startIndex){
        if(*summ > target){
            return;
        }
        if(*summ == target){
            res++;
            return;
        }

        for(int i = 0; i < nums.size(); i++){
            *summ += nums[i];
            backTrack(nums, target, summ, 0);
            *summ -= nums[i];
        }
    }
    int combinationSum4(vector<int>& nums, int target) {
        int summ = 0;
        backTrack(nums, target, &summ, 0);
        return res;
    }
};
```

### [电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

**为什么` string.insert()`这个函数调用不了？**

这一题自己的想法在于审题错误，在回溯的树中，树的每一层横向搜索是不一样的。

```c++
class Solution {
private:
    unordered_map<char, string> umap;
public:
    vector<string> res;
    string path;

    void backTrack(string& digits, int k, int startIndex){
        if(path.size() > k){
            return;
        }
        if(path.size() == k){
            res.push_back(path);
            return;
        }
        string chars = umap[digits[startIndex]]; // 首先找到要横向遍历的字符串

        for(int i = 0;i < chars.size(); i++){
            path.push_back(chars[i]);
            backTrack(digits, k, startIndex+1); // index+1 找到下一个按键对应的字符串
            path.pop_back(); 
        }
    }
    vector<string> letterCombinations(string digits) {
        if(digits.size() == 0){
            return {};
        }
        umap.insert(pair<char, string>{'2', "abc"});
        umap.insert(pair<char, string>{'3', "def"});
        umap.insert(pair<char, string>{'4', "ghi"});
        umap.insert(pair<char, string>{'5', "jkl"});
        umap.insert(pair<char, string>{'6', "mno"});
        umap.insert(pair<char, string>{'7', "pqrs"});
        umap.insert(pair<char, string>{'8', "tuv"});
        umap.insert(pair<char, string>{'9', "wxyz"});
        
        int k = digits.size();
        backTrack(digits, k, 0);
        return res;
    }
};
```

## 切割问题

切割问题和组合问题类似，组合问题是选择元素，切割问题是在元素间==“添加分隔符”==。

代码里怎么表示 ==分隔符== 呢？答案是==stratIndex==

代码里怎么表示 ==“分割的子串”== 呢？这里说的是子串，其实说是==前缀==的话更合适。用==[startIndex, i]== 来表示。

### [分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)

判断回文串 用双指针

```c++
class Solution {
public:
    vector<vector<string>> res;
    vector<string> path;
    bool isHuiwen(string& s,int left, int right){
        if(s.size() == 1){
            return true;
        }

        
        while(left < right){
            if(s[left] != s[right]){
                return false;
            }
            left++;right--;
        }
        return true;
    }

    void backTrack(string& s, int startIndex){
        if(startIndex >= s.size()){
            res.push_back(path);
            return;
        }

        for(int i = startIndex; i < s.size(); i++){
            if(isHuiwen(s, startIndex, i)){
                path.push_back(s.substr(startIndex, i-startIndex+1));
            }else{
                continue;
            }
            backTrack(s, i+1);
            path.pop_back();
        }

    }
    vector<vector<string>> partition(string s) {
        backTrack(s, 0);
        return res;
        
    }
};
```

### [复原 IP 地址](https://leetcode.cn/problems/restore-ip-addresses/)

要考虑的是结束条件：一定为4个字段，并且是全部的字符都用上了。

考虑不合法字段的情况：

```c++
bool isSubIP(string& s, int left, int right){
    string temp = s.substr(left, right-left+1);
    if((temp[0] == '0' && temp.size() != 1) || temp.size() > 3 || stoi(temp) > 255){
        return false;
    }else{
        return true;
    }
}
```

全部代码

```c++
class Solution {
public:
    vector<string> res; // 所有结果
    vector<string> IP;// 每一个结果的不同段
    bool isSubIP(string& s, int left, int right){
        string temp = s.substr(left, right-left+1);
        if((temp[0] == '0' && temp.size() != 1) || temp.size() > 3 || stoi(temp) > 255){
            return false;
        }else{
            return true;
        }
    }
    bool isAllUsed(vector<string>& IP, string& s){
        string sIP;
        for(auto ip:IP){
            sIP += ip;
        }
        return sIP.size() == s.size() ? true:false;
    }

    void backTrack(string& s, int startIndex, int pointNum){
        if(pointNum == 4 && isAllUsed(IP, s)){
            // 将 vector<string> IP 转化成 string IP
            string sIP;
            for(int i = 0; i < IP.size() - 1; i++){
                sIP += IP[i];
                sIP += ".";
            }
            sIP += IP[IP.size() - 1];
            res.push_back(sIP);
            return;
        }

        for(int i = startIndex; i < s.size(); i++){
            // 判断该字段[startIndex, i]是否满足要求
            if(isSubIP(s, startIndex, i)){
                // 如果是一个合法的IP字段
                IP.push_back(s.substr(startIndex, i-startIndex+1));
                pointNum++;
            }else{
                // 否则，continue
                continue;
            }
            //递归
            backTrack(s, i+1, pointNum);
            //回溯
            IP.pop_back();
            pointNum--;
        }
        
    }
    vector<string> restoreIpAddresses(string s) {
        backTrack(s, 0, 0);
        return res;
    }
};
```

## 子集问题

### [子集](https://leetcode.cn/problems/subsets/)

```c++
class Solution {
public:
    vector<int> subset;
    vector<vector<int>> res;
    void backTrack(vector<int>& nums, int startIndex){
        //结束条件，注意不是 nums.zie()-1
        if(startIndex == nums.size()){

            return;
        }

        for(int i = startIndex; i < nums.size(); i++){
            subset.push_back(nums[i]);
            res.push_back(subset);
            backTrack(nums, i+1);
            subset.pop_back();
        }

    }
    vector<vector<int>> subsets(vector<int>& nums) {
        res.push_back(subset);
        backTrack(nums, 0);
        
        return res;
    }
};
```

### [子集 II](https://leetcode.cn/problems/subsets-ii/)

该题的主要考虑的是去重，结果中不能包括重复子集。那么去重的思路也就是横向搜索的时候，不要回溯元素相同的节点了，遇到该元素和前一个元素相同的情况，就直接把这一支剪掉即可，该方法首先需要对元素进行排序，维护used数组来判断相同元素是在同一层中被使用还是在同一分支上被使用。这个去重思路与 ==组合总和 II== 相似

```c++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> subset;
    void backTrack(vector<int>& nums, int startIndex, vector<bool>& used){
        // 结束条件
        if(startIndex >= nums.size()){
            return;
        }

        //
        for(int i = startIndex; i < nums.size(); i++){
            //先去重
            if(i > startIndex && nums[i-1] == nums[i] && used[i-1] == false){
                continue;
            }else{
                subset.push_back(nums[i]);
                res.push_back(subset);
                used[i] = true;
                backTrack(nums, i+1, used);
                subset.pop_back();
                used[i] = false;
            }
        }
    }
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<bool> used(nums.size(), false);
        res.push_back(subset);
        backTrack(nums, 0, used);
        return res;
    }
};
```

### [递增子序列](https://leetcode.cn/problems/non-decreasing-subsequences/)

这个题要注意：

保证递增的逻辑如下：

```c++
if(!subset.empty() && nums[i] < subset.back()){
    continue;
}
```

而去重的逻辑则不能用之前的used数组了，因为用used数组需要先对原来的used进行排序才行，但是这样就不满足题意了，所以这里的去重逻辑为

```c++
unordered_set<int> uset;
for(int i = startIndex; i < nums.size(); i++){
    // 去重
    if(uset.find(nums[i])  != uset.end() ){
        continue;
    }
    uset.insert(nums[i]);
    XXX       
}
```

总代码

```c++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> subset;
    void backTrack(vector<int>& nums, int startIndex){
        if(startIndex >= nums.size()){
            return;
        }        
        unordered_set<int> uset;
        for(int i = startIndex; i < nums.size(); i++){
            // 去重
            
            if(uset.find(nums[i])  != uset.end() ){
                continue;
            }
            uset.insert(nums[i]);
            // 保证递增
            if(!subset.empty() && nums[i] < subset.back()){
                continue;
            }
            subset.push_back(nums[i]);
            if(subset.size() > 1){
                res.push_back(subset);
            }
            backTrack(nums, i+1);
            subset.pop_back();            
        }
    }
    vector<vector<int>> findSubsequences(vector<int>& nums) {

        backTrack(nums, 0);
        return res;
    }
};
```

## 排列问题

### [全排列](https://leetcode.cn/problems/permutations/)

用used数组标记每一个数是否被用了，并且去掉重复使用同一个数

```c++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> path;
    
    void backTrack(vector<int>& nums, int startIndex, vector<bool>& used){
        
        if(path.size() == nums.size() ){
            
            res.push_back(path);
            return;
        }

        for(int i = 0; i < nums.size(); i++){
            if(used[i] == true) continue;
            path.push_back(nums[i]);
            used[i] = true;
            backTrack(nums, 0, used);
            path.pop_back();
            used[i] = false;
        }
    }
    vector<vector<int>> permute(vector<int>& nums) {
        vector<bool> used(nums.size(), false);
        backTrack(nums, 0, used);
        return res;
    }
};
```



### [全排列 II](https://leetcode.cn/problems/permutations-ii/)

```c++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> path;
    void backTrack(vector<int>& nums, vector<bool>& used){
        if(path.size() == nums.size()){
            res.push_back(path);
            return;
        }
        
        for(int i = 0;i < nums.size(); i++){
            if(i > 0 && nums[i-1] == nums[i] && used[i-1] == false){
                continue;
            }
            if(used[i] == true){
                continue;
            }
            path.push_back(nums[i]);
            used[i] = true;
            backTrack(nums, used);
            path.pop_back();
            used[i] = false;

        }
    }
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        sort(nums.begin(), nums.end()); // 排序
        vector<bool> used(nums.size(), false);
        backTrack(nums, used);
        return res;
    }
};
```

### [重新安排行程](https://leetcode.cn/problems/reconstruct-itinerary/)



### [N 皇后](https://leetcode.cn/problems/n-queens/)



### [解数独](https://leetcode.cn/problems/sudoku-solver/)



# 贪心算法

## 基础理论

贪心算法是从局部最优推导出全局最优的方法。

贪心算法一般分为如下四步：

- 将问题分解为若干个子问题
- 找出适合的贪心策略
- 求解每一个子问题的最优解
- 将局部最优解堆叠成全局最优解

## 例题

### [分发饼干](https://leetcode.cn/problems/assign-cookies/)

**这里的局部最优就是大饼干喂给胃口大的，充分利用饼干尺寸喂饱一个，全局最优就是喂饱尽可能多的小孩**。

```c++
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        sort(g.begin(), g.end());
        sort(s.begin(), s.end());

        int res = 0;
        for(int i = s.size()-1, j = g.size()-1; i >= 0 && j >= 0;){
            if(s[i] >= g[j]){
                res++;                
                i--;
                j--;
            }else{
                j--;
            }            
        }
        return res;

    }
};
```

### [摆动序列](https://leetcode.cn/problems/wiggle-subsequence/)

要找到一段序列中最大的摆动序列，其实就是**统计这个序列中的波峰和波谷的个数。**

```c++
class Solution {
public:
    int wiggleMaxLength(vector<int>& nums) {
        int res = 1;
        int preDiff = 0;
        int curDiff = 0;
        //             不判断最右侧了
        for(int i = 0;i < nums.size()-1;i++){
            curDiff = nums[i+1] - nums[i];  
            // 波谷                             波峰
            if((preDiff <= 0 && curDiff > 0) || (preDiff >= 0 && curDiff < 0)){
                res++;
                preDiff = curDiff;
            }
            
        }
        return res;
    }
};
```

### [最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。**子数组** 是数组中的一个连续部分。

这个用动态规划好像更能说得通，要注意**连续这一个特点**，所以如果dp[i-1]提供的作用是副作用，即`dp[i-1] < 0`，那就从当前位置`nums[i]`再开始。

```c++
class Solution {
public:
    
    int maxSubArray(vector<int>& nums) {
        
        vector<int> dp(nums.size(), 0);
        dp[0] = nums[0];
        int res = nums[0];
        for(int i = 1; i < nums.size(); i++){            
            dp[i] = max(nums[i], dp[i-1] + nums[i]);
            res = max(res, dp[i]);
        }

        return res;

    }
};
```

### [买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

要注意一定是先买入再卖出，直接求最值是不对的。这也是动态规划的思想，首先动态规划维护==[0,i-1]==这个子序列的最小值minPrice，代表最大利润的dp数组的转移方程则为

```c++
dp[i] = max(dp[i-1], nums[i] - minPrice);
```

```c++
class Solution {
public:
    void printVec(vector<int>& dp){
        for(auto d:dp){
            cout << d << " ";
        }
        cout << endl;
    }
    int maxProfit(vector<int>& prices) {
        vector<int> dp(prices.size(), 0);
        int minPrice = INT_MAX;
        for(int i = 1;i < prices.size();i++){
            minPrice = min(minPrice, prices[i-1]);
            dp[i] = max(dp[i-1], prices[i]-minPrice);
        }
        printVec(dp);
        return dp[dp.size()-1];
    }
};
```

### [买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

在每一天，你可以决定是否购买和/或出售股票。你在任何时候 **最多** 只能持有 **一股** 股票。你也可以先购买，然后在 **同一天** 出售。==可以在抛出之后，继续购买==，区别是可以重复购买。

陷入上面的循环购买的思想怪圈之后这个题就有些复杂了，代码随想录中的给的贪心方法是：==只收集每一天的正利润==，先计算每一天的利润，正利润求和。

```c++
class Solution {
public:
    void printVec(vector<int>& dp){
        for(auto d:dp){
            cout << d << " ";
        }
        cout << endl;
    }
    int maxProfit(vector<int>& prices) {
        vector<int> ProfitDay(prices.size()-1, 0);
        int res;
        for(int i = 0; i < ProfitDay.size();i++){
            ProfitDay[i] = prices[i+1]-prices[i];
            if(ProfitDay[i]>0) res += ProfitDay[i];
        }
        printVec(ProfitDay);
        return res;
    }
};
```

### [跳跃游戏](https://leetcode.cn/problems/jump-game/)

给定一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。数组中的每个元素代表你在该位置可以跳跃的最大长度。判断你是否能够到达最后一个下标。

不用考虑是跳几步，只考虑可能覆盖的最大范围。只要最大范围可以包括最后一个下标，那就返回true。

```cpp
class Solution {
public:   
    bool canJump(vector<int>& nums) {
        if(nums.size()==1) return true;
        int cover = 0;
        for(int i = 0; i <= cover; i++){
            cover = max(cover, i + nums[i]);
            cout << cover << endl;
            if(cover >= nums.size()-1) return true;
        }
        return false;
    }
};
```

### [跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/)

返回到达 `nums[n - 1]` 的最小跳跃次数。

==要想清楚什么时候步数才一定要加一呢？==**这里需要统计两个覆盖范围，当前这一步的最大覆盖和下一步最大覆盖**。如果移动下标达到了当前这一步的最大覆盖最远距离了，还没有到终点的话，那么就必须再走一步来增加覆盖范围，直到覆盖范围覆盖了终点。

```c++
class Solution {
public:   
    int jump(vector<int>& nums) {
        if(nums.size()==1) return 0;
        int nextCover = 0;
        int curCover = 0;
        int res = 0;
        for(int i = 0; i <= nums.size()-1; i++){
            nextCover = max(nextCover, i + nums[i]);
            // 如果 i 已经到了当前cover的最大索引
            if(i == curCover){
                // 如果当前cover没能到达终点
                if(curCover < nums.size()-1){
                    //就加一步
                    res++;
                    curCover = nextCover;
                    //if(nextCover >= nums.size()-1) break;
                }else{
                    break;
                }
            }            
        }
        return res;
    }
};
```

### [K 次取反后最大化的数组和](https://leetcode.cn/problems/maximize-sum-of-array-after-k-negations/)

给你一个整数数组 `nums` 和一个整数 `k` ，按以下方法修改该数组：

- 选择某个下标 `i` 并将 `nums[i]` 替换为 `-nums[i]` 。

重复这个过程恰好 `k` 次。可以多次选择同一个下标 `i` 。

以这种方式修改数组后，返回数组 **可能的最大和** 。

贪心，局部最优到全局最优

那这道题贪心的策略就是每一次选择的都是绝对值最大的负数(小于零的最小的数)，如果没有小于零的，那么就反转最小的非负整数。==总的来说就是反转最小的数。==

```c++
class Solution {
public:
    void reverseMin(vector<int>& nums){
        int min = INT_MAX;
        int index = 0;
        for(int i = 0;i < nums.size();i++){
            min = min > nums[i] ? nums[i] : min;
        }
        for(int i = 0;nums[i] != min;i++){
            index++;
        }
        nums[index] = -nums[index];
    }
    int largestSumAfterKNegations(vector<int>& nums, int k) {
        for(int j = 0;j < k;j++){
            reverseMin(nums);
        }
        int res = 0;
        for(auto n:nums){
            res += n;
        }
        return res;
    }
};
```

### [加油站](https://leetcode.cn/problems/gas-station/)

如果总油量大于总的消耗量，那么就说明每一步的剩余油量的和是大于0的，说明可以跑完一圈。

用curSum记录`[0,i]`的剩余油量的和，如果`curSum < 0`，则说明`[0,i]`都不可以作为起始点。此时应该认为可以从`start= i+1` 开始，并将curSum置0。如果遍历完整个数组之后，没有遇到`curSum < 0`的情况，又因为该题肯定存在解，那么就可以认为start为起点。如果遇到了`curSum < 0`，就重新更新start和curSum。

```c++
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int curSum = 0, totalSum = 0,start = 0;
        for(int i = 0;i < gas.size();i++){
            curSum += gas[i] - cost[i];
            totalSum += gas[i] - cost[i];
            if(curSum < 0){
                start = i+1;
                curSum = 0;
            }
        }
        if(totalSum < 0) return -1;
        return start;
    }
};
```



### [分发糖果](https://leetcode.cn/problems/candy/)

这里面需要先从左到右比较，如果右大于左，则糖果加一；否则，就发一个。

然后需要从左到右比较，如果左大于右，则要`res[i] = max(res[i+1] + 1, res[i])`，而不是简单加一；

比较左是否大于右时，要从右往左遍历，这样才会利用到上一次的比较结果。同理，比较右是否大于左时，要从左往右遍历。

```c++
class Solution {
public:
    int candy(vector<int>& ratings) {
        vector<int> res(ratings.size(), 1);
        // 从左到右
        for(int i = 1; i < ratings.size();i++){
            if(ratings[i] > ratings[i-1]){
                res[i] = res[i-1] + 1;
            }
        }

        // 从右到左
        for(int i = res.size()-2; i >= 0;i--){
            if(ratings[i+1] < ratings[i]){
                res[i] = max(res[i+1] + 1, res[i]);
            }
        }

        int resSum = 0;
        for(auto r:res){
            resSum += r;
        }
        return resSum;

    }
};
```

### [柠檬水找零](https://leetcode.cn/problems/lemonade-change/)

在给20找零时，优先消耗10美元+5美元

账单是20的情况，为什么要优先消耗一个10和一个5呢？

**因为美元10只能给账单20找零，而美元5可以给账单10和账单20找零，美元5更万能！**

```c++
class Solution {
public:
    bool cannot10(unordered_map<int, int>& umap){
        if(umap[5] > 0){
            umap[5]--;
            return false;
        }else{
            return true;
        }
    }
    bool cannot20(unordered_map<int, int>& umap){
        if(umap[5] >= 1 && umap[10] >= 1){
            umap[5]--;
            umap[10]--;
            return false;
        }
        else if(umap[5] >= 3){
            umap[5] -= 3;
            return false;
        }
        else{
            return true;
        }
    }
    bool lemonadeChange(vector<int>& bills) {
        unordered_map<int, int> umap;
        umap.insert(pair<int, int>{5,0});
        umap.insert(pair<int, int>{10,0});
        umap.insert(pair<int, int>{20,0});

        if(bills[0] != 5){
            return false;
        }
        for(int i = 0;i < bills.size();i++){
            
            if(bills[i] == 10 && cannot10(umap)){
                return false;
            }
            if(bills[i] == 20 && cannot20(umap)){
                return false;
            }
            umap[bills[i]]++;
        }
        return true;
    }
};
```

### [根据身高重建队列](https://leetcode.cn/problems/queue-reconstruction-by-height/)

遇到这种两个维度的，需要将两个维度分开考虑。首先按照身高进行排序，身高高的在前，如果身高相同，则第二个属性小的在前。然后按照身高从高到低的顺序，将每一个元素按第二个属性提示的位置插入到新的数组中。

```c++
class Solution {
public:
    static bool camp(vector<int>& a, vector<int>& b){
        if(a[0] == b[0]) return a[1] < b[1];
        return a[0] > b[0];
    }
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        sort(people.begin(), people.end(), camp);
        vector<vector<int>> queue;
        for(int i = 0;i < people.size();i++){
            queue.insert(queue.begin() + people[i][1], people[i]);
        }
        return queue;
    }
};
```

### [用最少数量的箭引爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)

```c++
class Solution {
public:
    static bool camp(vector<int>& a, vector<int>& b){
        return a[0] < b[0];
    }
    int findMinArrowShots(vector<vector<int>>& points) {
        if(points.size() <= 1) return points.size();
        int res = 0;
        sort(points.begin(), points.end(), camp);

        for(int i = 1;i < points.size();i++){
            // 第i个的起点已经大于第i-1个的终点了，就说明不在一堆，要放箭。
            if(points[i][0] > points[i-1][1]){
                res++;
            }else{
                // 把在一堆的气球的末尾统一
                points[i][1] = min(points[i-1][1], points[i][1]);
            }
        }
        ++res; //最后一箭
        return res;

    }
};
```

### [无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/)

这个题自己没有思路。

题解思路：按照**右边界排序**，就要**从左向右遍历**，因为右边界越小越好，只要右边界越小，留给下一个区间的空间就越大，所以从左向右遍历，优先选右边界小的。

按照**左边界排序**，就要**从右向左遍历**，因为左边界数值越大越好（越靠右），这样就给前一个区间的空间就越大，所以可以从右向左遍历。

找到非重叠区域的个数，总个数-非重叠区域的个数 == 结果。

```c++
class Solution {
public:
    // 按照右边界排序
    static bool cmp(vector<int>& a, vector<int>& b){
        return a[1] < b[1];
    }
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        // 按照右边界排序
        sort(intervals.begin(), intervals.end(), cmp);

        // 从左向右遍历 找到非重叠区域个数
        int nonOverNum = 0;
        for(int i = 1;i < intervals.size();i++){
            // i与i-1重叠
            if(intervals[i][0] < intervals[i-1][1]){
                intervals[i][1] = intervals[i-1][1];
            }else{
                nonOverNum++; // i与i-1不重叠 nonOverNum++
            }
        }
        nonOverNum++; // 考虑最后一个区间
        //cout << nonOverNum << endl;
        return intervals.size() - nonOverNum;
    }
};
```

### [划分字母区间](https://leetcode.cn/problems/partition-labels/)

把同一个字母的都圈在同一个区间里。

```c++
class Solution {
public:

    vector<int> partitionLabels(string s) {
        // 统计每个字母出现的最大下标
        int hash[26] = {0};
        for(int i = 0;i < s.size();i++){
            hash[s[i] - 'a'] = i;
        }
        
        // 如果 , 那么就可以分割
        vector<int> res;
        int left = 0;
        int right = 0;
        for(int i = 0;i < s.size();i++){
            // 确定右指针，这一点要理解一下
            right = max(right, hash[s[i] - 'a']);
            if(right == i){
                res.push_back(right - left + 1);
                left = i + 1;
            }
        }
        return res;
    }
};
```



### [合并区间](https://leetcode.cn/problems/merge-intervals/)

```c++
class Solution {
public:
    //按照起始位置排序
    static bool camp(vector<int>& a, vector<int>& b){
        return a[1] < b[1];
    }
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(), camp);

        vector<vector<int>> res;
        res.push_back(intervals.back());
        for(int i = intervals.size()-1;i >= 0;i--){
            // 是否有重叠
            if(res.back()[0] <= intervals[i][1]){
                res.back()[0] = min(intervals[i][0], res.back()[0]);
                //res.back()[1] = max(intervals[i][1], res.back()[1]);
            }else{
                res.push_back(intervals[i]);
            }
        }
        //res.push_back(intervals[intervals.size()-1]);
        return res;
    }
};
```



```c++
class Solution {
public:
    //按照末尾位置排序
    static bool camp(vector<int>& a, vector<int>& b){
        return a[0] < b[0];
    }
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(), camp);

        vector<vector<int>> res;
        res.push_back(intervals[0]);
        for(int i = 1;i < intervals.size();i++){
            // 是否有重叠
            if(res.back()[1] >= intervals[i][0]){
                //res.back()[0] = min(intervals[i][0], res.back()[0]);
                res.back()[1] = max(intervals[i][1], res.back()[1]);
            }else{
                res.push_back(intervals[i]);
            }
        }
        //res.push_back(intervals[intervals.size()-1]);
        return res;
    }
};
```



### [单调递增的数字](https://leetcode.cn/problems/monotone-increasing-digits/)

思路：遇到前一位比当前位大的，则前一位减一，当前位设为9.

注意，==`sNum[i] = '9'`==

```c++
class Solution {
public:
    int monotoneIncreasingDigits(int n) {
        string sNum = to_string(n);
        int flag = sNum.size();
        for(int i = sNum.size()-1; i > 0; i--){

            if(sNum[i-1] > sNum[i]){
                sNum[i-1]--;
                flag = i;
                //sNum[i] = '9';
            }
        }
        for(int i = flag;i < sNum.size();i++){
            sNum[i] = '9';
        }
      
        return stoi(sNum);

    }
};
```

### [买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

这一题就是考虑手续费的买卖股票。回顾一下买卖股票的第一题，是只能有一次的买卖，维护一个动态规划数组。买卖股票的第二题，是可以重复购买，贪心的策略是收集每一天的正利润。但是在有手续费的情况下，之前的贪心策略就不适用了。

思路：动态规划考虑手续费的思路应该是最直接的。该题动态规划的思路为

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        // dp[i][1]第i天持有的最多现金
        // dp[i][0]第i天持有股票所剩的最多现金
        int n = prices.size();
        vector<vector<int>> dp(n, vector<int>(2,0));
        dp[0][0] = -prices[0];
        for(int i = 1;i < prices.size();i++){
            //不持有股票		第i天不操作       第i天卖掉股票
            dp[i][1] = max(dp[i-1][1], dp[i-1][0] + prices[i] - fee);
            //持有股票 		第i天不操作       第i天买掉股票
            dp[i][0] = max(dp[i-1][0], dp[i-1][1] -prices[i]);
        }
        return max(dp[n-1][0], dp[n-1][1]);
    }
};
```

### [监控二叉树](https://leetcode.cn/problems/binary-tree-cameras/)

```c++
class Solution {
public:
    int result;
    int travel(TreeNode* cur){
        // 空节点应该师有覆盖的状态，才不会在叶子节点处放置相机
        if(cur == nullptr){
            return 2;
        }
        //后序遍历，从树的一侧 自低向上遍历
        int left = travel(cur->left);
        int right = travel(cur->right);

        // 如果左右孩子都是有覆盖的，因为相机是从低向上放置的，那么相机应该在孙子节点处，那么该节点是没有被覆盖的状态
        if(left == 2 && right == 2){
            return 0;
        }
        // 如果左右孩子中有没有被覆盖的，那么该节点应该放置相机
        if(left == 0 || right == 0){
            result++;
            return 1;
        }

        // 如果左右孩子中有相机的，那么该节点就是被覆盖的状态
        if(left == 1 || right == 1){
            return 2;
        }
        
        return -1;
    }
    int minCameraCover(TreeNode* root) {
        result = 0;
        //如果根节点是没有被覆盖的状态
        if(travel(root) == 0){
            result++;
        }      
   
        return result;

    }
};
```

# 动态规划

## 基础理论

**对于动态规划问题，我将拆解为如下五步曲，这五步都搞清楚了，才能说把动态规划真的掌握了！**

1. 确定dp数组（dp table）以及下标的含义
2. 确定递推公式
3. dp数组如何初始化
4. 确定遍历顺序
5. 举例推导dp数组

## 基础例题

### [斐波那契数](https://leetcode.cn/problems/fibonacci-number/)

1. 确定dp数组（dp table）以及下标的含义

   dp[i]：第 i 个数的斐波那契数是dp[i]

2. 确定递推公式

   dp[i] = dp[i-1] + dp[i-2]

3. dp数组如何初始化

   dp[0] = 0; dp[1] = 1; 

4. 确定遍历顺序

   从0开始，正向到i

5. 举例推导dp数组

   0 1 1 2 3 5 8 13 21 34 55

```c++
class Solution {
public:
    int fib(int n) {
        if(n <= 1) return n;

        vector<int> dp(n+1, 0);
        
        dp[0] = 0;
        dp[1] = 1;
        for(int i = 2; i <= n;i++){
            dp[i] = dp[i-1] + dp[i-2];
        }
        return dp[n];
    }
};
```

### [爬楼梯](https://leetcode.cn/problems/climbing-stairs/)

1. 确定dp数组（dp table）以及下标的含义

   dp[i]：爬到第i层楼梯，有dp[i]种方法。

2. 确定递推公式

   dp[i] = dp[i-1] + dp[i-2] **通过手写前四种情况就可以找出规律**

3. dp数组如何初始化

   dp[0] = 0; dp[1] = 1; 

4. 确定遍历顺序

   从0开始，正向到i

5. 举例推导dp数组

   0 1 2 3 5 8 13 21 34 55

```c++
class Solution {
public:
    int climbStairs(int n) {
        if (n <= 1) return n; // 因为下面直接对dp[2]操作了，防止空指针
        vector<int> dp(n + 1);
        dp[1] = 1;
        dp[2] = 2;
        for (int i = 3; i <= n; i++) { // 注意i是从3开始的
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        return dp[n];
    }
};
```

### [使用最小花费爬楼梯](https://leetcode.cn/problems/min-cost-climbing-stairs/)

1. 确定dp数组（dp table）以及下标的含义

   dp[i]：爬到第i层楼梯的最小花费dp[i]

2. 确定递推公式

   因为是每一次只能往上爬一节或者两节，所以只考虑前两个状态即可

   dp[i] = min(dp[i-1]+cost[i-1], dp[i-2]+cost[i-2]);

3. dp数组如何初始化

   dp[0] = 0; dp[1] = 0; 

4. 确定遍历顺序

   从0开始，正向到i

5. 举例推导dp数组

   | cost | 1    | 100  | 1    | 1    | 1    | 100  | 1    | 1    | 100  | 1    | 楼顶 |
   | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
   | dp   | 0    | 0    | 1    | 2    | 2    | 3    | 3    | 4    | 4    | 5    | 6    |

   ```cpp
   class Solution {
   public:
       int minCostClimbingStairs(vector<int>& cost) {
           if(cost.size() <= 1) return 0;
           vector<int> dp(cost.size()+1, 0);
           for(int i = 2; i < cost.size()+1;i++){
               dp[i] = min(dp[i-1]+cost[i-1], dp[i-2]+cost[i-2]);
           }
           return dp.back();
       }
   };
   ```



### [不同路径](https://leetcode.cn/problems/unique-paths/)

只能向右或者向下移动。

1. 确定dp数组（dp table）以及下标的含义

   `dp[i][j]`：移动到`[i,j]`的路径数`dp[i][j]`

2. 确定递推公式

   因为是每一次只能往右和往下走，所以只考虑前两个状态即可

   `dp[i][j] = dp[i-1][j] + dp[i][j-1] `

3. dp数组如何初始化

   `dp[0][j] = 1;dp[i][0] = 0; `

4. 确定遍历顺序

   从0,0开始，正向到i,j

5. 举例推导dp数组

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m, vector<int>(n, 0));

        for(int i = 0; i < m;i++){
            dp[i][0] = 1;
        }
        for(int j = 0; j < n;j++){
            dp[0][j] = 1;
        }

        for(int i = 1; i < m;i++){
            for(int j = 1; j < n;j++){
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        
        return dp.back().back();

    }
};
```

### [不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)

只能向右或者向下移动，并且中间有障碍物。

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size();
        int n = obstacleGrid[0].size();
        vector<vector<int>> dp(m, vector<int>(n, 0));

        bool flag = true;
        for(int i = 0; i < m;i++){
            if(obstacleGrid[i][0] == 1){
                dp[i][0] = 0;
                flag = false;
            }else if(flag == true){
                dp[i][0] = 1;
            }
        }
        flag = true;
        for(int j = 0; j < n;j++){
            if(obstacleGrid[0][j] == 1){
                dp[0][j] = 0;
                flag = false;
            }else if(flag == true){
                dp[0][j] = 1;
            }
        }

        for(int i = 1; i < m;i++){
            for(int j = 1; j < n;j++){
                if(obstacleGrid[i][j] == 1){
                    dp[i][j] = 0;
                }else{
                    dp[i][j] = dp[i-1][j] + dp[i][j-1];
                }
            }
        }
        
        return dp.back().back();
    }
};
```

### [不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/)

1. 确定dp数组（dp table）以及下标的含义

   `dp[i]`：`[0,i]`的数字构成的二叉搜索树的个数`dp[i]`

2. 确定递推公式

   从 0 开始到 i ，每一个数字 j 轮流做头节点，剩下的 `[0, j-1]` 和 `[j+1, i]` 分别在左右两个子树， `[0, j-1]` 有 j 个节点，有 `dp[j-1]` 种排列方式， `[j+1, i]`  有 `i-j` 个节点，有 `dp[i-j]`种排列方式，所以当  j 做头节点，按这种情况有 `dp[j-1] * dp[i-j] ` 种方式.

   `dp[i] += dp[j-1] * dp[i-j]`

3. dp数组如何初始化

   `dp[0] = 1 `

4. 确定遍历顺序

   从0,0开始，正向到i,j

5. 举例推导dp数组

```cpp
class Solution {
public:
    int numTrees(int n) {

        vector<int> dp(n+1, 0);
        dp[0] = 1;
        for(int i = 1; i <= n;i++){
            for(int j = 1; j <= i;j++){
                dp[i] += dp[j-1]*dp[i-j];
            }
        }
        return dp.back();

    }
};
```

### 背包问题

#### 01背包问题

1. 确定dp数组（dp table）以及下标的含义

   `dp[i][j]`：从下标为`[0,i]`的物品里任意取，放入容量为`j`的背包里，可以获得的最大价值为`dp[i][j]`

2. 确定递推公式

   面对一个下标为 i 的物品，就是有两种选择，拿或者不拿。

   不拿物品 i : 那么最大的价值就是 `dp[i-1][j]`

   拿物品 i : 那么最大价值就是 `dp[i-1][j-weight[i]]+value[i]`

   所以递推公式为 `dp[i][j]=max(dp[i-1][j], dp[i-1][j-weight[i]]+value[i])`

3. dp数组如何初始化

   - 首先是背包容量为0，`dp[i][0]=0`
   - 当容量小于最小物品（已排序，下标为0）的体积时，一个物品也拿不了，`dp[0][j]=0`；当容量大于最小物品时， `dp[0][j]=value[0]`

   ```cpp
   vector<vector<int>> dp(n, vector<int>(bag, 0));
   for(int j = weight[0];j <= bagCap;j++){
       dp[0][j] = value[0];
   }
   ```

   

4. 确定遍历顺序

   从0,0开始，正向到i,j

   **要注意是先遍历物品还是先遍历背包容量：从更新公式来看，只依赖于前一个物品（0~i-1个）的状态，背包容量则是会有一定的跳跃（j-weight[i]），所以还是先遍历物品再遍历背包容量更直观。**

   ```cpp
   for(int i = 1;i < n;i++){
   	for(int j = 1;j <= bagCap;j++){
           if(j < weight[i]){
               dp[i][j] = dp[i-1][j];
           }
           else{
               dp[i][j]=max(dp[i-1][j], dp[i-1][j-weight[i]]+value[i]);
           }
           
       }
   }
   ```

   

5. 举例推导dp数组

```cpp

#include <iostream>
#include <algorithm>
#include <vector>

class Solution {
public:
    static bool cmp(std::vector<int>& a, std::vector<int>& b) {
        return a[0] < b[0];
    }
    int solve01BagProblem(std::vector<std::vector<int>>& things, int bagCap) {
        // things 是一个二维数组，n * 2，第一列是weight，第二列是value
        int n = things.size();
        std::vector<int> weight(n, 0);
        std::vector<int> value(n, 0);
        std::sort(things.begin(), things.end(), cmp);
        for (int i = 0; i < things.size(); i++) {
            weight[i] = things[i][0];
            value[i] = things[i][1];
        }

        std::vector<std::vector<int>> dp(n, std::vector<int>(bagCap+1, 0));
        for (int j = weight[0]; j <= bagCap; j++) {
            dp[0][j] = value[0];
        }

        for (int i = 1; i < n; i++) {
            for (int j = 1; j <= bagCap; j++) {
                if (j < weight[i]) {
                    dp[i][j] = dp[i - 1][j];
                }
                else {
                    dp[i][j] = std::max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
                }

            }
        }

        return dp.back().back();
    }
};

int main()
{
    
    int bagCap = 4;
    std::vector<std::vector<int>> things = { {3,20},{4,30}, {1,15}};
    Solution S;
    int res = 0;
    res = S.solve01BagProblem(things, bagCap);
    std::cout << res << std::endl;
    return 0;
}

```

#### 01背包问题进阶，改成dp一维数组

原来的dp数组只用到了这一行和上一行，我们可以进一步把上一行拷贝到这一行，压缩到一个一维的dp数组。递推公式为 `dp[j]=max(dp[j], dp[j-weight[i]]+value[i])`

初始化 `dp[0]=0`，都初始化为0.

**需要从后向前遍历，倒序遍历**的原因是，本质上还是一个对二维数组的遍历，并且右下角的值依赖上一层左上角的值，因此需要保证左边的值仍然是上一层的，从右向左覆盖，所以倒序遍历。

### [分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)

看成是每一个物体的价值是`nums[i]`，每一个物体的体积是`nums[i]`

要理解 `dp[target]==target` 则说明可以将其分割成等和的子集，就理解了为什么这道题可以看作是01背包问题：背包问题是要在容量为 `j` 的背包里获取最大的价值。在我们这道题里，每一个物体的价值和体积是一样的。所以 `dp[j] <= j`是恒成立的，而我们背包问题就是求解使得 `dp[j] == j`的。那如果 `j == target == sum/2`即 `dp[target]==target` 就说明是可以把这个数组分成两个等和的子集的。

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        vector<int> dp(10001,0);
        int sum = 0;
        for(int n:nums){
            sum += n;
        }
        if(sum % 2 == 1) return false;

        int target = (sum >> 1);
        for(int i = 0; i < nums.size(); i++) {
            for(int j = target; j >= nums[i]; j--){
                dp[j] = max(dp[j], dp[j-nums[i]]+nums[i]);
            }
        }

        if(dp[target]==target) return true;
        return false;

    }
```

### [最后一块石头的重量 II](https://leetcode.cn/problems/last-stone-weight-ii/)

本题其实就是尽量让石头分成重量相同的两堆，相撞之后剩下的石头最小

```cpp
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        vector<int> dp(15001, 0);
        int sum = 0;
        for(int n:stones){
            sum += n;
        }

        int target = (sum >> 1);
        for(int i = 0;i < stones.size();i++){
            for(int j = target;j >= stones[i];j--){
                dp[j] = max(dp[j], dp[j-stones[i]]+stones[i]);
            }
        }
        int res = (sum - dp[target]) - dp[target];
        return res;


    }
};
```

### [目标和](https://leetcode.cn/problems/target-sum/)

首先对题目化解，要给给定的数组添加正负号，最终的结果就是 `子集1 - 子集2`，也就是要让`target = sum1 - sum2 = sum1 - (sum - sum1)  `，所以`sum1 = (sum + target)/2`，也就是找到数组中 和为 `(sum + target)/2`的子集的个数。

dp[j] 表示 和为 j 的子集的个数。

背包问题求组合的个数的递推公式 ` dp[j] += dp[j-nums[i]]`

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum = 0;
        for(int n:nums){
            sum += n;
        }
        int tar = (sum + target) / 2;
        if((sum + target) % 2 != 0 || abs(target) > sum){
            return 0;
        }
       

        vector<int> dp(tar+1, 0);
        dp[0] = 1;
        for(int i = 0;i < nums.size();i++){
            for(int j = tar;j >= nums[i];j--){
                dp[j] += dp[j-nums[i]];
            }
        }
        return dp[tar];

    }
};
```

### [一和零](https://leetcode.cn/problems/ones-and-zeroes/)

给你一个二进制字符串数组 `strs` 和两个整数 `m` 和 `n` 。

请你找出并返回 `strs` 的最大子集的长度，该子集中 **最多** 有 `m` 个 `0` 和 `n` 个 `1` 。

1. 确定dp数组（dp table）以及下标的含义

   `dp[i][j]`：子集中包含 `i` 个 `0` 和 `j` 个 `1` 时，该子集的最大长度`dp[i][j]`

2. 确定递推公式

   面对一个下标为 i 的str，就是有两种选择，拿或者不拿。

   不拿str i : 那么最大长度就是 `dp[i][j]`

   拿str i : 那么最大价值就是 `dp[i-zeroNum][j-oneNum]+1`； zeroNum 和 oneNum 分别为字符串str i 的0的个数 和 1 的个数。

   所以递推公式为 `dp[i][j]=max(dp[i][j], dp[i-zeroNum][j-oneNum]+1)`

3. dp数组如何初始化

   均初始化为0

4. 遍历顺序，i 和 j 均倒序遍历。m-->zeroNum; n-->oneNum

```cpp
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        vector<vector<int>> dp(m+1, vector<int>(n+1,0));
        for(string str:strs){
            int zeroNum = 0, oneNum = 0;
            for(char s:str){
                if(s == '0') zeroNum++;
                else oneNum++;
            }

            for(int i = m;i >= zeroNum;i--){
                for(int j = n;j >= oneNum;j--){
                    dp[i][j] = max(dp[i][j], dp[i-zeroNum][j-oneNum]+1);
                }
            }
        }
        return dp.back().back();

    }
};
```

### 完全背包问题

有N件物品和一个最多能背重量为W的背包。第i件物品的重量是weight[i]，得到的价值是value[i] 。**每件物品都有无限个（也就是可以放入背包多次）**，求解将哪些物品装入背包里物品价值总和最大。

**完全背包和01背包问题唯一不同的地方就是，每种物品有无限件**。我们知道01背包内嵌的循环是从大到小遍历，为了保证每个物品仅被添加一次。而**完全背包的物品是可以添加多次的，所以要从小到大去遍历**。

**01背包内嵌的循环从倒序遍历，可以保证每个物品仅被添加一次。**因为把dp压缩成一维数组之后，j 从后往前遍历更新就可以等效于由二维dp数组的`dp[i-1][j], dp[i-1][j-weight[i]]`推导出`dp[i][j]`的过程，因为 j 是从后往前遍历更新，`dp[j-weight[i]]`就是还没有更新，还是`dp[i-1][j-weight[i]]`的状态。

而完全背包从前向后遍历，就是每一个物品不止被添加一次了，是可以重复添加的。

```cpp
// 先遍历物品，再遍历背包
for(int i = 0; i < weight.size(); i++) { // 遍历物品
    for(int j = weight[i]; j <= bagWeight ; j++) { // 遍历背包容量
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);

    }
}
```

背包问题模板汇总

| 问题类型           | 模板注意要点                                                 |
| ------------------ | ------------------------------------------------------------ |
| 01背包             | 先遍历物品或先遍历背包都可以；<br />当dp数组是一维数组时，要倒序遍历背包。 |
| 完全背包           | 当dp数组是一维数组时，要正序遍历背包，这样才是无限拿物品     |
| 完全背包求组合问题 | 正序遍历背包；<br />先遍历物品，后遍历背包；                 |
| 完全背包求排列问题 | 正序遍历背包；<br />先遍历背包，后遍历物品，这样才会把不同的物品次序看成是不同的。 |



### [零钱兑换](https://leetcode.cn/problems/coin-change/)

这一题用回溯会超时

```cpp
class Solution {
public:
    int res = INT_MAX;
    vector<int> path;
    void backTrack(vector<int>& coins, int amount, long* sum, int startIndex){
        if(*sum == amount){
            res = res < path.size()? res: path.size();
            return;
        }
        if(*sum > amount){
            return;
        }

        for(int i = startIndex; i < coins.size(); i++){
            path.push_back(coins[i]);
            *sum += coins[i];
            backTrack(coins, amount, sum, i);
            path.pop_back();
            *sum -= coins[i];
        }
    }
    int coinChange(vector<int>& coins, int amount) {
        path.clear();
        long sum = 0;
        backTrack(coins, amount, &sum, 0);
        if(res == INT_MAX){
            return -1;
        }
        return res;
    }
};
```



### [零钱兑换 II](https://leetcode.cn/problems/coin-change-ii/)

1. 确定dp数组（dp table）以及下标的含义

   `dp[j]`：背包容量为 j 构成的组合数`dp[j]`

2. 确定递推公式

   `dp[j] += dp[j-nums[i]]`

3. dp数组如何初始化

   `dp[0] = 1`

4. 确定遍历顺序

   本题是**完全背包**，要求的是**组合数，所以遍历顺序是先正序遍历物品数，然后正序遍历背包。**

   **如果要求的是排列数，就要先正序遍历背包，然后正序遍历物品数。**

```cpp
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        vector<int> dp(amount+1, 0);
        dp[0] = 1;

        for(int i = 0;i < coins.size();i++){
            for(int j = coins[i]; j <= amount;j++){
                dp[j] += dp[j - coins[i]];
            }
        }

        return dp.back();
    }
};
```

### [单词拆分](https://leetcode.cn/problems/word-break/)

单词拆分，这里相当于把背包容量改成了用字符串来表示了。没思路。

1. 确定dp数组（dp table）以及下标的含义

   `dp[i]`：背包容量的前 i 位部分可不可以用字典表示。可以，则`dp[i] = true`

2. 确定递推公式

   显然，`dp[i]`的状态与`dp[i-word.size()]`有关。如果`dp[i-word.size()]=true`，并且背包的 `[i-word.size(),i]`的子串与word相等，那么`dp[i]=true`

3. dp数组如何初始化

   `dp[0] = true`

4. 确定遍历顺序

   这个题是 排列问题，因为单词之间的顺序对结果有影响，所以是排列问题，所以要先遍历背包，再遍历物品。

   因为是完全背包，所以正向遍历背包。

```cpp
class Solution {
public:
    bool isSame(string s1, string s2){
        if(s1.size() != s2.size()){
            return false;
        }
        for(int i = 0;i < s1.size();i++){
            if(s1[i] != s2[i]){
                return false;
            }
        }
        return true;
    }
    bool wordBreak(string s, vector<string>& wordDict) {
        vector<bool> dp(s.size() + 1, false);
        dp[0] = true;

        for(int i = 1; i <= s.size();i++){
            for(int j = 0;j < wordDict.size();j++){
                int size = wordDict[j].size();
                if(i - size >= 0){
                    string str = s.substr(i - size, size);
                    if(isSame(str, wordDict[j]) && dp[i - size]){
                        dp[i] = true;
                    }
                }
            }
        }

        return dp.back();
    }
};
```



### [打家劫舍 III](https://leetcode.cn/problems/house-robber-iii/)（递归+DP）

二叉树的节点A的孩子何其父节点不可以同时偷。

```cpp
class Solution {
public:
    vector<int> robTree(TreeNode* cur){
        //vector<int> dp  dp[0]代表不偷当前节点，可以获得的最大金额； dp[1]代表偷当前节点，可以获得的最大金额；
        if(cur == nullptr){
            return {0,0};
        }

        vector<int> left = robTree(cur->left);
        vector<int> right = robTree(cur->right);

        vector<int> res(2,0);
        res[1] = cur->val + left[0] + right[0];
        res[0] = max(left[0], left[1]) + max(right[0], right[1]);
        return res;

    }
    int rob(TreeNode* root) {
        vector<int> res = robTree(root);
        return max(res[0], res[1]);
    }
};
```

### [买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)(只能买卖一次股票)

1. 确定dp数组（dp table）以及下标的含义

   `dp[i][0]`：第 i 天不持有股票，手中的钱数为`dp[i][0]`

   `dp[i][1]`：第 i 天持有股票，手中的钱数为`dp[i][1]`

2. 确定递推公式

   显然，`dp[i]`的状态与`dp[i-1]`有关。

   如果第 i 天不持有股票，那么有可能是 i-1 天就没有持有股票，或者是在第 i 天把股票卖掉，更新公式为
   $$
   dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
   $$
   如果第 i 天持有股票，由于本题只能买卖一次股票，一开始的钱数是 0， 所以第 i 天持有股票要么是第 i 天买入的，要么之前买入的，更新公式
   $$
   dp[i][1] = max(dp[i-1][1], -price[i])
   $$
   

3. dp数组如何初始化

   `dp[0][0] = 0; dp[0][1] = -prices[0]`

4. 确定遍历顺序

   正序遍历

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        vector<vector<int>> dp(prices.size(), vector<int>(2,0));
        dp[0][0] = 0;
        dp[0][1] = -prices[0];
        for(int i = 1;i < prices.size();i++){
            dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i]);
            dp[i][1] = max(dp[i-1][1], -prices[i]);
        }
        return dp.back()[0];
    }
};
```

### [买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)(可以多次买卖股票，但是只能持有一支)

1. 确定dp数组（dp table）以及下标的含义

   `dp[i][0]`：第 i 天不持有股票，手中的钱数为`dp[i][0]`

   `dp[i][1]`：第 i 天持有股票，手中的钱数为`dp[i][1]`

2. 确定递推公式

   显然，`dp[i]`的状态与`dp[i-1]`有关。

   如果第 i 天不持有股票，那么有可能是 i-1 天就没有持有股票，或者是在第 i 天把股票卖掉，更新公式为
   $$
   dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
   $$
   如果第 i 天持有股票，第 i 天持有股票要么是第 i 天买入的，要么之前买入的，又因为可以多次购买，所以更新公式在上一题的基础上修改为
   $$
   dp[i][1] = max(dp[i-1][1], dp[i-1][0]-price[i])
   $$
   

3. dp数组如何初始化

   `dp[0][0] = 0; dp[0][1] = -prices[0]`

4. 确定遍历顺序

   正序遍历

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        // dp[i][1]第i天持有股票的最多现金
        // dp[i][0]第i天不持有股票的最多现金
        vector<vector<int>> dp(prices.size(), vector<int>(2,0));
        dp[0][0] = 0;
        dp[0][1] = -prices[0];
        for(int i = 1;i < prices.size();i++){
            dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i]);
            dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i]);
        }
        return dp.back()[0];
    }
};
```





### [买卖股票的最佳时机 III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)(最多进行2笔交易)



