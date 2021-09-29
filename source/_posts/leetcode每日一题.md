---
title: leetcode每日一题
date: 2021-09-24 01:54:56
categories: leetcode
tags: leetcode
thumbnail: https://tva1.sinaimg.cn/large/008i3skNgy1gur3n02kllj61900u0k2l02.jpg
---

好家伙，一次过了

![](https://tva1.sinaimg.cn/large/008i3skNgy1gur3lu3o16j61ab0u0tee02.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gur3kz0gnzj61p90u0tep02.jpg)
---
9月25
![](https://tva1.sinaimg.cn/large/008i3skNgy1guujmkp8xtj61m20mywi802.jpg)
---
9月26
![](https://tva1.sinaimg.cn/large/008i3skNgy1guujk6gelpj60sm0asq3f02.jpg)
---
9月27，hard题 瑟瑟发抖
![](https://tva1.sinaimg.cn/large/008i3skNgy1guujkt2smlj60rw0pujug02.jpg)

hard题纯手打
```java
class Solution {
    public int numDecodings(String s) {
        //组合成的数字需要小于26，dfs
        // 如果遇到*需要遍历9种结果
        //数字不可以0开头
        
        //居然不可以用dfs？复习一下IP题的dfs
        int len=s.length();
        if(len==0) return 1;
        long[] dp=new long[len+1];
        if(s.charAt(0)=='0'){
            dp[1]=0;
        }else{
            dp[1]=s.charAt(0)=='*'?9:1;
        }

        if(len==1) {
            return (int)dp[1];
        }

        dp[0]=1;
        
        for(int i=2;i<=len;i++){
            int j=i-1;
            int f1=0,f2=0;
            char c=s.charAt(j), d=s.charAt(j-1);
            if(c=='*') f1=9;
            else if(c>='1'&&c<='9') f1=1;
            else f1=0;
            //System.out.print(f1+" ");
            if(c=='*'&&d=='*') f2=15;//*不能为0
            else if(d=='*'&&c!='*'){
                if(c>='0'&&c<='6') f2=2;
                else f2=1;
            }
            else if(d!='*'&&c=='*'){
                if(d=='1') f2=9;
                else if(d=='2') f2=6;
                else f2=0;
            }
            else if(d!=0){
                if(d=='1'||d=='2'&&c>='0'&&c<='6') f2=1;
           
            } else f2=0;
            // System.out.println(f1+" "+dp[i-1]);

            // System.out.println(f2+" "+dp[i-2]);

            dp[i]=(f1*dp[i-1]+f2*dp[i-2])%(1000000007);
            System.out.println(f1*dp[i-1]+f2*dp[i-2]);

        }
        //多了一个for循环就超时了
        return (int)dp[len];
    }
}
```
case
![](https://tva1.sinaimg.cn/large/008i3skNgy1guujl8wbblj60sa0qqdic02.jpg)

---
9月28
![](https://tva1.sinaimg.cn/large/008i3skNgy1guwrx3c1sej61ja0mon2a02.jpg)

---
9月29
又是hard题，人没了
![](https://tva1.sinaimg.cn/large/008i3skNgy1guwtf4wlmcj60sg0rkdie02.jpg)
