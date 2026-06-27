# Loops Explained: Claude, GPT, Mira and What Actually Works

## Summary
Loops are systems that let AI move from a one-step prompt to a goal that it can work toward. Loops teach AI to not just give an answer but to keep working  unless it clears a verified condition. This article explains loops in abstract terms, how to build a basic loop ourselves, what loops actually cost, and why loops are relevant for everyday work - inside and outside code bases.

## Key Takeaways
- A loop is a goal the AI keeps working toward until it gets there; it has five parts: discover, plan, execute, verify, iterate.
- Verify is the heart of the loop; without it, the agent only confirms its own work.
- State makes a loop learn; each agent revision learns from its mistakes by visiting a documented state.
- A stop condition keeps the loop sane; no exit means the loop will run until it succeeds or breaks your bank.
- Coding loops mostly matter because code can be verified; a loop is worth it only when repeatable work exists, something can automatically reject bad output, the agent can do the work end to end, and "done" is objective.
- Building blocks of a loop: automation (automation), skill (what), sub-agents (make checker), connectors (act), verifier (the gate).
- Cost compounds; track cost per accepted change, not cost per call.
- Loops fail quietly when the verifier is loose or missing; they can drain money while doing nothing.
- Build a basic loop by giving the model a goal, strict success criteria, and a self-checking protocol.
- The simple version of loops is available in free tools like Mira; the heavy version needs engineering guardrails.
- The loop pattern is shifting from asking users to predict everything to sharing context briefly then getting the agent to take the reins.
- Start with manual runs, turn them into skills, wrap them in loops, then schedule them.

## Related
- [[loop-engineering/from-prompting-agents-to-loop-engineering]]
- [[loop-engineering/build-an-ai-that-codes-while-you-sleep]]
- [[ai-agents/overview]]