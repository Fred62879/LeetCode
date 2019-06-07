## Specification
Similar to LT\312
Given several boxes with different colors represented by different positive numbers. You may experience several rounds to remove boxes until there is no box left. Each time you can choose some continuous boxes with the same color (composed of k boxes, k >= 1), remove them and get k*k points. Find the maximum points you can get.

Example 1:
Input:
[1,2,2,2,2,3,4,3,1]
Output:
23
Explanation:
[1, 3, 2, 2, 2, 3, 4, 3, 1] 
----> [1, 3, 3, 4, 3, 1] (3*3=9 points) 
----> [1, 3, 3, 3, 1] (1*1=1 points) 
----> [1, 1] (3*3=9 points) 
----> [] (2*2=4 points)
Note: The number of boxes n would not exceed 100.



## Ideas
# (3) indep_2nd visit
The base case is that we remove each element on its own regardless of identical boxes. To increase the result, we need to remove boxes identical in color together. We can think of it in this way, at the last removal, box(es) of one color are
removed. Therefore, we can simulate box of each color to be such last removal. And for each color, if there are x slots of boxes not immediately adjacent in the array, we check possibility for the comination of each. 

For instance, it we have x,x,x,3,3,3,x,x,x,3,3,x,x,x,3,x,x,x; then we check the result if the last removal is 3-3 & 2-3; 3-3 & 2-3 & 1-3; 2-3 & 1-3. Since that should be the last removal, the slots besides that or interclating that should be removed as a unit and can be recursively solved.

To implement this idea efficiently, we record mid as the max for the interclating sequence between two removal boxes and as we recurse, we update this part.

In the 2nd implementation, solve is not used to calculate the value of the subarray from lo to hi but instead take into account the previously accumulated mid. This is problematic as there can be many combinations of final-box-pop that include calculting the same subarray. If we insist use this implem, we have to define each touch uniquely, possibly using a 4D array, with an extra dimension redorcing the accumulated mid although this still may guarantee an absolute unique of each touch of the particular subarray.

If we use solve to calculate only the value of lo->hi there is no problem as this can be shared by all combinations entails it, each of which separately calculate the mid value at the call previous to the current recursion and thus are independent.
Conclusion, still have to insist the 1st implementation although idea differs


# (2) indep
Ajacent boxes with same color must be removed together to maximize the result. We can remove them as we reach them or remove them after removing certain other boxes in the middle. The former return an immediate result, the latter entails the possibility that the adjacent same boxes will further increase length.

Moreover, boxes in the last removal should be all of the same color. Thus, taking the following testcase as an example, we start with 4, assuming it to be the color of the last removed box(es). If only the adjacent two 4 are still presented in the last removal, we count it and traceback recursively to find the max contribution of the rest elements.

If box 4 in loc 8 is also presented with the first two, then all elements in middle must have been removed some time. We thus count this. Then, we also check the rest elements.

Moreover, it is also possible that only box 4 in Loc 0, 1 and 11 are presented in the last removal. We do a recursion for elements from 2 to 10 in this case. Notably, the case where box 4 in 0, 1, 4, and 11 are all presented in last removal is already counted in the recursion from element 5 onward in case of the presence of 0,1,4 in the last removal.

Then, we may also assume that box 3 is the last removed. However, as this indicates the removal of box 4 in 0 and 1, and as we have already figured out max(0,1) + max(2,n-1) this is regarded as repetition.

id   0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
ele  4 4 3 3 2 2 2 3 4 6  5  4  3  2  2  1
 |
shrink
 |
      0 1 2 3 4 5 6 7 8 
count 2 2 3 1 1 1 1 1 2
ele   4 3 2 3 4 6 5 3 2


# (1) CITED
Removed.



## Testcases
[1,2,1]
[5, 9, 6, 5, 6, 9, 6, 9, 5]
[3, 8, 8, 5, 5, 3, 9, 2, 4, 4, 6, 5, 6, 9, 6, 2, 8, 6, 4, 1, 9, 5, 3, 10]
[3, 8, 8, 5, 5, 3, 9, 2, 4, 4, 6, 5, 8, 4, 8, 6, 9, 6, 2, 8, 6, 4, 1, 9, 5, 3, 10, 5, 3, 3  , 9, 8, 8, 6, 5, 3, 7, 4,   9, 6, 3, 9, 4, 3, 5, 10, 7, 6, 10, 7]
[4,4,3,3,2,2,2,3,4,6,5,4,3,2,2,1] - 44
[1,1,1,1,1,1,1,1,1,1,2,1,3,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]



## Codes
# (I) DP_bottom-up
// derived
class Solution {
    
    public int removeBoxes(int[] boxes) {
        int n = boxes.length, i = 0, j = 0;
        List<Integer>count = new ArrayList<Integer>(), ele = new ArrayList<Integer>();
        while (i < n) {
            while (i + 1 < n && boxes[i] == boxes[i+1]) i++;
            count.add(++i -j);
            ele.add(boxes[j]);
            j = i;
        }
        int m = count.size();
        
        int[][][] dp = new int[m][m][n];
        
        // subarray starts from j, ends at j with rep-many same boxes remained
        for (j = 0; j < m; j++) for(int rep = 0; rep < j; rep++) 
                dp[j][j][rep] = (rep+count.get(j))*(rep+count.get(j));
        // bottom up dp
        for (int len = 1; len < m; len++) { // dist between j and i
            for (i = len; i < m; i++) {     // subarray ends at i
                j = i - len;                //          starts from j
                dp[j][i][0] = dp[j][j][0] + dp[j+1][i][0];
                for (int rep = 0; rep <= j; rep++) {
                    for (int c = j + 1; c <= i; c++) { // c between j and i, same as j
                        if (ele.get(j) != ele.get(c)) continue;
                        int curRep = rep + count.get(c);
                        int immed = dp[j+1][c-1][0] + curRep*curRep + dp[c+1][i][0];
                        int delay = dp[j+1][c-1][0] + dp[c][i][rep];
                        dp[j][i][rep] = Math.max(dp[j][i][rep], Math.max(immed, delay));
                    }
                }
            }
        }
        return dp[0][m-1][0];
    }
}
