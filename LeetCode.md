# LeetCode

## 1.两数之和

```c++
利用哈希表可让搜索时间复杂度从O(n)->O(1)
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        //创建哈希表
        unordered_map<int,int>hashTable;
        
        //遍历数组
        for(int i=0;i<nums.size();i++){
            //查找是否有与nums[i]互补的值
            auto it=hashTable.find(target-nums[i]);
            //如果找到，则返回对应的指针，没找到就返回end指针，因此可以以此判断是否找到互补的值
            
            if(it!=hashTable.end()){//如果在哈希表中找到互补的值，就直接返回索引即可
                return{it->second,i};
            }
            
            //但如果没找到就插入哈希表中，以便后续查找
            hashTable[nums[i]]=i;
        }
        
        return {};
    }
};
```

## 2.两数相加

```c++
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode*pre=new ListNode(-1);//创建前驱指针，防止头指针丢失
        ListNode*cur=pre;//创建副本指针，用来做相应的操作
        int carry=0;
        while(l1||l2){//循环条件是指针l1和l2至少有一个不为空
            
            //缺的位用0补齐，例如，194+3可以看作：194+003
            int val1=l1?l1->val:0;
            int val2=l2?l2->val:0;

            int sum=val1+val2+carry;//注意要将前一位相加的进位加入
            carry=sum/10;//进位
            sum%=10;//和要取模10，不然可能会溢出(大于10)

            cur->next=new ListNode(sum);//让cur的next指针指向值等于我们上一步求出的和的节点

            cur=cur->next;//后移cur
            
            //只有当指针不为空时，才能后移，不然会报错
            if(l1){
                l1=l1->next;
            }
            if(l2){
                l2=l2->next;
            }
        }
        if(carry==1){//注意不要忘记最高位相加进位是1的话，要在最后为填补1
            cur->next=new ListNode(1);
        }
        
        return pre->next;
    }
};
```

## 3.无重复字符的最长子串

![image-20221008231011250](D:\Code(Github)\C-\LeetCode-img\1.png)

![image-20221008231138459](D:\Code(Github)\C-\LeetCode-img\2.png)

```c++
思路：
    滑动窗口
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int length=s.size();//记录字符串s的长度
        if(length==0){
            return 0;//边界情况
        }
        unordered_set<char>st;//创建哈希表st，以便后续操作
        int maxLength=0;//记录最长子串长度，初值赋值为0
        int right=-1;//用来移动查看字符

        for(int i=0;i<length&&right+1<length;i++){
            //每右移一位，删去最左边的字符，但开始时不删去，因为此时哈希表st为空
            if(i!=0){
                st.erase(s[i-1]);
            }
            
            //由于在哈希表中的字符必然是不重复的，所以无需让right回溯
            while(right+1<length&&st.count(s[right+1])==0){
                st.insert(s[right+1]);//满足条件，插入哈希表中
                right++;//right指针右移
            }
            maxLength=max(maxLength,right-i+1);//如果有更长的子串就更新子串长度
        }
        return maxLength;
    }
};
```

## 4.寻找两个正序数组的中位数

