---
title:  Hnoi2016 day1最小公倍数
date: 2018-03-18 16:50:55
tags: [分块,并查集]
---

# [Hnoi2016 day1]最小公倍数

###### 问题描述

给定一张N个顶点M条边的无向图(顶点编号为1,2,…,n)，每条边上带有权值。所有权值都可以分解成2^a*3^b的形式。现在有q个询问，每次询问给定四个参数u、v、a和b，请你求出是否存在一条顶点u到v之间的路径，使得路径依次经过的边上的权值的最小公倍数为2^a*3^b。注意：路径可以不是简单路径。
下面是一些可能有用的定义：
最小公倍数：K个数a1,a2,…,ak的最小公倍数是能被每个ai整除的最小正整数。路径：路径P:P1,P2,…,Pk是顶点序列，满足对于任意1<=i<k，节点Pi和Pi+1之间都有边相连。
简单路径：如果路径P:P1,P2,…,Pk中，对于任意1<=s≠t<=k都有Ps≠Pt，那么称路径为简单路径。

###### 输入格式

第一行包含两个整数N和M，分别代表图的顶点数和边数。接下来M行，每行包含四个整数u、v、a、b代表一条顶点u和v之间、权值为2^a*3^b的边。接下来一行包含一个整数q，代表询问数。接下来q行，每行包含四个整数u、v、a和b，代表一次询问。询问内容请参见问题描述。

###### 输出格式

对于每次询问，如果存在满足条件的路径，则输出一行Yes，否则输出一行 No（注意：第一个字母大写，其余字母小写） 。

###### 样例输入

4 5 
1 2 1 3 
1 3 1 2 
1 4 2 1 
2 4 3 2 
3 4 2 2 
5 
1 4 3 3 
4 2 2 3 
1 3 2 2 
2 3 2 2 
1 3 4 4 

###### 样例输出

Yes 
Yes 
Yes 
No 
No 

###### 数据规模
1<=n,q<=50000、1<=m<=100000、0<=a,b<=10^9



---



首先一个比较暴力的想法是，对于每一个询问$(U,V,A,B)$，把$a\leq A$且$ b\leq B$的所有边都加进一个并查集，最后判断$U,V$是否连通、$U,V$所在连通块中$a,b$的最大值是否能够分别取到$A,B$即可。

虽然这样会超时，但是感觉是可优化的，很容易想到把边按照某一维排序，再考虑另一维的处理。

这道题没有强制在线，也可以考虑是否能把询问排序从而优化算法。

将边按照$a$排序并分块，将询问分到$a$的最小值不大于它、$a$的最大值大于它的块中，同一块里的询问按照$b$排序（好像有种莫队的感觉……）。（为方便实现，可以先把边按照$b$排序，再依次分配到对应的块中）

对于一个询问，它能够用到的边是之前的块里$b$满足要求的边、当前块里$a,b$都满足要求的边。对于后一种显然是暴力搞；对于前一种，由于我们已经保证了块内询问是对于$b$有序的了，所以如果之前的块也按照$b$排序，那么就可以利用单调性搞完同一块内的关于$b$的问题了。这里对前面的块的边按$b$排序，可以采用归并的方法略微减少复杂度，也可以直接暴力排序，因为排序次数是不超过块的个数的。

由于对于同一块内的询问关于$a$不单调，所以在当前块内，不同询问可以使用的边是不同的。那么并查集就不能路径压缩了，而要使用按秩合并、支持撤销操作的并查集。

于是这道题就用分块神奇地降低了复杂度。



---

```c++
#include<stdio.h>
#include<algorithm>
#include<cstring>
#include<cmath>
#define MAXN 51234
#define MAXM 123456 
using namespace std;

int tmax(int a,int b,int c){
	if(a>=b&&a>=c)return a;
	if(b>=c)return b;
	return c;
}

int N,M,K,S;
bool Ans[MAXM];

struct node{
	int u,v,a,b,id;
}E[MAXM],Q[MAXM],sel[MAXM];int tot;
struct op{
	int x,y,fy,va,vb,sa;
}stk[MAXM];int cnt;

bool cmpa(node x,node y){
	return x.a==y.a?x.b<y.b:x.a<y.a;
}
bool cmpb(node x,node y){
	return x.b==y.b?x.a<y.a:x.b<y.b;
}

int fa[MAXN],sz[MAXN],maxa[MAXN],maxb[MAXN];

int gf(int x){
	return fa[x]==x?x:gf(fa[x]);
}

void Merge(int x,int y,int a,int b){
	x=gf(x);y=gf(y);
	if(sz[x]<sz[y])swap(x,y);
	stk[++cnt]=(op){x,y,fa[y],maxa[x],maxb[x],sz[x]};
	fa[y]=x;
	maxa[x]=tmax(maxa[x],maxa[y],a);
	maxb[x]=tmax(maxb[x],maxb[y],b);
	if(x!=y)sz[x]+=sz[y];
}

void Del(){
	int i;
	for(i=cnt;i;i--){
		fa[stk[i].y]=stk[i].fy;
		maxa[stk[i].x]=stk[i].va;
		maxb[stk[i].x]=stk[i].vb;
		sz[stk[i].x]=stk[i].sa;
	}
}

int main(){
	int i,j,k,l;
	scanf("%d%d",&N,&M);
	for(i=1;i<=M;i++)scanf("%d%d%d%d",&E[i].u,&E[i].v,&E[i].a,&E[i].b);
	scanf("%d",&K);
	for(i=1;i<=K;i++)scanf("%d%d%d%d",&Q[i].u,&Q[i].v,&Q[i].a,&Q[i].b),Q[i].id=i;
	sort(E+1,E+M+1,cmpa);sort(Q+1,Q+K+1,cmpb);
	S=sqrt(M);
	for(i=1;i<=M;i+=S){
		for(j=1;j<=N;j++)fa[j]=j,maxa[j]=maxb[j]=-1,sz[j]=1;
		tot=0;
		for(j=1;j<=K;j++)if(Q[j].a>=E[i].a&&(i+S>M||E[i+S].a>Q[j].a))sel[++tot]=Q[j];
		sort(E+1,E+i+1,cmpb);
		for(j=1,k=1;j<=tot;j++){
			while(k<i&&E[k].b<=sel[j].b)
			Merge(E[k].u,E[k].v,E[k].a,E[k].b),k++;
			cnt=0;
			for(l=i;l<i+S&&l<=M;l++)if(E[l].a<=sel[j].a&&E[l].b<=sel[j].b)
			Merge(E[l].u,E[l].v,E[l].a,E[l].b);
			int x=gf(sel[j].u),y=gf(sel[j].v);
			if(x==y&&maxa[x]==sel[j].a&&maxb[x]==sel[j].b)Ans[sel[j].id]=true;
			Del();
		}
	}
	for(i=1;i<=K;i++)Ans[i]?puts("Yes"):puts("No");
}
```