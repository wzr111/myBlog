---
title: leetcode每日一题
date: 2021-09-24 01:54:56
categories: leetcode
tags: leetcode
thumbnail: https://tva1.sinaimg.cn/large/008i3skNgy1gur3n02kllj61900u0k2l02.jpg
---
10月3 国庆差不多要过去一半了

![](https://tva1.sinaimg.cn/large/008i3skNgy1gv1iy0frmfj626a0pgdm402.jpg)

---

回忆一下dfs的题，复原IP地址
![](https://tva1.sinaimg.cn/large/008i3skNgy1gv1ecz6qm7j612m0t8q5o02.jpg)

```java
class Solution {
    List<String> ans=new ArrayList<String>();
    int[] segments=new int[4];
    public List<String> restoreIpAddresses(String s) {
        dfs(s,0,0);
        return ans;

    }
    public void dfs(String s, int segId, int segStart){
        //如果找到了4段IP地址并且遍历完了字符串，那么就是一种答案
        if(segId==4){
            if(segStart==s.length()){

                //刚好遍历完且是4段，则作为结果存储
                StringBuffer sb=new StringBuffer();
                for(int i=0;i<4;i++){
                    sb.append(segments[i]);
                    if(i!=3) sb.append(".");
                }
                ans.add(sb.toString());
            }
            return;
        }
        if(segStart==s.length()) return;//没找到结果，返回

        //由于不能有前导0，如果当前数字为0，那么这一段IP只能为0
        if(s.charAt(segStart)=='0'){
            segments[segId]=0;
            dfs(s,segId+1,segStart+1);
        }

        //一般情况，dfs回溯遍历
        int addr=0;
        for(int segEnd=segStart; segEnd<s.length();segEnd++){
            addr=addr*10+s.charAt(segEnd)-'0';
            if(addr>0&&addr<=255){//增加大于0的条件是 四个0的时候，会重复输出。或者有0开头的非0IP
                segments[segId]=addr;
                dfs(s,segId+1,segEnd+1);

            }else return;
        }
    }

}

```
![](https://tva1.sinaimg.cn/large/008i3skNgy1gv1gmsk0bmj60ym0u0wjs02.jpg)

---
10月2号
国庆都是简单题
![](https://tva1.sinaimg.cn/large/008i3skNgy1gv1eazizpfj61sm0omgps02.jpg)

---
10月1号 国庆快乐 看长津湖去了，忘了打卡 补上

![](https://tva1.sinaimg.cn/large/008i3skNgy1gv1ebjpmkkj613u0aeaar02.jpg)
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

```java
class Solution {
    public int findMinMoves(int[] machines) {
        int len=machines.length;
        int sum=0,max=Integer.MIN_VALUE;
        for(int i=0;i<len;i++){
            sum+=machines[i];
        }
        if(sum%len!=0) return -1;
        int evg=sum/len;
        //次数为差值数组中出现的绝对值最大的数字

        //忽略了一点，当machine[i]<0,是可以从左右两边获得衣服的。取左右中最大值
        //当machine[i]>0 需要把洗衣机的衣服向左右转移，移动次数为与evg当前差值
        
        int balance=0;//balance记录当前洗衣机要经过的流量，因为非相邻的洗衣机的衣服也要流通
        //balance<0需要向右边拿衣服，>0需要向右边输送
        for(int i=0;i<len;i++){
            balance+=machines[i]-evg;
            
            //当前位置的最大流量 比如[9,1,8,8,9],7.
            //2向右输送，2号位置既可以从1进，也可以从3进。下面的左参数就是往右出，右参数就是从左右进，左参数有可能为负数，可以无视。balance为-4，其实意思是-6，已经从左边取了2.还需要从右边取4.
            int current=Math.max(machines[i]-evg,Math.abs(balance));
            max=Math.max(max,current);
        }
        return max;
    }
}
```
![](https://tva1.sinaimg.cn/large/008i3skNgy1guy0zzbrimj614k0n40uy02.jpg)
---
