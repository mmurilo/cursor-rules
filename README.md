# cursor-rules
Compilation of Cursor rules




## Links

https://github.com/eyaltoledano/claude-task-master/tree/main/.cursor/rules

https://dev.to/dpaluy/mastering-cursor-rules-a-developers-guide-to-smart-ai-integration-1k65





## PRD

### What Makes a Good PRD?

- Clear objective — what’s the outcome or feature?
- Context — what’s already in place or assumed?
- Constraints — what limits or requirements need to be respected?
- Reasoning — why are you building it this way?

The more context you give the model, the better the breakdown and results.
​
### Writing a PRD

PRD template: [example_prd.md](tamplates/example_prd.md)

co-write your PRD with an LLM model using the following workflow:

- Chat about requirements — explain what you want to build.
- Show an example PRD — share the example PRD so the model understands the expected format. The example uses formatting that work well with Task Master’s code. Following the example will yield better results.
- Iterate and refine — work with the model to shape the draft into a clear and well-structured PRD.

This approach works great in Cursor, or anywhere you use a chat-based LLM.

### Creating tasks

In Cursor’s AI chat, instruct the agent/planning to generate tasks from your PRD:

```
Please use the task-master parse-prd command to generate tasks from my PRD. The PRD is located at .taskmaster/docs/<prd-name>.md.
```
