## **Question 1**

👉 Human prompt for ChatGPT:  

```text
I'm currently reading the following book with NotebookLM.  
Please generate a prompt to help me understand the mental models in Chapter 10.
The book (616 pages): https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/

Prompt example:  

I am now studying Chapter 9: Faults and Partial Failures.
Assume I am an experienced software/backend engineer, but I have not systematically studied distributed systems.
Teach me this chapter as a mental model, not as a chapter summary.

Answer:

1. What is the fundamental problem this chapter is trying to solve?
2. Why is this problem fundamentally different from failures in a single-machine system?
3. What assumptions do backend engineers commonly make that become invalid in a distributed system?
4. What types of failures does the chapter discuss?
5. For each failure type:
   - What can go wrong?
   - Why is it difficult to detect?
   - What can the system safely assume?
   - What can it NOT assume?
6. What does the chapter teach about unreliable networks?
7. What does the chapter teach about clocks and time?
8. What does the chapter teach about detecting whether another node is alive?
9. What engineering techniques are used to deal with these problems?
10. What trade-offs do those techniques introduce?

Then build one coherent mental model connecting:
network failures → partial failures → unreliable failure detection → unreliable clocks/time → distributed coordination → system design decisions.

Finally, give me 5 rules of thumb that I should remember when designing distributed systems.
Base the answer on the actual chapter, including important examples and arguments from the book. Use source citations where appropriate.
```

## **Question 2**

👉 Human prompt for ChatGPT: 

```text
I'm currently reading the following book with NotebookLM.  
Please generate a prompt for NotebookLM to identify the most important engineering trade-offs in Chapter 10.
The book (616 pages): https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/

Prompt example: 

Analyze Chapter 9 and identify the 5 most important engineering trade-offs.
For each trade-off use this structure:

Problem
→ Why the naive solution fails
→ Possible approaches
→ What each approach optimizes for
→ What each approach sacrifices
→ Failure modes
→ When to use it
→ When NOT to use it

Focus especially on:

- timeouts and failure detection
- retries and duplicate requests
- network reliability
- clocks and timestamps
- detecting whether a node is alive
- safety vs availability

Use concrete examples from the chapter and provide source citations.
Do not merely summarize the chapter. Explain the engineering decisions behind these trade-offs.
```

## **Question 3**

```text
I'm currently reading the following book with NotebookLM.  
Please generate a prompt for NotebookLM to generate 3 inverview questions to test my understanding of Chapter 10.
The book (616 pages): https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/

Prompt example: 

Act as my distributed-systems interviewer.
I have just studied Chapter 9: Faults and Partial Failures.
Test whether I actually understand the chapter.

Rules:

- Ask me ONE question at a time.
- Do not reveal the answer before I respond.
- Do not give me hints unless I explicitly ask.
- Prefer reasoning over memorization.
- Ask follow-up questions when my answer is incomplete.
- After each answer:
  1. Evaluate my reasoning.
  2. Identify what I got right.
  3. Identify what is missing or incorrect.
  4. Explain the correct mental model.
  5. Give me a stronger version of the answer that an experienced backend engineer should be able to give.
- Then ask the next question.

Across the session, test:

- partial failures
- unreliable networks
- timeouts
- retries
- failure detection
- clocks and time
- safety vs availability
- distributed-system assumptions
- real-world backend scenarios

Include at least 3 system-design scenarios and several "why" questions.
Start with the first question now.
```