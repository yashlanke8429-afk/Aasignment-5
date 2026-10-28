# Assignment-5
import time
from functools import lru_cache


def build_lcs_table(seq1, seq2):
    m, n = len(seq1), len(seq2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if seq1[i - 1] == seq2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp


def reconstruct_lcs(dp, seq1, seq2):
    i, j = len(seq1), len(seq2)
    result = []
    while i > 0 and j > 0:
        if seq1[i - 1] == seq2[j - 1]:
            result.append(seq1[i - 1])
            i -= 1
            j -= 1
        elif dp[i - 1][j] >= dp[i][j - 1]:
            i -= 1
        else:
            j -= 1
    result.reverse()
    return "".join(result) if isinstance(seq1, str) else result


def lcs(seq1, seq2):
    dp = build_lcs_table(seq1, seq2)
    length = dp[len(seq1)][len(seq2)]
    return length, reconstruct_lcs(dp, seq1, seq2), dp


def print_dp_table(dp, seq1, seq2):
    header = ["Ø"] + [str(x) for x in seq2]
    width = max(len(h) for h in header) + 1
    print(" " * (width + 1) + "".join(h.rjust(width) for h in header))
    labels = ["Ø"] + [str(x) for x in seq1]
    for label, row in zip(labels, dp):
        print(label.rjust(width) + " " + "".join(str(v).rjust(width) for v in row))


def memoized_lcs(seq1, seq2):
    @lru_cache(maxsize=None)
    def helper(i, j):
        if i == 0 or j == 0:
            return 0
        if seq1[i - 1] == seq2[j - 1]:
            return 1 + helper(i - 1, j - 1)
        return max(helper(i - 1, j), helper(i, j - 1))
    return helper(len(seq1), len(seq2))


def brute_force_lcs(seq1, seq2):
    def helper(i, j):
        if i == 0 or j == 0:
            return 0
        if seq1[i - 1] == seq2[j - 1]:
            return 1 + helper(i - 1, j - 1)
        return max(helper(i - 1, j), helper(i, j - 1))
    return helper(len(seq1), len(seq2))


if __name__ == "__main__":
    # Example 1: classic
    s1, s2 = "ABCBDAB", "BDCABA"
    print(f'Sequences: "{s1}" vs "{s2}"\n')
    length, subseq, dp = lcs(s1, s2)
    print_dp_table(dp, s1, s2)
    print(f"\nLCS length     : {length}")
    print(f"LCS subsequence: {subseq}")

    # Cross-checks
    print("\nMemoized length   :", memoized_lcs(s1, s2))
    print("Brute-force length:", brute_force_lcs(s1, s2))

    # Example 2: lists of words
    p1 = "walk to the store and buy milk".split()
    p2 = "drive to store buy some milk".split()
    length, subseq, _ = lcs(p1, p2)
    print(f"\nPlaylist example -> length {length}: {subseq}")

    # Timing: brute force vs DP (kept small so it finishes quickly)
    a, b = "AGGTAB" * 2, "GXTXAYB" * 2
    t0 = time.perf_counter()
    bf = brute_force_lcs(a, b)
    t1 = time.perf_counter()
    dp_len = lcs(a, b)[0]
    t2 = time.perf_counter()
    print(f"\nTiming on lengths {len(a)} and {len(b)}:")
    print(f"  Brute force: length {bf}, {t1 - t0:.4f} s")
    print(f"  DP table   : length {dp_len}, {t2 - t1:.6f} s")
