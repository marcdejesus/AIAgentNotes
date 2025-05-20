# 🧠 Selective Memory Sharing: Using LLM Understanding for Context Selection

In multi-agent systems, there are times when an agent must share part of its memory with another agent to provide context for a delegated task. Instead of relying on rigid, rule-based filtering, we can use the LLM’s reasoning ability to intelligently select the most relevant memories. This creates smarter, more adaptive collaboration between agents.

We'll demonstrate how to implement **selective memory sharing** using a self-prompting approach, where the LLM analyzes memory items and determines which are most relevant to the task.

---

## 🔧 Implementing `call_agent_with_selected_context`

```python
@register_tool(description="Delegate a task to another agent with selected context")
def call_agent_with_selected_context(action_context: ActionContext, agent_name: str, task: str) -> dict:
    """Call another agent, providing only relevant memories selected by an LLM."""
    agent_registry = action_context.get_agent_registry()
    agent_run = agent_registry.get_agent(agent_name)

    current_memory = action_context.get_memory()
    memory_with_ids = [
        {**item, "memory_id": f"mem_{idx}"}
        for idx, item in enumerate(current_memory.items)
    ]

    selection_schema = {
        "type": "object",
        "properties": {
            "selected_memories": {
                "type": "array",
                "items": {"type": "string", "description": "ID of a memory to include"}
            },
            "reasoning": {
                "type": "string",
                "description": "Explanation of why these memories were selected"
            }
        },
        "required": ["selected_memories", "reasoning"]
    }

    memory_text = "\n".join([
        f"Memory {m['memory_id']}: {m['content']}"
        for m in memory_with_ids
    ])

    selection_prompt = f"""Review these memories and select the ones relevant for this task:

Task: {task}

Available Memories:
{memory_text}

Select memories that provide important context or information for this specific task.
Explain your selection process."""

    selection = prompt_llm_for_json(
        action_context=action_context,
        schema=selection_schema,
        prompt=selection_prompt
    )

    filtered_memory = Memory()
    selected_ids = set(selection["selected_memories"])
    for item in memory_with_ids:
        if item["memory_id"] in selected_ids:
            item_copy = item.copy()
            del item_copy["memory_id"]
            filtered_memory.add_memory(item_copy)

    result_memory = agent_run(user_input=task, memory=filtered_memory)

    current_memory.add_memory({
        "type": "system",
        "content": f"Memory selection reasoning: {selection['reasoning']}"
    })

    for memory_item in result_memory.items:
        current_memory.add_memory(memory_item)

    return {
        "result": result_memory.items[-1].get("content", "No result"),
        "shared_memories": len(filtered_memory.items),
        "selection_reasoning": selection["reasoning"]
    }
```

---

## 🧪 Example: Delegating a Budget Review Task

Let’s say a project manager agent wants to delegate a budget review to a finance agent. Here’s what the memory might contain:

```python
memories = [
    {"type": "user", "content": "We need to build a new reporting dashboard"},
    {"type": "assistant", "content": "Initial cost estimate: $50,000"},
    {"type": "user", "content": "That seems high"},
    {"type": "assistant", "content": "Breakdown: $20k development, $15k design..."},
    {"type": "system", "content": "Project deadline updated to Q3"},
    {"type": "user", "content": "Can we reduce the cost?"}
]
```

The LLM might return the following selection:

```json
{
  "selected_memories": ["mem_1", "mem_3", "mem_5"],
  "reasoning": "Selected memories containing cost information and the request for cost reduction, excluding project timeline and general discussion as they're not directly relevant to the budget review task."
}
```

The finance agent now receives only the cost-related memories—resulting in focused context and a cleaner task handoff.

---

## ✅ Advantages of LLM-Based Memory Selection

* **Context Awareness**: The LLM can understand relevance beyond keyword matches.
* **Structured Reasoning**: The model explains its selections, improving transparency.
* **Task Adaptability**: Works across diverse tasks without needing custom rules.
* **Auditable Sharing**: Original agent logs what was shared and why.

This pattern is especially useful when you want to **minimize noise** and avoid overwhelming the receiving agent with irrelevant context.

---

## 🧭 Recap: Four Memory Sharing Patterns

Each pattern offers a different benefit for inter-agent collaboration:

| Pattern                      | Description                                                      |
| ---------------------------- | ---------------------------------------------------------------- |
| **Message Passing**          | Simple input/output exchange between agents                      |
| **Memory Reflection**        | Agents learn from another’s decision-making history              |
| **Memory Handoff**           | One agent passes full memory to continue a complex, ongoing task |
| **Selective Memory Sharing** | Only relevant context is shared, with reasoning preserved        |

Use these guiding questions to decide which pattern fits best:

* How much context does the next agent need?
* Should the original agent maintain a record of the interaction?
* Are there privacy or security concerns around memory sharing?
* Does the receiving agent need a clean memory scope?

---

By leveraging selective memory sharing, you enable agents to collaborate more intelligently—maintaining focus, reducing overhead, and providing just the right amount of context for each task.

---
