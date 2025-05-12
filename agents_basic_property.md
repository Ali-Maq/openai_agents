I've created a comprehensive system diagram that maps out all the components mentioned in the Agents documentation. Here's what the diagram shows:

## Main Components:

1. **Agent (Central Entity)**
   - All the properties: name, instructions, model, model_settings, tools, output_type, handoffs, hooks, guardrails
   - Shows it's generic on context type `Agent[UserContext]`
   - Includes reset_tool_choice property

2. **Context System**
   - Shows how UserContext is used as a generic type parameter
   - Demonstrates dependency injection pattern

3. **ModelSettings**
   - Temperature, top_p configurations
   - Tool choice options (auto, required, none, specific tool)

4. **Tools**
   - Function tool example with @function_tool decorator
   - Shows parameter and return type

5. **Output Types**
   - Pydantic model example (CalendarEvent)
   - Notes support for dataclasses, TypedDict, lists

6. **Handoffs**
   - Shows list of other agents (booking_agent, refund_agent)
   - Delegation pattern for specialized tasks

7. **Dynamic Instructions**
   - Function signature showing context and agent parameters
   - Returns string for instructions

8. **Additional Features**
   - AgentHooks for lifecycle events
   - Guardrails for parallel validation
   - Clone method for duplicating agents
   - Tool behavior settings (stop_on_first_tool)

9. **Runner**
   - Shows how agents are executed
   - Takes agent and context as parameters

All relationships and connections are shown with arrows, and I've included a legend explaining the key concepts. The diagram captures every detail from the documentation without leaving anything out.


You're correct about the Agent structure! Let me clarify and give you an example based on the documentation:

## Your understanding is accurate:
- **name**: string identifier
- **instructions**: system prompt (string or function)
- **model**: string (like "o3-mini")
- **model_settings**: ModelSettings object (includes temperature, top_p, tool_choice)
- **tools**: list of functions
- **output_type**: any type (Pydantic, dataclass, TypedDict, etc.)
- **handoffs**: list of other agents
- **guardrails**: The doc says they validate "user input" specifically

## About `reset_tool_choice`:

The documentation explains:
> "To prevent infinite loops, the framework automatically resets tool_choice to "auto" after a tool call. This behavior is configurable via agent.reset_tool_choice. The infinite loop is because tool results are sent to the LLM, which then generates another tool call because of tool_choice, ad infinitum."

This means:
- If you set `tool_choice="my_tool"`, the agent MUST use that tool
- After using the tool, if `reset_tool_choice=True` (default), it changes back to "auto"
- This prevents: Tool call → Result → Forced tool call again → Result → Infinite loop

## Example from the documentation:

```python
# Step 1: Create tools
@function_tool
def get_weather(city: str) -> str:
    return f"The weather in {city} is sunny"

# Step 2: Create sub-agents for handoffs
booking_agent = Agent(
    name="Booking Agent",
    instructions="Handle booking requests"
)

refund_agent = Agent(
    name="Refund Agent", 
    instructions="Handle refund requests"
)

# Step 3: Create main agent with everything
triage_agent = Agent(
    name="Triage Agent",
    instructions=(
        "Help the user with their questions."
        "If they ask about booking, handoff to the booking agent."
        "If they ask about refunds, handoff to the refund agent."
    ),
    model="o3-mini",
    model_settings=ModelSettings(
        temperature=0.7,
        tool_choice="auto"  # or "required", "none", "get_weather"
    ),
    tools=[get_weather],
    handoffs=[booking_agent, refund_agent],
    output_type=str,  # default
    reset_tool_choice=True  # default behavior
)

# Step 4: Clone an agent with modifications
pirate_agent = Agent(
    name="Pirate",
    instructions="Write like a pirate",
    model="o3-mini",
)

robot_agent = pirate_agent.clone(
    name="Robot",
    instructions="Write like a robot",
)
```

## How it works:
1. User asks: "What's the weather in Paris?"
2. Triage agent sees it has `get_weather` tool
3. Based on `tool_choice="auto"`, it decides to use the tool
4. Tool returns: "The weather in Paris is sunny"
5. `reset_tool_choice` ensures next interaction isn't forced to use tools
6. Agent responds with the weather info

If user asks: "I need to book a flight"
- Triage agent recognizes "booking" keyword
- Hands off to `booking_agent`
- Booking agent takes over the conversation

The `tool_use_behavior="stop_on_first_tool"` mentioned means:
- Normal: Tool → Result → LLM processes result → Final response
- With this setting: Tool → Result → Stop (result IS the response)
