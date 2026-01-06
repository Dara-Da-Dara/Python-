# Parlant: A Comprehensive 20-Hour Course Module
## Introduction to Conversational AI Control Framework

---

## Course Overview

This comprehensive 20-hour course module covers the complete Parlant framework for building controlled, compliant, and purpose-driven conversational AI agents. Parlant is an open-source Alignment Engine designed specifically for production-ready customer-facing AI applications.

### What You'll Learn:
- **Hours 1-3**: Core concepts and architecture
- **Hours 4-6**: Guidelines system (the heart of Parlant)
- **Hours 7-9**: Tools and integrations
- **Hours 10-12**: Journeys and orchestration
- **Hours 13-15**: Advanced features (glossary, variables, retrievers, canned responses)
- **Hours 16-18**: Production deployment and observability
- **Hours 19-20**: Real-world case studies and best practices

### Prerequisites:
- Python 3.8+
- Basic understanding of LLMs and prompt engineering
- Familiarity with async/await patterns
- REST API concepts

---

## Part 1: Foundation & Architecture (Hours 1-3)

### Hour 1: Why Parlant? - The Problem It Solves

#### The Challenge of Production AI Agents

In traditional LLM-based agents:
- ❌ Agents work well in testing but fail with real users
- ❌ System prompts are frequently ignored
- ❌ Hallucinations occur at critical moments
- ❌ Behavior is inconsistent across conversations
- ❌ Edge cases create unpredictable responses
- ❌ Adding more guidelines creates conflicts

#### The Parlant Philosophy

Parlant shifts from **prompt engineering** to **behavior modeling**:
- ✅ Structure defines reliability
- ✅ Control is built into the engine
- ✅ Compliance is enforced, not hoped for
- ✅ Scalability is deterministic
- ✅ Explanability is built-in

#### Real-World Use Cases

Parlant excels in regulated industries:
- **Financial Services**: Compliance-first agents
- **Healthcare**: HIPAA-compliant appointment schedulers
- **Legal**: Document analysis with audit trails
- **Customer Service**: Brand-aligned support bots
- **Insurance**: Claims processing with guardrails

### Hour 2: Core Architecture Overview

#### The Parlant Architecture

```
┌─────────────────────────────────────────┐
│     Parlant REST API / SDK              │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│    Parlant AI Response Engine           │
├──────────────────────────────────────────┤
│  ┌─────────────────────────────────────┐│
│  │ 1. Load Domain Terminology          ││
│  │    (Glossary Store)                 ││
│  └─────────────────────────────────────┘│
│  ┌─────────────────────────────────────┐│
│  │ 2. Match Relevant Guidelines        ││
│  │    (Guideline Matcher)              ││
│  └─────────────────────────────────────┘│
│  ┌─────────────────────────────────────┐│
│  │ 3. Infer & Call Tools               ││
│  │    (Tool Caller)                    ││
│  └─────────────────────────────────────┘│
│  ┌─────────────────────────────────────┐│
│  │ 4. Tailor Message & Self-Critique   ││
│  │    (Message Composer)               ││
│  └─────────────────────────────────────┘│
└──────────────────────────────────────────┘
               │
        ┌──────▼──────┐
        │  LLM Model  │
        └─────────────┘
```

#### Key Components

1. **Agents**: Customized AI personalities with their own configuration
2. **Guidelines**: Behavior rules (condition-action pairs)
3. **Tools**: External functions/APIs the agent can call
4. **Journeys**: Multi-step conversation flows
5. **Sessions**: Persistent conversation containers
6. **Customers**: User entities with profiles and history

#### Design Philosophy: Single vs. Multi-Agent

```python
# Parlant's recommendation: Model agents like real people
# If it would be ONE person in real-life, make it ONE agent

# ✅ GOOD: Single comprehensive agent
async with p.Server() as server:
    agent = await server.create_agent(
        name="Emma",  # One consistent personality
        description="Comprehensive customer support specialist"
    )

# ❌ LESS IDEAL: Multi-agent for simple coordination
# Multi-agent systems often create coordination overhead
# Parlant's architecture handles complexity in single agents
```

### Hour 3: Getting Started & Setup

#### Installation & Quick Start

```bash
# Install Parlant
pip install parlant

# Start the server
parlant-server run

# Visit http://localhost:8800 in your browser
```

#### Your First Agent

```python
import asyncio
import parlant.sdk as p

async def main():
    # Create a Parlant server
    async with p.Server() as server:
        # Create an agent
        agent = await server.create_agent(
            name="Alex",
            description="A helpful customer support specialist"
        )
        
        # Add a simple guideline
        await agent.create_guideline(
            condition="The customer greets you",
            action="Greet them warmly and ask how you can help"
        )
        
        # Start the server
        await server.serve()

if __name__ == "__main__":
    asyncio.run(main())
```

#### Programmatic Session Management

```python
import asyncio
import parlant.sdk as p

async def interact_with_agent():
    # Create API client
    client = p.ParlantClient(
        base_url="http://localhost:8800",
        api_key="your-api-key"  # Required in production
    )
    
    # Create a session
    session = await client.sessions.create(
        agent_id="alex",
        customer_id="cust-123",
        title="Support Ticket #001"
    )
    
    # Send a message from customer
    event = await client.sessions.create_event(
        session_id=session.id,
        kind="message",
        source="customer",
        message="Hello, I need help with my order"
    )
    
    # Listen for agent response
    events = await client.sessions.list_events(session_id=session.id)
    for event in events:
        if event.source == "agent":
            print(f"Agent: {event.message}")

asyncio.run(interact_with_agent())
```

---

## Part 2: Guidelines System - Core Control Mechanism (Hours 4-6)

### Hour 4: Understanding Guidelines

#### What Are Guidelines?

Guidelines are the **heart** of Parlant. They are condition-action pairs that define how your agent should behave.

```
When <CONDITION>, Then <ACTION>
```

#### Anatomy of a Guideline

```python
import parlant.sdk as p

# A guideline has two essential parts:
await agent.create_guideline(
    # CONDITION: When should this apply?
    # (Natural language description matched semantically)
    condition="The customer asks about refund policy",
    
    # ACTION: What should the agent do?
    # (Instructions for how to respond)
    action="Explain that we offer full refunds within 30 days"
)
```

#### How Guidelines Are Matched

Parlant uses **semantic matching** to determine when guidelines apply:

```python
# This guideline has a semantic condition
await agent.create_guideline(
    condition="Customer wants to return an item",
    action="First get the order number, then help them return it"
)

# These customer messages would match:
customer_messages = [
    "I want to send this back",  # ✓ Matches
    "Can I return this?",        # ✓ Matches
    "This doesn't work",          # ✓ Matches (context-aware)
    "What's your return policy?", # ✗ Doesn't quite match
]
```

