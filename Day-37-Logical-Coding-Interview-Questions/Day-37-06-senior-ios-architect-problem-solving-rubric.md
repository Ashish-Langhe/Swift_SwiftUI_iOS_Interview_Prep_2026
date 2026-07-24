# Day 37: Senior iOS Architect Problem-Solving Rubric

As a senior iOS architect, I do not test only whether a candidate can write a loop. I test whether they can turn unclear product behavior into safe, maintainable, testable Swift. Good iOS engineers solve the problem in front of them. Senior engineers also protect the codebase from the next six months of change.

## What I Would Test

I would test seven areas:

1. **Core logic**
   - Can the candidate reason through arrays, strings, dictionaries, sets, sorting, searching, and edge cases?

2. **Swift correctness**
   - Do they use optionals safely?
   - Do they understand value semantics?
   - Do they avoid force unwraps?
   - Can they write small, pure functions?

3. **State modeling**
   - Can they model loading, empty, success, failed, paginating, refreshing, and partially loaded states?

4. **Concurrency**
   - Can they use async/await safely?
   - Can they handle cancellation?
   - Can they avoid data races?
   - Do they know where UI updates belong?

5. **Memory and lifecycle**
   - Can they avoid retain cycles?
   - Can they reason about ownership?
   - Can they cancel work when screens go away?

6. **Architecture boundaries**
   - Can they separate DTO, domain model, view model, repository, service, and view state?
   - Can they design code that is testable?

7. **Production behavior**
   - Can they handle offline state, retry, duplicate requests, token refresh, cache invalidation, and error mapping?

## How I Score A Coding Answer

| Level | What I Expect |
|---|---|
| Junior | Solves the happy path, uses basic syntax correctly, explains simple edge cases. |
| Mid-level | Handles edge cases, writes readable Swift, explains complexity, avoids unsafe code. |
| Senior | Models the problem cleanly, handles failure, cancellation, identity, testing, and future change. |
| Architect | Identifies hidden product/system risks, creates boundaries, discusses tradeoffs, and makes the solution evolvable. |

## What Makes A Problem Senior-Level

A problem becomes senior-level when it includes:

- Ambiguous requirements.
- Async work.
- Multiple sources of truth.
- Caching.
- Identity.
- Partial failure.
- Retry.
- Cancellation.
- Memory ownership.
- UI state.
- Testability.
- Backward compatibility.

Example:

```text
Easy:
Find duplicates in an array.

Senior:
Given cached products and remote products, merge them by stable ID, preserve local favorite state, remove deleted items, keep sort order, and produce a diffable-data-source-ready list.
```

## Interview Signal I Look For

Strong candidates say things like:

- "What should happen for empty input?"
- "Can IDs repeat?"
- "Should this preserve order?"
- "Is this user-facing state or domain state?"
- "Can this request be cancelled?"
- "What happens if two refreshes happen together?"
- "Should cached data appear before network completes?"
- "How do we test this without the network?"
- "What is the stable identity for this row?"

Weak signals:

- Starts coding immediately without clarifying.
- Solves only the happy path.
- Uses `!` casually.
- Captures `indexPath` in async callbacks.
- Ignores cancellation.
- Makes everything a singleton.
- Mixes API models directly into UI.
- Cannot explain complexity.

## My Favorite iOS Coding Assessment Themes

### 1. Collection Transformation

Example prompt:

```text
Given a list of expenses, group them by month, sort months descending, and calculate total per category.
```

Why I ask it:

- Tests dictionaries.
- Tests sorting.
- Tests date handling.
- Tests model design.
- Maps directly to real iOS dashboards.

### 2. View State Modeling

Example prompt:

```text
Design the state for a product list screen with initial loading, pull-to-refresh, pagination, empty state, error state, and cached data.
```

Why I ask it:

- Tests product thinking.
- Tests architecture.
- Tests UI correctness.
- Reveals whether candidate only knows algorithms or can build apps.

### 3. Token Refresh Coordination

Example prompt:

```text
Multiple API requests receive 401 at the same time. Design logic so only one token refresh happens, and all requests continue after refresh.
```

Why I ask it:

- Tests concurrency.
- Tests shared state.
- Tests failure handling.
- Tests senior backend/client boundary thinking.

### 4. Search With Cancellation

Example prompt:

```text
Build search logic that debounces typing, cancels old requests, handles empty query, and prevents stale results from replacing newer results.
```

Why I ask it:

- Tests async/await.
- Tests task cancellation.
- Tests user experience.
- Tests stale-response handling.

### 5. Diffable Identity

Example prompt:

```text
Create section and row models for a mixed collection view. Rows must have stable identity and support content updates.
```

Why I ask it:

- Tests Hashable design.
- Tests UI architecture.
- Tests list correctness.
- Reveals whether candidate knows that index paths are not durable identity.

### 6. Cache Policy

Example prompt:

```text
Show cached data immediately, fetch fresh data, update cache, and handle stale or failed response.
```

Why I ask it:

- Tests repository design.
- Tests persistence boundary.
- Tests offline behavior.
- Tests user trust.

### 7. Memory Leak Debugging

Example prompt:

```text
Find and fix a retain cycle in a view controller using a view model, closure callback, timer, and async task.
```

Why I ask it:

- Tests ARC.
- Tests closure capture.
- Tests lifecycle.
- Tests practical debugging.

## How To Answer Architect-Level Questions

Use this structure:

1. **Clarify behavior**
   - "Should cached data be shown if network fails?"
   - "Should pagination errors replace the whole screen or show footer retry?"

2. **Define models**
   - DTO.
   - Domain.
   - View state.
   - Row model.

3. **Define ownership**
   - Repository owns data fetching.
   - View model owns screen state.
   - View owns rendering.
   - Coordinator owns navigation.

4. **Define failure behavior**
   - Empty.
   - Offline.
   - Unauthorized.
   - Retryable.
   - Non-retryable.

5. **Define concurrency**
   - Which tasks can run together?
   - Which tasks cancel old work?
   - Which shared state needs actor isolation?

6. **Define test cases**
   - Happy path.
   - Empty path.
   - Failure path.
   - Cancellation.
   - Duplicate request.
   - Cached data.

## Problems I Would Not Overvalue

These are useful warmups but not enough for senior iOS:

- Reverse string.
- Check prime.
- Print pattern.
- Bubble sort.
- Factorial.

I may ask them only to warm up. For senior roles, I care more about how the candidate handles realistic app complexity.

## Final Senior Architect Takeaway

Basic logic gets you through the first screen. Production judgment gets you through the job. Practice logical questions, but always connect them back to real iOS: state, identity, lifecycle, cancellation, memory, networking, persistence, and testability.

