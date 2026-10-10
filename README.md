# single number: the XOR trick that solves LeetCode 136 in one pass, plus the II and III follow-ups without memorizing bit hacks

People search "single number" for two different reasons. Some are staring at LeetCode 136 with a hash set that passes on time but blows the "constant extra space" requirement. Others have already seen the three-line XOR solution and don't believe it should work.

Both questions have the same answer, and it fits in three lines of code. What follows is the problem as it's actually asked, the four approaches ranked by what the interviewer is grading, the math behind XOR that makes the trick obvious instead of magic, and then the two follow-ups that trip up people who learned the trick by memorizing it.

## What the problem actually asks

The statement is short, which is part of the trap.

> Given a non-empty array of integers `nums`, every element appears twice except for one. Find that single one. You must implement a solution with linear runtime complexity and use only constant extra space.

Examples:

- `[2,2,1]` → `1`
- `[4,1,2,1,2]` → `4`
- `[1]` → `1`

Constraints worth noticing: the array length goes up to 3 × 10⁴, values can be negative (down to −3 × 10⁴), and exactly one element appears once. Every other element appears exactly twice.

The two requirements are what separate the real solution from the one most people write first. "Linear runtime" rules out the nested-loop scan. "Constant extra space" rules out the hash set, which is the answer most candidates reach for because it's the one that comes to mind immediately.

## Four ways to solve it, ranked by what gets accepted

| Approach | Time | Space | Where it breaks |
| --- | --- | --- | --- |
| Nested loop scan | O(n²) | O(1) | Times out on large inputs |
| Hash set toggle | O(n) | O(n) | Fails the constant-space requirement |
| Sort, then scan pairs | O(n log n) | O(1) to O(n) | Destroys the input order; still not linear |
| XOR accumulation | O(n) | O(1) | Nothing. This is the intended answer |

**Brute force** compares each element against every other one. It's correct and it's the version you'd write in a first pass, but it's O(n²) and will time out.

**Hash set toggle** is the intuitive optimization. Iterate through the array; if the number is already in the set, remove it, otherwise add it. Duplicates pair up and cancel, leaving only the single number. It's O(n) time and O(n) space — clean, and a perfectly reasonable answer if the question doesn't care about space. It does care here.

**Sorting** groups duplicates next to each other, so you can walk the array two at a time and return the element that doesn't match its neighbor. O(n log n), which misses the linear requirement, and you've modified the caller's array.

**XOR accumulation** is the answer the question is built around. Initialize a result to 0, XOR every element into it, return the result. One pass, one variable.

## Why XOR works, explained without hand-waving

XOR (the `^` operator) has three properties that matter:

- `x ^ 0 = x` — XOR with zero changes nothing
- `x ^ x = 0` — a number XORed with itself disappears
- XOR is commutative and associative, so the order of operations doesn't matter

Put those together and the whole problem collapses. Every element that appears twice cancels itself out. The only value with no partner left behind is the single number.

Watch it happen on `[4,1,2,1,2]`, processing left to right:

1. Start with `result = 0`
2. `0 ^ 4 = 4`
3. `4 ^ 1 = 5`
4. `5 ^ 2 = 7`
5. `7 ^ 1 = 6`
6. `6 ^ 2 = 4`

The intermediate values look like noise, and that's the part that makes people distrust the solution. The ordering is a distraction. Because XOR is associative, you're allowed to regroup:

`4 ^ 1 ^ 2 ^ 1 ^ 2  =  4 ^ (1 ^ 1) ^ (2 ^ 2)  =  4 ^ 0 ^ 0  =  4`

Both pairs vanish, and 4 is what's left, regardless of where the duplicates sit in the array.

## The solution in five languages

Python:

python
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        res = 0
        for num in nums:
            res ^= num
        return res


Java:

java
class Solution {
    public int singleNumber(int[] nums) {
        int res = 0;
        for (int num : nums) res ^= num;
        return res;
    }
}


C++:

cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res = 0;
        for (int num : nums) res ^= num;
        return res;
    }
};


Go:

go
func singleNumber(nums []int) int {
    res := 0
    for _, num := range nums {
        res ^= num
    }
    return res
}


