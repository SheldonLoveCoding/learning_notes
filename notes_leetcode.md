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

3. 处理 b[j] == b[i] 的情况，并给next[i]赋值。

   跳出步骤2的循环之后，

   要么就是b[j] == b[i]，**这里我们把视角移到对于i的这一层循环上**，我们想，经历了步骤2中的while循环，那么在上一轮对i的循环时（i-1的时候），这个j要么是0，要么是落在j-1的位置上，并且b[j-1] == b[i-1]。这里就有一点动态规划的意思了，因为可以保证在当前循环中，要么 i 的前一位和 j 的前一位是相等的，要么 j==0，所以当b[i]==b[j]的时候，我们直接j++即可找到 最大前缀和后缀完全相等的位数，后面赋值next[i]=j。

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

# 栈与队列（stack & queue）

## 基础知识

1. C++中stack 是容器么？

   并不是，C++的STL中提供了七种容器。分别是三个序列式容器：向量（vector）、双端队列（deque）、列表（list），此外你也可以把 string 和 array 当做一种序列式容器。四个关联式容器：集合（set）、多重集合（multiset）、映射（map）和多重映射（multimap）。所以STL中栈往往不被归类为容器，而被归类为container adapter（容器适配器）。

2. 我们使用的stack是属于哪个版本的STL？

3. 我们使用的STL中stack是如何实现的？

   栈的内部结构，栈的底层实现可以是vector，deque，list 都是可以的， 主要就是数组和链表的底层实现。stack和queue都是默认使用deque来实现的。

4. stack 提供迭代器来遍历stack空间么？

   不提供，栈提供push 和 pop 等等接口，所有元素必须符合先进后出规则，**所以栈不提供走访功能，也不提供迭代器(iterator)**。 不像是set 或者map 提供迭代器iterator来遍历所有元素。队列也不允许有便利行为，不提供迭代器。

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
        preorderTraversal(cur->left, result);
        result.push_back(cur->val);        
        preorderTraversal(cur->right, result);
        
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
        preorderTraversal(cur->left, result);   
        preorderTraversal(cur->right, result);
        result.push_back(cur->val);     
        
    }
    
    vector<int> Traversal(TreeNode* root) {
        vector<int> result;
        postorderTraversal(root, result);
        return result;
    }
};
```

