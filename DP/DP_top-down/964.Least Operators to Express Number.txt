## Specification
Similar: LT818

Given a single positive integer x, we will write an expression of the form x (op1) x (op2) x (op3) x ... where each operator op1, op2, etc. is either addition, subtraction, multiplication, or division (+, -, *, or /).  For example, with x = 3, we might write 3 * 3 / 3 + 3 - 3 which is a value of 3. When writing such an expression, we adhere to the following conventions:
The division operator (/) returns rational numbers.
There are no parentheses placed anywhere.
We use the usual order of operations: multiplication and division happens before addition and subtraction.
It's not allowed to use the unary negation operator (-).  For example, "x - x" is a valid expression as it only uses subtraction, but "-x + x" is not because it uses negation.

We would like to write an expression with the least number of operators such that the expression equals the given target.  Return the least number of operators used.

Example 1:
Input: x = 3, target = 19
Output: 5
Explanation: 3 * 3 + 3 * 3 + 3 / 3.  The expression contains 5 operations.

Example 2:
Input: x = 5, target = 501
Output: 8
Explanation: 5 * 5 * 5 * 5 - 5 * 5 * 5 + 5 / 5.  The expression contains 8 operations.

Example 3:
Input: x = 100, target = 100000000
Output: 3
Explanation: 100 * 100 * 100 * 100.  The expression contains 3 operations.

Note:
2 <= x <= 100
1 <= target <= 2 * 10^8



## Ideas
# (5) Dijkstra

# (4) Recursion

# (3) DP_bottom-up

# (2) DP_top-down
// indep
Notable points here are: (a) division is only performed when generating 1 since it does not make sense to write sth like: a * a / a. (b) multiplication is the only operator that should be constantly employed. (c) plus and minus are carried out when separating blocks of mulipl and divi operations.

Thus, differing from LT818, for a given integer, there is no need here to worry about how much to exceed and how much from the target shall we stop and resume. As we are trying to achieve target in shortest time possible, now there are only two options each time, e.g. for target = 16, x = 5, we only consider 15 and 20 because what else may be chosen (10/25) would require extra steps to reach 15 or 20 from which on the time to reach target would be the same.

In this case, we are sort of disclosing the series of blocks of operations sum up to target (e.g. 94 = 3^4 + 3^2 + 3 + 1 or ...). One way to do this is to constantly zero-out blocks. For instance, for the single 1 above, it must be generated based on 3/3 which decisively require two steps (/ and + by others). After removing "1" block, divide the rest by 3 gives 3^3 + 3 + 1. Now this new "1" can also be removed but it does not involve division. 

Instead, if we record how many block-wise division by 3 have been done (1 by now), we would know that this "1" originates from 3. And if we block-wise divide further, when we have 3^2 + 1, the "1" is the residue of 3*3.

// formal proof
https://leetcode.com/problems/least-operators-to-express-number/discuss/209552/C%2B%2B-DFS-using-memorization-with-detailed-explanation-O(log(target))-time-4ms-and-beats-93

