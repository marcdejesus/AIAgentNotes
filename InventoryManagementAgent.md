# 📦 An Inventory Management Agent

Traditional software development involves writing complex code to handle every possible scenario. With LLM-powered agents, we can take a different approach: provide clear goals, simple tools, and let the agent’s intelligence bridge the gap. Let’s explore this through building an inventory management system.

---

## 🔄 Rethinking Software Architecture

Instead of building complex inventory logic, we break down the system into simple tools and let the agent handle the complexity.

### 🛠️ Simple Tools That Focus on Data Operations

```python
@register_tool(description="Save an item to inventory")
def save_item(action_context: ActionContext,
              item_name: str,
              description: str,
              condition: str,
              estimated_value: float) -> dict:
    inventory = action_context.get("inventory_db")
    item_id = str(uuid.uuid4())

    item = {
        "id": item_id,
        "name": item_name,
        "description": description,
        "condition": condition,
        "estimated_value": estimated_value,
        "added_date": datetime.now().isoformat()
    }

    inventory[item_id] = item
    return {"item_id": item_id}

@register_tool(description="Get all inventory items")
def get_inventory(action_context: ActionContext) -> List[dict]:
    inventory = action_context.get("inventory_db")
    return list(inventory.values())

@register_tool(description="Get specific inventory item")
def get_item(action_context: ActionContext, item_id: str) -> dict:
    inventory = action_context.get("inventory_db")
    return inventory.get(item_id)

These tools handle basic CRUD operations. The intelligence comes from the agent’s goals and system prompt.

---

## 🎯 Defining the Agent's Goals

```python
goals = [
    Goal(
        name="inventory_management",
        description="""Maintain an accurate inventory of items including:
        - Detailed descriptions
        - Condition assessment
        - Value estimates
        - Historical tracking"""
    )
]
```

### 🧠 Creating the Agent

```python
agent = Agent(
    goals=goals,
    agent_language=JSONAgentLanguage(),
    action_registry=registry,
    capabilities=[
        SystemPromptCapability("""You are an expert inventory manager.
        When shown items:
        1. Identify the item type and key features
        2. Assess condition from visual cues
        3. Estimate market value based on condition and features
        4. Maintain organized records with consistent descriptions

        Always be thorough in descriptions and conservative in value estimates.""")
    ]
)
```

---

## 🧑‍💬 Using the System

Users can interact naturally using plain language.

### 💬 Example Interaction

```python
result = agent.run("""I have a pair of Air Jordan basketball shoes.
                     They're red with the Jumpman logo, showing some wear
                     and slight discoloration.""")
```

### 🧾 Agent Response (Example)

```
I'll help you add those shoes to inventory.

First, let me analyze the item details you've provided:
- Item: Air Jordan Basketball Shoes
- Color: Red
- Notable Features: Jumpman logo
- Condition: Used with visible wear and discoloration

Based on these details and current market values, I'll create an inventory entry.

Action: save_item
{
    "item_name": "Air Jordan Basketball Shoes",
    "description": "Red colorway with iconic Jumpman logo",
    "condition": "Used - visible wear and slight discoloration",
    "estimated_value": 85.00
}

The shoes have been added to inventory. Would you like to add any additional items?
```

---

## 🖼️ Extending with Images (Future Enhancement)

This system can be extended to handle images with additional tools.

```python
@register_tool(description="Analyze an image and describe what you see")
def process_inventory_image(action_context: ActionContext,
                            image_path: str) -> str:
    with open(image_path, "rb") as image_file:
        image_data = base64.b64encode(image_file.read()).decode("utf-8")

    response = completion(
        model="gpt-4o",
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": """Please describe this item for inventory purposes.
                        Include details about:
                        - What the item is
                        - Its key features
                        - The condition it's in
                        - Any visible wear or damage
                        - Anything notable about it"""
                    },
                    {
                        "type": "image",
                        "image_url": {
                            "url": f"data:image/jpeg;base64,{image_data}"
                        }
                    }
                ]
            }
        ],
        max_tokens=1000
    )

    return response
```

---

## ✅ Why This Approach Works

### 🔹 Simple Tools, Complex Understanding

* Tools handle basic operations.
* The LLM does the heavy lifting via reasoning.

### 🔹 Natural Interaction

* Users describe items in plain language or images.
* No need for structured forms.

### 🔹 Flexible Intelligence

* Identify items from descriptions.
* Assess condition from details.
* Estimate value with market insight.
* Maintain consistent records.

### 🔹 Easy Extension

* Update system prompt.
* Add new tools.
* Enhance goals.

---

## 🌍 Real-World Applications

This framework extends far beyond inventory systems:

* ✅ Policy compliance agents
* ✅ Document processing
* ✅ Customer service
* ✅ Data analysis

The key to success:

* Define **clear goals**
* Provide **simple, focused tools**
* Let the **agent handle the complexity**

> The future of software development lies not in handling every edge case manually, but in empowering intelligent agents to adapt within a simple, robust framework.
