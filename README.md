# Container With Most Water: Two Pointers Explained Step by Step, With the Proof, the Code, and the Mistakes That Cost Offers

The solution to Container With Most Water is eight lines long. Writing it isn't the hard part. The hard part is answering the follow-up question interviewers actually ask: why is it safe to throw away the shorter line? If you can't answer that, you've memorized a solution instead of learning a pattern, and the next array problem will knock you over.

What follows is the full breakdown of LeetCode 11 (Medium, tagged Array, Two Pointers, Greedy): what the problem is really asking, why brute force isn't acceptable, why the greedy pointer move is provably correct, a full step-by-step trace, working code in Python, Java, and C++, the complexity math, and the specific bugs that show up in practice. At the end there's a short section on how to actually retain this pattern, since that's the part most solutions pages skip.

## What the problem is actually asking

You get an array `height` of non-negative integers. Each element is a vertical line standing at position `i`, running from `(i, 0)` to `(i, height[i])`. Pick two lines. Together with the x-axis they form a container shaped like a rectangle, and you want the one that holds the most water.

Two rules matter:

- The container cannot be slanted, so the water level is capped by the **shorter** of the two lines.
- The width is the index distance between them, which is `j - i`, **not** `j - i + 1`.

That gives the only formula you need:


area = (j - i) * min(height[i], height[j])


Constraints on the LeetCode version: `2 <= height.length <= 100000` and `0 <= height[i] <= 10000`. Those bounds are the whole point. At `10^5` elements, an `O(n²)` scan is 5 billion pair evaluations, which times out well before it finishes.

Example: `height = [1,8,6,2,5,4,8,3,7]` returns `49`. The winning pair is index 1 (`8`) and index 8 (`7`): width 7, shorter side 7, area 49.

## Why brute force gets rejected

The direct approach checks every pair:

python
def maxArea_brute(height):
    best = 0
    for i in range(len(height)):
        for j in range(i + 1, len(height)):
            best = max(best, (j - i) * min(height[i], height[j]))
    return best


It's correct, and it's useless at scale. **Time: O(n²). Space: O(1).** Interviewers won't accept it, but it's worth writing down anyway because it gives you a reference implementation to test the fast version against.

## The two-pointer move that makes it O(n)

Start with the widest container available: `left = 0`, `right = n - 1`. Compute the area. Then move **one** pointer, and it has to be the one sitting on the shorter line.

The first half of that is easy to see. Starting at the two extremes gives you the maximum possible width, and every subsequent step trades width for a chance at more height. The second half is where people guess.

### The correctness argument, in three sentences

Suppose `height[left] <= height[right]` at some point. The current area is `height[left] * (right - left)`.

Now consider any other container that still uses `left` as its boundary: it pairs `left` with some index `k` where `left < k <= right`. Its area is `min(height[left], height[k]) * (k - left)`, which is at most `height[left] * (k - left)`, which is at most `height[left] * (right - left)` — because `k <= right`, so `k - left <= right - left`.

That's it. Every container using `left` is bounded by the area you just computed, so `left` can never appear in a better answer. Discard it permanently and move on. The same argument applies symmetrically when `height[right] < height[left]`.

The intuition in one line: the shorter line is a hard ceiling on height. Moving the taller line inward shrinks the width while the ceiling stays exactly where it was, so the area cannot improve.

### Why moving the taller line is always a dead end

Keep the shorter line `h[l]` and move the taller one inward to some `k < r`. Width drops from `r - l` to `k - l`. Height is still capped at `h[l]`, or possibly lower. Two factors, one strictly smaller and the other non-increasing, so the product cannot go up. There is no case where this helps, which is why "just move both inward" or "move the taller one and see" versions of the solution fail on tests rather than fail in theory.

### The algorithm, written out

1. `left = 0`, `right = n - 1`, `best = 0`.
2. While `left < right`: compute `area = min(height[left], height[right]) * (right - left)`, update `best`.
3. If `height[left] < height[right]`, increment `left`. Otherwise decrement `right`. (When the two heights are equal, move either; the argument above still holds.)
4. Return `best`.

