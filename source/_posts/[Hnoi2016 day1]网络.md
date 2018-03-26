---
title:  Hnoi2016day1网络 
date: 2018-03-18 16:51:55
tags: [树链剖分,线段树,堆]
---
#[Hnoi2016 day1]网络 

###### 问题描述

一个简单的网络系统可以被描述成一棵无根树。每个节点为一个服务器。连接服务器与服务器的数据线则看做
一条树边。两个服务器进行数据的交互时，数据会经过连接这两个服务器的路径上的所有服务器（包括这两个服务
器自身）。由于这条路径是唯一的，当路径上的某个服务器出现故障，无法正常运行时，数据便无法交互。此外，
每个数据交互请求都有一个重要度，越重要的请求显然需要得到越高的优先处理权。现在，你作为一个网络系统的
管理员，要监控整个系统的运行状态。系统的运行也是很简单的，在每一个时刻，只有可能出现下列三种事件中的
一种：1.  在某两个服务器之间出现一条新的数据交互请求；2.  某个数据交互结束请求；3.  某个服务器出现故
障。系统会在任何故障发生后立即修复。也就是在出现故障的时刻之后，这个服务器依然是正常的。但在服务器产
生故障时依然会对需要经过该服务器的数据交互请求造成影响。你的任务是在每次出现故障时，维护未被影响的请
求中重要度的最大值。注意，如果一个数据交互请求已经结束，则不将其纳入未被影响的请求范围。

###### 输入格式

第一行两个正整数n,m，分别描述服务器和事件个数。服务器编号是从1开始的，因此n个服务器的编号依次是1
,2,3,…,n。接下来n-1行，每行两个正整数u,v，描述一条树边。u和v是服务器的编号。接下来m行，按发生时刻依
次描述每一个事件；即第i行（i=1,2,3,…,m）描述时刻i发生的事件。每行的第一个数type描述事件类型，共3种
类型：（1）若type=0，之后有三个正整数a,b,v，表示服务器a,b之间出现一条重要度为v的数据交互请求；（2）
若type=1，之后有一个正整数t，表示时刻t（也就是第t个发生的事件）出现的数据交互请求结束；（3）若type=2
，之后有一个正整数x，表示服务器x在这一时刻出现了故障。对于每个type为2的事件，就是一次询问，即询问“
当服务器x发生故障时，未被影响的请求中重要度的最大值是多少？”注意可能有某个服务器自身与自身进行数据
交互的情况。

###### 输出格式

对于每个type=2的事件，即服务器出现故障的事件，输出一行一个整数，描述未被影响的请求中重要度的最大
值。如果此时没有任何请求，或者所有请求均被影响，则输出-1。

###### 数据规模

2 ≤ n ≤ 10^5, 1 ≤ m ≤ 2×10^5，其他的所有输入值不超过 10^9

---



路径问题，容易想到树分治，这道题容易想到树链剖分。

题目中要求维护**未被影响**的请求中重要度的最大值。看似不好做，实际上直接维护即可：利用重链剖分，树上的一条路径被分成了$O(logn)$段连续的区间。这条交互请求不受路径外的点的影响，而路径外的点就是这条路径被分成的区间以外的点。容易发现：**这些点同样形成了若干段区间，而且个数也是$O(logn)$的**，这样就可以用线段树维护了。只需要取出路径对应区间，暴力排序后取出想要的区间即可。

由于要维护最大值，容易想到给每个线段树的节点开一个堆。利用标记永久化思想，每次只修改$O(logn)$个节点上的堆，由于是单点查询，查询时只要一直跳到树根，取路径上堆顶元素的最大值就好。单点查询，区间修改，使用zkw线段树非常好。



时间复杂度：

每一次询问，找到对应区间；$O(logn)$对这些区间排序$O(lognlog(logn))$，可以视为$O(logn)$；修改完所有区间的时间复杂度上界理论上是$O(log^3n)$的，但是由于$zkw$线段树和二叉堆往往跑不满$logn$，所以官方数据是可过的。



正解貌似是整体二分？复杂度和常数都优于这种做法，于是有人加了一组数据把这种做法卡到复杂度上界直接TLE。**卡前向星存图**的还是第一次= =。由于前向星存边是逆序的，用vector顺序存边的话，这组数据就卡不掉了……或者是像下面的代码一样，把树链剖分里找重儿子的判断条件取个等就好（达到与顺序存边相同的效果）。



---



学习了支持删除操作的堆的优秀写法。



