# 🧠 Building a Simple Agent Framework (Part 2)

Now, we are going to put the components together into a reusable `Agent` class. This class will encapsulate the GAME components and provide a simple interface for running the agent loop. The agent will be responsible for constructing prompts, executing actions, and managing memory. We can create different agents simply by changing the goals, actions, and environment without modifying the core loop.

## 🏗️ Agent Class Definition

```python
class Agent:
    def __init__(self,
                 goals: List[Goal],
                 agent_language: AgentLanguage,
                 action_registry: ActionRegistry,
                 generate_response: Callable[[Prompt], str],
                 environment: Environment):
        self.goals = goals
        self.generate_response = generate_response
        self.agent_language = agent_language
        self.actions = action_registry
        self.environment = environment

    def construct_prompt(self, goals: List[Goal], memory: Memory, actions: ActionRegistry) -> Prompt:
        return self.agent_language.construct_prompt(
            actions=actions.get_actions(),
            environment=self.environment,
            goals=goals,
            memory=memory
        )

    def get_action(self, response):
        invocation = self.agent_language.parse_response(response)
        action = self.actions.get_action(invocation["tool"])
        return action, invocation

    def should_terminate(self, response: str) -> bool:
        action_def, _ = self.get_action(response)
        return action_def.terminal

    def set_current_task(self, memory: Memory, task: str):
        memory.add_memory({"type": "user", "content": task})

    def update_memory(self, memory: Memory, response: str, result: dict):
        new_memories = [
            {"type": "assistant", "content": response},
            {"type": "user", "content": json.dumps(result)}
        ]
        for m in new_memories:
            memory.add_memory(m)

    def prompt_llm_for_action(self, full_prompt: Prompt) -> str:
        response = self.generate_response(full_prompt)
        return response

    def run(self, user_input: str, memory=None, max_iterations: int = 50) -> Memory:
        memory = memory or Memory()
        self.set_current_task(memory, user_input)

        for _ in range(max_iterations):
            prompt = self.construct_prompt(self.goals, memory, self.actions)
            print("Agent thinking...")
            response = self.prompt_llm_for_action(prompt)
            print(f"Agent Decision: {response}")
            action, invocation = self.get_action(response)
            result = self.environment.execute_action(action, invocation["args"])
            print(f"Action Result: {result}")
            self.update_memory(memory, response, result)

            if self.should_terminate(response):
                break

        return memory
```

---

## 🌀 Step-by-Step Breakdown of the Agent Loop

### 🔹 Step 1: Constructing the Prompt

```python
def construct_prompt(self, goals: List[Goal], memory: Memory, actions: ActionRegistry) -> Prompt:
    return self.agent_language.construct_prompt(
        actions=actions.get_actions(),
        environment=self.environment,
        goals=goals,
        memory=memory
    )
```

* Goals
* Available Actions
* Memory Context
* Environment Details

### 🔹 Step 2: Generating a Response

```python
def prompt_llm_for_action(self, full_prompt: Prompt) -> str:
    response = self.generate_response(full_prompt)
    return response
```

### 🔹 Step 3: Parsing the Response

```python
def get_action(self, response):
    invocation = self.agent_language.parse_response(response)
    action = self.actions.get_action(invocation["tool"])
    return action, invocation
```

### 🔹 Step 4: Executing the Action

```python
result = self.environment.execute_action(action, invocation["args"])
```

### 🔹 Step 5: Updating Memory

```python
def update_memory(self, memory: Memory, response: str, result: dict):
    new_memories = [
        {"type": "assistant", "content": response},
        {"type": "user", "content": json.dumps(result)}
    ]
    for m in new_memories:
        memory.add_memory(m)
```

### 🔹 Step 6: Termination Check

```python
def should_terminate(self, response: str) -> bool:
    action_def, _ = self.get_action(response)
    return action_def.terminal
```

---

## 🔄 The Flow of Information Through the Loop

1. **Memory** provides historical context.
2. **Goals** define what to achieve.
3. **ActionRegistry** defines available actions.
4. **AgentLanguage** constructs the prompt.
5. **LLM** generates a response.
6. **AgentLanguage** parses the response into an invocation.
7. **Environment** executes the chosen action.
8. **Memory** stores the result.
9. Loop continues until termination condition is met.

---

## 🧪 Creating Specialized Agents

### 🔬 Research Agent

```python
research_agent = Agent(
    goals=[Goal("Find and summarize information on topic X")],
    agent_language=ResearchLanguage(),
    action_registry=ActionRegistry([SearchAction(), SummarizeAction(), ...]),
    generate_response=openai_call,
    environment=WebEnvironment()
)
```

### 💻 Coding Agent

```python
coding_agent = Agent(
    goals=[Goal("Write and debug Python code for task Y")],
    agent_language=CodingLanguage(),
    action_registry=ActionRegistry([WriteCodeAction(), TestCodeAction(), ...]),
    generate_response=anthropic_call,
    environment=DevEnvironment()
)
```

Each agent uses the same loop but behaves differently by swapping in different GAME components.