AlgoMonster's editorial for this problem builds the same reasoning with its own smaller trace on `[1,3,2,4,2]` returning 6, which is worth reading alongside a longer example because the shorter array makes the pruning visible. 👉 [Read AlgoMonster's full walkthrough for Container With Most Water](https://algo.monster/liteproblems/11?fpr=li69)

## Full trace on [1,8,6,2,5,4,8,3,7]

| Step | left | right | h[left] | h[right] | Width | Area | Best | Move |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 0 | 8 | 1 | 7 | 8 | 8 | 8 | left++ (1 < 7) |
| 2 | 1 | 8 | 8 | 7 | 7 | **49** | **49** | right-- (7 < 8) |
| 3 | 1 | 7 | 8 | 3 | 6 | 18 | 49 | right-- |
| 4 | 1 | 6 | 8 | 8 | 5 | 40 | 49 | right-- (tie) |
| 5 | 1 | 5 | 8 | 4 | 4 | 16 | 49 | right-- |
| 6 | 1 | 4 | 8 | 5 | 3 | 15 | 49 | right-- |
| 7 | 1 | 3 | 8 | 2 | 2 | 4 | 49 | right-- |
| 8 | 1 | 2 | 8 | 6 | 1 | 6 | 49 | right-- |

After step 8, `left` and `right` both land on index 1, the loop condition fails, and the function returns 49. Eight iterations, no pair checked twice, and the optimum found at step 2 is never beaten afterward. That's the shape of the whole algorithm: it takes at most `n - 1` iterations regardless of the input.

## Working code

**Python**

python
class Solution:
    def maxArea(self, height: list[int]) -> int:
        left, right = 0, len(height) - 1
        best = 0
        while left < right:
            area = (right - left) * min(height[left], height[right])
            if area > best:
                best = area
            if height[left] < height[right]:
                left += 1
            else:
                right -= 1
        return best


**Java**

java
class Solution {
    public int maxArea(int[] height) {
        int left = 0, right = height.length - 1, best = 0;
        while (left < right) {
            int width = right - left;
            int h = Math.min(height[left], height[right]);
            best = Math.max(best, width * h);
            if (height[left] < height[right]) left++;
            else right--;
        }
        return best;
    }
}


**C++**

cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0, right = height.size() - 1, best = 0;
        while (left < right) {
            int width = right - left;
            best = max(best, width * min(height[left], height[right]));
            if (height[left] < height[right]) left++;
            else right--;
        }
        return best;
    }
};


## Complexity

**Time: O(n).** Each iteration moves one pointer one step inward, and the loop ends when the pointers meet. Total pointer movement is at most `n - 1`, so the work is linear.

**Space: O(1).** Two integer indices plus a running maximum, no auxiliary array.

O(n) is also optimal here, and it's worth saying out loud in an interview. Any correct algorithm has to consider what the maximum element could contribute, and with the array unsorted there's no way to know that without reading it, so the problem is `Ω(n)`.

## Common mistakes that show up in practice

**Moving the taller pointer.** The most common error, and the one that produces wrong answers on real test cases rather than a crash, which makes it harder to catch.

**Off-by-one on width.** The width is `right - left`. Using `right - left + 1` inflates every area by the minimum height of that pair, and on `[1,8,6,2,5,4,8,3,7]` it returns 56 instead of 49.

**Confusing this with Trapping Rain Water.** Same array shape, same water metaphor, completely different problem. Here you pick exactly two walls and compute a single rectangle. Trapping Rain Water sums the water sitting above *every* position between the walls. If you start writing a running maximum of the left and right sides, you've switched problems.

**Forgetting to track the maximum.** Returning the last computed area, or the area at termination, gives you a small value instead of the answer. The maximum is usually found early, not at the end.

**Assuming the tallest line matters.** Two tall lines close together can lose to two shorter lines far apart. The height 8 at index 1 is the tallest line in the array, and it participates in the winning pair, but step 6 shows it paired with a height of 5 for an area of 15, seven times worse than the winner.

## Edge cases worth testing

- `[1,1]` → 1. The smallest valid input.
- `[1,1,1,1]` → 3. Width 3, height 1. Confirms the width formula.
- `[2,2,2]` → 4. All heights equal, so every move is a tie and either pointer is valid.
- Arrays containing zeros, e.g. `[0,5,0]` → 0. The container is capped by zero, so the answer is legitimately 0. Code that initializes `best` to a nonzero value or treats 0 as "no answer" gets this wrong.
- `height = [4,3,2,1,4]` → 16. Width 4, height 4.

## Container With Most Water vs Trapping Rain Water

|  | Container With Most Water (LeetCode 11) | Trapping Rain Water (LeetCode 42) |
| --- | --- | --- |
| What you compute | Area of one container picked from two lines | Total water trapped across all positions |
| Formula | `(j - i) * min(h[i], h[j])` | Sum of `min(maxLeft, maxRight) - h[i]` over interior indices |
| Difficulty | Medium | Hard |
| Time / space | O(n) / O(1) | O(n) / O(1) with the two-pointer version, O(n) / O(n) with prefix arrays |
| Pointer rule | Move the shorter line | Move the pointer with the smaller running max |
| Typical mistake | Moving the taller line | Mixing up the two problems and summing a single rectangle |

Both show up in the same interview loops, and doing them back to back is the fastest way to make sure your brain routes them to different patterns.