JavaScript:

js
var singleNumber = function(nums) {
    return nums.reduce((a, b) => a ^ b, 0);
};


Python one-liner, if you'd rather use the standard library:

python
from functools import reduce
from operator import xor

class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        return reduce(xor, nums, 0)


All of them are O(n) time and O(1) space.

## Where people actually lose points

The XOR solution is four lines, so the interview signal isn't in the code. It's in whether you can say why it's correct.

**Reaching for the hash set first and stopping there.** It's a fine starting point to say out loud, and it shows you can solve the problem. Stopping there skips the entire point of the question. Use it as a stepping stone: "Here's O(n) time with O(n) space — can I get rid of the space?"

**Confusing `^` with `|` or `&`.** In a few languages `^` is easy to mistype when you're writing quickly under pressure. `|` accumulates bits and never cancels anything.

**Assuming the input is sorted.** It isn't. The solution doesn't need it, but any pair-scanning approach you sketch as an alternative does.

**Worrying about negative numbers.** XOR operates on the bit representation of the value, and two's complement numbers are still self-inverse, so `-5 ^ -5 = 0` and the algorithm handles negative single numbers correctly. That comfort does not carry over to the bit-counting approach in Single Number II, as you'll see below.

## Single Number II: three copies instead of two

Now every element appears three times except one, which appears once. Example: `[2,2,3,2]` → `3`.

Double XOR doesn't help — three copies of a number don't cancel to zero. The standard approach switches from XOR to bit counting: for each of the 32 bit positions, count how many numbers have a 1 there. Every bit contributed by the tripled numbers is a multiple of three. Take the count modulo 3, and whatever remains belongs to the single number.

python
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        ans = 0
        for i in range(32):
            cnt = sum(num >> i & 1 for num in nums)
            if cnt % 3:
                if i == 31:
                    ans -= 1 << 31   # sign bit, Python needs this form
                else:
                    ans |= 1 << i
        return ans


That bit-31 special case is not decoration. In Python's two's complement representation, setting the sign bit with `|= 1 << 31` gives you the wrong number; subtracting does it correctly. This is exactly the kind of detail that shows up in the constraints (values go down to −2³¹) and breaks solutions that were tested only on positive inputs.

Complexity is O(32n), which is O(n), with O(1) space.

## Single Number III: two uniques in one array

The third variation: every element appears twice except two elements, which each appear exactly once. Return both. Example: `[4,1,2,1,2,3]` → `[3,4]`.

XOR the whole array and the duplicates cancel, leaving `xs = a ^ b` where `a` and `b` are the two unique values. That single number isn't either answer, but it does tell you something useful: every bit where `xs` has a 1 is a bit where `a` and `b` differ.

Isolate the lowest such bit with `lb = xs & -xs`, then partition the array into numbers that have that bit set and numbers that don't. The two unique values are guaranteed to land in different groups, and all the pairs stay together. XOR each group separately and you have both answers.

python
class Solution:
    def singleNumber(self, nums: List[int]) -> List[int]:
        xs = 0
        for num in nums:
            xs ^= num
        lb = xs & -xs          # lowest bit where the two uniques differ
        a = b = 0
        for num in nums:
            if num & lb:
                a ^= num
            else:
                b ^= num
        return [a, b]


Same linear time, same constant space. The interesting move isn't the XOR — it's realizing the XOR result is a map of where the two answers disagree.

## The pattern underneath all three

LeetCode 136, 137, and 260 are one idea wearing three costumes: use bit-level properties to track state without allocating memory. Once you've internalized `a ^ a = 0` and the count-mod-k generalization, the follow-ups stop looking like separate problems you have to memorize.

