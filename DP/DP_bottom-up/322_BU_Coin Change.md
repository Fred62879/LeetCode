## Specification
You are given coins of different denominations and a total amount of money amount. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return -1.

Example 1:
Input: coins = [1, 2, 5], amount = 11
Output: 3 
Explanation: 11 = 5 + 5 + 1

Example 2:
Input: coins = [2], amount = 3
Output: -1

Note: You may assume that you have an infinite number of each kind of coin.



## Testcases
[1]
0



## Ideas
# (II) DFS_pruning/backtracking

# (I) DP_bottom-up
Trivial



## Code
# (II) DFS
// optimized
class Solution {
public:
    // min # of coins to cover t using coins[idx:0]
    int dfs(int t, int idx, int num, vector<int>& coins) {
        if (idx < 0) return -1;
        if (!(t % coins[idx])) return t / coins[idx];
        
        int n = t / coins[idx], cur = INT_MAX;
        for (int i = n; i >= 0; i--) {
            int need = dfs(t - i * coins[idx], idx - 1, i + num, coins);
            if (need != -1) cur = min(cur, need);
        }
        return cur == INT_MAX ? -1 : cur;
    }
    
    int coinChange(vector<int>& coins, int amount) {
        sort(coins.begin(), coins.end());
        return dfs(amount, coins.size() - 1, 0, coins);
    }
};

// mem_TLE
class Solution {
public:
    vector<vector<int>> dp;
    
    // min # of coins to cover t using coins[idx:0]
    int dfs(int t, int idx, vector<int>& coins) {
        if (idx < 0) return -1;
        if (dp[idx][t] != -1) return dp[idx][t];
        if (!(t % coins[idx])) return t / coins[idx];
        
        int n = t / coins[idx], cur = INT_MAX;
        for (int i = n; i >= 0; i--) {
            int need = dfs(t - i * coins[idx], idx - 1, coins);
            if (need != -1) cur = min(cur, i + need);
        }
        return (dp[idx][t] = cur == INT_MAX ? -1 : cur);
    }
    
    int coinChange(vector<int>& coins, int amount) {
        sort(coins.begin(), coins.end());
        dp.assign(coins.size(), vector<int>(amount + 1, -1));
        return dfs(amount, coins.size() - 1, coins);
    }
};

// prototype_indep_TLE
class Solution {
public:
    // min # of coins to cover t using coins[idx:0]
    int dfs(int t, int idx, vector<int>& coins) {
        if (idx < 0) return -1;
        if (!(t % coins[idx])) return t / coins[idx];
        
        int n = t / coins[idx], cur = INT_MAX;
        for (int i = n; i >= 0; i--) {
            int need = dfs(t - i * coins[idx], idx - 1, coins);
            if (need != -1) cur = min(cur, i + need);
        }
        return cur == INT_MAX ? -1 : cur;
    }
    
    int coinChange(vector<int>& coins, int amount) {
        sort(coins.begin(), coins.end());
        return dfs(amount, coins.size() - 1, coins);
    }
};


# (I) DP
// indep
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, INT_MAX);
        dp[0] = 0;
        for (int i = 1; i <= amount; i++)
            for (int coin : coins) if (coin <= i && dp[i - coin] != INT_MAX)
                dp[i] = min(dp[i], dp[i - coin] + 1);
        // for (int i : dp) cout << i << endl;
        return dp[amount] == INT_MAX ? -1 : dp[amount];
    }
};