## How to actually retain the pattern instead of re-deriving it

Container With Most Water is a **converging two-pointer** problem: two indices at opposite ends, one monotonic rule deciding which one moves, and a running best value. The same skeleton answers container-style maximization, sorted-pair-sum problems, and a handful of string problems. What separates people who can solve it cold from people who can solve it the second time is whether the rule is stored as a rule or as a memorized answer.

That's the gap AlgoMonster is built around. Instead of a bank of problems with per-problem editorials, it groups problems under reusable patterns and adds decision flowcharts for the "which pattern is this?" step, which is usually the actual bottleneck under interview pressure. The editorial for LeetCode 11 walks the two-pointer reasoning, the greedy justification, and a traced example, and the platform's content credits Google engineers and competitive programmers.

Worth knowing before you pay:

- Explanations are **text and diagrams**, not video. If you learn faster from video walkthroughs, budget for that separately.
- The curriculum is pattern-organized rather than a 3,000-problem bank. It's built for understanding categories of problems, not for grinding volume.
- System design coverage is introductory. For senior-level loops you'll need something else alongside it.
- Speedrun mode and the AI assistant are useful for spaced review once the patterns are already in your head. They aren't a substitute for the first pass.

Reviews of the platform generally land on the same conclusion: it's a good fit for developers who already write code fluently and need to get faster at recognizing which approach applies, and a poor fit for anyone two weeks out from an interview or still shaky on language fundamentals.

## AlgoMonster plans and pricing

AlgoMonster has three plans. There's no monthly option, so the choice is annual versus one-time lifetime access. Prices move with promotions, so verify the current number on the checkout page rather than trusting an old coupon roundup.

| Plan | Price | Billing | What you get | Get it |
| --- | --- | --- | --- | --- |
| Free | $0 | — | Limited access to a subset of patterns and problems; enough to see whether the text-and-diagram teaching style works for you | [Start with the free plan](https://algo.monster/subscribe?fpr=li69) |
| Pro – Yearly | $99/year (about $8.25/month) | Billed annually, 12 months of full access | All patterns, lessons and problems; unlimited AI assistant usage; unlimited Speedrun; real questions from top tech companies updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials; member discount on live bootcamps | [Check the current Pro-Yearly price](https://algo.monster/subscribe?fpr=li69) |
| Pro – Lifetime | $189 | Billed once | Everything in the annual plan, plus lifetime updates and an internal referral to a top tech company for qualified candidates | [Grab lifetime access](https://algo.monster/subscribe?fpr=li69) |

Two things to check before buying, because the numbers on the subscribe page have changed during sales: the yearly plan has been listed as low as $69.99/year (about $5.84/month), and lifetime access has appeared at $89. Those are promotion-dependent, not permanent prices. Also note that a couple of specialized courses, such as the quant interview prep track, are sold separately and aren't covered by a Pro subscription.

Which plan makes sense depends on one question: will you use this more than once? Complete the curriculum in a six-week sprint and never open it again, and the annual plan is the cheaper, better-fitting purchase. Expect to interview again in two or three years during a job search, and lifetime access stops being a splurge. Both paid tiers include the bonus Foundation course and the build-an-LLM course at no extra cost, which is worth a look if you're also shoring up fundamentals.

## FAQ

**Can I move either pointer when the two heights are equal?**
Yes. Run the argument again with `height[left] == height[right]`. Any container keeping `left` and pairing it with `k < right` has area at most `height[left] * (k - left)`, which is strictly less than the current `height[left] * (right - left)`. So `left` is safe to discard, and the identical argument clears `right`.

**Is O(n) really optimal, or is there a trick?**
There's no trick. You need to examine every line to know the maximum height, so linear time is a floor. Any sublinear claim would be wrong.

**Do I need to sort anything?**
No, and sorting would be a mistake. Sorting destroys the index positions, and the width depends on the original indices.

**Why is this problem considered harder than its code suggests?**
Because the code is trivial once you know the rule, and the rule isn't obvious. It's a good test of whether a candidate can justify a greedy choice rather than pattern-match to one.

## The short version

Work outward in. Start at both ends, compute `min(h[l], h[r]) * (r - l)`, and always move the pointer on the shorter line, because every container still using that shorter line is capped by the area you just measured. One pass, two integer pointers, `O(n)` time and `O(1)` space. Test it on `[1,8,6,2,5,4,8,3,7]` for 49, and on `[0,5,0]` for 0 to make sure your initialization isn't hiding a bug.

If the goal is to have this rule available on demand in an interview rather than after twenty minutes of re-deriving it, that's a practice problem, and it's the kind of practice AlgoMonster is organized for. 👉 [See the current plans and pricing](https://algo.monster/subscribe?fpr=li69)