```c++
方法一：直接法
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        vector<double>vec(nums1.size()+nums2.size());
        int i=0;
        int j=0;
        int k=0;
        
        //合并数组
        while(j<nums1.size()&&k<nums2.size()){
            vec[i++]=nums1[j]<nums2[k]?nums1[j++]:nums2[k++];
        }
        while(j<nums1.size()){
            vec[i++]=nums1[j++];
        }
        while(k<nums2.size()){
            vec[i++]=nums2[k++];
        }
        if(vec.size()%2!=0){
            return vec[vec.size()/2];
        }else{
            return (vec[vec.size()/2-1]+vec[vec.size()/2])/2;
        }
    }
};

方法二：无需合并数组
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int index1=0;
        int index2=0;

        int preVal=0;
        int postVal=0;

        int len=nums1.size()+nums2.size();
        for(int i=0;i<=len/2;i++){
            postVal=preVal;//保存前一次的值
            if(index1<nums1.size()&&(index2>=nums2.size()||nums1[index1]<nums2[index2])){
                preVal=nums1[index1++];
            }else{
                preVal=nums2[index2++];
            }
        }

        if(len%2==0){
            return (preVal+postVal)/2.0;//主要要除小数，这样才能得到double
        }else{
            return preVal;
        }
    }
};

方法三：类二分
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int n=nums1.size();
        int m=nums2.size();
        int left=(n+m+1)/2;
        int right=(n+m+2)/2;
        //如果n+m是奇数，那么left=right；
        return (getKth(nums1,0,n-1,nums2,0,m-1,left)+getKth(nums1,0,n-1,nums2,0,m-1,right))*0.5;
    }

    int getKth(vector<int>&nums1,int start1,int end1,vector<int>&nums2,int start2,int end2,int k){
        int len1=end1-start1+1;
        int len2=end2-start2+1;

        //让len1小于len2，这样能保证有数组空时，一定是nums1
        if(len1>len2)return getKth(nums2,start2,end2,nums1,start1,end1,k);
        
        //nums1为空
        if(len1==0)return nums2[start2+k-1];
        
        if(k==1)return min(nums1[start1],nums2[start2]);

        //取min是防止访问溢出
        int i=start1+min(len1,k/2)-1;
        int j=start2+min(len2,k/2)-1;
        if(nums1[i]>nums2[j]){
            return getKth(nums1,start1,end1,nums2,j+1,end2,k-(j-start2+1));
        }else{
            return getKth(nums1,i+1,end1,nums2,start2,end2,k-(i-start1+1));
        }
    }
};
```

## 5.最长回文子串

```c++
方法一：暴力解法
class Solution {
public:
    string longestPalindrome(string s) {
        string longest="";
        int maxLen=0;
        for(int i=0;i<s.length();i++){
            for(int j=i+1;j<=s.length();j++){
                string tempS=s.substr(i,j-i);
                if(isPalindrome(tempS)&&tempS.length()>maxLen){
                    longest=tempS;
                    maxLen=longest.length();
                }
            }
        }
        return longest;
    }

    bool isPalindrome(string s){
        for(int i=0;i<s.length()/2;i++){
            if(s[i]!=s[s.length()-i-1]){
                return false;
            }
        }
        return true;
    }
};

方法二：动态规划
class Solution {
public:
    string longestPalindrome(string s) {
        int n=s.length();
        if(n<2){
            return s;
        }

        int maxLen=1;
        int begin=0;
        vector<vector<bool>>dp(n,vector<bool>(n));
        
        for(int i=0;i<n;i++){
            dp[i][i]=true;
        }

        for(int L=2;L<=n;L++){
            for(int i=0;i<n;i++){
                int j=i+L-1;
                if(j>=n){
                    break;
                }
                if(s[i]!=s[j]){
                    dp[i][j]=false;
                }else{
                    if(j-i<3){
                        //i和j之间的字符数小于3的话，就无法通过前一次的来判断这一次
                        dp[i][j]=true;
                    }else{
                        dp[i][j]=dp[i+1][j-1];
                    }
                }
                if(dp[i][j]&&j-i+1>maxLen){
                    maxLen=j-i+1;
                    begin=i;
                }
            }
        }
        return s.substr(begin,maxLen);
    }
};

方法三：中心扩展算法
    
边界情况:P(i,i),p(i,i+1),即中心长度为1或2时，因为每次判断时必定是两边各添加一个字符，总共就是两个字符。
假设边界情况为1，那每次长度为：1，3，5，7，2n+1
假设边界情况为2，那每次长度为：2，4，6，8，2n
这样奇偶情况都有了，即涵盖了所有情况

class Solution {
public:
    string longestPalindrome(string s) {
        int start=0,end=0;
        for(int i=0;i<s.size();i++){
            auto [left1,right1]=expandAroundCenter(s,i,i);
            auto [left2,right2]=expandAroundCenter(s,i,i+1);
            if(right1-left1>end-start){
                start=left1;
                end=right1;
            }
            if(right2-left2>end-start){
                start=left2;
                end=right2;
            }
        }
        return s.substr(start,end-start+1);
    }

    pair<int,int>expandAroundCenter(const string& s,int left,int right){
        while(left>=0&&right<s.size()&&s[left]==s[right]){
            left--;
            right++;
        }
        return {left+1,right-1};
    }
};
```

