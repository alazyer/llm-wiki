---
title: "20 prompting rules that finally make Claude work the way you expected"
source: "https://x.com/AnatoliKopadze/status/2051665228044124548"
author:
  - "[[@AnatoliKopadze]]"
published: 2026-05-05
created: 2026-06-27
description: "Most people use Claude like a search engine. They type a question, get an answer, move on.That is not how Claude works. Claude responds to..."
tags:
  - "clippings"
---
![[5cea228d281084673140c06f2e971890_MD5.jpg]]

Most people use Claude like a search engine. They type a question, get an answer, move on. That is not how Claude works. Claude responds to context, roles, constraints, and framing. The same question asked two different ways gets two completely different answers. One of them is actually useful. These 20 rules are not specific prompts for specific tasks. They are the method. Learn these once and every conversation you have with Claude gets better, no matter what you are working on.

The difference between a mediocre Claude response and one that replaces an hour of work is almost always in how the request was written. Not in the model. In the prompt.

## Setup

**1 - Give Claude a role before the task**

Before you tell Claude what to do, tell Claude who to be. A role changes the entire lens Claude uses to approach the task. It shifts vocabulary, tone, depth, and what Claude considers relevant. The role does not need to be elaborate. Three words are enough. But without a role, Claude defaults to a generic assistant mode that is useful for everything and exceptional at nothing.

```text
You are a [specific role] who [key trait or value].

[Your actual task]
```

**2 - Tell Claude who will read this**

Claude has no idea who your audience is unless you say so. Without an audience, it writes for everyone, which means it writes for no one in particular. The more specific the audience description, the sharper the output. Age, background, what they already know, what they are skeptical about. Any detail you give makes the output fit better.

```text
Write this for a [specific person or profile] who already knows [what they know] and is skeptical about [their main objection].
```

**3 - Show Claude what you do NOT want**

Describing what you want is useful. Describing what you do not want is more useful. Negatives help Claude cut out the defaults it falls into without thinking: corporate language, excessive hedging, generic openers, unnecessary bullet points. If there is a pattern you hate seeing in Claude responses, name it explicitly. Claude will avoid it.

```text
Do not use corporate language.

Do not start with "Great question" or "Certainly".

Do not use bullet points.

Do not hedge every sentence with "it depends" or "it's important to note".
```

**4 - Set the length before you start**

Claude does not know whether you want three sentences or three pages unless you say so. Left to its own defaults, it will write something that feels complete but may be twice as long as what you actually needed, or half as developed. Be specific: a word count, a number of sentences, a number of paragraphs, or a description like "short enough to read in 30 seconds."

```text
Keep this under 150 words.

Write exactly 3 paragraphs.

Short enough to read in under 30 seconds.

Long enough to cover all the main objections, no longer.
```

**5 - Give Claude your own writing first**

When Claude writes in your voice without examples, it writes in its own voice. The output is grammatically fine and logically sound, but it does not sound like you. Paste three to five samples of your own writing and ask Claude to match the style before giving it any new task. Claude will study rhythm, sentence length, vocabulary, how you open paragraphs, how you end them. The output will be closer to what you would have written than anything Claude produces from scratch.

```text
Here are 3 examples of my writing:

[paste example 1]
[paste example 2]
[paste example 3]

Analyze my style: sentence length, vocabulary, tone, how I open and close paragraphs. 

Then write [new piece] in exactly that style.
```

## Context

**6 - Paste the full document, not a summary** When you summarize a document before giving it to Claude, you have already made editorial decisions about what matters. Claude is then working with your interpretation, not the source. Paste the full thing. Claude's context window is large enough to handle it. This matters most when asking Claude to find problems, inconsistencies, or things you might have missed. A summary hides exactly the things Claude would have caught.

```text
Here is the full document. Read all of it before responding.

[paste full document]

Now: [your actual question]
```

**7 - Tell Claude what you already know**

Claude does not know your background unless you tell it. If you are an expert asking about your own field, say that. If you are a complete beginner, say that too. Claude calibrates explanation depth to what it knows about you. Without this, Claude picks an assumed level that may be too basic or too technical. One sentence about your background eliminates an entire category of bad answers.

```text
I have been working in [field] for [X years]. 

I understand [what you know] but I am not familiar with [what you need explained]. 

Explain this at the right level.
```

**8 - Tell Claude the purpose, not just the task**

There is a difference between "write a subject line for this email" and "write a subject line for this email. The goal is to get a response from someone who has ignored three previous emails." The second one tells Claude why, which changes what Claude optimizes for. Purpose changes structure, tone, word choice, and what Claude decides to include or cut. When Claude knows why you need something, it makes better decisions about how to deliver it.

```text
[Task]. The goal is [specific outcome].

The person reading this will [what they will do or feel after].
```

**9 - Give Claude a constraint it has to work within**

Constraints make Claude more creative, not less. When Claude has unlimited space and freedom, it produces something complete and average. When it has a specific limitation to work around, it finds solutions you would not have thought of. Any real constraint works: word limit, format, tone, vocabulary level, medium, reading time. The tighter the constraint, the more interesting the result.

```text
Explain this using only words a 10-year-old would know.

Write this as a single sentence.

Make the argument using only numbers and facts, no opinions.

Write this so it works as both a tweet and an email subject line.
```

**10 - Start fresh when you switch topics**

Claude carries context from everything earlier in the conversation. When you switch to a completely different task in the same chat, that context can pull the new answer in the wrong direction. Claude may apply tone, framing, or assumptions from the previous task when they no longer apply. A new topic deserves a new conversation. It takes ten seconds to open one and it eliminates an entire category of subtle contamination in Claude's responses.