#### The Criticality Level

```python
# Guidelines have priority levels that affect how hard
# the agent tries to follow them

await agent.create_guideline(
    condition="Customer provides credit card information",
    action="NEVER display the full credit card number in responses",
    criticality=p.Criticality.HIGH  # ← Must follow this!
)

await agent.create_guideline(
    condition="Customer seems frustrated",
    action="Offer empathy and patience",
    criticality=p.Criticality.MEDIUM  # ← Try to follow
)

await agent.create_guideline(
    condition="It's a holiday",
    action="Wish them a happy holiday",
    criticality=p.Criticality.LOW  # ← Nice to have
)
```

### Hour 5: Advanced Guideline Patterns

#### Guideline Activation Modes

```python
# 1. GLOBALLY ACTIVATED: Always active
await agent.create_guideline(
    condition="Customer asks anything",
    action="Be polite and professional"
)

# 2. CONDITIONALLY ACTIVATED: Active when condition is met
await agent.create_guideline(
    condition="Customer is identified as VIP",
    action="Offer premium support options"
)

# 3. DYNAMICALLY ACTIVATED: Based on real-time context
# (Parlant handles this automatically through semantic matching)
```

#### Guideline Relationships & Dependencies

```python
# Create relationships between guidelines using observations
# (Guidelines without actions)

# Observation: Create a guideline that just establishes context
ambiguous_request = await agent.create_observation(
    condition="Customer wants limits but unclear which type"
)

# Set up dependencies
await ambiguous_request.disambiguate([
    fetch_atm_limit_guideline,
    fetch_credit_card_limit_guideline
])

# Example: When the ambiguous observation matches,
# Parlant will ask for clarification before proceeding
```

#### Handling Edge Cases

```python
# Guidelines for out-of-scope interactions
await agent.create_guideline(
    condition="Customer asks about topics unrelated to our service",
    action="Politely decline and redirect to relevant topics",
    criticality=p.Criticality.HIGH
)

# Guidelines for policy conflicts
await agent.create_guideline(
    condition="Customer explicitly asks for something against policy",
    action="Explain why we cannot do that, then offer alternatives"
)

# Guidelines with state tracking
await agent.create_guideline(
    condition="The user has already asked this question in this session",
    action="Reference your previous answer instead of repeating"
)
```

### Hour 6: Guidelines in Production

#### Best Practices for Guideline Design

```python
# ✅ GOOD: Specific and bounded conditions
await agent.create_guideline(
    condition="Customer asks for product recommendations without specifying preferences",
    action="Ask about their specific needs before making recommendations"
)

# ❌ POOR: Vague and unbounded conditions
await agent.create_guideline(
    condition="Customer asks about products",
    action="Recommend something they might like"
)

# ✅ GOOD: Clear, actionable instructions
await agent.create_guideline(
    condition="Customer expresses dissatisfaction with service",
    action="Acknowledge their frustration specifically, express empathy, ask for details"
)

# ❌ POOR: Ambiguous instructions
await agent.create_guideline(
    condition="Customer is unhappy",
    action="Be nice"
)
```

#### Guideline Versioning & Monitoring

```python
# Guidelines should be part of version control
# Add metadata for tracking changes

guideline_metadata = {
    "owner": "customer_success_team",
    "created": "2024-01-15",
    "updated": "2025-01-06",
    "reason": "Updated refund policy per new compliance requirements",
    "related_jira": "COMP-4521"
}

# Monitor guideline performance through OpenTelemetry
export OTEL_EXPORTER_OTLP_TRACES_ENDPOINT=http://localhost:4317
```

#### Testing Guidelines

```python
# Create test scenarios for guidelines
test_scenarios = [
    {
        "input": "I want to return my order",
        "expected_guideline": "return_process",
        "should_match": True
    },
    {
        "input": "What's your weather forecast?",
        "expected_guideline": "return_process",
        "should_match": False
    }
]

# Use Parlant's policy graph builder to validate
# relationships between guidelines
policy_graph = await agent.get_policy_graph()
```

---

## Part 3: Tools System - External Integrations (Hours 7-9)

### Hour 7: Building Tools

#### What Are Tools?

Tools are functions that agents can call to:
- Fetch data from APIs
- Query databases
- Execute external actions
- Integrate with business systems

#### Basic Tool Structure

```python
import parlant.sdk as p
from typing import Dict, List

# Define a tool using the @p.tool decorator
@p.tool
async def check_order_status(
    context: p.ToolContext,
    order_id: str
) -> p.ToolResult:
    """
    Check the status of a customer's order.
    
    Args:
        order_id: The unique identifier for the order
    
    Returns:
        Detailed order information including status
    """
    
    # Tool logic here
    try:
        order_info = await fetch_from_database(order_id)
        return p.ToolResult(
            data={
                "order_id": order_id,
                "status": order_info["status"],
                "items": order_info["items"],
                "estimated_delivery": order_info["eta"]
            }
        )
    except OrderNotFound:
        return p.ToolResult(
            data=None,
            error="Order not found"
        )
```

#### Tool Parameters & Type Safety

```python
@p.tool
async def book_appointment(
    context: p.ToolContext,
    doctor_name: str,
    appointment_date: str,
    time_slot: str
) -> p.ToolResult:
    """Book an appointment with a doctor."""
    
    # Parameters are automatically validated
    # Type hints provide the agent with parameter information
    # The agent understands what information is needed
    
    return p.ToolResult(
        data={
            "confirmation_number": "APT-12345",
            "doctor": doctor_name,
            "date": appointment_date,
            "time": time_slot
        }
    )
```

#### Tool Context: Accessing Session Data

```python
@p.tool
async def get_customer_info(
    context: p.ToolContext
) -> p.ToolResult:
    """Get customer information from the context."""
    
    # Access customer from context
    customer_id = context.customer_id
    customer_tags = context.customer_tags
    
    # Access conversation history
    messages = context.interaction.messages
    last_message = context.interaction.last_customer_message
    
    # Access session variables
    account_tier = context.variables.get("account_tier")
    
    # Access the customer object
    customer = context.customer
    
    return p.ToolResult(
        data={
            "customer_id": customer_id,
            "account_tier": account_tier,
            "conversation_context": len(messages)
        }
    )
```

### Hour 8: Tool Integration with Guidelines

#### Conditional Tool Activation

```python
# Tools are only exposed when their guideline is active
# This prevents unnecessary tool calls and reduces context overload

await agent.create_guideline(
    condition="Customer wants to return an item",
    action="Get the order number and item name and then help them return it",
    tools=[check_order_status, process_return]  # ← Only available when matched
)

# Different tools for different guidelines
await agent.create_guideline(
    condition="Customer wants to schedule an appointment",
    action="Help them find an available slot",
    tools=[get_available_slots, book_appointment]
)

await agent.create_guideline(
    condition="Customer asks about refund status",
    action="Check their refund status",
    tools=[check_refund_status]
)
```

