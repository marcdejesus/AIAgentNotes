# 🤝 `call_agent`: Enabling Multi-Agent Collaboration

Imagine building a system where multiple specialized agents work together, each contributing its unique capabilities to solve complex problems. For instance, a **primary agent** may oversee high-level coordination, but delegate specialized tasks—like scheduling or document analysis—to other agents. To support this, agents need a way to invoke one another.

The most effective way to enable this kind of modular collaboration is by **exposing inter-agent communication as a tool**. This makes the architecture extensible and allows easy composition of complex systems through well-defined interfaces. The `call_agent` tool allows one agent to call another and receive its results using the shared `ActionContext`.

---

## 🛠️ `call_agent` Tool Implementation

Here’s how `call_agent` works inside an agent’s execution:

```python
@register_tool()
def call_agent(action_context: ActionContext, agent_name: str, task: str) -> dict:
    """
    Invoke another agent to perform a specific task.
    """
    agent_registry = action_context.get_agent_registry()
    if not agent_registry:
        raise ValueError("No agent registry found in context")

    agent_run = agent_registry.get_agent(agent_name)
    if not agent_run:
        raise ValueError(f"Agent '{agent_name}' not found in registry")

    invoked_memory = Memory()

    try:
        result_memory = agent_run(
            user_input=task,
            memory=invoked_memory,
            action_context_props={
                'auth_token': action_context.get('auth_token'),
                'user_config': action_context.get('user_config'),
                # Prevent infinite recursion by not passing the registry itself
            }
        )

        if result_memory.items:
            return {
                "success": True,
                "agent": agent_name,
                "result": result_memory.items[-1].get("content", "No result content")
            }
        else:
            return { "success": False, "error": "Agent failed to run." }

    except Exception as e:
        return { "success": False, "error": str(e) }
```

This tool provides structured inter-agent communication while ensuring:

* **Memory Isolation**: Each agent operates with a clean memory context.
* **Controlled Context Sharing**: Only necessary properties are passed forward.
* **Safe Result Handling**: The last memory item is returned as the agent’s final result.

---

## 📅 Example: Meeting Scheduling with Specialized Agents

Let’s look at a collaboration example between two agents in a project management system:

### 1. **The Scheduler Agent**

Handles the logistics of scheduling:

```python
@register_tool()
def check_availability(...): 
    """Find available time slots for all attendees."""
    return calendar_service.find_available_slots(...)

@register_tool()
def create_calendar_invite(...): 
    """Create and send a calendar invitation."""
    return calendar_service.create_event(...)

scheduler_agent = Agent(goals=[
    Goal(name="schedule_meetings", description="""
        1. Find available times
        2. Create and send calendar invites
        3. Handle scheduling conflicts
    """)
])
```

### 2. **The Project Manager Agent**

Coordinates project progress and uses the `call_agent` tool to delegate scheduling:

```python
@register_tool()
def get_project_status(...): 
    """Retrieve project status."""
    return project_service.get_status(...)

@register_tool()
def update_project_log(...): 
    """Log project updates."""
    return project_service.log_update(...)

@register_tool()
def call_agent(...): 
    """Delegate to a specialist agent."""
    # Reuses the earlier `call_agent` implementation

project_manager = Agent(goals=[
    Goal(name="project_oversight", description="""
        1. Monitor project progress
        2. Determine when meetings are needed
        3. Use 'scheduler_agent' to schedule meetings
        4. Record project updates
    """)
])
```

This division of labor ensures each agent stays focused:

* **The project manager** handles project logic.
* **The scheduler** handles time and invites.
* **The `call_agent` tool** connects them seamlessly.

---

## 🧠 Agent Registry Setup

Before agents can communicate, they must be registered:

```python
class AgentRegistry:
    def __init__(self):
        self.agents = {}

    def register_agent(self, name: str, run_function: callable):
        self.agents[name] = run_function

    def get_agent(self, name: str) -> callable:
        return self.agents.get(name)

# Register agents
registry = AgentRegistry()
registry.register_agent("scheduler_agent", scheduler_agent.run)

# Provide registry to the ActionContext
action_context = ActionContext({
    'agent_registry': registry,
    'auth_token': '...',  # Optional shared resources
    'user_config': {...}
})
```

---

## ✅ Benefits of the `call_agent` Tool

* **🔒 Memory Isolation**
  Each agent gets a clean working memory, avoiding cross-contamination.

* **🧭 Controlled Context**
  Only essential context is passed—avoiding infinite loops or unnecessary data exposure.

* **📤 Clean Result Interface**
  The last memory entry is returned as a structured result for the calling agent.

---

By exposing `call_agent` as a modular tool, you enable sophisticated **multi-agent collaboration** while maintaining clean boundaries, safer execution, and testable interfaces. This architecture scales naturally—allowing you to grow a constellation of specialized agents that cooperate effectively, just like a well-coordinated team.
