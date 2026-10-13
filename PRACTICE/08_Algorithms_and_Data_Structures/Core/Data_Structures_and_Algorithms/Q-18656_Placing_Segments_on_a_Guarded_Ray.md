---
type: "practice"
difficulty: "Other"
---

# Placing Segments on a Guarded Ray

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Segment Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Segment_Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

There is a one-dimensional ray that begins at coordinate 0 and extends without bound in the positive direction. You process a list of operations, given in order, of two kinds:

Type 1, written [1, p]: install a peg at coordinate p on the ray. It is guaranteed that no peg currently sits at p when this operation is issued.
Type 2, written [2, p, len]: a hypothetical measurement. Decide whether a straight segment of length len could be laid down somewhere inside the interval [0, p] so that the entire segment stays within [0, p] and does not cross the interior of any peg. The segment is allowed to merely touch a peg (share an endpoint with it) but never overlap it. The segment is not actually placed; every type-2 operation is independent, and pegs are never removed.

A segment of length len can be placed inside [0, p] if and only if there exists a free stretch of the ray of length at least len that lies fully inside [0, p] and contains no peg in its interior. The pegs currently in [0, p] split the interval into consecutive free stretches: from 0 up to the first peg, between neighbouring pegs, and from the last peg up to p. The answer is true iff the widest of these stretches is at least len.

Return an array of booleans, one entry per type-2 operation in the order the operations appear. The i-th returned entry is true if the segment described by the i-th type-2 operation can be placed, and false otherwise.

#### Cons
traints

1 <= number of operations <= 1.5 * 10^4 ... 1.5 * 10^5 (i.e. 1 <= ops <= 150000).
Each operation has either 2 fields (type 1) or 3 fields (type 2).
The first field of every operation is 1 or 2.
1 <= p, len <= min(5 * 10^4, 3 * number_of_operations).
For every type-1 operation, no peg exists at p at the moment the operation is processed.
At least one type-2 operation is present.

#### Example
s & test cases
TEXT
Input: ops = [[1,4],[2,6,5],[2,6,3],[2,4,4]] -> Output: [false, true, true]
Why: after pegging 4, the free stretches inside [0,6] are [0,4] (width 4) and [4,6] (width 2); length 5 does not fit (false), 3 fits (true); inside [0,4] the stretch [0,4] has width 4, so length 4 fits exactly (true).

Input: ops = [[1,9],[2,9,8],[1,3],[2,9,6],[2,9,7]] -> Output: [true, true, false]
Why: with only peg 9, [0,9] gives width 9 so length 8 fits; after pegging 3 the widest stretch in [0,9] is [3,9] of width 6, so length 6 fits but 7 does not.

Input: ops = [[2,5,5],[2,5,6]] -> Output: [true, false]
Why: edge case with no pegs yet — the whole [0,5] is free (width 5), so length 5 fits and length 6 does not.

Input: ops = [[1,2],[1,7],[2,7,5],[2,7,6]] -> Output: [true, false]
Why: pegs at 2 and 7 leave the stretch [2,7] of width 5; a length-5 segment fits by touching both pegs (touching is allowed), but length 6 does not.

#### Approach

Process the operations in their original order (temporal correctness is automatic because a measurement only sees pegs installed before it). Keep two structures over coordinates. (1) An "ordered positions" segment tree that, for a range, records the max and min present peg index; this yields the nearest peg to the left of x (floor) and to the right (higher) in O(log M). (2) A "gap" segment tree that stores, at each peg position q, the width of the free stretch immediately to its left (q - previousPeg), and answers range-max. When a peg is installed at x between neighbours L and R, the stretch that ended at R shrinks (assign gapT[R] = R - x) and the new peg's left stretch is gapT[x] = x - L (with L = 0 meaning the origin). For a measurement [2, p, len], the widest usable stretch inside [0, p] is max( range-max of left-gaps over [1, p], p - (largest peg <= p) ); the second term is the free tail after the last peg up to p. Answer true iff that maximum is at least len. The underlying pattern is offline-style range-max maintenance with an ordered set of positions, i.e. segment tree + sorted-set neighbour queries.

Complexity

Time: O((Q + Q log M)) = O(Q log M), where Q is the number of operations and M is the largest coordinate that appears. Each operation performs a constant number of segment-tree walks, each O(log M). Space: O(M) for the two segment trees. Justification: every insert does two neighbour lookups plus up to two point-assigns; every measurement does one range-max plus one floor lookup — all logarithmic.