#### Tool Parameter Handling

```python
# Tools have sophisticated parameter management

@p.tool
async def transfer_funds(
    context: p.ToolContext,
    amount: float,  # Required parameter
    recipient_account: str,
    memo: str = ""  # Optional parameter
) -> p.ToolResult:
    """Transfer funds to another account."""
    
    return p.ToolResult(data={"status": "success"})

# Define how parameters should be obtained
await agent.create_guideline(
    condition="Customer wants to send money",
    action="Initiate a transfer",
    tools=[transfer_funds],
    tool_configuration={
        "transfer_funds": {
            "amount": {
                "source": "customer",  # Ask customer
                "description": "How much do you want to transfer?"
            },
            "recipient_account": {
                "source": "context",  # Extract from conversation
                "description": "The recipient's account from context"
            }
        }
    }
)
```

#### Error Handling in Tools

```python
@p.tool
async def query_database(
    context: p.ToolContext,
    query: str
) -> p.ToolResult:
    """Query the database safely."""
    
    try:
        result = await execute_query(query)
        return p.ToolResult(
            data=result,
            control={
                "success": True,
                "message": f"Query returned {len(result)} results"
            }
        )
    except DatabaseError as e:
        # Return error but allow agent to handle gracefully
        return p.ToolResult(
            data=None,
            error=str(e),
            control={"retry_allowed": True}
        )
    except SecurityException as e:
        # Critical errors that should stop execution
        return p.ToolResult(
            data=None,
            error="Security violation",
            control={"retry_allowed": False}
        )
```

### Hour 9: Advanced Tool Patterns

#### Async Tools & Performance

```python
# All Parlant tools are async by default

@p.tool
async def fetch_multiple_sources(
    context: p.ToolContext,
    customer_id: str
) -> p.ToolResult:
    """
    Fetch data from multiple sources in parallel.
    This is much faster than sequential calls.
    """
    
    import asyncio
    
    # Run multiple async operations in parallel
    orders, invoices, tickets = await asyncio.gather(
        fetch_orders(customer_id),
        fetch_invoices(customer_id),
        fetch_support_tickets(customer_id)
    )
    
    return p.ToolResult(
        data={
            "orders": orders,
            "invoices": invoices,
            "tickets": tickets
        }
    )
```

#### Tool Insights & Message Composition

```python
# Tool insights bridge tool calling and message composition
# They tell the message composer about tool execution results

@p.tool
async def validate_email(
    context: p.ToolContext,
    email: str
) -> p.ToolResult:
    """
    Validate an email address.
    Return insights for the message composer.
    """
    
    is_valid = await email_validation_service.check(email)
    
    return p.ToolResult(
        data={
            "email": email,
            "valid": is_valid
        },
        # Provide insights about the result
        metadata={
            "could_not_extract_parameter": False,
            "tool_execution_failed": False,
            "missing_required_parameters": []
        }
    )
```

#### Canned Response Fields in Tools

```python
# Tools can provide field values for canned responses

@p.tool
async def get_account_balance(
    context: p.ToolContext
) -> p.ToolResult:
    """Get the customer's account balance."""
    
    balance = await database.get_balance(context.customer_id)
    
    return p.ToolResult(
        data={
            "current_balance": balance,
            "currency": "USD"
        },
        canned_response_fields={
            "account_balance": balance,
            "currency_symbol": "$"
        }
    )

# Use these fields in canned responses
await agent.create_canned_response(
    template="Your current account balance is {{currency_symbol}}{{account_balance}}"
)
```

---

## Part 4: Journeys - Multi-Step Orchestration (Hours 10-12)

### Hour 10: Journey Fundamentals

#### What Are Journeys?

Journeys guide agents through **multi-turn conversational flows** like:
- Booking appointments
- Processing returns
- Onboarding customers
- Troubleshooting issues

```
Journey = State Machine + Conversation Guide
```

#### Journey Structure

A journey consists of:
- **States**: Points in the conversation flow
- **Transitions**: Paths between states with conditions
- **Chat States**: Conversational milestones
- **Tool States**: Automated actions

#### Creating Your First Journey

```python
import parlant.sdk as p
import asyncio

async def create_simple_booking_journey(agent: p.Agent) -> p.Journey:
    """Create a journey to help book appointments."""
    
    # Create the journey
    journey = await agent.create_journey(
        title="Book Appointment",
        description="Guide customers through booking an appointment",
        conditions=[
            "The customer wants to book an appointment",
            "The customer wants to schedule a meeting"
        ]
    )
    
    # State 1: Ask for preferred date
    state1_transition = await journey.initial_state.transition_to(
        chat_state="Ask what date and time work best for them"
    )
    
    # State 2: Check availability
    state2_transition = await state1_transition.target.transition_to(
        tool_state=check_availability,  # Call a tool
        condition="The customer has specified their preferred time"
    )
    
    # State 3: Confirm booking
    state3_transition = await state2_transition.target.transition_to(
        chat_state="Confirm the appointment details",
        condition="Availability was found"
    )
    
    # State 4: Complete booking
    await state3_transition.target.transition_to(
        tool_state=book_appointment,
        condition="The customer confirmed the details"
    )
    
    return journey
```

#### Journey Activation

```python
# Journeys are automatically activated when their conditions match
await agent.create_journey(
    title="Return Process",
    conditions=[
        "The customer wants to return an item",
        "The customer asks about returns"
    ],
    # ... journey states ...
)

# When a customer says "I want to return this"
# The journey automatically activates!
```

### Hour 11: Advanced Journey Patterns

#### Conditional Transitions

```python
async def create_complex_booking_journey(agent: p.Agent) -> p.Journey:
    """Journey with multiple paths based on conditions."""
    
    journey = await agent.create_journey(
        title="Flight Booking",
        conditions=["The customer wants to book a flight"],
        description="Guide through complete flight booking"
    )
    
    # Ask for destination
    t1 = await journey.initial_state.transition_to(
        chat_state="Ask where they want to go"
    )
    
    # Ask for dates
    t2 = await t1.target.transition_to(
        chat_state="Ask for travel dates"
    )
    
    # PATH A: Direct flight available
    t3a = await t2.target.transition_to(
        chat_state="Offer direct flights",
        condition="Direct flights are available"
    )
    
    # PATH B: No direct flights, show connections
    t3b = await t2.target.transition_to(
        chat_state="Offer connecting flights",
        condition="Only connecting flights are available"
    )
    
    # PATH C: No flights at all
    t3c = await t2.target.transition_to(
        chat_state="Apologize and suggest alternative dates",
        condition="No flights are available"
    )
    
    # All paths converge to confirmation
    confirm_state = await t3a.target.transition_to(
        chat_state="Confirm booking details"
    )
    await t3b.target.transition_to(state=confirm_state)
    await t3c.target.transition_to(state=confirm_state)
    
    return journey
```

