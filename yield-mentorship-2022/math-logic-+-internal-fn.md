---
description: https://github.com/hashedMae/mentorshipProjects/pull/4
---

# math logic + internal fn

Instead of repeating math logic, you should call the `convertToAssets` function.

Even better, for gas cost. You should put the logic on an internal `_convertToAssets` function, and call `_convertToAssets` from the external `convertToAssets` (which becomes a user-facing function), `mint` and `redeem`.

The same pattern should be followed for `convertToShares`, `deposit` and `withdraw`.
