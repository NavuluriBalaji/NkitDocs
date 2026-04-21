<div class="mixpanel-hero">
    <h1>Autonomous execution is possible</h1>
    <p>Simple and powerful multi-agent frameworks that help everyone make better decisions and build smarter code.</p>
    <a href="getting-started/quick-start/" class="mixpanel-btn">Let's Build</a>
    
    <div class="tunnel-container">
        <div class="layer layer-1">
            <div class="layer layer-2">
                <div class="layer layer-3">
                    <div class="layer layer-4">
                        <div class="layer layer-5">
                            <div class="layer layer-6">
                                <div class="layer layer-7">
                                    <div class="layer layer-8">
                                        <div class="layer layer-9">
                                            <div class="layer layer-10"></div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

---

## ⚡ Interactive Engine Capabilities

Instead of dumping a wall of text, explore our architecture by expanding the modules below:

???+ abstract "CodeACT Dynamic Sandboxing"
    Go beyond hardcoded tools! NKit agents can dynamically write Python functions, execute them in local sandboxes, and inject them into their own memory registry mid-flight. If a tool doesn't exist, **the agent builds it.**

??? info "Enterprise Safety & Auditing"
    - **`SafetyGate`**: Evaluates and intercepts all destructive tool calls *before* they execute.
    - **`WhyLog`**: Maintains a secure, tamper-proof JSONL audit trail of the agent's exact reasoning.

??? tip "Real-Time Observability"
    The **`LiveObserver`** class exposes an event streaming interface to pipe standard thoughts, actions, and failures directly to your frontend UI, terminal, or graphical IDE in real-time.

??? quote "Unified LLM Providers"
    Seamless, switchable integration for cloud, local, and universal models passing through standard HTTP architecture (GeminiLLM, LMStudioLLM, OpenAILLM, AnthropicLLM).

---

## 💻 Quick Start Example

You can initialize your environment exactly how you want it. Select your deployment style below:

=== "Local Network (LM Studio)"

    ```python
    from NKit import Agent, LiveObserver
    from NKit.llms import LMStudioLLM
    from NKit.safety import SafetyGate

    # Bypasses API keys and connects to your local inference server natively
    local_backend = LMStudioLLM(base_url="http://localhost:1234")

    # Inject your parameters
    agent = Agent(llm=local_backend, observer=LiveObserver(), safety_gate=SafetyGate())
    ```

=== "Cloud Network (Google Gemini)"

    ```python
    import os
    from NKit import Agent, LiveObserver
    from NKit.llms import GeminiLLM

    # Uses Google's ultra-fast API endpoints
    os.environ["GEMINI_API_KEY"] = "your-api-key"
    cloud_backend = GeminiLLM(model="gemini-2.5-flash")

    # Inject your parameters
    agent = Agent(llm=cloud_backend, observer=LiveObserver())
    ```

!!! success "Ready to Execute!"
    Once your agent is initialized, it operates across the async event loop instantly.
    ```python
    import asyncio
    result = asyncio.run(agent.run_async("Fetch the weather and write a short poem about it."))
    ```

---

## 📖 Architecture Documentation

Dive straight into the modules you need:

<div class="grid cards" markdown>

-   __Getting Started__
    ---
    Start spinning up agents in seconds.
    [:octicons-arrow-right-24: Installation](getting-started/installation.md)<br>
    [:octicons-arrow-right-24: Your First Agent](getting-started/quick-start.md)

-   __Core Concepts__
    ---
    Understand the engine block.
    [:octicons-arrow-right-24: Architecture](core-concepts/architecture.md)<br>
    [:octicons-arrow-right-24: Agents & CodeACT](core-concepts/agents.md)

-   __API Reference__
    ---
    Examine the strict internal protocols.
    [:octicons-arrow-right-24: Agent Lifecycle](api-reference/agent/agent.md)<br>
    [:octicons-arrow-right-24: Telemetry & Tracing](api-reference/telemetry/tracing.md)

</div>