#### Fork States

```python
async def create_support_journey(agent: p.Agent) -> p.Journey:
    """Journey with explicit fork for decision branches."""
    
    journey = await agent.create_journey(
        title="Technical Support",
        conditions=["The customer needs technical help"]
    )
    
    # Initial diagnostic
    t1 = await journey.initial_state.transition_to(
        chat_state="Describe the technical issue you're experiencing"
    )
    
    # Fork: Branch based on issue type
    fork = await t1.target.create_fork_state()
    
    # Billing issues
    billing_branch = await fork.transition_to(
        chat_state="I'll help with your billing issue",
        condition="The issue is billing-related"
    )
    
    # Technical issues
    tech_branch = await fork.transition_to(
        chat_state="I'll help troubleshoot the technical problem",
        condition="The issue is technical"
    )
    
    # Account issues
    account_branch = await fork.transition_to(
        chat_state="I'll help with your account",
        condition="The issue is account-related"
    )
    
    return journey
```

#### Journey-Scoped Guidelines

```python
async def setup_booking_journey(agent: p.Agent):
    """Journey with its own guidelines."""
    
    journey = await agent.create_journey(
        title="Hotel Booking",
        conditions=["The customer wants to book a hotel"]
    )
    
    # Add states...
    t1 = await journey.initial_state.transition_to(
        chat_state="Ask for check-in date"
    )
    
    # Add a guideline that ONLY applies during this journey
    # This guideline won't activate outside the journey
    await journey.create_guideline(
        condition="The customer hasn't selected a room type yet",
        action="Ask about room preferences",
        criticality=p.Criticality.MEDIUM
    )
    
    # Handle digressions from the journey flow
    await journey.create_guideline(
        condition="The customer changes their mind",
        action="Acknowledge and ask if they want to continue or cancel",
        tools=[cancel_booking]
    )
    
    return journey
```

### Hour 12: Journey Performance & Context Management

#### Managing Context in Journeys

```python
# Parlant automatically manages context to avoid overload

# When journey starts:
# ✓ Only the journey's states are visible to agent
# ✓ Only journey-relevant guidelines are active
# ✓ Context is trimmed to conversational relevant facts

# This keeps the agent's "attention" focused
# Reducing errors and improving response quality

# Context management happens in phases:

# Phase 1: EARLY (message acknowledged)
# - Parallel guideline matching
# - Retriever execution
# - Journey prediction

# Phase 2: MIDDLE (LLM processing)
# - State matching for active journeys
# - Tool availability checking
# - Dynamic context injection

# Phase 3: FINAL (before response)
# - Canned response matching
# - Message composition
# - Quality checks
```

#### Journey State Scoped Responses

```python
async def setup_journey_with_canned_responses(agent: p.Agent):
    """Journey states with canned responses."""
    
    journey = await agent.create_journey(
        title="FAQ Support",
        conditions=["The customer asks a frequently asked question"]
    )
    
    # State: Handling billing questions
    billing_state = await journey.initial_state.transition_to(
        chat_state="Answer billing questions"
    )
    
    # Responses only available in this state
    await billing_state.create_canned_response(
        template="Our billing runs on a monthly cycle, charged on the 1st of each month."
    )
    
    await billing_state.create_canned_response(
        template="You can update your payment method in account settings."
    )
    
    return journey
```

#### Adaptive Journey Flow

```python
# Parlants journeys are NOT rigid - they adapt!

# Customers can:
# ✓ Skip states if they provide information early
# ✓ Revisit previous states for clarification
# ✓ Jump ahead if context is clear
# ✓ Go off-path and return with guidelines

async def journey_with_natural_flow(agent: p.Agent):
    """Journey that feels natural despite structure."""
    
    journey = await agent.create_journey(
        title="Natural Booking Flow",
        conditions=["Customer wants to book"]
    )
    
    # Normal flow: date → time → confirm
    t1 = await journey.initial_state.transition_to(
        chat_state="When would you like to book?"
    )
    
    t2 = await t1.target.transition_to(
        chat_state="What time works best?"
    )
    
    t3 = await t2.target.transition_to(
        chat_state="Confirm your booking?"
    )
    
    # BUT: If customer says "I want Tuesday at 2pm"
    # Agent skips the separate date/time states
    # Parlant's journey system handles this gracefully!
    
    return journey
```

---

## Part 5: Advanced Features (Hours 13-15)

### Hour 13: Glossary & Domain Terminology

#### What Is the Glossary?

The glossary is your agent's **professional dictionary** - it teaches the agent domain-specific terms.

```
Glossary = Static Knowledge About Your Domain
Tools = Dynamic Data From Your Systems
```

#### Creating Glossary Terms

```python
import parlant.sdk as p

async def setup_hotel_glossary(agent: p.Agent):
    """Set up domain terminology for hotel booking agent."""
    
    # Basic glossary entry
    await agent.create_term(
        name="Ocean View Room",
        description="Premium rooms on floors 15-20 facing the Atlantic Ocean",
        synonyms=["seaside room", "beach view", "oceanfront"]
    )
    
    # Term with specific product details
    await agent.create_term(
        name="Sunrise Package",
        description="Includes complimentary breakfast and early check-in (8 AM) for Ocean View bookings",
        synonyms=["morning special", "sunrise special"]
    )
    
    # Service term
    await agent.create_term(
        name="Club Member",
        description="A guest who has stayed with us more than 5 times, entitled to 15% discount",
        synonyms=["loyal guest", "regular guest", "VIP member"]
    )
    
    # Policy term
    await agent.create_term(
        name="Cancellation Policy",
        description="Free cancellation up to 48 hours before arrival. After 48 hours, one night charged.",
        synonyms=["cancellation terms", "cancellation rules"]
    )
```

#### Glossary vs Guidelines vs Tools

```python
# All three work together but serve different purposes:

# GLOSSARY: What is this thing?
await agent.create_term(
    name="Express Checkout",
    description="Automated checkout process that takes <5 minutes"
)

# GUIDELINE: How should I behave when this comes up?
await agent.create_guideline(
    condition="Customer asks about Express Checkout",
    action="Highlight that it's fast and convenient"
)

# TOOL: How do I access current data about this?
@p.tool
async def enable_express_checkout(context: p.ToolContext) -> p.ToolResult:
    """Enable Express Checkout for this customer."""
    # Access real-time system
    return p.ToolResult(data={"enabled": True})
```

