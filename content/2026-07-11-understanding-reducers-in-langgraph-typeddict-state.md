Title: Understanding Reducers in LangGraph TypedDict State
Date: 2026-07-11
Category: GenAI
Tags: GenAI, LangGraph, LLM, agents, state-management, Python, TypedDict
Slug: understanding-reducers-in-langgraph-typeddict-state

In LangGraph, state is the shared data structure that nodes read from and write to. How that state gets updated — when multiple nodes touch the same key — is controlled by reducers. Get them wrong and you lose data. Get them right and your workflow accumulates exactly what it needs.

## What is a Reducer?

**Reducer** — A function that combines the existing state value with a new value returned by a node. LangGraph calls it automatically whenever a node updates a key. The reducer decides whether to replace, append, merge, or apply any custom logic.

**Default behavior** — Without a reducer, LangGraph replaces the old value with the new one. The previous value is gone.

## Without a Reducer

```python
from typing import TypedDict

class State(TypedDict):
    messages: list[str]
```

No reducer means last write wins.

```
Current:  messages = ["Hello"]
Node returns: {"messages": ["World"]}
Result:   messages = ["World"]   # "Hello" is lost
```

Use this when only the latest value matters — current user input, active page, latest prediction, session status.

## With a Reducer Annotation

```python
from typing import TypedDict, Annotated
import operator

class State(TypedDict):
    messages: Annotated[list[str], operator.add]
```

`operator.add` on lists concatenates instead of replacing.

```
Current:  messages = ["Hello"]
Node returns: {"messages": ["World"]}
Result:   messages = ["Hello", "World"]   # both values kept
```

Use `operator.add` whenever you need to accumulate — conversation history, logs, collected outputs across nodes.

## Default vs Annotated

| | Without Reducer | `Annotated[list, operator.add]` |
|---|---|---|
| Behavior | Replaces old value | Appends new values |
| Previous data | Lost | Preserved |
| Best for | Current state | History or logs |

## Custom Reducers

The built-in options cover simple cases. For everything else, write your own.

**Remove duplicates**

```python
def unique_merge(old, new):
    return list(dict.fromkeys(old + new))

# ["A", "B"] + ["B", "C"] → ["A", "B", "C"]
```

**Keep only the latest N items**

```python
def last_five(old, new):
    return (old + new)[-5:]
```

Useful for sliding chat history windows or recent notification feeds.

**Merge dictionaries**

```python
def merge_dict(old, new):
    return {**old, **new}

# {"name": "Alice"} + {"age": 22} → {"name": "Alice", "age": 22}
```

**Sum numerical values**

```python
def add_scores(old, new):
    return old + new
```

Token counts, running scores, cumulative statistics.

**Keep the highest value**

```python
def max_score(old, new):
    return max(old, new)
```

Confidence scores, rankings, peak sensor readings.

## Wiring a Custom Reducer

```python
from typing import TypedDict, Annotated

def unique_messages(old, new):
    return list(dict.fromkeys(old + new))

class State(TypedDict):
    messages: Annotated[list[str], unique_messages]
```

Now every state update to `messages` automatically deduplicates — no extra logic needed in any node.

## When to Use Each

**Default reducer** — Latest value is all that matters, overwriting is correct behavior.

**`operator.add`** — Simple accumulation of lists, straightforward append semantics.

**Custom reducer** — Deduplication, size limits, dictionary merging, business-specific merge rules, or anything that requires inspecting both old and new values before deciding the result.

> Keep reducers pure and deterministic. A reducer that produces different output for the same inputs will make your workflow unpredictable and very hard to debug.
