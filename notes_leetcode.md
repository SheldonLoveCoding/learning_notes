# 链表

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

`.find(x)`在哈希表内查找x，如果查找到，则返回对应的迭代器；如果没有查到，则返回`.end()`的结果。 `.end()`返回的是最后一个元素的下一个迭代器。



# 字符串 string

## cpp基础用法

| 函数                                                       | 用法                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| `string& insert (size_t pos, const string& str);`          | 插入字符串。pos 表示要插入的位置，也就是下标；str 表示要插入的字符串，它可以是 string 字符串，也可以是C风格的字符串。 |
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

