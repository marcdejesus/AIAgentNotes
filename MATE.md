# ♟️ The MATE Design Principles

In chess, a checkmate represents the perfect execution of strategy—every piece positioned with intent, every move calculated for maximum impact. When designing AI agents, we can apply similar principles to achieve robust, safe, and efficient outcomes. Enter the **MATE framework**: **Model efficiency**, **Action specificity**, **Token efficiency**, and **Environmental safety**.

---

## 🧠 Model Efficiency: *Choose Your Pieces Wisely*

In chess, every piece has its purpose. You don’t deploy a queen when a pawn will do. Likewise, **model efficiency** means using the right model for the task at hand—leveraging lightweight models for simple jobs and more powerful ones for complex reasoning.

```python
@register_tool(description="Extract basic contact information from text")
def extract_contact_info(action_context: ActionContext, text: str) -> dict:
    """Use a smaller model to extract name, email, and phone."""
    response = action_context.get("fast_llm")(Prompt(messages=[
        {"role": "system", "content": "Extract contact information in JSON format."},
        {"role": "user", "content": text}
    ]))
    return json.loads(response)
```

```python
@register_tool(description="Analyze complex technical documentation")
def analyze_technical_doc(action_context: ActionContext, document: str) -> dict:
    """Use a powerful model for deeper analysis."""
    response = action_context.get("powerful_llm")(Prompt(messages=[
        {"role": "system", "content": "Analyze this documentation to identify contradictions or problems."},
        {"role": "user", "content": document}
    ]))
    return json.loads(response)
```

---

## 🎯 Action Specificity: *Control the Board*

Just as precise positioning in chess limits your opponent's moves, **action specificity** limits ambiguity and misuse. Generic actions may introduce risk or unclear behavior, while well-scoped tools improve safety and reliability.

❌ *Too generic* – overly broad and harder to control:

```python
@register_tool(description="Modify calendar events")
def update_calendar(action_context: ActionContext, event_id: str, updates: dict) -> dict:
    return calendar.update_event(event_id, updates)
```

✅ *Specific and constrained* – easier to validate and test:

```python
@register_tool(description="Reschedule a meeting you own to a new time")
def reschedule_my_meeting(action_context: ActionContext, event_id: str, new_start_time: str, new_duration_minutes: int) -> dict:
    event = calendar.get_event(event_id)
    if event.organizer != action_context.get("user_email"):
        raise ValueError("Can only reschedule meetings you organize")
        
    new_start = datetime.fromisoformat(new_start_time)
    if new_start < datetime.now():
        raise ValueError("Cannot schedule meetings in the past")
        
    return calendar.update_event_time(
        event_id,
        new_start_time=new_start_time,
        duration_minutes=new_duration_minutes
    )
```

---

## 🔢 Token Efficiency: *Maximize Every Move*

Every move in chess should advance your position. Similarly, every token in a prompt should serve a clear purpose. Bloated prompts waste compute and increase costs. Efficient prompts focus the model’s attention where it matters.

❌ *Token-inefficient* – overly verbose and unfocused:

```python
@register_tool(description="Analyze sales data to identify trends and patterns...")
def analyze_sales(action_context: ActionContext, data: str) -> str:
    return prompt_llm(action_context, f"""
        Analyze this sales data thoroughly. Consider monthly trends,
        seasonal patterns, year-over-year growth, product categories,
        regional variations, and customer segments. Provide detailed insights.
        
        Data: {data}
    """)
```

✅ *Token-efficient* – concise and to the point:

```python
@register_tool(description="Analyze sales data for key trends")
def analyze_sales(action_context: ActionContext, data: str) -> str:
    return prompt_llm(action_context, f"""
        Sales Data: {data}
        1. Calculate YoY growth
        2. Identify top 3 trends
        3. Flag significant anomalies
    """)
```

---

By following the MATE principles, you structure your AI agent design with the same discipline and foresight that a grandmaster brings to a chessboard. The result? Systems that are smarter, safer, faster, and more strategic.

---