#### Glossary in Condition Matching

```python
async def glossary_driven_behavior(agent: p.Agent):
    """The glossary improves guideline and condition matching."""
    
    # Define glossary terms
    await agent.create_term(
        name="High-Value Customer",
        description="A customer with lifetime value > $50,000",
        synonyms=["premium customer", "VIP"]
    )
    
    # Use the term in guideline conditions
    await agent.create_guideline(
        condition="Customer is a High-Value Customer and asks for support",
        action="Offer premium support options and prioritize their request"
    )
    
    # Now when a customer logs in with high lifetime value,
    # The agent recognizes them as a "High-Value Customer"
    # Even if the exact term wasn't used!
```

### Hour 14: Context Variables & Personalization

#### What Are Context Variables?

Variables enable **personalized, context-aware** agent behavior.

```python
import parlant.sdk as p

async def setup_context_variables(server: p.Server, agent: p.Agent):
    """Create variables for personalized behavior."""
    
    # Create a variable for subscription tier
    await server.variables.create(
        agent_id=agent.id,
        name="subscription_tier",
        description="The customer's subscription level"
    )
    
    # Create a variable for user preference
    await server.variables.create(
        agent_id=agent.id,
        name="communication_style",
        description="Preferred communication style (formal/casual/brief)"
    )
    
    # Create a variable for usage history
    await server.variables.create(
        agent_id=agent.id,
        name="feature_adoption",
        description="Which features the customer has tried"
    )
```

#### Assigning Variable Values

```python
async def set_customer_variables(client: p.ParlantClient):
    """Set variable values for a specific customer."""
    
    # Get the variable
    variables = await client.variables.list(agent_id="alex")
    subscription_var = [v for v in variables if v.name == "subscription_tier"][0]
    
    # Set value for a specific customer
    await client.variables.set_value(
        variable_id=subscription_var.id,
        customer_id="cust-john-123",
        value="premium"
    )
    
    # Set value for a customer tag/group
    await client.variables.set_value(
        variable_id=subscription_var.id,
        tag="vip_customers",
        value="enterprise"
    )
```

#### Variables in Guidelines

```python
async def use_variables_in_guidelines(agent: p.Agent):
    """Use context variables in guideline conditions."""
    
    # Guideline that depends on subscription tier
    await agent.create_guideline(
        condition="The customer's subscription_tier is 'free' AND they ask about advanced features",
        action="Encourage them to upgrade to access advanced features"
    )
    
    # Guideline for premium customers
    await agent.create_guideline(
        condition="The customer's subscription_tier is 'premium' OR 'enterprise'",
        action="Offer concierge-level support and priority routing"
    )
    
    # Guideline based on feature adoption
    await agent.create_guideline(
        condition="The customer has NOT tried the 'automation' feature",
        action="Mention automation as a time-saving capability"
    )
```

#### Tool-Enabled Variables

```python
# Variables can be refreshed from tools!
# This ensures always-current information

@p.tool
async def check_account_status(
    context: p.ToolContext
) -> p.ToolResult:
    """Fetch current account status."""
    status = await database.get_account_status(context.customer_id)
    return p.ToolResult(data=status)

async def create_dynamic_variable(server: p.Server, agent: p.Agent):
    """Create a variable that refreshes from a tool."""
    
    # The 'account_status' variable will call check_account_status
    # periodically to stay fresh
    await server.variables.create(
        agent_id=agent.id,
        name="account_status",
        description="Current account status (from tool)",
        service_name="default",
        tool_name="check_account_status",
        freshness_rules={
            "max_age_seconds": 300  # Refresh every 5 minutes
        }
    )
```

### Hour 15: Retrievers & Canned Responses

#### Understanding Retrievers (RAG)

Retrievers implement **Retrieval-Augmented Generation** for contextual knowledge.

```python
import parlant.sdk as p

async def simple_retriever(
    context: p.RetrieverContext
) -> p.RetrieverResult:
    """Fetch relevant documents based on conversation."""
    
    # Get the last customer message
    if last_message := context.interaction.last_customer_message:
        message_text = last_message.content
        
        # Embed the message
        vector = await embedder.embed(message_text)
        
        # Search vector database for similar documents
        documents = await vector_db.search(vector, top_k=3)
        
        return p.RetrieverResult(documents)
    
    return p.RetrieverResult(None)

# Attach retriever to agent
await agent.attach_retriever(simple_retriever)
```

#### Deferred Retrievers

```python
async def conditional_retriever(
    context: p.RetrieverContext
) -> p.DeferredRetriever:
    """Retrieve documents conditionally based on matched guidelines."""
    
    # This part runs early (parallel with guideline matching)
    documents = await fetch_candidate_documents(
        context.interaction.last_customer_message
    )
    
    # Define what to do after guidelines are matched
    async def deferred(engine_ctx: p.EngineContext) -> p.RetrieverResult | None:
        # Check which guidelines matched
        matched_guidelines = engine_ctx.matched_guidelines
        
        # Only include retriever results if relevant guideline matched
        if "technical_support" in [g.name for g in matched_guidelines]:
            return p.RetrieverResult(documents)
        
        # Otherwise, skip adding to context
        return None
    
    return deferred
```

#### Canned Responses

```python
async def setup_canned_responses(agent: p.Agent):
    """Create pre-approved response templates."""
    
    # Basic canned response
    await agent.create_canned_response(
        template="Thank you for contacting us. How can I help you today?"
    )
    
    # Response with fields from tools
    await agent.create_canned_response(
        template="Your current balance is {{currency_symbol}}{{account_balance}}. Is there anything else?"
    )
    
    # Response with signals (examples of when to use it)
    await agent.create_canned_response(
        template="We offer full refunds within 30 days of purchase.",
        signals=[
            "What's your refund policy?",
            "Can I get my money back?",
            "Is this returnable?"
        ]
    )
```

#### Composition Modes

```python
async def setup_composition_modes(agent: p.Agent):
    """Different strategies for response composition."""
    
    # FLUID (default): Use canned responses when they fit
    await agent.create_guideline(
        condition="Customer asks general questions",
        action="Provide helpful information",
        composition_mode=p.CompositionMode.FLUID
    )
    
    # COMPOSITED: Blend canned response style into generated response
    await agent.create_guideline(
        condition="Customer interaction is brand-sensitive",
        action="Respond professionally",
        composition_mode=p.CompositionMode.COMPOSITED
    )
    
    # STRICT: Only use canned responses (no hallucinations!)
    await agent.create_guideline(
        condition="Customer asks about prices or legal terms",
        action="Only provide pre-approved information",
        composition_mode=p.CompositionMode.STRICT
    )
```