Python
PYTHON
import sys
from typing import List

class Solution:
    def getResults(self, queries: List[List[int]]) -> List[bool]:
        sys.setrecursionlimit(1 << 20)
        N = 1
        for q in queries:
            if q[1] > N:
                N = q[1]
        size = 4 * (N + 1)
        gapT = [0] * size          # range-max of left-gaps
        pMax = [0] * size          # present pegs: max index (0 = none)
        pMin = [N + 1] * size      # present pegs: min index (N+1 = none)

        def gap_assign(nd, lo, hi, idx, v):
            if lo == hi:
                gapT[nd] = v
                return
            mid = (lo + hi) // 2
            if idx <= mid:
                gap_assign(2 * nd, lo, mid, idx, v)
            else:
                gap_assign(2 * nd + 1, mid + 1, hi, idx, v)
            gapT[nd] = gapT[2 * nd] if gapT[2 * nd] > gapT[2 * nd + 1] else gapT[2 * nd + 1]

        def gap_query(nd, lo, hi, l, r):
            if r < lo or hi < l:
                return 0
            if l <= lo and hi <= r:
                return gapT[nd]
            mid = (lo + hi) // 2
            a = gap_query(2 * nd, lo, mid, l, r)
            b = gap_query(2 * nd + 1, mid + 1, hi, l, r)
            return a if a > b else b

        def pos_set(nd, lo, hi, idx):
            if lo == hi:
                pMax[nd] = idx
                pMin[nd] = idx
                return
            mid = (lo + hi) // 2
            if idx <= mid:
                pos_set(2 * nd, lo, mid, idx)
            else:
                pos_set(2 * nd + 1, mid + 1, hi, idx)
            pMax[nd] = max(pMax[2 * nd], pMax[2 * nd + 1])
            pMin[nd] = min(pMin[2 * nd], pMin[2 * nd + 1])

        def pos_qmax(nd, lo, hi, l, r):        # 0 if none
            if r < lo or hi < l:
                return 0
            if l <= lo and hi <= r:
                return pMax[nd]
            mid = (lo + hi) // 2
            return max(pos_qmax(2 * nd, lo, mid, l, r),
                       pos_qmax(2 * nd + 1, mid + 1, hi, l, r))

        def pos_qmin(nd, lo, hi, l, r):        # N+1 if none
            if r < lo or hi < l:
                return N + 1
            if l <= lo and hi <= r:
                return pMin[nd]
            mid = (lo + hi) // 2
            return min(pos_qmin(2 * nd, lo, mid, l, r),
                       pos_qmin(2 * nd + 1, mid + 1, hi, l, r))

        res: List[bool] = []
        for q in queries:
            if q[0] == 1:
                x = q[1]
                L = pos_qmax(1, 1, N, 1, x - 1) if x - 1 >= 1 else 0
                R = pos_qmin(1, 1, N, x + 1, N) if x + 1 <= N else N + 1
                if R <= N:
                    gap_assign(1, 1, N, R, R - x)
                gap_assign(1, 1, N, x, x - L)
                pos_set(1, 1, N, x)
            else:
                x, sz = q[1], q[2]
                p = pos_qmax(1, 1, N, 1, x)
                tail = x - p
                g = gap_query(1, 1, N, 1, x)
                res.append(max(g, tail) >= sz)
        return res
JavaScript
JAVASCRIPT
/**
 * @param {number[][]} queries
 * @return {boolean[]}
 */
