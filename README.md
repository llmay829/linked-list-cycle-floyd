# linked list cycle: Floyd's tortoise and hare explained, the entry-point proof, and the four bugs that break your six-line solution

Ask ten people who searched "linked list cycle" what they actually want and you'll get two answers. Half want to understand the problem well enough to write `hasCycle` from scratch. The other half can already write it and want to know why the version that returns the *entry node* works — because the code is four more lines but the reasoning behind it feels like something you're supposed to already know.

Both of those are solvable. What trips people up is the stuff in between: the null check that has to be in the loop condition and not somewhere else, the comparison that has to happen after you advance the pointers, and the fact that you're comparing node *identity* rather than node *values*. Get one of those wrong and the code compiles, runs, and returns the wrong answer on a test case you didn't think to write.

## The detection loop is genuinely six lines

Floyd's algorithm, usually called the tortoise and hare, puts two pointers at the head of the list. The slow one moves one node per step. The fast one moves two.

python
def hasCycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False


If there's no cycle, the fast pointer runs off the end of the list and the loop exits normally. If there is one, the fast pointer re-enters the cycle ahead of the slow pointer and eventually lands on the same node.

The `O(1)` space is the whole point. The obvious alternative — walk the list and keep every visited node in a hash set — also runs in `O(n)` time but costs `O(n)` memory. LeetCode 141's follow-up question is literally "can you solve it without using extra space," and this is the answer they're fishing for.

## Four ways those six lines break

**The loop guard is `fast and fast.next`, both of them.** Writing `while fast:` and then touching `fast.next.next` crashes the moment the fast pointer reaches the final node of an acyclic list. This is the single most common failure, and it's a null-dereference, not a logic error — so it shows up as a runtime exception rather than a wrong answer, which is at least honest about being broken.

**Advance first, compare second.** If you compare before moving, `slow` and `fast` are both sitting on `head` and the function immediately returns `True` for a list with no cycle at all. The comparison has to see the pointers *after* a step.

**Compare identity, not value.** In Python that means `is`, not `==` on `.val`. In Java, `slow == fast` on references, not `slow.val == fast.val`. The constraints on these problems allow node values in the range `[-10⁵, 10⁵]` across up to 10,000 nodes, so duplicate values are guaranteed on any long list. Two different nodes with the same number are not a cycle, and a cycle whose meeting point happens to hold a repeated value is not something value comparison can distinguish.

**Both pointers start at `head`.** Some implementations start `fast` at `head.next` to save a compare. That works, but it shifts every subsequent distance relation by one step, which is exactly the kind of off-by-one that makes the second phase — finding the entry node — fail in ways that look mysterious. Starting both at `head` costs one extra iteration and removes the entire class of bug.

## LeetCode 142: how you get from a meeting point to the actual entry node

Detecting a cycle and finding where it starts are different problems. The two pointers meet *somewhere* inside the loop, not at its entrance.

The trick is short: once they meet, leave the fast pointer where it is, send the slow pointer back to `head`, and move both of them one step at a time. They converge on the cycle's entry node.

python
def detectCycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            slow = head
            while slow is not fast:
                slow = slow.next
                fast = fast.next
            return slow
    return None


Nobody believes this the first time. The reason it works is one substitution, and it's worth being able to derive rather than recall.

Let `L` be the distance from `head` to the cycle's entry node, `C` the cycle length, and `k` the distance from the entry node forward to wherever the pointers met.

When they meet, the slow pointer has walked `L + k` steps. The fast pointer has walked twice that, `2(L + k)`. It's also walked the tail plus `k` steps past entry plus some whole number of extra laps — call it `n` laps — so `L + k + nC`.

Set those equal:


2(L + k) = L + k + nC
     L + k = nC
         L = nC − k


`L = nC − k` is the whole story. Walking `L` steps from `head` lands on the entry node, by definition. Walking `L` steps from the meeting point means walking `nC − k` steps — which is `n` full laps minus the `k` steps you're standing past the entry. Net result: you're standing on the entry node.

So a pointer starting from `head` and a pointer starting from the meeting point, both moving one step at a time, arrive at the same node after the same number of steps. That node is the entrance.

## Why fast moves two and not three

This is a fair follow-up question and the answer is a parity argument, not a preference.

Once both pointers are inside the cycle, what matters is the gap between them measured along the loop. Every iteration, the fast pointer gains one node on the slow pointer, so the gap shrinks by exactly one and must hit zero. It can't jump over zero, because it never jumps.

Move the fast pointer three steps instead and the gap shrinks by two. If the cycle length is odd, three is coprime enough with the loop to still guarantee a meeting. If the cycle length is even, the gap can oscillate through the same set of positions forever without ever landing on zero. Same algorithm, different arithmetic, no guaranteed termination.

## Hash set or two pointers — which one you say out loud

Start with the hash set in an interview if you want, but name its cost out loud before you write it: `O(n)` time, `O(n)` space, and it doesn't answer the "no extra space" follow-up. Then say the two-pointer version and the `O(1)` space. Interviewers at this level are grading whether you know the space tradeoff exists, not whether you can recite Floyd's from memory on the first try.