That's the argument behind pattern-based interview prep, and it's the framing AlgoMonster is built on. Rather than treating these as three unrelated questions, the platform groups them under a bit manipulation pattern, walks through the XOR properties first, then applies them to the single-number family. If you want to see how the walkthrough is structured, you can 👉 [open the Single Number walkthrough on AlgoMonster](https://algo.monster/liteproblems/136?fpr=li69) and the follow-ups at 👉 [Single Number II](https://algo.monster/liteproblems/137?fpr=li69) and 👉 [Single Number III](https://algo.monster/liteproblems/260?fpr=li69) — those pages are free to read.

The honest tradeoffs, based on what third-party reviews consistently report: the platform is text and diagrams, with no video walkthroughs, and its problem bank is far smaller than LeetCode's (third-party breakdowns cite roughly 48 patterns and a few hundred problems versus LeetCode's thousands). Its system design coverage is introductory, which won't be enough for senior-level loops. What it does have is a defined curriculum — commonly described as a 4 to 8 week path — which is the main reason people pick it over grinding without a plan.

## AlgoMonster's plans and what they cost

AlgoMonster sells one product in three tiers. All three are on the same subscribe page, and the pricing is more volatile than most platforms because the site runs frequent percentage-off sales. The figures below are what the official subscribe page listed when this was written.

| Plan | What you get | Price | Billing |
| --- | --- | --- | --- |
| Free | Limited access to a subset of lessons and problems. Enough to judge whether the text-and-diagram format works for you. | $0 | Free |
| Pro – Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; company questions updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials; member discount on live bootcamps; 12 months of access | $5.84/month, billed annually ($69.99/year) | Annual, renews |
| Pro – Lifetime | Everything in the yearly plan, plus lifetime updates on all content added later. One-time payment, no renewals. | $89 | One-time |

Plan links:

- 👉 [Start with the free plan](https://algo.monster/subscribe?fpr=li69)
- 👉 [Get Pro – Yearly access](https://algo.monster/subscribe?fpr=li69)
- 👉 [Get Pro – Lifetime access](https://algo.monster/subscribe?fpr=li69)

One warning that saves people money or confusion: other sites quote AlgoMonster at $99/year and $189 lifetime, or $459 lifetime, or $119–$299 annual. Those numbers come from different points in the platform's promo cycle. Third-party pages go stale quickly here. The checkout page is the only price that matters, and it's worth checking it directly rather than trusting a comparison table.

## Which plan makes sense for you

If the single-number family is the first time bit manipulation has clicked, you're probably early enough that you don't know yet whether the reading format suits you. Use the free tier on one pattern before paying for anything.

If you have an interview in the next two months, the yearly plan covers everything you need and costs less than a year of most alternatives. Nothing in the lifetime perks helps you inside a 4-to-8 week prep window.

The lifetime plan is the odd-looking one, since $89 is only about $19 more than the annual price. If you expect to interview again in two or three years — which is normal for engineers who switch jobs — paying once and keeping access through future updates is the cheaper long-run option. The case against it: if you finish the curriculum, land the job, and never revisit the material, you've paid $89 for a few weeks of use. That's a real possibility, and it's the honest counterargument.

## Quick answers

**Is XOR required to pass?** No. But the problem statement asks for constant extra space, so a hash set solution answers a different question than the one being asked. XOR is the intended answer.

**Does the XOR solution work with negative numbers and zeros?** Yes. `0` is a valid element and XORs harmlessly, and negative numbers are self-inverse under XOR.

**Can I use `sum` tricks instead?** For two copies, `2 * sum(set(nums)) - sum(nums)` gives the answer in O(n) time, but it allocates a set, so it uses O(n) space. It's a legitimate alternative to mention, not a way around the constraint.

**Do I need to handle an empty array?** No. The input is non-empty by definition.

**Is AlgoMonster free?** There's a free tier with limited access to samples. Full access requires the yearly or lifetime plan.

## The short version

The single number problem is a two-line loop with a lot of reasoning behind it, and that gap between the code and the reasoning is exactly what gets tested. Learn why the pair cancellation works instead of where to type the caret, and Single Number II and III turn into variations of the same idea rather than two more things to memorize before an interview.

If you'd rather work through the pattern with structured lessons than reverse-engineer it from discussion threads, 👉 [check the current AlgoMonster plans and start on the free tier](https://algo.monster/subscribe?fpr=li69). The bit manipulation section is free to preview, and the XOR properties the whole family depends on are covered before you touch a single-number problem.