var getResults = function(queries) {
    let N = 1;
    for (const q of queries) if (q[1] > N) N = q[1];

#### cons
t size = 4 * (N + 1);

#### cons
t gapT = new Array(size).fill(0);   // range-max of left-gaps

#### cons
t pMax = new Array(size).fill(0);   // present pegs: max index (0 = none)

#### cons
t pMin = new Array(size).fill(N + 1); // present pegs: min index (N+1 = none)

    function gapAssign(nd, lo, hi, idx, v) {
        if (lo === hi) { gapT[nd] = v; return; }

#### cons
t mid = (lo + hi) >> 1;
        if (idx <= mid) gapAssign(2*nd, lo, mid, idx, v);
        else gapAssign(2*nd+1, mid+1, hi, idx, v);
        gapT[nd] = Math.max(gapT[2*nd], gapT[2*nd+1]);
    }
    function gapQuery(nd, lo, hi, l, r) {
        if (r < lo || hi < l) return 0;
        if (l <= lo && hi <= r) return gapT[nd];

#### cons
t mid = (lo + hi) >> 1;
        return Math.max(gapQuery(2*nd, lo, mid, l, r), gapQuery(2*nd+1, mid+1, hi, l, r));
    }
    function posSet(nd, lo, hi, idx) {
        if (lo === hi) { pMax[nd] = idx; pMin[nd] = idx; return; }

#### cons
t mid = (lo + hi) >> 1;
        if (idx <= mid) posSet(2*nd, lo, mid, idx);
        else posSet(2*nd+1, mid+1, hi, idx);
        pMax[nd] = Math.max(pMax[2*nd], pMax[2*nd+1]);
        pMin[nd] = Math.min(pMin[2*nd], pMin[2*nd+1]);
    }
    function posQMax(nd, lo, hi, l, r) {     // 0 if none
        if (r < lo || hi < l) return 0;
        if (l <= lo && hi <= r) return pMax[nd];

#### cons
t mid = (lo + hi) >> 1;
        return Math.max(posQMax(2*nd, lo, mid, l, r), posQMax(2*nd+1, mid+1, hi, l, r));
    }
    function posQMin(nd, lo, hi, l, r) {     // N+1 if none
        if (r < lo || hi < l) return N + 1;
        if (l <= lo && hi <= r) return pMin[nd];

#### cons
t mid = (lo + hi) >> 1;
        return Math.min(posQMin(2*nd, lo, mid, l, r), posQMin(2*nd+1, mid+1, hi, l, r));
    }

#### cons
t res = [];
    for (const q of queries) {
        if (q[0] === 1) {

#### cons
t x = q[1];

#### cons
t L = (x - 1 >= 1) ? posQMax(1, 1, N, 1, x - 1) : 0;

#### cons
t R = (x + 1 <= N) ? posQMin(1, 1, N, x + 1, N) : (N + 1);
            if (R <= N) gapAssign(1, 1, N, R, R - x);
            gapAssign(1, 1, N, x, x - L);
            posSet(1, 1, N, x);
        } else {

#### cons
t x = q[1], sz = q[2];

#### cons
t p = posQMax(1, 1, N, 1, x);

#### cons
t tail = x - p;

#### cons
t g = gapQuery(1, 1, N, 1, x);
            res.push(Math.max(g, tail) >= sz);
        }
    }
    return res;
};
Java
JAVA
import java.util.*;

class Solution {
    int N;
    long[] gapT;
    int[] pMax, pMin;