One practical note on the hash set: store node objects or their `id()`/reference, never the values. A set of values will tell you a duplicate number appeared and nothing about whether the list cycles.

## Where this pattern shows up when nobody says "linked list"

Cycle detection is one instance of a broader fast/slow pointer pattern, and the disguise is usually the interesting part of the problem:

- **Find the Duplicate Number** — an array of `n + 1` integers in the range `[1, n]` can be read as an implicit linked list where index `i` points to `arr[i]`. The repeated value is a cycle entry. Same algorithm, no pointers in sight.
- **Happy Number** — repeatedly summing squared digits either reaches 1 or eventually repeats, which is a cycle.
- **Middle of the Linked List** — when the fast pointer falls off the end, the slow pointer is at the midpoint. Same mechanics, different output.
- **Remove Nth Node From End** — advance fast `n` steps first, then move both together.
- **Circular Array Loop** — same idea with direction changes layered on top.

The recognition step is the skill. If you can write Floyd's from memory but can't look at "array of `n+1` values in `[1,n]`" and see a linked list, the memorized code doesn't help you.

That gap — pattern recognition rather than implementation — is what platforms like AlgoMonster are built around. Their curriculum organizes problems under an explicit **Fast and Slow Pointers** pattern instead of leaving you to notice the connection across five unrelated LeetCode tags, and the flowcharts exist to route a new problem to the right pattern before you start writing. It's text and diagrams, no video, which is either the best or worst thing about it depending on how you learn.

## What AlgoMonster costs right now

AlgoMonster is created by ex-Google engineers and competitive programmers, and it runs on two paid tiers plus a free one. There's no monthly option — you're choosing between a year and a one-time payment. Both paid plans include the same content; the difference is how long you keep it.

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems — enough to judge whether the text-and-diagram format works for you | $0 | — | [Explore AlgoMonster free](https://algo.monster?fpr=li69) |
| Pro – Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; company questions updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials | $99/year (about $8.25/mo billed annually) | 12 months, renews | [Get Pro Yearly access](https://algo.monster/subscribe?fpr=li69) |
| Pro – Lifetime | Everything in the yearly plan plus lifetime updates — current content and everything added later | $189 one-time | Once | [Get Pro Lifetime access](https://algo.monster/subscribe?fpr=li69) |

Both paid tiers also include the Foundation course (data structures and algorithms from the ground up) and a build-an-LLM course, at no extra cost. AlgoMonster runs frequent promotions, and third-party reviewers have reported seeing the yearly plan anywhere from roughly $89 to $299 and lifetime pricing well above $189 depending on the sale running at the time — so the numbers above reflect what the subscribe page shows, not a fixed price list. If you're price-sensitive, the sale timing matters more than the plan comparison.

## Who this actually fits, and who should skip it

The honest split, based on what the platform covers and what reviewers consistently flag:

**It fits** if you can already write working code in one language and your problem is *selection speed* — you stare at a problem for ten minutes and pick the wrong approach. Structured pattern coverage directly targets that, and a 4-to-8 week curriculum has an end point, unlike 3,000+ problems.

**It doesn't fit** if you're still learning what a hash map is. This is not a programming fundamentals course.

**And it's not enough on its own** if you're targeting senior loops. Reviewers who've gone through the whole thing repeatedly point at the same three gaps: system design coverage is introductory rather than L5-ready, behavioral preparation is thin, and there's no timed, watched, follow-up-question-pressure environment. Pattern recognition is necessary for a coding round. It isn't sufficient. Practicing a pattern at your own pace with no timer is a different skill from applying it in 45 minutes with someone asking why your second phase terminates.

Who's it for, per the platform's own framing? Students hunting internships, working engineers prepping a job change, and self-taught developers who need the algorithmic layer fast.

## Short answers to the questions people usually search next

**What's the time complexity of cycle detection?** `O(n)` time and `O(1)` space with the two-pointer method. The hash set version is `O(n)` time and `O(n)` space.

**How many nodes can the list have?** LeetCode 142 allows up to 10,000 nodes with values between -100,000 and 100,000, and requires that you don't modify the list.

**Does the cycle have to start at the first node?** No. That's the entire reason the second phase exists — the tail before the cycle can be any length, including zero.

**What if the list is empty or has one node?** `while fast and fast.next` handles both. An empty list fails the guard immediately, and a single node without a self-loop fails on `fast.next`. A single node that points to itself passes the guard one time and gets caught by the identity comparison.

**Should I use a visited set instead?** Only if the interviewer hasn't asked about space. It's simpler to write correctly and strictly worse on the constraint the problem is actually testing.

If you want the pattern layer built rather than assembled from scattered solutions, 👉 [see what the full AlgoMonster curriculum covers](https://algo.monster?fpr=li69) — the Fast and Slow Pointers section is one of 48 patterns, and the flowcharts are the part that generalizes past this one problem.

The six-line loop is the easy part. Knowing when the problem in front of you *is* a linked list cycle in disguise is the part that pays.