---

## Part 6: Production Deployment & Observability (Hours 16-18)

### Hour 16: Deployment Strategies

#### Local Development Setup

```bash
# 1. Install Parlant
pip install parlant

# 2. Start server
parlant-server run

# 3. Access UI at http://localhost:8800

# Optional: Use Ollama for local LLM
docker run -d -v ollama:/root/.ollama -p 11434:11434 ollama/ollama
ollama pull llama2
```

#### Docker Deployment

```dockerfile
# Dockerfile for Parlant agent
FROM python:3.10-slim

WORKDIR /app

# Install dependencies
RUN pip install parlant

# Copy agent code
COPY agent.py .

# Configure environment
ENV PARLANT_LLM_MODEL=gpt-4
ENV PARLANT_API_PORT=8800

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD python -c "import requests; requests.get('http://localhost:8800/health')"

# Run agent
CMD ["python", "agent.py"]
```

#### Kubernetes Deployment

```yaml
# kubernetes deployment for Parlant
apiVersion: apps/v1
kind: Deployment
metadata:
  name: parlant-agent
spec:
  replicas: 3
  selector:
    matchLabels:
      app: parlant-agent
  template:
    metadata:
      labels:
        app: parlant-agent
    spec:
      containers:
      - name: parlant
        image: parlant-agent:latest
        ports:
        - containerPort: 8800
        env:
        - name: PARLANT_LLM_MODEL
          value: "gpt-4"
        - name: PARLANT_LOG_LEVEL
          value: "INFO"
        - name: OTEL_EXPORTER_OTLP_TRACES_ENDPOINT
          value: "http://otel-collector:4317"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8800
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8800
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Hour 17: API Hardening & Security

#### Authorization Policies

```python
import parlant.sdk as p
from fastapi import HTTPException
import jwt

class CustomAuthorizationPolicy(p.ProductionAuthorizationPolicy):
    """Custom authorization with JWT validation."""
    
    async def check_permission(
        self,
        request,
        permission: p.AuthorizationPermission
    ) -> bool:
        # Extract JWT token
        auth_header = request.headers.get("Authorization", "")
        if not auth_header.startswith("Bearer "):
            return False
        
        try:
            token = auth_header.split(" ")[1]
            payload = jwt.decode(
                token,
                secret_key="your-secret-key",
                algorithms=["HS256"]
            )
            
            # Check permissions based on token claims
            user_role = payload.get("role")
            required_role = self.get_required_role(permission)
            
            return self.has_role(user_role, required_role)
        except jwt.InvalidTokenError:
            return False
    
    def get_required_role(self, permission):
        """Map permissions to required roles."""
        if permission == p.AuthorizationPermission.CREATE_EVENT:
            return "customer"
        elif permission == p.AuthorizationPermission.DELETE_SESSION:
            return "admin"
        return "customer"
    
    def has_role(self, user_role, required_role):
        role_hierarchy = {"admin": 2, "moderator": 1, "customer": 0}
        return role_hierarchy.get(user_role, -1) >= role_hierarchy.get(required_role, 0)

# Configure server with custom policy
async def main():
    def configure_container(container):
        container.authorization_policy.override(CustomAuthorizationPolicy)
    
    async with p.Server(configure_container=configure_container) as server:
        agent = await server.create_agent(
            name="Secure Agent",
            description="Authorization-protected agent"
        )
        await server.serve()
```

#### Rate Limiting

```python
from limits import RateLimitItemPerMinute

class RateLimitedAuthPolicy(p.ProductionAuthorizationPolicy):
    """Authorization with rate limiting."""
    
    def __init__(self):
        super().__init__()
        
        # Global rate limits
        self.global_limits = {
            "create_event": RateLimitItemPerMinute(100),
            "create_session": RateLimitItemPerMinute(50)
        }
        
        # Per-user rate limits
        self.per_user_limits = {
            "create_event": RateLimitItemPerMinute(10),
            "create_session": RateLimitItemPerMinute(5)
        }
    
    async def check_rate_limit(self, request, permission):
        # Check global limit
        if not self.limiter.try_consume(
            self.global_limits[permission.name]
        ):
            raise HTTPException(status_code=429, detail="Rate limit exceeded")
        
        # Check per-user limit
        user_id = self.extract_user_id(request)
        if not self.user_limiter.try_consume(
            user_id,
            self.per_user_limits[permission.name]
        ):
            raise HTTPException(status_code=429, detail="User rate limit exceeded")
        
        return True
```

### Hour 18: Observability & Monitoring

#### OpenTelemetry Integration

```bash
# Set up Jaeger locally
docker run -d --name jaeger \
  -p 16686:16686 \
  -p 4317:4317 \
  jaegertracing/all-in-one:latest

# Configure Parlant
export OTEL_EXPORTER_OTLP_TRACES_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_INSECURE=true
export OTEL_SERVICE_NAME=parlant-agent

# View traces at http://localhost:16686
```

#### Custom Instrumentation

```python
import asyncio
import parlant.sdk as p
from opentelemetry import trace

async def instrumented_agent():
    """Agent with custom instrumentation."""
    
    tracer = trace.get_tracer(__name__)
    
    async with p.Server() as server:
        agent = await server.create_agent(
            name="Instrumented Agent",
            description="Agent with observability"
        )
        
        # Add custom span
        with tracer.start_as_current_span("agent_initialization"):
            # Your setup code
            pass
        
        # Monitor guideline performance
        async def monitored_guideline_creation():
            with tracer.start_as_current_span("create_guideline"):
                await agent.create_guideline(
                    condition="Customer asks about pricing",
                    action="Provide pricing information"
                )
        
        await monitored_guideline_creation()
        await server.serve()

asyncio.run(instrumented_agent())
```

#### Metrics & Dashboards

```python
# Metrics exported by Parlant

IMPORTANT_METRICS = {
    "message_generation_duration": "Time to generate response (ms)",
    "ttfm_duration": "Time to first message (ms)",
    "match_duration": "Time for guideline matching (ms)",
    "tool_call_duration": "Time for tool execution (ms)",
    "guideline_match_count": "Number of matched guidelines per request",
    "tool_error_rate": "Percentage of failed tool calls"
}

# Example: Dashboarding with Prometheus + Grafana
DASHBOARD_CONFIG = """
dashboard:
  title: Parlant Agent Monitoring
  panels:
    - title: Response Latency
      metric: message_generation_duration
      
    - title: Guideline Match Rate
      metric: guideline_match_count
      
    - title: Tool Success Rate
      metric: 1 - tool_error_rate
"""
```

---

## Part 7: Real-World Case Studies & Best Practices (Hours 19-20)

### Hour 19: Case Study - Insurance Claims Agent

#### Requirements

```python
"""
Insurance Claims Processing Agent

Requirements:
1. File new claims with validation
2. Track claim status  
3. Provide policy information
4. Ensure compliance (PII handling)
5. Escalate complex cases
"""
```

#### Implementation

```python
import parlant.sdk as p
from datetime import datetime
import asyncio

