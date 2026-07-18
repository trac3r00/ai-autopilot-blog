---
title: "Why Computer-Use Agents Feel Slow — and the Cache That Removes a Round Trip"
date: 2026-07-18T20:15:00-04:00
description: "A measured note from building a macOS computer-use agent: the hidden cost of repeatedly walking the accessibility tree, and how action-aware caching changes the execution path."
tags: ["computer-use", "macos", "accessibility", "agent-engineering", "performance"]
categories: ["Field Notes"]
author: "Minseo"
ShowToc: true
TocOpen: false
---

A computer-use agent can make the right decision and still feel unusable if every small action starts by rediscovering the entire screen.

In our macOS agent, that rediscovery meant invoking System Events through AppleScript to enumerate the accessibility (AX) tree. One baseline run against the foreground app took **0.47 seconds** and returned no usable elements. That is not a model problem. It is an execution-path problem.

This note documents the change we made, what it does, and what has actually been verified so far.

## The expensive loop

The original cognitive loop asked for UI elements repeatedly:

1. Walk the current app's AX tree through System Events.
2. Convert those elements into a scene graph.
3. Ask the model to choose an action.
4. Perform a key press, type, scroll, or click.
5. Start from step 1 again.

That is defensible after a navigation click. It is wasteful after typing one character into a text field or scrolling inside the same view. The element structure usually has not changed.

There is a second cost hidden behind the first one: scene-graph construction derives positional relationships between elements. Recomputing those relationships for the same AX tree adds work without adding information.

## The change: cache the structure, not the screen

We added an `AXCache` to the CUA cognitive loop. It stores the most recent AX element list and its scene graph, scoped to the active app.

The cache survives actions that normally preserve layout:

```python
STABLE_ACTIONS = frozenset({
    "type", "key", "scroll", "wait",
})
```

It is cleared after actions that may change the page or open a dialog:

```python
def after_action(self, action_kind: str) -> None:
    if action_kind not in self.STABLE_ACTIONS:
        self.invalidate()
```

It also has two safety guards:

- **App switch:** a different app invalidates the cache.
- **Time limit:** cached UI data expires after five seconds, even if no invalidating action was reported.

That gives the agent a cheap fast path for a run like “focus editor → type → type → scroll”, while preserving a fresh read after “click Save” or “click Next.”

## Scene graph gets the same treatment

The cache does not only save the AX walk. If the element list is unchanged, the agent reuses the already-computed scene graph instead of rebuilding positional relations.

```python
cached_graph = cache.get_scene_graph(len(elements))
if cached_graph is not None:
    graph = cached_graph
else:
    graph = scene_graph_from_ui_elements(elements)
    cache.put_scene_graph(graph, len(elements))
```

The important boundary is that we cache **structure**, not a screenshot and not a guessed application state. Text content can change while the UI map stays valid; a navigation action discards that map.

## What we verified

This is deliberately narrower than a benchmark claim.

- A baseline AX enumeration run took **0.47 seconds** in the observed foreground-app case.
- The cache has **7 focused tests**, all passing.
- Tests cover a cold cache, cache hit, expiration, app change, explicit invalidation, stable actions, and navigation invalidation.
- The relevant CUA test slice passed after the change.

We have **not** yet published an end-to-end median latency reduction or a success-rate increase. Those require repeated runs across real applications, not one local timing and a unit-test suite.

## What comes next

The next experiment is straightforward:

1. Run the same multi-step task with caching disabled and enabled.
2. Record AX-walk count, scene-graph rebuild count, task latency, and completion rate.
3. Split results by action type and app.
4. Keep the cache only where it improves latency without hurting verification.

The lesson is simple: before paying for another vision or model call, check whether the agent already has a valid map of the interface. Most perceived CUA slowness is not intelligence. It is unnecessary rediscovery.

---

*Evidence: local AX timing on July 18, 2026; `AXCache` focused test suite (7 passed). This post will be updated with an end-to-end benchmark once the repeated-run harness exists.*

[View the implementation context on GitHub](https://github.com/trac3r00/bob). 

---

*This is a field note, not a product claim. Measurements and source links are updated when the experiment changes.*

## Source trail

- macOS Accessibility API: [Apple Developer Documentation](https://developer.apple.com/documentation/applicationservices/axuielement)
- System Events accessibility scripting: [AppleScript Language Guide](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/)
- Implementation: internal Bob CUA `AXCache` change, tested locally on July 18, 2026.

## What’s Next?

The next note will cover the other half of the problem: how a computer-use agent independently verifies that an action worked instead of asking vision to re-interpret the whole screen.