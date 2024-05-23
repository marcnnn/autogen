# Autogen

This is a work-in-progress Elixir port for Microsoft's agent framework: [Autogen](https://microsoft.github.io/autogen/).
This is highly experimental and not at all ready for use in production.

## Usage

### Conversation between a user (on command-line) and a simple LLM assistant:
```elixir
    assistant = %XAgent{name: "Assistant", system_prompt: "You are a helpful chatbot", type: :conversable_agent}
    user = %XAgent{name: "user", type: :user_proxy_agent}

    XAgent.initiate_chat(from_agent: assistant, to_agent: user, message: "How can I help you today?")
```

### A back-and-forth conversation between two AI comedians:
```elixir
    joe = %XAgent{
      name: "Joe",
      system_prompt: "Your name is Joe and you are a part of a duo of comedians.",
      type: :conversable_agent,
      llm: %{temperature: 0.9},
      human_input_mode: :never,
      max_consecutive_auto_reply: 1,
      is_termination_msg: fn msg -> String.contains?(String.downcase(msg.content), "bye") end
    }

    cathy = %XAgent{
      name: "Cathy",
      system_prompt: "Your name is Cathy and you are a part of a duo of comedians.",
      type: :conversable_agent,
      llm: %{temperature: 0.7},
      human_input_mode: :never
    }

    XAgent.initiate_chat(from_agent: joe, to_agent: cathy, message: "Cathy, tell me a joke and then say the words GOOD BYE..", max_turns: 2)
```

### Code writer agent and Code executor agents collaborating to perform a task:

```elixir
    code_executor_agent = %XAgent{
      name: "code_executor_agent",
      type: :conversable_agent,
      is_code_executor: true,
      human_input_mode: :always,
      is_termination_msg: fn msg -> String.contains?(msg.content, "TERMINATE") end
    }

    code_writer_agent = %XAgent{
      name: "code_writer_agent",
      system_prompt: ~S"""
      You are a helpful AI assistant.
      Solve tasks using your coding and language skills.
      In the following cases, suggest Elixir code (in a elixir coding block) for the user to execute.
      The last expression in your code should compute and return the result (rather than print it).
      Check the execution result returned by the user.
      If the result indicates there is an error, fix the error and output the code again. Suggest the full code instead of partial code or code changes. If the error can't be fixed or if the task is not solved even after the code is executed successfully, analyze the problem, revisit your assumption, collect additional info you need, and think of a different approach to try.
      When you find an answer, verify the answer carefully. Include verifiable evidence in your response if possible.
      If the execution was successful and everything is done, reply with 'TERMINATE'.
      """,
      type: :conversable_agent
    }

    XAgent.initiate_chat(
      from_agent: code_executor_agent,
      to_agent: code_writer_agent,
      message: "Write Elixir code to calculate the 14th Fibonacci number."
    )
```


## Installation

This library has not yet been published on Hex. 
If you have cloned this repo, you can try it with `OPENAI_API_KEY=your_openai_key mix test` which will invoke the demos.