async def build_insurance_agent():
    async with p.Server() as server:
        # Create agent
        agent = await server.create_agent(
            name="Claims Assistant",
            description="A helpful insurance claims specialist. Professional, empathetic, and detail-oriented."
        )
        
        # === GLOSSARY ===
        await agent.create_term(
            name="Deductible",
            description="The amount you must pay out-of-pocket before insurance coverage begins",
            synonyms=["out-of-pocket cost", "cost-sharing"]
        )
        
        await agent.create_term(
            name="Coverage Period",
            description="The timeframe during which your policy is active",
            synonyms=["policy period", "active dates"]
        )
        
        # === CONTEXT VARIABLES ===
        await server.variables.create(
            agent_id=agent.id,
            name="customer_tier",
            description="Customer insurance tier (basic/standard/premium)"
        )
        
        # === TOOLS ===
        @p.tool
        async def file_claim(
            context: p.ToolContext,
            claim_type: str,
            incident_date: str,
            description: str
        ) -> p.ToolResult:
            """File a new insurance claim."""
            claim_id = f"CLM-{context.customer_id}-{datetime.now().timestamp()}"
            
            return p.ToolResult(
                data={
                    "claim_id": claim_id,
                    "status": "submitted",
                    "next_steps": "We'll review and contact you within 48 hours"
                },
                canned_response_fields={
                    "claim_number": claim_id
                }
            )
        
        @p.tool
        async def check_claim_status(
            context: p.ToolContext,
            claim_id: str
        ) -> p.ToolResult:
            """Check the status of a claim."""
            # Mock database lookup
            claims = {
                "CLM-123-456": {"status": "under_review", "progress": 50},
                "CLM-789-012": {"status": "approved", "amount": 5000}
            }
            
            if claim := claims.get(claim_id):
                return p.ToolResult(data=claim)
            return p.ToolResult(data=None, error="Claim not found")
        
        @p.tool
        async def get_policy_details(
            context: p.ToolContext
        ) -> p.ToolResult:
            """Get customer's policy information."""
            return p.ToolResult(
                data={
                    "policy_number": "POL-12345",
                    "deductible": 500,
                    "coverage": ["medical", "dental"],
                    "renewal_date": "2025-12-31"
                }
            )
        
        # === GUIDELINES ===
        
        # Greeting
        await agent.create_guideline(
            condition="The customer greets you",
            action="Warmly greet them and ask how you can assist with their insurance needs"
        )
        
        # File claim guideline
        await agent.create_guideline(
            condition="The customer wants to file a new claim",
            action="Help them through the claim process by collecting: claim type, incident date, and description",
            tools=[file_claim],
            criticality=p.Criticality.HIGH
        )
        
        # PII Protection (CRITICAL)
        await agent.create_guideline(
            condition="Customer provides sensitive information (SSN, credit card, health details)",
            action="NEVER display or repeat this sensitive information. Acknowledge receipt and reassure of privacy.",
            criticality=p.Criticality.HIGH,
            composition_mode=p.CompositionMode.STRICT
        )
        
        # Status check guideline
        await agent.create_guideline(
            condition="Customer wants to check claim status",
            action="Ask for their claim ID and provide current status",
            tools=[check_claim_status]
        )
        
        # Policy info guideline
        await agent.create_guideline(
            condition="Customer asks about their policy or coverage",
            action="Retrieve and explain their policy details clearly",
            tools=[get_policy_details]
        )
        
        # Escalation guideline
        await agent.create_guideline(
            condition="The customer's issue is complex or requires human judgment",
            action="Explain that you're escalating to a specialist and provide a reference number",
            tools=[escalate_to_human],
            criticality=p.Criticality.HIGH
        )
        
        # === CANNED RESPONSES ===
        
        await agent.create_canned_response(
            template="I understand this is stressful. Let me help you get this resolved quickly."
        )
        
        await agent.create_canned_response(
            template="Your claim {{claim_number}} has been submitted successfully. We'll review it and contact you within 48 hours.",
            signals=["claim submitted", "claim filed"]
        )
        
        await agent.create_canned_response(
            template="For data security, I cannot display full details like SSNs or credit card numbers. Rest assured, all your information is securely encrypted."
        )
        
        # === JOURNEYS ===
        
        # Journey: File New Claim
        filing_journey = await agent.create_journey(
            title="File New Claim",
            conditions=["The customer wants to file a claim"],
            description="Guide through claim filing process"
        )
        
        t1 = await filing_journey.initial_state.transition_to(
            chat_state="Ask what type of claim they want to file (medical, dental, etc.)"
        )
        
        t2 = await t1.target.transition_to(
            chat_state="Ask when the incident occurred"
        )
        
        t3 = await t2.target.transition_to(
            chat_state="Ask for a brief description of what happened"
        )
        
        t4 = await t3.target.transition_to(
            tool_state=file_claim,
            condition="All required information has been collected"
        )
        
        await t4.target.transition_to(
            chat_state="Confirm claim was filed and provide next steps"
        )
        
        await server.serve()

asyncio.run(build_insurance_agent())
```

### Hour 20: Best Practices & Common Pitfalls

#### Best Practices Checklist

```markdown
## ✅ DO's

1. **Start Simple**
   - Begin with basic guidelines
   - Add complexity gradually
   - Test thoroughly at each stage

2. **Be Specific in Conditions**
   - ✓ "Customer asks about Ocean View rooms"
   - ✗ "Customer asks about rooms"

3. **Be Clear in Actions**
   - ✓ "Ask about their room preferences before offering options"
   - ✗ "Be helpful"

4. **Separate Concerns**
   - Glossary: What things are
   - Guidelines: How to behave
   - Tools: How to access data

5. **Use Criticality Appropriately**
   - HIGH: Compliance, security, brand voice
   - MEDIUM: Normal business logic
   - LOW: Enhancements, nice-to-haves

6. **Version Control Everything**
   - Store agent configs in git
   - Track guideline changes
   - Use semantic versioning

7. **Monitor in Production**
   - Set up OpenTelemetry
   - Track key metrics
   - Alert on anomalies

## ❌ DON'Ts

1. **Don't Overuse Canned Responses**
   - ✗ 100% strict mode from day one
   - ✓ Start fluid, tighten based on issues

2. **Don't Create Vague Guidelines**
   - ✗ "When customer has an issue"
   - ✓ "When customer reports a technical error and needs troubleshooting"