## Output

**11 - Ask Claude to think before it answers**

For complex tasks, tell Claude to work through the problem before giving you a final answer. This is not a trick. It genuinely changes what Claude produces. When Claude reasons through something step by step, it catches errors, finds gaps, and arrives at better conclusions than when it generates a response immediately. This matters most for anything involving decisions, plans, analysis, or creative problems with multiple possible approaches.

```text
Before you answer, think through this step by step. 

List your reasoning. 

Then give me your final answer at the end.

[your question or task]
```

**12 - Ask for multiple versions, not one**

When Claude gives you one version of something, it has already made every editorial decision for you. You get its best guess at what you want. When you ask for three versions with different angles or tones, you get options, and options let you choose or combine. This works especially well for headlines, subject lines, hooks, opening sentences, and anything where the first version is rarely the best one.

```text
Give me 3 versions of this.

Version 1: [tone or angle]

Version 2: [different tone or angle]

Version 3: [different tone or angle]

Then tell me which one you think is strongest and why.
```

**13 - Ask Claude to argue against you**

Before you commit to any plan, decision, or piece of writing, ask Claude to push back on it as hard as it can. Not politely. Not with caveats. With the strongest possible case against what you just said. The holes Claude finds are the ones that would have caused problems later. This is one of the highest-value things Claude can do and almost nobody does it.

```text
Here is my plan / idea / argument:

[paste it]

Now argue against it as hard as you can. 

Find every weak point. 

Assume I am wrong and prove it. 

Do not soften the criticism.
```

**14 - Tell Claude how you will use the output**

A summary for a CEO presentation is different from a summary for your own notes. A tweet draft that will be posted as-is is different from one you will heavily edit. Claude formats and calibrates differently when it knows the destination. This is one of the simplest rules and one of the most consistently ignored. One sentence about how the output will be used changes the entire structure of what Claude delivers.

```text
This will be read aloud in a 5-minute presentation.

This is for my own reference, not for anyone else to read.

This will be sent as-is to a client, no edits after.

This is a first draft I will rewrite heavily.
```

**15 - Ask Claude to rate its own answer**

After Claude gives you something, ask it to rate the response and explain what it would do differently if it had more information or more time. Claude will often identify the weakest parts of its own output, and that tells you exactly where to push for a better version. This also works as a quality filter. If Claude rates its answer a 6 out of 10 and explains why, you know the direction to take the next prompt.

```text
On a scale of 1-10, how good is the response you just gave me?

What are the weakest parts?

What would you do differently if you had more context or more time?
```

## Iteration

**16 - Never say "make it better"**

"Make it better" is the most common feedback given to Claude and the least useful. Better how? Better for whom? Better in what direction? Claude will make changes, but they will be guesses about what you meant.

When you want a revision, say exactly what to change and why. The more specific the feedback, the closer the next version gets to what you actually want.

```text
The [specific part] is too [problem]. 

Make it [what you want instead]. 

Keep everything else exactly as it is.
```

**17 - Ask Claude what it needs**

Before starting a complex task, ask Claude what information would help it do this better. Claude will tell you exactly what context it is missing. You give it that context. The output improves significantly.

This works especially well for long documents, detailed analysis, and anything where the quality of the output depends heavily on details you might not think to include.

```text
I want you to help me with [task].

Before you start, tell me: what information would help you do this significantly better? 

What am I probably not telling you that you need to know?
```

**18 - Chain tasks instead of combining them**

When you give Claude three tasks in one prompt, it completes all three at average quality. When you give it one task, review the output, then give the next task using that output as input, each step gets full attention.

Chaining is slower in terms of messages but faster in terms of getting to a result you can actually use. Complex work benefits from sequential steps, not parallel ones.

```text
Step 1: Summarize this document in 5 bullet points.

[after Claude responds]

Step 2: Based on that summary, write 3 key questions a skeptical reader would ask.

[after Claude responds]

Step 3: Answer each of those questions in one paragraph each.
```

## Advanced

**19 - Use "explain this as if I am..."**

This is one of the most underused prompting patterns and one of the most powerful for learning. Asking Claude to explain something as if you are a specific person, in a specific situation, with a specific level of knowledge forces Claude to translate complex ideas into the exact form you can actually absorb.

It also works in reverse for creative tasks: write this as if you are a journalist, a lawyer, a 10-year-old, a skeptic. The persona unlocks a completely different register.

```text
Explain [concept] as if I am a [profession] who has never heard of this.

Explain [concept] using only analogies from [familiar field].

Explain this so clearly that a smart 12-year-old could explain it to someone else.
```

**20 - End every important prompt with "what did I miss?"**

After Claude completes any significant task, ask it what you did not think to ask. What assumptions did you make that might be wrong? What questions did you not ask that you should have? What would a smarter version of you have included? Claude will find things. It almost always does. This single habit catches more errors and blind spots than any other prompting technique, because it specifically asks Claude to look for what you could not see yourself.

```text
What did I miss?

What should I have asked that I did not?

What assumptions am I making that might be wrong?

What would change your answer if I told you something different?
```

These are not hacks. They are the fundamentals of how Claude actually works. Claude responds to context. It responds to roles, constraints, purpose, and framing. Every one of these rules gives Claude more of what it needs to produce something useful instead of something average. Most people apply none of them. They type a sentence, get a mediocre answer, and decide Claude is not that impressive.

Pick three of these rules. Apply them in your next session. That is enough to see a difference. The other seventeen are here when you need them.