(I) If target is not divisible by x, say the residual is r, we define c = x-r then we can first express target as target = t1 + r or target = t2 - c, where t1 and t2 are divisble by x, and 0 < r, c < x. The only way to express r and c is to use / or - or +. Let's take r for example, we can either express it as summation of r 1's (2r x's and 2r-1 operators), or as x substract c 1's (1+2c x's and 2c operators). Take the min from these two.

(II) If target is divisble by x, suppose target = a1*x^{p1} + a2*x^{p2} + ... + ak*x^{pk} is the optimal expression, (1 <= p1 < p2 < ... < pk), and we use nx = a1*p1 + a2*p2 + ... + ak*pk x's and nx - 1 operators, then we have |ai| < x for each i. 

Why? Let's say |ai|>=x for some i, and without loss of generality, we assume ai is positive, then we can express ai*x^pi by (ai-x)*x^pi + x^{pi+1}, which uses (ai-x)*pi + pi + 1 < ai*pi x's, and this contradicts the assumption that the expression is optimal. With this constraint we can easily show that the most significant x-git, ak must be positive.

Let's back up a little bit. If we enforce that all the x-gits should be non-negative, then clearly the optimal expression is the standard definition of x-ary expression: target = b1*x^{q1} + b2*x^{q2} + ... + bt*x^{qt}, where q1 < q2 < ... < qt and bi = target % (x^{qi}) for each i. What is its relation to the optimal expression?

Well, it can be a candiate for the optimal solution, but can we do better? Obviously, we should not introduce an x-git lower than q1. For the least significant x-git component b1*x^{q1}, we can either keep as it as, and try to find the optimal expression of target - b1*x^{q1}, or we reformulate it as x^{q1+1} - (x - b1)*x^{q1}, with - (x - b1) being the q1-th x-git, and try to find the optimal expression of x^{q1+1} + b2*x^{q2} + ... + bt*x^{qt}. Note that in either way we clear one least significant x-git component from target, and we are trying to convert its standard x-ary expression to an optimal expression. Since any number contains limited x-gits, the recursion will exit eventually.

Now let's do it in a programmatic way. Since given any positive number target, its standard x-ary expression is unique, we can define a routine that takes target and x as parameters, and returns the minimum number of x's used to express target. Let's name the routine as f(target, x), and inside the routine we compute the least significant x-git component target, say lsc=b1*x*{q1}, then we recursively call f(target-lsc, x) and f(target+x^{q1}-lsc, x), and return min(b1*q1+f(target-lsc, x), (x-b1)*q1+f(target+x^{q1}-lsc, x)). Since f routine returns number of x's used, the final solution will be f(target, x) - 1. Note that target may have been already evaluated, so we can use memorization (dynamic programming) to improve time efficiency.

Time and space complexity: target has O(log(target)) x-gits, which is the recursion depth, and in each recursion we change 2 x-gits, so the total number of disticnt values we evaluate is O(log(target)). Since we use memorization, the time complexity is linear to the number of evaluations, which is O(log(target)) and the space complexity is also O(log(target)).

Proof of correctness: when target is divisble by x, the optimal expression cannot include division. If there is "... + x*x/x + ...", we can have an expression "... + x + ..." using less operators. If there is "...+x/x+...", then the expression will split target into two numbers undivisable by x, say the residuals are r1 and r2 respectively, where r1+r2=x, then seperately expressing them requires at least three operators --- two '/' and one '+', as discussed in the second paragraph, which is far warse than the case that we do not split them. Therefore, the optimal expression can only contain +, -, and *. Since we cannot use parentheses, the multiplication will always take priority. So the expression is natuarally an x-ary expression expression of target, where each non-zero x-git can be either positive or negative.

# (1) BFS


## Code
# (2) DP-top-down
// derived
class Solution {
public:
    unordered_map<string, int> dp;
    int x;
    
    int value(int i) {
        return i > 0 ? i : 2;
    }
    
    // min # of operators to nullify all blocks given v# division performed
    // every block is preceded with an extra "+"
    int solve(int t, int v) {
        string s = to_string(t) + to_string(v); // use int string for mapping
        if (dp.find(s) != dp.end()) return dp[s];
        
        int ans;
        if (t == 1) ans = value(v);
        else if (t == x) ans = 1 + v;
        else if (t == 0) ans = 0;
        else {
            int p = t / x, r = t % x;
            int a = value(v)*r + solve(p, v + 1);
            int b = value(v)*(x-r) + solve(p + 1, v + 1);
            ans = min(a, b);
        }
        dp[s] = ans;
        return ans;
    }
    
    int leastOpsExpressTarget(int x, int target) {
        this->x = x;
        return solve(target, 0) - 1; // preced block with "+", need to minus that of the 1st blk
    }
};

// indep_adapted
class Solution {
public:
    // min # of operators to achieve ct given current value being cv
    int solve(long t, int x, unordered_map<int, int>& dp) {
        if (dp.find(t) != dp.end()) return dp[t];
        int power = log(t) / log(x); long lo = pow(x, power), hi = pow(x, power + 1);
        
        int cur = power + solve(t - lo, x, dp); // furtherest not exceeding
        if (t > hi / 2) cur = min(cur, power + 1 + solve(hi - t, x, dp)); // just exceed
        
        dp[t] = cur;
        return cur;
    }
    
    int leastOpsExpressTarget(int x, int target) {
        if (x == target) return 0;
        int power = log(target) / log(2), lo = pow(x, power), hi = x * lo;
        unordered_map<int, int> dp;
        dp[0] = 0; dp[1] = 2;
        for (int i = 2; i < x; i++) dp[i] = min(2*i, 1 + (x-i)*2); // *** NOTE ***
        for (int i = 1; i <= power; i++) dp[pow(x, i)] = i;
        return solve((long)target, x, dp) - 1;
        cout << dp[40] << ", " << dp[243] << ", " << dp[81] << ", " << dp[1];
        // return 0;
    }
};



## Testcases
78
19840328

2
125046

2
28125082

3
5 (or 14)
