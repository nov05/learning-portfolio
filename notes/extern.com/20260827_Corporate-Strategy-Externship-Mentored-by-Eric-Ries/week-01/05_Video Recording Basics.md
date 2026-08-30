# Video Recording Basics

*Estimated Reading Time: 10 minutes*

## Why This, Why Now?

You've done the analysis. You've written the script. Now you need to make the thing.

This module gives you the minimum viable production setup — professional enough to be taken seriously, not polished enough to take forever.

## The Principle: Good Enough Is Good Enough

Your analysis is the product. Production quality just needs to not distract from it.

| Priority    | Focus                                 |
| ----------- | ------------------------------------- |
| **Highest** | Clear audio                           |
| **High**    | Readable text                         |
| **Medium**  | Stable video                          |
| **Low**     | Perfect lighting, expensive equipment |

> **Reality check:** Some of the most-viewed TikToks and Reels are filmed on phones with no special equipment. Your analysis matters more than your production value.

## Option A: Video Format

### Equipment Needed

| Item       | Minimum    | Ideal            |
| ---------- | ---------- | ---------------- |
| **Camera** | Your phone | Your phone       |
| **Audio**  | Phone mic  | Earbuds with mic |

### Recording Setup

1. **Find a quiet space** — This matters most.
2. **Face a light source** — Window during the day, lamp at night.
3. **Stabilize your phone** — Don't handheld record.
4. **Use the back camera** — Better quality than the front camera.
5. **Test audio first** — Record 10 seconds and verify clarity.

### Recording Tips

| Do                                          | Don't                               |
| ------------------------------------------- | ----------------------------------- |
| Speak slightly slower than normal           | Rush through your script            |
| Look at the camera lens                     | Watch yourself while recording      |
| Leave 2 seconds of silence at start and end | Start talking immediately           |
| Record in one take if possible              | Do 50 takes seeking perfection      |
| Re-record if you fumble a key line          | Re-record because you blinked weird |

## Option B: Carousel Slides

### Tools Needed

| Tool             | Free Options           |
| ---------------- | ---------------------- |
| **Design**       | Google Slides or Canva |
| **Stock images** | Unsplash               |

### Design Guidelines

| Element            | Guideline               |
| ------------------ | ----------------------- |
| **Slides**         | 3–4 slides              |
| **Text per slide** | 1–2 sentences max       |
| **Font size**      | 24pt minimum equivalent |
| **Colors**         | High contrast           |
| **Images**         | Optional                |

### Slide-by-Slide Structure

| Slide | Purpose          |
| ----- | ---------------- |
| **1** | Hook             |
| **2** | Mission          |
| **3** | Corruption Event |
| **4** | Diagnosis        |
| **5** | Evidence         |
| **6** | Lesson           |

### Carousel Tips

| Do                              | Don't                                    |
| ------------------------------- | ---------------------------------------- |
| Make each slide standalone      | Require previous slides to understand it |
| Use consistent fonts and colors | Change the design every slide            |
| Put key words at the start      | Bury the point in long sentences         |
| Include your name on slide 6    | Forget to take credit                    |

## Choosing Your Format

| Choose Video If                   | Choose Carousel If                   |
| --------------------------------- | ------------------------------------ |
| Your company has dramatic visuals | Your analysis is data-heavy          |
| You're comfortable on camera      | You prefer writing to speaking       |
| Your hook is punchy when spoken   | Your evidence is a quote or stat     |
| You want practice with video      | You want practice with visual design |

Both formats are equally valid.

## Time Management

| Phase        |     Video |  Carousel |
| ------------ | --------: | --------: |
| **Setup**    |    10 min |     5 min |
| **Creation** | 15–30 min | 20–40 min |
| **Editing**  | 10–20 min |  5–10 min |
| **Total**    | 35–60 min | 30–55 min |

> **Pro tip:** Set a timer. Your first evaluation doesn't need to be perfect. It needs to ship.

## TL;DR

* Production quality matters less than you think.
* Audio clarity is the one non-negotiable for video.
* Basic editing tools are free and sufficient.
* Carousels need readable text and consistent design.
* Both formats can effectively deliver your analysis.

**Next up:** Time to ship. Share your finished diagnostic.

<br><br><br>

👉 Prompt for NotebookLM short video generation  

