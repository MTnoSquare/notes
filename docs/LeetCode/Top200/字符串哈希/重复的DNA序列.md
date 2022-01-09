## 187.重复的DNA序列
 所有 DNA 都由一系列缩写为` 'A'，'C'，'G' 和 'T' `的核苷酸组成，例如：`"ACGAATTCCG"`。在研究 DNA 时，识别 DNA 中的重复序列有时会对研究非常有帮助。

编写一个函数来找出所有目标子串，目标子串的长度为 10，且在 DNA 字符串 s 中出现次数超过一次。

### 思路
**TLE**:  
    利用哈希表存储子串以及对应出现次数，当子串出现次数大于1次时添加到答案中（子串长度过大时会导致TLE）

**字符串哈希**：
    为了在O(1)时间得出判断子串，采用**字符串哈希+前缀和**的思想

具体做法为，使用一个与字符串 s 等长的哈希数组` h[]`，以及次方数组 `p[]`。

由字符串预处理得到这样的哈希数组和次方数组复杂度为` O(n)`。当我们需要计算子串 `s[i...j] `的哈希值，只需要利用前缀和思想` h[j] - h[i - 1] * p[j - i + 1] `即可在 `O(1)` 时间内得出哈希值（与子串长度无关）。



### 题解
```java
class Solution {
    public List<String> findRepeatedDnaSequences(String s) {
        List<String> a=new ArrayList<>();
        int n=s.length();
        Map<Integer,Integer> map=new HashMap<>();
        int P=13131;
        int []hash=new int[n+1];
        int []p=new int[n+1];
        p[0]=1;
        for(int i=1;i<=n;i++){
            hash[i]=hash[i-1]*P+s.charAt(i-1);
            p[i]=p[i-1]*P;
        }
        for(int i=1;i+10-1<=n;i++){
            int r=i+10-1;
            int cur=hash[r]-hash[i-1]*p[r-i+1];
            int count=map.getOrDefault(cur,0);
            if(count==1)
                a.add(s.substring(i-1,r));
            map.put(cur,count+1);
        }
        return a;
    }
}
```
## 参考链接