3. **Don't Forget About Edge Cases**
   - ✗ Guidelines only for happy path
   - ✓ Guidelines for common error scenarios

4. **Don't Ignore Context Management**
   - ✗ Create unlimited retrievers
   - ✓ Only retrieve when necessary

5. **Don't Mix Responsibilities**
   - ✗ Tools that also create canned responses
   - ✓ Clear tool → result → response flow

6. **Don't Deploy Without Testing**
   - ✗ Push to production directly
   - ✓ Test in staging, gradual rollout

7. **Don't Ignore Latency**
   - ✗ Add unlimited tools and retrievers
   - ✓ Monitor p99 latency, optimize hot paths
```

#### Common Pitfalls & Solutions

```python
# PITFALL 1: Overlapping Guidelines
# ❌ Multiple guidelines matching at once causing conflicts
await agent.create_guideline(
    condition="Customer asks about products"  # Too broad
)
await agent.create_guideline(
    condition="Customer asks about products"  # Duplicate!
)

# ✅ Solution: Make conditions specific
await agent.create_guideline(
    condition="Customer asks about premium products specifically"
)
await agent.create_guideline(
    condition="Customer asks about budget products specifically"
)

# PITFALL 2: Context Overload
# ❌ Attaching too many retrievers
await agent.attach_retriever(retriever1)  # FAQ documents
await agent.attach_retriever(retriever2)  # User history
await agent.attach_retriever(retriever3)  # Related products
await agent.attach_retriever(retriever4)  # Team knowledge
# → Context explodes, agent gets confused

# ✅ Solution: Use deferred retrievers
async def deferred_retriever(ctx):
    documents = await fetch_candidates()
    async def deferred(engine_ctx):
        if "faq_question" in [g.name for g in engine_ctx.matched_guidelines]:
            return p.RetrieverResult(documents)
        return None
    return deferred

# PITFALL 3: Poor Tool Error Handling
# ❌ Tool fails, agent has no recovery path
@p.tool
async def get_data(context) -> p.ToolResult:
    result = await external_api.call()  # What if this fails?
    return p.ToolResult(data=result)

# ✅ Solution: Comprehensive error handling
@p.tool
async def get_data(context) -> p.ToolResult:
    try:
        result = await external_api.call(timeout=5)
        return p.ToolResult(data=result)
    except TimeoutError:
        return p.ToolResult(
            data=None,
            error="Service is temporarily slow",
            control={"retry_allowed": True}
        )
    except Exception as e:
        return p.ToolResult(
            data=None,
            error=f"Could not retrieve data: {type(e).__name__}",
            control={"retry_allowed": False}
        )

# PITFALL 4: Inflexible Journeys
# ❌ Journey locks customer into rigid flow
journey_state = "collect_date"
# Customer provides all info upfront → Stuck

# ✅ Solution: Adaptive journeys
# Parlant journeys naturally adapt
# Customer can skip states if they provided info early
# This just works!
```

#### Performance Optimization

```python
# Monitor these metrics
KEY_METRICS = {
    "message_generation_duration": "Target: < 1000ms",
    "ttfm_duration": "Target: < 300ms (p95)",
    "guideline_match_duration": "Target: < 100ms",
    "tool_call_duration": "Target: varies by integration"
}

# Optimization techniques

# 1. Parallel Execution
async def optimized_retriever(context):
    import asyncio
    # Run all in parallel, not sequentially
    results = await asyncio.gather(
        fetch_faq(),
        fetch_user_history(),
        fetch_recommendations()
    )
    return p.RetrieverResult(results)

# 2. Early Exit
@p.tool
async def expensive_lookup(context):
    # Check cache first
    if cached := await cache.get(context.customer_id):
        return p.ToolResult(data=cached)
    
    # Only do expensive work if not cached
    result = await expensive_operation()
    await cache.set(context.customer_id, result, ttl=3600)
    return p.ToolResult(data=result)

# 3. Deferred Processing
async def deferred_retriever(ctx):
    async def deferred(engine_ctx):
        # Only compute heavy retrieval if actually needed
        return await expensive_retrieval()
    return deferred

# 4. Batch Operations
# Instead of multiple tool calls:
# ❌ Slow
for item in items:
    await process_item(item)

# ✅ Fast
await batch_process_items(items)
```

#### Testing Strategy

```python
import pytest
from unittest.mock import AsyncMock

class TestInsuranceAgent:
    """Test suite for insurance claims agent."""
    
    @pytest.mark.asyncio
    async def test_claim_filing_flow(self):
        """Test complete claim filing journey."""
        async with p.Server() as server:
            agent = await server.create_agent(
                name="Test Agent"
            )
            
            # Verify guideline exists
            guidelines = await agent.list_guidelines()
            assert any("claim" in g.condition for g in guidelines)
    
    @pytest.mark.asyncio
    async def test_pii_protection(self):
        """Verify sensitive data is not exposed."""
        # Create agent with PII protection guideline
        # Test that SSN, credit cards, etc. are never repeated
        pass
    
    @pytest.mark.asyncio
    async def test_tool_error_handling(self):
        """Test graceful tool error handling."""
        # Mock tool failure
        # Verify agent handles gracefully
        pass

# Run tests
# pytest tests/test_insurance_agent.py -v
```

---

## Conclusion: Your Parlant Journey

### What You've Learned

- ✅ **Hours 1-3**: Architecture and design philosophy
- ✅ **Hours 4-6**: Guidelines system (core control mechanism)
- ✅ **Hours 7-9**: Tools and integrations
- ✅ **Hours 10-12**: Journeys and multi-turn orchestration
- ✅ **Hours 13-15**: Advanced features (glossary, variables, retrievers, canned responses)
- ✅ **Hours 16-18**: Production deployment and observability
- ✅ **Hours 19-20**: Real-world implementation and best practices

### Next Steps

1. **Start Small**: Build a simple agent with one guideline
2. **Iterate**: Add complexity based on real-world feedback
3. **Monitor**: Set up observability from day one
4. **Collaborate**: Version control your agent configurations
5. **Share**: Join the Parlant community on GitHub

### Resources

- **GitHub**: https://github.com/emcie-co/parlant
- **Documentation**: https://parlant.io/docs
- **Community**: Discord (link on website)
- **Examples**: Check the GitHub repository for example agents

### Key Principles to Remember

1. **Structure enables scale**: Behavior modeling beats prompt engineering
2. **Compliance is paramount**: Build it in from day one
3. **Context matters**: Keep agent context focused and relevant
4. **Test thoroughly**: Production agents need validation at every step
5. **Monitor obsessively**: You can't fix what you can't measure

Happy building with Parlant! 🚀