```text
Create a short-form video of approximately 60–70 seconds.
Use the provided source script as the complete voiceover. Read it essentially verbatim, word for word.
Do NOT rewrite, summarize, paraphrase, expand, reorder, or add any narration. Do not introduce additional facts, examples, background, or commentary.
The source script is already complete and follows the required four-beat structure:
Hook → Mission → Corruption Event → Diagnosis → Structural Lesson.
Your job is to visualize and present the existing script, not to create a new explanation.
Use concise, relevant visuals that directly illustrate what is being said: Airbnb listings, hotel comparisons, price breakdowns, service fees, cleaning fees, checkout chores, and simple business-model visuals.
Keep the pacing tight and professional. Prioritize clear narration and readable on-screen text.
Do not expand into Airbnb's broader history, housing regulation, local communities, or COVID.
Target approximately 60–70 seconds. Do not add content to increase runtime.
```

⚠️ 勉强能用，但是内容会被压缩改变。时长无法控制，explainer 5-8 分钟，short 1 分钟。   

<br><br><br>

👉 Prompt for NotebookLM explainer video generation

⚠️ 时长无法控制，explainer 5-8 分钟，short 1 分钟。 

```text
Create a 60-second video about Airbnb's business model deterioration.

The central thesis is: Airbnb gradually lost its value advantage over traditional hotels while remaining able to increase revenue. Rising prices, service fees, cleaning fees, and guest checkout chores can make an Airbnb stay comparable to or more expensive than a hotel while requiring more effort and offering less standardized service.

Structure the video around four beats:

1. Mission: Airbnb originally offered travelers a cheaper, more flexible, and more diverse alternative to traditional hotels.

2. Corruption Event: As the platform matured, rising prices and fees combined with increasingly demanding checkout chores. Guests could end up paying substantial cleaning fees while still being asked to perform cleaning-related tasks.

3. Diagnosis: The primary failure was Mission Drive. Airbnb could make more money from transactions even when the customer's price-to-value proposition became weaker relative to hotels.

4. Structural Lesson: A business becomes vulnerable when its revenue can keep growing even as the customer value that originally made the business attractive declines.

Make the opening hook provocative and concise:
"Airbnb didn't lose its edge because hotels got better. It lost its edge because the Airbnb experience became harder to justify."

Keep the tone analytical and sharp, not sensational. Focus on the economics of price, value, fees, and customer effort. Do not make the video primarily about housing regulation, local communities, COVID, or Airbnb's overall history. The goal is to explain one structural business problem clearly within 60 seconds.
```

A byproduct of using NotebookLM  

https://youtu.be/Sj4iIKYaUyo    
Title: **Explain Corporate Failure in 90 Seconds**   

```text
How do you explain a complex corporate failure without telling the entire story?

This video introduces a four-beat framework for turning corporate accountability research into a clear, compelling story:

• Mission — What was the company supposed to be?
• Corruption Event — What specific moment revealed what went wrong?
• Diagnosis — Which structural weakness was the root cause?
• Lesson — What broader lesson can be applied beyond the company?

The framework is designed to move beyond simply listing scandals, bad decisions, or individual failures. The goal is to identify the pivotal event, connect it to a root cause, and extract a structural lesson.

The method can be applied to corporate failures, governance problems, financial scandals, and other cases of institutional or organizational breakdown.

Source: Incorruptible: Why Good Companies Go Bad... and How Great Companies Stay Great 
by Eric Ries (Author), Publication date: May 26, 2026
https://www.simonandschuster.com/books/Incorruptible/Eric-Ries/9798893311860

This video was generated with Google NotebookLM based on the source materials listed above.

#CorporateFailure #BusinessAnalysis 
```

Quizzes generated by ChatGPT 

```text
What is the first beat in the framework for explaining corporate failure?
A. Financial performance
B. The Mission
C. The Corruption Event
D. The Lesson
Answer: B. The Mission

What should a “corruption event” identify?
A. The company's founding date
B. The company's biggest competitor
C. A specific moment that revealed what went wrong
D. The company's final financial results
Answer: C. A specific moment that revealed what went wrong

What is the purpose of the “Diagnosis” beat?
A. To summarize the company's entire history
B. To identify the root cause of the failure
C. To describe the CEO's personality
D. To list every problem the company faced
Answer: B. To identify the root cause of the failure

Why should a corporate failure story focus on one primary lens?
A. To make the story longer
B. To avoid using evidence
C. To demonstrate a clear diagnosis
D. To make the company look worse
Answer: C. To demonstrate a clear diagnosis

What is the purpose of the final “Structural Lesson”?
A. To explain what happened to the stock price
B. To assign blame to an individual
C. To extract a lesson that applies beyond the company
D. To summarize the company's founding story
Answer: C. To extract a lesson that applies beyond the company
```