```c++
#include<stdio.h>
#include<queue>
#include<algorithm>
#define MAXN 200005
#define MAXM 400005
#define MAXT 800005
#define pr pair
#define fi first
#define se second
#define mp make_pair
using namespace std;

char buf[1<<20],*p1,*p2;
#define GC (p1==p2&&(p2=(p1=buf)+fread(buf,1,1000000,stdin),p1==p2)?0:*p1++)
inline int _R(){
	int v=0,sign=0;char s=GC;
	while((s!='-')&&(s>57||s<48))s=GC;
	if(s=='-')s=GC,sign=1;
	for(;s>47&&s<58;s=GC)v=v*10+s-48;
	return sign?-v:v;
}

struct Heap{
	priority_queue<int>q,h;
	int top(){
		while(q.size()&&h.size()&&q.top()==h.top())q.pop(),h.pop();
		return q.size()?q.top():-1;
	}
	void ins(int v){
		q.push(v);
	}
	void del(int v){
		h.push(v);
	}
};

struct node{
	int a,b,v;
}A[MAXN];

pr<int,int>s[105];int cnt;

int N,M;

int en[MAXM],nex[MAXM],las[MAXN],tot;
void Add(int x,int y){
	en[++tot]=y;
	nex[tot]=las[x];
	las[x]=tot;
}

int sz[MAXN],hson[MAXN],fa[MAXN],anc[MAXN],id[MAXN],dep[MAXN],lab;

void GetSon(int x,int f){
	int i,y;
	sz[x]=1;fa[x]=f;dep[x]=dep[f]+1;
	for(i=las[x];i;i=nex[i]){
		y=en[i];if(y==f)continue;
		GetSon(y,x);
		sz[x]+=sz[y];
		if(sz[y]>=sz[hson[x]])hson[x]=y;
	}
}

void Connect(int x,int f,int aa){
	anc[x]=aa;id[x]=++lab;
	if(hson[x])Connect(hson[x],x,aa);
	int i,y;
	for(i=las[x];i;i=nex[i]){
		y=en[i];if(y==f||y==hson[x])continue;
		Connect(y,x,y);
	}
}

int m=1;
Heap Q[MAXT];

void Modify(int l,int r,int v,int op){
	if(op==1){
		for(l+=m-1,r+=m+1;l^r^1;l>>=1,r>>=1){
			if(~l&1)Q[l^1].ins(v);
			if(r&1)Q[r^1].ins(v);
		}
	}
	else{
		for(l+=m-1,r+=m+1;l^r^1;l>>=1,r>>=1){
			if(~l&1)Q[l^1].del(v);
			if(r&1)Q[r^1].del(v);
		}
	}
}

int GetAns(int p){
	int ret=-1;
	p+=m;
	while(p)ret=max(ret,Q[p].top()),p>>=1;
	return ret;
}

void Build(){
	while(m<=N)m<<=1;
}

void GetSeg(int x,int y){
	cnt=0;
	while(anc[x]!=anc[y]){
		if(dep[anc[x]]<dep[anc[y]])swap(x,y);
		s[++cnt]=mp(id[anc[x]],id[x]);
		x=fa[anc[x]];
	}
	if(dep[x]<dep[y])swap(x,y);
	s[++cnt]=mp(id[y],id[x]);
	sort(s+1,s+cnt+1);
}

void Insert(int x,int y,int v){
	GetSeg(x,y);
	int i;
	if(s[1].fi!=1)Modify(1,s[1].fi-1,v,1);
	for(i=1;i<cnt;i++)if(s[i].se+1<=s[i+1].fi-1)Modify(s[i].se+1,s[i+1].fi-1,v,1);
	if(s[cnt].se!=N)Modify(s[cnt].se+1,N,v,1);
}

void Delete(int x,int y,int v){
	GetSeg(x,y);
	int i;
	if(s[1].fi!=1)Modify(1,s[1].fi-1,v,-1);
	for(i=1;i<cnt;i++)if(s[i].se+1<=s[i+1].fi-1)Modify(s[i].se+1,s[i+1].fi-1,v,-1);
	if(s[cnt].se!=N)Modify(s[cnt].se+1,N,v,-1);
}

void Init(){
	GetSon(1,0);
	Connect(1,0,1);
	Build();
}

int main(){
	int i,x,y,v,ty;
	N=_R();M=_R();
	for(i=1;i<N;i++){
		x=_R();y=_R();
		Add(x,y);Add(y,x);
	}
	Init();
	for(i=1;i<=M;i++){
		ty=_R();
		if(ty==0){
			x=_R();y=_R();v=_R();
			Insert(x,y,v);
			A[i]=(node){x,y,v};
		}
		else if(ty==1){
			x=_R();
			Delete(A[x].a,A[x].b,A[x].v);
		}
		else{
			x=_R();
			printf("%d\n",GetAns(id[x]));
		}
	}
}
```

