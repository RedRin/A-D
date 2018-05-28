#最长回文子串问题

###解1：动态规划
O(N^2) time and O(N^2) space:

避免重复相同的子串

 Consider the case “ababa”. If we already knew that “bab” is a palindrome, it is obvious that “ababa” must be a palindrome since the two left and right end letters are the same.

即  

![](imgs/Palindromic0.png)

	假设1000长度的串   66ms
	class Solution {
	public:
	    string longestPalindrome(string s) {
	       
	        int maxLen = 1;
	        int start = 0;
	        bool table[1000][1000] = {false};
	        
	        for(int i=0; i<s.size(); i++){
	            table[i][i] = true;
	        }
	        
	        for(int i=0; i<s.size()-1; i++) {
	            if(s[i] == s[i+1]){
	                table[i][i+1] = true;
	                maxLen = 2;
	                start = i;
	            }
	            
	        }
	        
	        for(int len=3; len <= s.size(); len++){       // it must be equal.
	            for(int i=0; i<s.size()-len+1; i++) {
	                int j = i+len-1;
	                if(s[i]==s[j] && table[i+1][j-1]==true) {
	                    table[i][j] = true;
	                    maxLen = len;
	                    start = i;
	                }
	            }
	        }
	        
	        return s.substr(start, maxLen);
	    }
	};

###解2：
由回文子串的中点向两边扩展

A simpler approach, O(N^2) time and O(1) space:   

	25ms

	class Solution {
	public:
	    string longestPalindrome(string s) {
	       
	        string ret = s.substr(0,1);
	        for(int i=0; i<s.size(); i++){
	            string s1 = expandAroundCenter(s, i, i);
	            string s2 = expandAroundCenter(s, i, i+1);
	            if(s1.size() > ret.size()) {
	                ret = s1;
	            }
	            if(s2.size() > ret.size()) {
	                ret = s2;
	            }
	        }
	        return ret;
	    }
	    
	    static string expandAroundCenter(string s, int left, int right){
	        int len = s.size();
	        while(left>=0 && right<len && s[left] == s[right]) {
	            left--;
	            right++;
	        }
	        return s.substr(left+1, right-left-1);
	    }
	};

#解3：Manacher算法
O(N) time and O(N) space  
原理：回文串的对称性。

预处理：在字符串每个字符的空隙间插入没用过的字符，通常为"#"，将它变为有奇数个字符的串    

![](imgs/Palindromic1.png)

定义一个 `数组p` 用来表示以 `i` 为中心的回文串最长半径。   

`C` 指示当前右边界最远的回文串的中心，R为其右边界，L为其左边界   

	如果 P[ i’ ] ≤ R – i,  则 P[ i ] = P[ i’ ]

![](imgs/Palindromic2.png)

	否则，P[ i ] ≥ P[ i’ ] , 并延伸 i 为中心的字符串，
	直至不是回文，然后更新 C 为 i, R 为当前 i 的右边界。

![](imgs/Palindromic3.png)

另一篇文章  

	RL数组即为p数组，pos为C
	由对称有 j=2*pos-i
	step 1: 令RL[i]=min(RL[2*pos-i], MaxRight-i)
	step 2: 以i为中心扩展回文串，直到左右两边字符不同，或者到达边界。
	step 3: 更新MaxRight和pos

![](imgs/Palindromic4.png)


<font color=red>P[ i ] 即为回文串的长度</font>


	class Solution {
	public:
	    string longestPalindrome(string s) {
	        string temp = preProcess(s);
	        const int length = temp.size();
	        int RL[length] = {0};
	        
	        int pos = 0;
	        int maxRight = 0;
	        
	        for(int i=0; i<length; i++){
	            if(i<maxRight) {
	                RL[i] = min(RL[2*pos-i], maxRight-i);
	            } else {
	                RL[i] = 0;
	            }
	            
	            while(i+RL[i]+1 < length && i-RL[i]-1>=0 && temp[i+RL[i]+1] == temp[i-RL[i]-1]) {
	                RL[i]++;
	            }
	            //RL[i]--;
	            if(i+RL[i] > maxRight){
	                pos = i;
	                maxRight = i+RL[i];
	            }
	        }
	        
	       
	        int retCenter = 0;
	        int maxLen = 0;
	        for(int i=0; i<length; i++){
	            if(RL[i] > maxLen) {
	                retCenter = i;
	                maxLen = RL[i];
	            }
	        }
	        
	      return s.substr((retCenter-maxLen)/2, maxLen);
	        
	        
	    }
	    
	    static string preProcess(string s){
	        string ret("#");
	        for(int i=0; i<s.size(); i++){
	            ret += s[i];
	            ret += "#";
	        }
	        return ret;
	    }
	};













#<font color=red>注意：</font>

	string ret;
	string s = "hello";
	for(int i=0; i<s.size(); i++) {
		ret += "#" + s[i];   //不允许，const char* 与  char 怎么能加呢！！！！！？？
	}

	要么
	ret += "#" + s.substr(i,1);
	要么
	ret += "#";
	ret += s[i];

#<font color=red>注意：</font>
	
	for(int i=0; i<s.size()-1; i++){  //s.size()-1 溢出了！！！！
	s.length() 也一样

	解决办法
	int length = s.size();
	for(int i=0; i<length-1; i++){