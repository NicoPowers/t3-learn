# t3-learn plan

- **Purpose:** Record everything decided so far about t3-learn, a local web app for learning any subject with AI tutors run by T3 Code, so it can be built in stages.
- **For:** The project owner, and the coding agents that build the app.
- **Version:** Draft 8, 2026-10-18. Status: planning. Nothing is built yet.
- **Scope:** t3-learn is a private tutor you talk with. It is for learning, not for writing code or building things.
- **Working name:** t3-learn. Rename it any time.

## Contents

Sections are numbered so later discussions can cite them, for example "see 9.3".

- [1. Summary](#1-summary)
- [2. Where this comes from](#2-where-this-comes-from)
- [3. Requirements](#3-requirements)
- [4. System architecture](#4-system-architecture)
- [5. How t3-learn uses T3 Code](#5-how-t3-learn-uses-t3-code)
- [6. Pipeline roles and settings](#6-pipeline-roles-and-settings)
- [7. Teaching method](#7-teaching-method)
- [8. Your books come first](#8-your-books-come-first)
- [9. Side tangents](#9-side-tangents)
- [10. Tools the tutor can use](#10-tools-the-tutor-can-use)
- [11. Plain language](#11-plain-language)
- [12. Spend and plan limits](#12-spend-and-plan-limits)
- [13. Dashboard layout](#13-dashboard-layout)
- [14. Data model](#14-data-model)
- [15. Build stages](#15-build-stages)
- [16. Decisions, open questions and risks](#16-decisions-open-questions-and-risks)
- [17. Learning research](#17-learning-research)
- [18. References](#18-references)

## 1. Summary

t3-learn is a local web app for learning any subject through conversation with a private AI tutor. The first topics will be technical, such as RF theory, linear algebra and robotics. The app builds nothing specific to those subjects. The tutor uses the tools its provider already has, such as scripts and your MCP servers, and the app shows the results (section 10).

Everything happens in one browser dashboard: chat, quizzes, math, diagrams, book passages, progress and cost. T3 Code runs the AI tutor in the background, and it is the only other program that needs to be open.

It starts from [amosblomqvist/learn](https://github.com/amosblomqvist/learn), a teaching setup for the pi coding agent that shows lessons in Obsidian. t3-learn keeps its teaching method and adds:

- a choice of provider, model, reasoning level and skill for each step of the pipeline
- your own books as the tutor's main source, quoted word for word
- a live circuit simulator for electrical topics, and a way to ask about anything you select
- side tangents, kept apart from the main lesson
- four progress meters per concept
- control of challenge level and emotion
- more Socratic follow-up questions
- plain-language output
- cost and plan-limit tracking per concept

**Next step:** build stage 1 in [section 15](#15-build-stages). It is a small test that proves the dashboard can talk to T3 Code.

## 2. Where this comes from

The upstream repository was read at commit `7cfd894` (2026-08-26). A copy is in `reference/learn`. Git does not track it, because the upstream repository has no license file. To restore it, run `git clone https://github.com/amosblomqvist/learn.git reference/learn` from this folder.

This table shows what happens to each part.

| Upstream part | What it does | In t3-learn |
| :--- | :--- | :--- |
| `skills/teach` | Teaching philosophy and the probe, plan, teach loop | Keep. Remove the pi and Obsidian parts. Extend it as described in section 7. |
| `skills/visualize` | Briefs a diagram maker to draw one correct, minimal picture | Keep. Diagrams render in the dashboard. |
| `agents/researcher` | Checks facts on the web. Runs on GLM 5.3 through OpenRouter. | Becomes the researcher role in section 6. |
| `agents/mermaid-maker`, `agents/svg-maker` | Draw a diagram, render it, look at it, fix it. Run on Sonnet 5. | Become the visual maker role. |
| `extensions/quiz.ts` | Graded multiple-choice question with "I don't know" and a note field | Becomes an MCP tool shown in the dashboard. |
| `extensions/ask-user-question.ts` | Asks a question that has no right answer | Becomes an MCP tool shown in the dashboard. |
| `extensions/md-log.ts` | Copies the session into an Obsidian note | Replaced by the app's database and pages. |
| `extensions/visual-tools` | Render tools the diagram makers use to check their work | Keep the render-and-look check. |

MCP (Model Context Protocol) is the plugin format that Claude, Codex and the other T3 providers all support. That is why the pi extensions become MCP tools.

Keep the upstream quiz rules:

- Ask one question per call.
- Require a correct answer and an explanation.
- Add "I don't know" automatically.
- Shuffle the options.
- Write wrong options this way: write the correct claim first, then change it into each wrong claim. Put no reasoning inside any option, and never bold one option only.

Upstream stores no problems and no solutions:

- **Writing quizzes:** the tutor writes every quiz on the spot, with the correct answer and the explanation taken from its own knowledge.
- **Grading:** the quiz tool grades by matching your choice against the answer the tutor supplied.
- **Checking facts:** the teach skill tells the tutor to check any fact it is "even slightly unsure" of with the researcher, and to survey the topic with the researcher before planning.

So upstream accuracy depends on the tutor noticing its own doubt. t3-learn checks every stored solution before use instead, as section 8.6 describes.

### 2.1. Changes to the upstream skills

The skills live in this repository's `skills/` folder and are copied into each provider's skills folder. This table lists the edits, by section of the upstream `teach` skill unless the row names another skill.

| Upstream section | Change | Plan section |
| :--- | :--- | :--- |
| Principle ii, "How could I have discovered this?" | Add the naive-approach move. For a step that adds complexity, try the simplest approach, show it break, then bring in the method as the fix. Build on the learner's own attempt. | 7.9 |
| Socratic vs expository | Replace with the problem-first or example-first rule. Example-first steps can use a worked solution that contains a mistake to find. | 7.6, 7.9 |
| Phase 2, Plan | The planner records the naive approach, breaking case and fix for each such step. The map shows why each step exists. | 7.9 |
| Phase 3, Teach loop | The "Motivate" step uses the breaking case for those steps. Add explain-back prompts, "what made you pick that?", confidence taps, and the split between learned today and mastered. | 7.2, 7.4 |
| Accuracy | Exercises use checked answer keys. The researcher still checks facts. | 8.6 |
| The philosophy: understanding over memorizing | Keep it for ideas that can be derived. Add the kinds of knowledge, so arbitrary facts get memory aids and spaced recall instead. | 7.11 |
| Writing quiz options | Keep as is. | 2 |
| Formatting | Replace Obsidian with the dashboard. Keep LaTeX. | 4 |
| Whole skill | Upstream is written for its author ("teach him"). Rewrite it for the learner using t3-learn, as the upstream README suggests. | 3 |
| `visualize` skill | Replace the Obsidian embed with a dashboard tool call. | 4 |

A prompt alone can drift, so the parts that must always happen are also enforced in code:

- **Plan check:** the `save_plan` tool rejects a step marked "adds complexity" unless it has a naive approach, a breaking case and a fix.
- **Hint ladder:** runs in the Learn server, not in the prompt (section 7.10).
- **Map labels:** the dashboard shows "exists because …" from the stored fields, not from the tutor's text.

New skills to write: exercise writer, solution checker, resource scout (`curate`), circuit maker (`circuit`) and the grader's rubric.

## 3. Requirements

### 3.1. Must have

1. All interaction happens in one browser dashboard served on your computer. T3 Code runs in the background. No Obsidian and no terminal interface.
2. The dashboard renders LaTeX, Mermaid diagrams and images.
3. Settings let you pick the provider, model, reasoning level and skills for each pipeline role, from T3's live model list.
4. The app uses the providers T3 already has set up. Today these are all subscriptions, not API keys.
5. You create topics. Each topic has a library of your references, mostly books converted from PDF to Markdown. The tutor uses them first and quotes them word for word.
6. Side tangents are stored apart from the main lesson, and they are easy to find and return from.
7. The tutor manages challenge and emotion. You should feel curiosity, productive struggle and satisfaction. Too much failure is discouraging, and too little challenge is boring.
8. The app tracks four sides of learning for each concept: awareness, understanding, fluency and application.
9. The tutor asks why you chose an answer instead of guessing why.
10. The tutor points you to existing high-quality material instead of summarising it.
11. Text shown in the app follows the plain-language rules in the iso-24495 skills.
12. The app tracks cost and plan-limit use per concept.
13. The app works for any subject. It builds no subject-specific tools, sandboxes or widgets. The tutor uses its provider's own tools, and results appear as text, diagrams and images. The one exception is the embedded circuit simulator (section 10.3).

### 3.2. Nice to have later

- A desktop wrapper, for example Tauri, so the app gets its own window and icon.
- Access from your phone.

## 4. System architecture

### 4.1. What each component is responsible for

| Component | Responsibility |
| :--- | :--- |
| Dashboard | Runs in the browser. Shows chat, quizzes, math, diagrams, passages, the map, tangents, settings and spend. Sends your input to the Learn server. |
| Learn server | One local Node.js process. Serves the dashboard and stores everything in SQLite. Offers MCP tools to the tutor, talks to T3 as an outside agent, and reads provider logs for cost. |
| T3 Code | Runs the tutor and helper threads on your providers. |
| Tutor thread | One T3 thread per topic, plus one per explored tangent. Teaches, calls the Learn server's tools and starts helpers. |
| Helper threads | Short tasks the tutor starts with `delegate_task`: research, planning, review, finding resources, diagrams, grading, notes and book conversion. |

There are two MCP connections, one in each direction:

- **Learn server to T3:** the Learn server connects to T3's own MCP endpoint as an "outside agent". It starts threads, sends messages, reads replies, forks threads and reads the model list.
- **Tutor to Learn server:** the tutor, which is Claude or Codex running inside T3, connects to the Learn server's MCP endpoint. It calls tools such as `quiz` and `show_passage`.

### 4.2. How one message flows through the system

1. You type a message or answer a quiz in the dashboard.
2. The Learn server sends it to the topic's tutor thread with `t3_thread_send`. A quiz answer includes your choice, your confidence and your note.
3. The tutor runs a normal turn. When it calls a tool such as `quiz` or `show_passage`, the Learn server shows the result in the dashboard straight away. Every tool returns at once, and no tool waits for you.
4. The Learn server reads the tutor's reply with `t3_thread_read` and shows it.
5. Your next input starts the next turn.

This design avoids tool calls that wait minutes for an answer. Providers may cut such calls off. T3's own documentation notes a 60-second default request timeout in pi's built-in MCP support.

### 4.3. Where data is stored

- **Source code:** this folder.
- **Database and library files:** a local data folder outside OneDrive, for example `%LOCALAPPDATA%\t3-learn`. Sync tools and open SQLite files don't mix well.

## 5. How t3-learn uses T3 Code

These findings come from the T3 Code source, [pingdotgg/t3code](https://github.com/pingdotgg/t3code), read at commit `9c161462` (2026-10-18).

### 5.1. The Learn server connects as an outside agent

T3 lets apps it did not start connect to its MCP endpoint. The steps are in `docs/user/outside-agents.md` in the T3 repository.

1. The Learn server connects to `http://127.0.0.1:<port>/mcp` and starts an OAuth sign-in. OAuth is the standard sign-in method that T3 uses for outside apps.
2. You approve it with a pairing code from **Settings → Connections** in T3.
3. You choose an access level. Read only is the default. Starting and messaging threads needs Supervised or Full access.
4. The sign-in lasts 30 days. Then you approve it again.

### 5.2. Tools the Learn server can call

These access rules come from `apps/server/src/mcp/toolkits/*/handlers.ts`.

| Tool | What t3-learn uses it for |
| :--- | :--- |
| `orchestrator_capabilities` | Fill the model menus in Settings |
| `t3_project_create`, `t3_project_list` | Register the workspace T3 runs the tutors in |
| `t3_thread_launch` | Start a tutor thread with a chosen model |
| `t3_thread_configure` | Change a thread's model |
| `t3_thread_send` | Send your message or quiz answer |
| `t3_thread_read`, `t3_thread_wait` | Read the tutor's replies |
| `t3_thread_interrupt` | Power the stop button |
| `t3_thread_fork` | Open a tangent, as described in section 9 |
| `t3_thread_merge_back` | Bring a tangent's full detail back, only when asked |
| `schedule_task` | Possible later use: a daily review reminder |

### 5.3. Tools only the tutor can call

`delegate_task`, `task_status`, `task_cancel`, `create_threads`, `html_render` and the preview tools only work for an agent running inside a T3 thread. So the tutor starts helpers, not the Learn server.

### 5.4. Usage data stays inside T3

T3 already prices token usage by day or hour, provider and model. That data is defined in `packages/contracts/src/usage.ts`. T3 also tracks subscription limit windows, such as session and weekly limits, in `packages/contracts/src/providerUsageLimits.ts`. Both are available only through T3's internal WebSocket API (`server.getUsageSummary`), not through MCP. The data is also not split per thread. Section 12 explains the workaround.

### 5.5. Fork and merge-back

`docs/orchestration-v2/thread-lineage-and-context-transfer.md` in the T3 repository describes forking a thread "to explore a tangent without polluting the source thread's context". Forks and merge-back are the basis for side tangents in section 9.

### 5.6. Giving the tutor the Learn server's tools

Register the Learn server's MCP URL once in each provider's settings:

```sh
claude mcp add --transport http t3-learn http://127.0.0.1:<learn-port>/mcp
codex mcp add t3-learn --url http://127.0.0.1:<learn-port>/mcp
```

Claude threads started by T3 already load your MCP servers: LTspice and Blender appeared in the planning session. Codex has not been checked yet.

### 5.7. Not tested yet

- Whether replies can stream word by word. If they can't, the Learn server checks the thread about every half second, so text appears in chunks.
- How long the delay is between your message and the start of the reply.
- Whether Codex threads load the Learn server's MCP tools.

## 6. Pipeline roles and settings

### 6.1. Roles

| Role | Runs when | How it runs | Suggested model tier | Skills |
| :--- | :--- | :--- | :--- | :--- |
| Tutor | Every turn | The topic's T3 thread | Strong, medium reasoning | teach, iso-24495-1 |
| Tangent tutor | In an explored tangent | A forked T3 thread | Same as tutor, or cheaper | teach, iso-24495-1 |
| Planner | Once per topic | Helper; the tutor waits for it | Strongest, high reasoning | teach (plan phase) |
| Plan critic | After each plan | Helper on a different provider | Strong, high reasoning | review rubric |
| Researcher | When a fact is uncertain | Helper, in the background | Cheap, with web access | research |
| Resource scout | Per concept, ahead of time | Helper, in the background | Mid, with web access | curate (new) |
| Librarian | When you add a book | Helper, in the background | Cheap | pdf-markdown |
| Visual maker | When a picture helps | Helper, in the background | Mid | visualize |
| Exercise writer | When a step needs exercises or new variants | Helper, in the background | Strong, medium reasoning | None yet |
| Solution checker | After each generated solution | Helper on a different provider | Strong, high reasoning | None yet |
| Grader | Free-text answers | Helper or the tutor | Mid | rubric |
| Note writer | End of session | Helper, in the background | Cheap | iso-24495-3 |
| Circuit maker | When a circuit helps | Helper, in the background | Mid | circuit (new) |

Each helper takes a few seconds to start. Live chat therefore stays with the tutor, and helpers do slow work in parallel.

### 6.2. What you set for each role

- provider and model, chosen from T3's live list
- reasoning level
- skills to load
- extra instructions
- on or off

### 6.3. How the tutor applies the settings

1. The tutor calls the Learn server tool `get_role_config` with the role name.
2. It passes the provider, model and reasoning level to `delegate_task`.
3. The task prompt names the skills to use. It starts with a tag such as `[t3-learn topic=… step=… role=…]` so cost can be matched to a concept.

A skill must be installed for every provider that uses it. The Learn server can copy its skills into each provider's skills folder.

## 7. Teaching method

### 7.1. Keep the upstream principles

- **Unconditional truths first:** start from facts you can accept at face value, then build on them.
- **"How could I have discovered this?":** motivate every step so nothing feels arbitrary.
- **Probe, plan, teach:** find the edge of what you know, plan a concept map, then teach one step at a time with a quiz check after each step.
- **Accuracy:** check any uncertain fact with the researcher before stating it.
- **Approval:** show the plan and wait for your go-ahead before teaching.

### 7.2. Four meters per concept

| Meter | Question it answers | Evidence |
| :--- | :--- | :--- |
| Awareness | Do you know it exists and when it applies? | Spotting which situations need it |
| Understanding | Can you explain why it works? | Explaining back, predicting, deriving |
| Fluency | Are you fast and right, weeks later? | Timed drills and spaced reviews |
| Application | Can you use it on something new? | New problems and new situations |

The tutor works on your weakest meter that you are ready for. Fluency drills come after understanding, and projects come after fluency. The upstream method mainly builds understanding. It has no spaced review, no speed practice and no transfer tasks.

Research update: a step has two levels of done.

- **Learned today:** you passed the quiz right after the step.
- **Mastered:** you solved a new variant of the problem with no hints, in a later session. If you can't, the tutor reteaches the part that failed.

Passing right after teaching overstates what stuck. A 2026 trial with more than 6,000 students found that "three correct in a row" in one session did not, by itself, improve a test a week later.

Test the application meter directly with problems in new contexts. Being fluent does not mean you can transfer the skill.

### 7.3. Challenge and emotion

The Learn server watches these signals:

- your recent success rate
- how often you choose "I don't know"
- how long you take to answer
- your confidence taps (section 7.4)
- a mood button with four choices: confused, stuck, bored or fine

With each message, the Learn server gives the tutor an instruction. For example: "4 misses in a row: give an easy win next, then retry with a hint."

- **Target:** 80% to 90% correct on the first try with no hints, tracked for each meter. Research update: the 85% figure from Wilson et al. 2019 was derived for a machine-learning model, not for people, and other assumptions move it to 82% or 75%. A band fits the evidence better than one number.
- **Session shape:** open with a puzzle that sparks curiosity, struggle in the middle, and end on a win.
- **Probe questions:** label them "mapping, not grading" and leave them out of the success rate. The upstream probe raises difficulty on purpose until you fail. Problem-first attempts (section 7.6) are left out too.
- **Simple rules, not an emotion model:** research update: after a few minutes of unresolved struggle, give a hint. When you seem bored, raise the difficulty or switch to an application problem. Boredom is the most harmful state. Confusion helps when it gets resolved, and frustration does less harm than expected. Models of how emotions follow each other often fail to replicate, so the app doesn't build one.

### 7.4. Socratic follow-up

- **Confidence tap:** every answer gets a tap for sure, unsure or guess. Sure and wrong points to a wrong mental model. Unsure and right means the knowledge is still shaky.
- **Ask before explaining:** after a wrong answer, and sometimes after a right one, ask "what made you pick that?" before showing the explanation.
- **Free-text answers:** allowed and graded by a model. The upstream quiz is multiple choice only.
- **Hard rule in the teach skill:** the tutor never states why you answered something unless you told it.

Research update:

- **After a confident error:** explain where your reasoning went wrong, not just the right answer. Then check a new variant of the problem within a few days.
- **Calibration chart:** show how your confidence compares with your results. AI help tends to make learners feel they learned more than they did.
- **Explain-back prompts:** use one or two per step, on key steps only, always in free text, and always with feedback. Uncorrected wrong explanations reduce the benefit, and multiple-choice "why" prompts show no reliable benefit.
- **Grading free text:** always give the grader the reference solution and a rubric. Grade several times and keep the answer that comes up consistently. You can dispute any grade.

### 7.5. Spaced review

Use FSRS (Free Spaced Repetition Scheduler) to schedule reviews for the fluency meter. The `ts-fsrs` library is one TypeScript implementation.

Research update:

- **New variants:** review with new versions of a problem, not the same item. Remembering one answer is not the skill.
- **Full solutions:** a review asks for a full solution, not just the final answer.
- **Mixed review:** mix problem types from several steps, so you must first decide which method applies. Pair problems that look alike but need different methods, such as substitution and integration by parts.
- **Feedback:** always give feedback after a review.
- **Retention target:** FSRS's default target of 90% recall fits the 80% to 90% band in section 7.3.

### 7.6. Choose problem-first or example-first for each step

Research update: the tutor picks one of two ways to start each step.

| The step is | Start with |
| :--- | :--- |
| Conceptual, its prerequisites are mastered, and the material is not overwhelming | **Problem first.** Pose the motivating question and let you try it for 10 to 15 minutes. Then derive the result by comparing your attempt with the standard one. |
| A procedure, heavy material, or built on weak prerequisites | **Example first.** Show a fully worked example, preferably from your book. Then show a similar problem with the later steps blank. Then give a bare problem. |

This puts "how could I have discovered this?" into practice. Trying a problem first helps understanding and transfer, but not procedural skill. Worked examples help beginners and get in the way once you know the material, so the tutor uses your probe results to skip examples you don't need.

### 7.7. Check each step of a solution

This applies to exercises with steps, such as calculations, derivations and proofs.

Research update: you type your solution steps in Markdown or LaTeX. The grader compares each step with a stored reference solution and gives a hint at the first wrong step, not the fix. Tutors that check each step come close to human tutors in the research, while tutors that check only the final answer do much worse.

### 7.8. Guardrails for the tutor

Research update: these rules come from recent trials of AI tutors.

- **Hints, not answers:** students who could ask GPT-4 for answers scored 17% lower once the AI was taken away. A version that gave only hints removed that harm. Section 7.10 sets the exact rule.
- **Reference material in the prompt:** give the tutor the reference solution and common mistakes for each problem. GPT-4 alone solved those problems correctly only 51% of the time.
- **Structure in code:** the Learn server enforces the order of a multi-part problem. A prompt alone could not keep the parts in order in the Harvard physics tutor study.
- **Help after errors:** you try first. After a wrong answer, the tutor offers help without waiting for you to ask, because learners rarely ask.
- **Delayed tests:** judge progress by the delayed, hint-free mastery checks in section 7.2, not by how a session felt.

### 7.9. Start from the naive approach

Many methods only make sense once you see the simple approach fail. Upstream's "how could I have discovered this?" asks the tutor to motivate each step, but it doesn't require showing a simple approach failing. t3-learn makes that an explicit move.

For every step that adds complexity, the planner records three things:

- **Naive approach:** the simplest thing that could work.
- **Breaking case:** a concrete case where the naive approach fails.
- **Fix:** what the real method adds to handle that case.

The tutor then runs this sequence:

1. Ask "What's the simplest thing you'd try?" If you have an idea, the tutor uses yours. If not, it offers the naive approach.
2. Try the approach on an easy case, where it works.
3. Try it on the breaking case. Where possible, the tutor runs it so you see it fail, for example with a short script or an LTspice simulation, and shows the result as an image. Examples: forward Euler blowing up on a stiff system, or a voltage divider's output sagging under load.
4. Ask "What would you need to fix this?" before showing the method.
5. Introduce the method as the fix for that exact failure, and show it passing the breaking case.

The move works outside technical subjects too. For example, translating word for word works for simple sentences and breaks on idioms.

On the map, each step shows the failure it fixes, for example: "Exists because forward Euler blows up on stiff systems". That answers "why the extra complexity?" at a glance.

For example-first steps (section 7.6), an erroneous worked example plays the same role. It is a solution that uses the naive approach and goes wrong, and you find and fix the error.

Research update: the move is supported, with one condition.

- **Build on your attempt:** a review of problem solving before instruction found that it helps only when the teaching contrasts cases or builds on the learner's own solutions (Loibl, Roll and Rummel 2017). A later replication found no benefit when the attempts were too shallow, or when they didn't match the solutions used in the teaching. So the tutor builds on your actual attempt, not a generic naive one.
- **Erroneous examples:** students who found and fixed errors in worked solutions scored higher on a test a week later (d = 0.62, Adams, McLaren and others 2014). An earlier classroom study found no advantage, so the evidence is mixed.

### 7.10. Give hints in a fixed order before the answer

The Learn server, not the tutor, controls when a solution appears. The rule applies to practice problems only:

- **Your own questions in chat:** the tutor answers them directly.
- **Multiple-choice checks:** they show the answer and explanation right after you answer, as upstream does.

Each practice problem has a hint ladder:

1. **Hint 1:** names the idea or principle that applies.
2. **Hint 2:** points to where your solution goes wrong, or to the next step to take.
3. **Hint 3:** works the next step for you.
4. **Solution:** shows the full worked solution.

Rules for the ladder:

- **New attempt between hints:** each hint unlocks only after a new attempt, even a partial one. Students who click straight through hints to reach the answer, called "gaming the system", learn less (Baker, Corbett and Koedinger 2004).
- **Setting:** the number of hints before the solution is a setting, default 3. No study fixes the best number.
- **No penalty:** seeing the solution is not a failure. The problem becomes a worked example. The step doesn't count as mastered, and the app queues a new variant later in the session and again next session.
- **Hints from the reference solution:** hints come from the stored reference solution and its common mistakes (section 8.6), so they point at real errors.
- **Challenge signal:** the app records the hints used on each attempt. Many hints make the next problem easier (section 7.3).

### 7.11. Match the teaching to the kind of knowledge

Not everything you learn can be derived. Upstream values understanding over memorizing. That suits ideas you can build from first principles, but not arbitrary facts such as vocabulary, names or conventions. The planner tags each step with one kind, and the tutor picks its moves from this table.

| Kind of step | Examples | How the tutor teaches it | How mastery is checked |
| :--- | :--- | :--- | :--- |
| Concept you can derive | Why a transmission line reflects a signal; why eigenvectors matter | Unconditional truths, the naive approach (7.9), problem-first (7.6) | Explain why, or predict what happens in a new case |
| Convention or fact to remember | Vocabulary, symbols, units such as dBm, names, dates | Link it to something you know, add a memory aid, then spaced recall (7.5) | Recall after a delay |
| Procedure or skill | Row reduction, reading a Smith chart, a grammar pattern | Example-first with fading (7.6), step checking (7.7), mixed practice | A new variant, solved with no hints, in a later session |
| Judgment or interpretation | An antenna design trade-off, reading a historical source, judging an argument | Contrasting cases, several perspectives, examples of strong and weak answers | An open-ended answer graded against a rubric |

Most topics mix kinds. RF theory has derivable concepts, conventions such as decibel units, procedures and design judgment.

## 8. Your books come first

### 8.1. Source order

1. Your library for the topic
2. Saved resource cards (section 8.4)
3. The open web

The researcher also checks facts against your books first.

### 8.2. How a book enters the library

1. You drag a PDF or Markdown file onto a topic.
2. The librarian helper converts a PDF to Markdown with the `pdf-markdown` skill, which uses MinerU. It keeps page numbers and figures.
3. The Learn server splits the book into passages by section and page.
4. It indexes the passages with SQLite's FTS5 full-text search. No extra services are needed.

### 8.3. How the tutor quotes a book

1. The tutor calls `search_library` and gets passage IDs with short snippets.
2. It calls `show_passage` with the ID of the passage it wants.
3. The dashboard shows the stored text as a quote card. The card includes math, figures, the page number and a link that opens the PDF at that page.

The model only picks the passage, and the app shows the stored text. So the AI can't misquote your book.

### 8.4. Outside material

The resource scout finds clips, chapters and articles for each concept, for example from 3Blue1Brown, MIT OpenCourseWare, Khan Academy or a textbook section. It saves a resource card only after checking two things: the link works, and the clip at that time covers the point. Each card stores the link, the start and end time, and the concept and meter it serves.

The dashboard plays a clip from the right second. You can also paste in your own links.

Research update: assign short, specific pieces, such as a section number or a clip with start and end times. Follow each piece with a question that makes you recall it. Reading or watching on its own works about as well as rereading, which is a weak way to study.

### 8.5. Reading along

Tell the tutor "I'm on page 143", or highlight text in the PDF viewer and ask "explain this". The tutor pulls up that section and asks you questions about it.

### 8.6. Where answer keys come from

Every exercise needs an answer key and a list of common mistakes, because the grader and the hint ladder depend on them. The key depends on the exercise type:

- **Exact or checkable answer:** a number, an expression, a word or a choice.
- **Reference solution:** a worked solution with steps.
- **Rubric:** for open-ended questions, which have no single right answer, a rubric plus one or two example answers.

Answer keys come from two sources, in this order:

1. **Your book:** if the exercise comes from your book and the book gives the answer or a worked solution, use it and store the page number.
2. **Generated, then checked:** otherwise, the exercise writer creates the key, and the key is checked before use.

A generated key goes through these checks in order:

1. **Code check, where the answer can be checked by code:** the solution checker writes and runs a short script. For example, SymPy can substitute an answer back, differentiate an integral or check units, and LTspice can simulate a circuit answer. This check is free and reliable.
2. **Independent solve:** the solution checker, on a different provider, solves the problem without seeing the first solution. If the final answers agree, the solution is accepted.
3. **Disagreement:** if the answers differ, the researcher checks against your library and the web. If it can't settle the question, the solution is marked unverified.

Each solution shows its status in words: "from your book, page 143", "checked by code", "checked by two models" or "unverified".

- **Unverified solutions:** never used to grade you. The tutor uses those problems for discussion, not tests.
- **Book answers:** the code check runs on them too. If the code disagrees with the book, the app flags it, because answer keys contain errors.
- **Cost:** checks run once per exercise and are saved, so the cost stays low.
- **Rubrics:** the solution checker grades the example answers against the rubric. If its grades don't match the intended ones, the rubric is unclear and gets rewritten before use.

## 9. Side tangents

### 9.1. The problem

While learning one concept, you often get related questions that lead somewhere else. If they all stay in one chat, it fills up, and you have to scroll a long way to find where you are on the main path. Tangents need their own place. They should be easy to save, explore and return from.

### 9.2. Sort each side question first

Not every side question is a tangent. The tutor suggests where a question belongs, and you confirm it with one tap. You can also mark a question as a tangent yourself, with a button or by typing `/tangent`.

| The question is | It goes |
| :--- | :--- |
| Needed for your goal, and you are missing it | Into the main path, as a new step before the current one |
| Covered later in the plan | Pinned to that later step, with a note such as "we reach this at step 6" |
| Related, but not needed for your goal | Into a tangent |
| About a different subject | Into a suggested new topic |

### 9.3. What you can do with a tangent

| Action | What happens | Cost |
| :--- | :--- | :--- |
| Quick answer | Two or three sentences in a collapsed side note under the current step | Small, in the main thread |
| Park | Saved with the point where it came up. Nothing runs. | None |
| Explore | Opens a tangent tab. T3 forks the tutor thread, so the tangent knows what you have learned, but its messages never enter the main thread. | Its own thread, tracked separately |
| Return | The tangent writes a short summary. The main tutor gets it with your next message, recaps where you were and asks one warm-up question. | Small |
| Promote | Turns the tangent into its own topic with its own map, linked back to where it started | None |

Tangents can contain tangents, up to 2 levels deep. Past that, the app suggests promoting the tangent to a topic.

On return, the app sends only the short summary, not T3's `t3_thread_merge_back`. Merge-back adds everything that changed in the fork to the main thread, which would make the main thread long again. Merge-back stays available when you ask for the full detail.

### 9.4. How you find your way around

- **Breadcrumb:** the top of the page always shows where you are, for example "Laplace transforms › Step 4: Transform of a derivative › Tangent: Why does e show up everywhere?" Click any part to jump there.
- **Tabs:** one tab for the main path, plus one for each open tangent. At most 3 tangents stay open, and the rest are parked.
- **Map pane:** shows the main path with a "You are here" marker and the goal. Each tangent appears as a side branch from the step where it came up, labelled with its status in words: parked, exploring or done. Click a branch to open it.
- **Chat split by step:** finished steps collapse into one-line cards, such as "Step 3 done: 4 of 5 right, 2 tangents". Click a card to expand it.
- **Parking lot page:** lists every parked tangent by topic and step, with how long ago you parked it.

### 9.5. Tangents and motivation

Tangents are curiosity, and curiosity drives learning, so the app keeps them instead of discouraging them.

- **Parking:** a parked tangent is a promise to come back.
- **Reward:** after a win, or at the end of a session, the tutor offers a parked tangent. For example: "You parked 3 questions. Want to explore one for 10 minutes?"
- **Rabbit-hole check:** after 20 minutes off the main path (you can change this), the tutor asks: "Keep going, or return to step 4?" It never forces a return.

### 9.6. Tangent record

| Field | Meaning |
| :--- | :--- |
| `question` | The side question, in your words |
| `anchor` | The topic, step and message where it came up |
| `kind` | The sort from 9.2: prerequisite, later step, tangent or new topic |
| `status` | parked, exploring, done or promoted |
| `threadId` | The forked T3 thread, once you explore it |
| `parentTangentId` | The tangent it came from, for nested tangents |
| `summary` | What you learned, written on return |
| `newSteps` | Concepts the tangent added, each with its own meters |

Each explored tangent is its own thread, so its cost is easy to track.

## 10. Tools the tutor can use

t3-learn is a place to learn through conversation with a private tutor. It is not a place to write code or build things. So the app builds no tools for particular subjects, no sandboxes and no interactive widgets. The one exception is an existing circuit simulator that the app hosts (section 10.3).

### 10.1. The tutor uses what its provider already has

The tutor and its helpers already have their provider's own tools:

- **Scripts:** they can write and run short scripts, for example to plot a function or check a result with SymPy. The scripts are throwaway and run in a scratch folder inside the topic's data folder. You never manage them.
- **Web search:** for the researcher and the resource scout.
- **Files:** your library, and any project you point the tutor at.
- **Your MCP servers:** for example, your LTspice server can build a schematic and run a simulation.

### 10.2. The dashboard shows the results

The dashboard renders these in the conversation:

- Markdown text
- LaTeX math, with KaTeX
- Mermaid diagrams
- images from any tool, such as a plot from a script or a waveform from an LTspice simulation
- book passages (section 8.3) and quizzes

To show an image, the tutor calls the Learn server's `show_image` tool with a file path and a caption. The Learn server copies the file into the topic and shows it in the conversation. This is the same idea as upstream's diagram makers, which save a PNG for the lesson.

Research update: put the labels on the figure itself, and follow each figure with a question that links it to the idea, such as "which term flattens this curve?" Pictures without that kind of support add effort without adding learning. Interactive simulations had stronger evidence than static images, but the app uses static images to stay simple. The circuit simulator in section 10.3 is the one exception.

### 10.3. The one exception: a live circuit simulator

For electrical topics, the dashboard embeds CircuitJS1, Paul Falstad's circuit simulator that runs in the browser. It was read at commit `5a70716` (2026-09-23). It animates current flow, which builds intuition that a static image can't. The app hosts this existing tool rather than building one, the same way the tutor uses LTspice. LTspice stays the tool for accurate numbers and waveforms, because CircuitJS1's device models are simpler than SPICE's.

What CircuitJS1 offers:

- **Text circuits:** it loads a plain-text format with one line per part and its grid coordinates. For example, `r 240 48 240 208 0 10000` is a 10 kΩ resistor. This version exports XML but still loads text, so the app writes text.
- **JavaScript interface:** when it runs in an iframe from the same site, the page can load and export circuits, read node voltages, read each part's voltage and current, start and stop the simulation, and export the schematic as an SVG image. The interface is in `JSInterface.java` and demonstrated in `war/jsinterface.html`.
- **Example circuits:** 368 of them, in categories such as basics, filters, op-amps, transistors, power converters and transmission lines.
- **Embedding options in the URL:** hide the menu, sidebar and info box, turn off editing, use a white background, and choose US or European resistor symbols or conventional current.

How it fits into t3-learn:

- **Hosting:** the Learn server hosts a pinned build of CircuitJS1, so the dashboard embeds it from the same site, as the JavaScript interface requires.
- **Tools for the tutor:** `find_circuit_example` searches the examples by name and category. `show_circuit` shows an example or a netlist the AI wrote. `circuit_values` reads voltages and currents from the running simulation. `circuit_snapshot` saves the schematic as an image, for the lesson record and for checking.
- **Circuit maker:** a helper that writes netlists and checks the result, like upstream's diagram makers (section 6.1).
- **Teaching moves:** you predict what will happen, then watch it happen. The naive approach from section 7.9 becomes live, for example a voltage divider sagging under load. The tutor reads the real values to confirm what you saw.

Selecting parts to ask about them (section 13.1) needs a small patch, because today's interface can't read the selection:

- **What exists:** the simulator already tracks which parts are selected (`CircuitElm.isSelected()`), but it doesn't expose this to JavaScript.
- **The patch:** add `isSelected` and `setSelected` to the methods in `CircuitElm.addJSMethods`. With `setSelected`, the tutor can also highlight parts to point at them.
- **Part names:** CircuitJS1 parts have no names such as R1. The Learn server numbers them in the order they appear in the circuit text.
- **License:** CircuitJS1 is GPL-2.0. A patched copy is fine for personal use. If t3-learn is ever published, publish the patch too, or offer it to the CircuitJS1 project as a pull request.

### 10.4. Lay out new circuits with a deterministic engine

AI-written coordinates make messy schematics. In one study, only 76% of schematics drawn entirely by a language model would even load. So the AI never writes coordinates. It writes a netlist, and a layout engine in the Learn server places the parts. The full findings are in [the schematic layout research report](docs/research/schematic-layout.md).

When the tutor only changes values in an example circuit, the example keeps its hand-drawn layout. The engine runs only for new circuits or changed structure.

The engine works in these steps:

1. **The AI writes a netlist:** parts, values, which pin connects to which net, and each net's role: input, output, ground, supply or signal. It may add coarse hints, such as a stage number, "vertical" or "above U1". It never writes coordinates.
2. **Ground and supply become symbols:** ground connections become ground symbols, and supply connections become one rail plus named labels, instead of long wires. Only signal nets go into the layout.
3. **Common structures use templates:** for example, a voltage divider is a vertical column, and an op-amp's feedback parts go above it.
4. **elkjs places the rest:** its layered algorithm puts signal flow left to right, with right-angle wires and few crossings. Its settings are fixed, so the same netlist always gives the same layout.
5. **Snap and write:** every point snaps to CircuitJS1's 16-pixel grid. Wires are split at every junction, because CircuitJS1 connects points only where coordinates match exactly. The result is written as CircuitJS1 text.
6. **Check automatically:** the engine reads its own output back and confirms the connections match the netlist exactly. It also checks for overlapping parts and labels, wires crossing parts, rails above ground, inputs on the left and outputs on the right.
7. **Fall back if needed:** if a check fails, the engine retries with simpler settings. A circuit that fails the connection check is never shown.

- **Prior art:** Weave does the same for LTspice schematics, with elkjs, a read-back check and fallback layouts. It reports correct connections for all 117 circuits in one test set and 88.4% of 3,460 Analog Devices demo circuits (Gulgonul 2026). No converter for CircuitJS1 exists yet.
- **Effort:** the researcher estimates about 1 week for a prototype with only ground and rail symbols, and 3 to 4 weeks for a solid first version.
- **Testing:** turn the bundled example circuits into netlists, lay them out again, and review the results.
- **Licenses:** elkjs is EPL-2.0. The engine is our own code, and it must not copy CircuitJS1's GPL wire router.

## 11. Plain language

- **The tutor's skills:** the tutor loads `iso-24495-1`, plus `iso-24495-3` for technical explanations.
- **Rule check:** the tutor sends lesson text through a `say` tool. The Learn server checks the text for long sentences, filler such as "Great question", and heavy hedging. A failing draft goes back to the tutor for a rewrite in the same turn, with no extra model call.
- **Protected text:** the check never rewrites LaTeX, code, book quotes or quiz options. Upstream builds wrong options to mirror the right one closely, and rewording them would make the right one stand out.
- **Longer notes:** the note writer uses `iso-24495-3`.

A second model rewriting every message was rejected, because it would add seconds to each reply.

## 12. Spend and plan limits

Your providers are subscriptions, so the real limit is your plan's usage cap, not dollars. The spend page shows two numbers per concept.

| Number | Meaning | Where it comes from |
| :--- | :--- | :--- |
| Share of plan limits used | Your real cost on a subscription | T3's limit tracking, through its internal API. Optional, because that API may change between versions. |
| API-equivalent cost | What the same work would cost with API keys | The Learn server reads provider logs and prices the tokens |

The provider logs are in these folders:

- **Claude Code:** `~/.claude/projects/**/*.jsonl`. These files contain token counts for every message, including cache and thinking tokens. This was confirmed on this machine.
- **Codex:** `~/.codex/sessions/**/*.jsonl`. These files exist on this machine, but their token fields are not confirmed yet.

To match cost to a concept, each thread's first prompt carries the tag from section 6.3. Tangents and helpers are separate threads, so each one is a separate log session.

## 13. Dashboard layout

| Area | Contents |
| :--- | :--- |
| Left pane | Topics and the parking lot |
| Top bar | Breadcrumb and tabs (section 9.4) |
| Centre | The conversation for the current tab, split by step |
| Right pane | Map, meters and library search |

Pages: Topic, Library, Review, Parking lot, Spend and Settings.

### 13.1. Ask about whatever you select

When you select something in the dashboard, a context chip appears above the chat box. For example: "Selected: R2 (10 kΩ), C1 (100 nF)", or the first words of a highlighted passage. You can remove the chip. Your next message carries the selection to the tutor, so it knows exactly what you mean.

It works for:

- text in book passages and earlier messages (section 8.5)
- parts in the circuit simulator (section 10.3), sent with their values, connections, live voltage and current, and their lines of circuit text

## 14. Data model

| Entity | Purpose | Related to |
| :--- | :--- | :--- |
| Topic | Something you want to learn, with a goal | Steps, library items, tutor thread |
| Step | One concept in the topic's map, with its kind of knowledge (section 7.11), and its naive approach, breaking case and fix (section 7.9) | Topic, edges, meters, attempts |
| Edge | "Step B builds on step A" | Two steps |
| Thread link | Connects a topic, tangent or helper to its T3 thread | Topic or tangent |
| Tangent | A side question (section 9.6) | Topic, step, thread link |
| Circuit | A circuit shown in a lesson: its netlist, hints, CircuitJS1 text and check results (section 10.4) | Step |
| Exercise | One exercise of a given type, such as multiple choice, short answer, numeric, step-by-step, open-ended or recall card. Holds its answer key, common mistakes, variants, key source and check status (section 8.6). | Step, attempts |
| Attempt | One answer to an exercise, with confidence, note and hints used | Step, exercise |
| Meter | One of the four scores for a step | Step |
| Review card | A spaced-review item with its FSRS state | Step |
| Library item | A book or document you added | Topic, passages |
| Passage | One section or page of a library item | Library item |
| Resource card | A checked outside link with start and end times | Step |
| Role config | Settings for one pipeline role | None |
| Usage record | Tokens and cost for one log session | Thread link |

## 15. Build stages

1. **Connection test.** The Learn server signs in to T3, starts a thread, sends messages and reads replies. The tutor calls one `quiz` tool, and the dashboard shows the quiz with rendered math. Test on Claude and on Codex, and measure the reply delay. This stage decides whether the architecture in section 4 works.
2. **Port the upstream setup.** Move and edit the two skills as listed in section 2.1, and keep the quiz rules. Run the helpers through `delegate_task`. Add the Settings page with T3's model list.
3. **Track your learning.** Add the map, the chat split by step, the four meters, confidence taps, free-text grading, FSRS review, and the `show_image` tool (section 10.2). Also add exercises with answer keys, kind-of-knowledge tags (7.11), problem-first and example-first starts, step checking, the delayed mastery check and the tutor guardrails from section 7.8.
4. **Add your books and tangents.** Add the library, quote cards, reading along, and tangents with fork, park and return.
5. **Add the extras.** Add the challenge rules, the resource scout and the plain-language check.
6. **Add spend tracking.** Add the log reader, the price table and the spend page.
7. **Add the circuit simulator.** Host CircuitJS1 with the selection patch, and add its four tools, the circuit maker and the context chip (sections 10.3 and 13.1). Then build the layout engine (section 10.4): the prototype first, then the full version. Until the engine exists, the tutor uses example circuits and edits their values in place.

## 16. Decisions, open questions and risks

### 16.1. Decisions made

| Date | Decision |
| :--- | :--- |
| 2026-10-18 | Build a local web app first. Add a desktop wrapper later if wanted. |
| 2026-10-18 | Chat happens fully in the dashboard. |
| 2026-10-18 | The Learn server talks to T3 through T3's outside-agent MCP endpoint. |
| 2026-10-18 | Use the providers T3 already has. They are all subscriptions for now. |
| 2026-10-18 | Start with math and engineering subjects. |
| 2026-10-18 | The app is for learning through conversation, not for writing code. It builds no subject-specific tools, sandboxes or interactive widgets. The tutor uses its provider's own tools, and results show as text, diagrams and images (section 10). |
| 2026-10-18 | One exception: embed the CircuitJS1 circuit simulator, hosted rather than built (section 10.3). |
| 2026-10-18 | The AI never writes circuit coordinates. A deterministic layout engine built on elkjs places the parts (section 10.4). |
| 2026-10-18 | Plain language comes from the tutor's skill plus a rule check, not from a rewrite by a second model. |
| 2026-10-18 | Each explored tangent is a forked thread. On return, only a short summary goes back. |
| 2026-10-18 | Adopt the research changes listed in section 17.2. |
| 2026-10-18 | Start steps that add complexity from the naive approach and a case where it breaks (section 7.9). |
| 2026-10-18 | Use your book's solutions when they exist. Check generated solutions with code and an independent solve before use (section 8.6). |
| 2026-10-18 | The Learn server shows a practice solution only after a fixed number of hints, default 3, each after a new attempt (section 7.10). |

### 16.2. Open questions

- What should the final project name be?
- Which frontend framework should the dashboard use?
- Which price table should the API-equivalent cost use?
- Should the app read plan limits from T3's internal API?
- Should the selection patch be offered to the CircuitJS1 project as a pull request?
- Are the AI's layout hints worth having? Measure layout quality with and without them.

### 16.3. Risks

| Risk | Effect | Mitigation |
| :--- | :--- | :--- |
| Replies can't stream | Text appears in chunks | Check the thread often. Test in stage 1. |
| T3's internal API changes | Plan-limit data breaks | Keep it optional and behind one module |
| Codex doesn't load the Learn server's tools | The tutor works only on Claude | Test in stage 1 |
| This folder is in OneDrive | Syncing `node_modules` is slow, and syncing an open database is unsafe | Keep the database outside OneDrive. Consider moving the code folder or excluding `node_modules` from sync. |
| Forking a long main thread copies a lot of context | Tangents cost more | Measure in stage 4. If needed, start tangents from a short brief instead of a fork. |
| The layout engine is the biggest single piece of work | New circuits arrive late | Build the prototype first. Until then, edit example circuits in place. |
| CircuitJS1 changes its file format | Circuits stop loading | Pin the build, write text rather than XML, and test on every update |
| A wire junction isn't split | The circuit silently breaks | The read-back connection check blocks it from being shown |

## 17. Learning research

A research helper reviewed evidence from 2018 to 2026 on learning methods for technical subjects, preferring meta-analyses and randomized trials. The full report, with every source, is in [the learning methods research report](docs/research/learning-methods.md).

### 17.1. Methods worth adding

The effect sizes below are standardized: 0.2 is small, 0.5 is medium and 0.8 is large.

| Method | Key evidence | Where in this plan |
| :--- | :--- | :--- |
| Faded worked examples, matched to your level | About 0.48 over plain problem solving in 55 math studies. Help aids beginners (0.51) and hurts advanced learners (−0.43). | 7.6 |
| Mixed, spaced review of problem types | 61% vs 38% on a test a month later in a large classroom trial (0.83) | 7.5 |
| Checking each step of a solution | Step-checking tutors 0.76, human tutors 0.79, answer-only tutors 0.31 | 7.7 |
| Explain-back prompts in free text, with feedback | 0.55 overall and 0.48 for problem solving across 69 effects | 7.4 |
| Mastery only on a delayed, new problem | "Three correct in a row" in one session gave no gain a week later | 7.2 |
| Problem first, for conceptual steps | 0.36 for understanding and transfer, about zero for procedural skill | 7.6 |
| Linked pictures, equations and simulations, with support | Simulations 0.62 over none; supported simulations 0.49 over plain ones | 10.2 |
| Teaching that builds on your own attempt | Problem-first helps only with contrasting cases or teaching built on the learner's solutions | 7.9 |
| Erroneous worked examples | 0.62 on a test a week later in one study; no advantage in an earlier one | 7.9 |
| A new attempt required between hints | Clicking through hints to the answer goes with lower learning | 7.10 |

### 17.2. What changed in the plan

In sections 7, 8 and 10, each changed passage is marked with the words "Research update".

- **7.2:** a step is only mastered after a hint-free pass on a new variant in a later session.
- **7.3:** the target is an 80% to 90% band, not 85%. Emotion handling uses simple rules.
- **7.4:** confident errors get an explanation of the reasoning, plus a calibration chart. Explain-back prompts are limited and always get feedback. Grading uses a reference solution, a rubric and repeated grading.
- **7.5:** reviews use new variants, full solutions, mixed problem types and feedback.
- **7.6:** new section on choosing problem-first or example-first.
- **7.7:** new section on checking each step.
- **7.8:** new section on tutor guardrails.
- **7.9:** new section on starting from the naive approach (Draft 3).
- **7.10:** new section on the hint ladder (Draft 3).
- **8.4:** assigned material is short and specific, and each piece is followed by a recall question.
- **10.2:** plots carry their own labels and come with a mapping question.
- **14:** new Exercise entity, called Problem in Draft 2.
- **15:** stage 3 gains the new features.

### 17.3. What to avoid

- **Matching lessons to "learning styles":** no measurable benefit.
- **Rereading, highlighting or reading summaries as the main way to study:** low value. Reading an AI summary is a form of rereading.
- **Discovery with no guidance:** loses to explicit teaching.
- **An AI that hands out answers on request:** exam scores dropped 17% once the AI was taken away.
- **"Three correct in a row" as the test of mastery:** no lasting gain by itself.
- **Growth-mindset messages as a way to raise learning:** close to zero effect. They are fine as tone.

### 17.4. How the findings were checked

Four sources the plan relies on most were opened again and checked:

- **Wilson 2019:** confirmed.
- **Bastani 2025:** confirmed.
- **Kestin 2025:** confirmed. The effect of 0.63 is a lower bound, and the authors estimate 0.73 to 1.3.
- **Oreopoulos 2026:** confirmed. The gain from combining AI with the mastery workflow is modest and only marginally significant, and the paper is not yet peer-reviewed.

Some other claims rest on abstracts or summaries, because the papers were paywalled. The report lists each one.

The three sources added in Draft 3 (Loibl 2017, Adams 2014 and Baker 2004) were found by web search and checked against abstracts and summaries only. They are listed in the addendum to the research report.

### 17.5. How far the evidence reaches

The research asked about technical, problem-solving subjects, so its findings lean that way:

- **General methods:** retrieval practice, spacing, feedback and mixed practice have evidence across many kinds of subjects. Mixed practice, for example, also helped people learn to recognise painters' styles (Kornell and Bjork 2008).
- **Mostly math and science:** worked examples, step checking and problem-first come mostly from math and science studies. For other subjects, treat them as untested.

If the app is used for very different kinds of subjects, a short research pass on those subjects would help.

## 18. References

- [Learning methods research report](docs/research/learning-methods.md): full findings and sources for section 17
- [amosblomqvist/learn on GitHub](https://github.com/amosblomqvist/learn): the upstream teaching setup for pi. A local copy is in `reference/learn`.
- [Schematic layout research report](docs/research/schematic-layout.md): full findings and sources for section 10.4
- [pfalstad/circuitjs1 on GitHub](https://github.com/pfalstad/circuitjs1): the CircuitJS1 circuit simulator
- [Weave paper on arXiv](https://arxiv.org/abs/2607.03835): Gulgonul 2026, "Weave: Verified Netlist-to-Schematic Conversion via Layered Graph Layout"
- [pingdotgg/t3code on GitHub](https://github.com/pingdotgg/t3code): T3 Code source, including `docs/user/outside-agents.md` and `docs/orchestration-v2/thread-lineage-and-context-transfer.md`
- [pi-dedumbify package page](https://pi.dev/packages/pi-dedumbify): another pi learning tool that uses FSRS
- Wilson, Shenhav, Straccia and Cohen, 2019, "The Eighty Five Percent Rule for optimal learning", *Nature Communications*