    void gapAssign(int nd,int lo,int hi,int idx,long v){
        if(lo==hi){ gapT[nd]=v; return; }
        int mid=(lo+hi)>>>1;
        if(idx<=mid) gapAssign(2*nd,lo,mid,idx,v); else gapAssign(2*nd+1,mid+1,hi,idx,v);
        gapT[nd]=Math.max(gapT[2*nd],gapT[2*nd+1]);
    }
    long gapQuery(int nd,int lo,int hi,int l,int r){
        if(r<lo||hi<l) return 0;
        if(l<=lo&&hi<=r) return gapT[nd];
        int mid=(lo+hi)>>>1;
        return Math.max(gapQuery(2*nd,lo,mid,l,r),gapQuery(2*nd+1,mid+1,hi,l,r));
    }
    void posSet(int nd,int lo,int hi,int idx){
        if(lo==hi){ pMax[nd]=idx; pMin[nd]=idx; return; }
        int mid=(lo+hi)>>>1;
        if(idx<=mid) posSet(2*nd,lo,mid,idx); else posSet(2*nd+1,mid+1,hi,idx);
        pMax[nd]=Math.max(pMax[2*nd],pMax[2*nd+1]);
        pMin[nd]=Math.min(pMin[2*nd],pMin[2*nd+1]);
    }
    int posQMax(int nd,int lo,int hi,int l,int r){       // 0 if none
        if(r<lo||hi<l) return 0;
        if(l<=lo&&hi<=r) return pMax[nd];
        int mid=(lo+hi)>>>1;
        return Math.max(posQMax(2*nd,lo,mid,l,r),posQMax(2*nd+1,mid+1,hi,l,r));
    }
    int posQMin(int nd,int lo,int hi,int l,int r){       // N+1 if none
        if(r<lo||hi<l) return N+1;
        if(l<=lo&&hi<=r) return pMin[nd];
        int mid=(lo+hi)>>>1;
        return Math.min(posQMin(2*nd,lo,mid,l,r),posQMin(2*nd+1,mid+1,hi,l,r));
    }
    public List<Boolean> getResults(int[][] queries){
        N=1;
        for(int[] q:queries) N=Math.max(N,q[1]);
        gapT=new long[4*(N+1)];
        pMax=new int[4*(N+1)];
        pMin=new int[4*(N+1)];
        Arrays.fill(pMin,N+1);
        List<Boolean> res=new ArrayList<>();
        for(int[] q:queries){
            if(q[0]==1){
                int x=q[1];
                int L=(x-1>=1)? posQMax(1,1,N,1,x-1):0;
                int R=(x+1<=N)? posQMin(1,1,N,x+1,N):(N+1);
                if(R<=N) gapAssign(1,1,N,R,(long)R-x);
                gapAssign(1,1,N,x,(long)x-L);
                posSet(1,1,N,x);
            } else {
                int x=q[1], sz=q[2];
                int p=posQMax(1,1,N,1,x);
                long tail=(long)x-p;
                long g=gapQuery(1,1,N,1,x);
                res.add(Math.max(g,tail)>=sz);
            }
        }
        return res;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
    int N;
    vector<long long> gapT;      // range-max of left-gaps
    vector<int> pMax, pMin;      // present pegs: max/min index in a range

    void gapAssign(int nd,int lo,int hi,int idx,long long v){
        if(lo==hi){ gapT[nd]=v; return; }
        int mid=(lo+hi)/2;
        if(idx<=mid) gapAssign(2*nd,lo,mid,idx,v); else gapAssign(2*nd+1,mid+1,hi,idx,v);
        gapT[nd]=max(gapT[2*nd],gapT[2*nd+1]);
    }
    long long gapQuery(int nd,int lo,int hi,int l,int r){
        if(r<lo||hi<l) return 0;
        if(l<=lo&&hi<=r) return gapT[nd];
        int mid=(lo+hi)/2;
        return max(gapQuery(2*nd,lo,mid,l,r),gapQuery(2*nd+1,mid+1,hi,l,r));
    }
    void posSet(int nd,int lo,int hi,int idx){
        if(lo==hi){ pMax[nd]=idx; pMin[nd]=idx; return; }
        int mid=(lo+hi)/2;
        if(idx<=mid) posSet(2*nd,lo,mid,idx); else posSet(2*nd+1,mid+1,hi,idx);
        pMax[nd]=max(pMax[2*nd],pMax[2*nd+1]);
        pMin[nd]=min(pMin[2*nd],pMin[2*nd+1]);
    }
    int posQMax(int nd,int lo,int hi,int l,int r){       // 0 if none
        if(r<lo||hi<l) return 0;
        if(l<=lo&&hi<=r) return pMax[nd];
        int mid=(lo+hi)/2;
        return max(posQMax(2*nd,lo,mid,l,r),posQMax(2*nd+1,mid+1,hi,l,r));
    }
    int posQMin(int nd,int lo,int hi,int l,int r){       // N+1 if none
        if(r<lo||hi<l) return N+1;
        if(l<=lo&&hi<=r) return pMin[nd];
        int mid=(lo+hi)/2;
        return min(posQMin(2*nd,lo,mid,l,r),posQMin(2*nd+1,mid+1,hi,l,r));
    }
public:
    vector<bool> getResults(vector<vector<int>>& queries){
        N=1;
        for(auto& q:queries) N=max(N,q[1]);
        gapT.assign(4*(N+1),0);
        pMax.assign(4*(N+1),0);
        pMin.assign(4*(N+1),N+1);
        vector<bool> res;
        for(auto& q:queries){
            if(q[0]==1){
                int x=q[1];
                int L=(x-1>=1)? posQMax(1,1,N,1,x-1):0;
                int R=(x+1<=N)? posQMin(1,1,N,x+1,N):(N+1);
                if(R<=N) gapAssign(1,1,N,R,(long long)R-x);
                gapAssign(1,1,N,x,(long long)x-L);
                posSet(1,1,N,x);
            } else {
                int x=q[1], sz=q[2];
                int p=posQMax(1,1,N,1,x);
                long long tail=(long long)x-p;
                long long g=gapQuery(1,1,N,1,x);
                res.push_back(max(g,tail)>=sz);
            }
        }
        return res;
    }
};
C
C
#include <stdlib.h>
#include <stdbool.h>

static long long *gapT;      /* range-max of left-gaps, 1-indexed positions */
static int *pMax, *pMin;     /* present pegs: max and min index in a range   */
static int SN;

static void gapAssign(int nd,int lo,int hi,int idx,long long v){
    if(lo==hi){ gapT[nd]=v; return; }
    int mid=(lo+hi)/2;
    if(idx<=mid) gapAssign(2*nd,lo,mid,idx,v);
    else gapAssign(2*nd+1,mid+1,hi,idx,v);
    long long a=gapT[2*nd], b=gapT[2*nd+1];
    gapT[nd]= a>b?a:b;
}
static long long gapQuery(int nd,int lo,int hi,int l,int r){
    if(r<lo||hi<l) return 0;
    if(l<=lo&&hi<=r) return gapT[nd];
    int mid=(lo+hi)/2;
    long long a=gapQuery(2*nd,lo,mid,l,r), b=gapQuery(2*nd+1,mid+1,hi,l,r);
    return a>b?a:b;
}
static void posSet(int nd,int lo,int hi,int idx){
    if(lo==hi){ pMax[nd]=idx; pMin[nd]=idx; return; }
    int mid=(lo+hi)/2;
    if(idx<=mid) posSet(2*nd,lo,mid,idx);
    else posSet(2*nd+1,mid+1,hi,idx);
    pMax[nd]= pMax[2*nd]>pMax[2*nd+1]?pMax[2*nd]:pMax[2*nd+1];
    pMin[nd]= pMin[2*nd]<pMin[2*nd+1]?pMin[2*nd]:pMin[2*nd+1];
}
static int posQMax(int nd,int lo,int hi,int l,int r){   /* 0 if none present */
    if(r<lo||hi<l) return 0;
    if(l<=lo&&hi<=r) return pMax[nd];
    int mid=(lo+hi)/2;
    int a=posQMax(2*nd,lo,mid,l,r), b=posQMax(2*nd+1,mid+1,hi,l,r);
    return a>b?a:b;
}
static int posQMin(int nd,int lo,int hi,int l,int r){   /* SN+1 if none present */
    if(r<lo||hi<l) return SN+1;
    if(l<=lo&&hi<=r) return pMin[nd];
    int mid=(lo+hi)/2;
    int a=posQMin(2*nd,lo,mid,l,r), b=posQMin(2*nd+1,mid+1,hi,l,r);
    return a<b?a:b;
}

bool* getResults(int** queries,int queriesSize,int* queriesColSize,int* returnSize){
    int N=1;
    for(int i=0;i<queriesSize;i++) if(queries[i][1]>N) N=queries[i][1];
    SN=N;
    gapT=(long long*)calloc(4*(N+1),sizeof(long long));
    pMax=(int*)calloc(4*(N+1),sizeof(int));            /* default 0 = none */
    pMin=(int*)malloc(4*(N+1)*sizeof(int));
    for(int i=0;i<4*(N+1);i++) pMin[i]=N+1;
    bool* res=(bool*)malloc((queriesSize>0?queriesSize:1)*sizeof(bool));
    int rc=0;
    for(int i=0;i<queriesSize;i++){
        if(queries[i][0]==1){
            int x=queries[i][1];
            int L=(x-1>=1)? posQMax(1,1,N,1,x-1):0;              /* nearest peg left, or origin 0 */
            int R=(x+1<=N)? posQMin(1,1,N,x+1,N):(N+1);         /* nearest peg right, if any     */
            if(R<=N) gapAssign(1,1,N,R,(long long)R-x);         /* R's left-gap shrinks to R-x   */
            gapAssign(1,1,N,x,(long long)x-L);                  /* new peg's left-gap is x-L     */
            posSet(1,1,N,x);
        } else {
            int x=queries[i][1], sz=queries[i][2];
            int p=posQMax(1,1,N,1,x);                            /* largest peg <= x, 0 if none  */
            long long tail=(long long)x-p;                       /* free stretch after last peg  */
            long long g=gapQuery(1,1,N,1,x);                     /* widest left-gap within [0,x] */
            long long best= g>tail?g:tail;
            res[rc++]= best>=sz;
        }
    }
    *returnSize=rc;
    free(gapT); free(pMax); free(pMin);
    return res;
}

#### Follow-up
questions
What if pegs can also be removed later? (Removals merge two stretches; maintain the gap and ordered-position trees under deletions, or process offline in reverse.)
Change the query to return the maximum segment length placeable in [0, p] instead of a yes/no for a given len.
What if the segment must fit inside an arbitrary window [a, p] rather than [0, p]?
Coordinates can be as large as 10^9 — apply coordinate compression over all coordinates that appear so the trees stay O(number of operations).

