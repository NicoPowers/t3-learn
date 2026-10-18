# Learning methods for technical subjects: research report

- **Purpose:** Keep the full research findings behind section 17 of [the t3-learn plan](../../PLAN.md), with every source.
- **For:** The project owner, and anyone changing the teaching method.
- **Date:** 2026-10-18.
- **How it was made:** A research subagent searched for evidence from 2018 to 2026, preferring meta-analyses and randomized trials. Its report is below, unchanged except for formatting. Four sources were then opened again to check the claims the plan relies on most.

## Spot-check results

| Source | Claim checked | Result |
| :--- | :--- | :--- |
| [Wilson et al. 2019, the 85% rule](https://pmc.ncbi.nlm.nih.gov/articles/PMC6831579/) | 85% is derived for binary classification learned by gradient descent. Other noise assumptions move it to 82% or 75%. | Confirmed. The paper does not say outright that the theory fails for humans. It says the result "remains to be generalized to a broader class of circumstances". |
| [Bastani et al. 2025, PNAS](https://pmc.ncbi.nlm.nih.gov/articles/PMC12232635/) | Plain GPT-4 cut exam scores 17%. A hint-giving version removed the harm but gave no gain. Plain GPT-4 was right 51% of the time. Students misjudged their learning. | Confirmed. |
| [Kestin et al. 2025, Harvard physics tutor](https://pmc.ncbi.nlm.nih.gov/articles/PMC12179260/) | 0.63 SD over in-class active learning, with pre-written solutions and structure enforced by the platform | Confirmed, with one change. The authors call 0.63 an underestimate caused by a ceiling effect, and their quantile regression gives 0.73 to 1.3 SD. |
| [Oreopoulos et al. 2026, NBER w35621](https://www.nber.org/papers/w35621) | "Three correct in a row" did not by itself improve a delayed test. AI plus the mastery workflow did best. | Confirmed, with one change. The combined result is the "most encouraging" one but only "marginally significant". This is a working paper, not yet peer-reviewed. |

## Report

I found strong evidence for six things this app can add easily. One needs more build work. The plan already gets most things right. The main changes: check skills at the step level, count a node as mastered only after a delayed test on a fresh problem, and keep the AI from handing out answers.

### Top methods to integrate (ranked)

**1. Faded worked examples, matched to the learner's level**
- **Method:** Show a fully solved problem. Then show the same kind of problem with the later steps blank. Then give a bare problem. Give less help as skill grows.
- **Evidence:** A 2023 meta-analysis of 55 math studies found g ≈ 0.48 versus plain problem solving (Barbieri et al., Educational Psychology Review). The paper is paywalled, so this figure comes from two summaries; a 2022 poster version reported 0.43. A 2025 meta-analysis of the expertise reversal effect (60 experiments, 5,924 people) found that extra help aids novices (d = 0.51) and hurts advanced learners (d = −0.43) (Tetzlaff et al., Learning and Instruction).
- **Why it works:** Novices have little working memory to spare, and a worked example frees it to take in the structure of the solution. Once the learner knows that structure, the example is redundant and gets in the way.
- **How this app would use it:** Open each procedural node with one worked example, taken from the user's textbook where possible, then blank out steps starting from the end. Use the probe score for each node to skip examples the user doesn't need.
- **Effort to add:** low.

**2. Interleaved, spaced review of problem types**
- **Method:** Mix problem types in review, so the learner must first decide which method applies. Spread practice across days.
- **Evidence:** A preregistered cluster RCT in 54 seventh-grade classes found 61% vs 38% on an unannounced test a month later, d = 0.83 (Rohrer et al., 2020). The U.S. What Works Clearinghouse rated it "meets standards without reservations." A meta-analysis found g = 0.34 for math tasks, with larger effects when categories look alike (Brunmair & Richter, 2019). A 2025 math-only meta-analysis found spacing g = 0.28 (Murray et al.).
- **Why it works:** Blocked practice tells you the method in advance. Mixing forces you to learn which method fits which problem.
- **How this app would use it:** Keep the quiz right after a node blocked. Build later review sets from several DAG nodes, pairing problems that look similar but need different methods (for example, substitution vs integration by parts).
- **Effort to add:** low.

**3. Step-level checking and hints**
- **Method:** Check each step of the learner's solution, not just the final answer. Give a hint at the first wrong step.
- **Evidence:** A 2011 review compared tutoring with no tutoring (VanLehn, Educational Psychologist). Tutors that work at the step level scored d = 0.76, close to human tutors at 0.79. Systems that check only answers scored 0.31.
- **Why it works:** Feedback on the faulty step points at the actual misconception. A wrong final answer only says something broke.
- **How this app would use it:** The user types their steps in Markdown or LaTeX. The grader compares each step with a stored reference solution and gives a hint on the first bad step, not the fix.
- **Effort to add:** medium.

**4. Self-explanation prompts (free text, with feedback)**
- **Method:** Ask the learner to explain, in their own words, why a step or principle works.
- **Evidence:** A meta-analysis of 69 effects found g = 0.55 overall and 0.48 when solving problems (Bisra et al., 2018). Explaining it yourself beat being given the explanation (g = 0.35). Multiple-choice prompts showed no reliable effect (g = 0.24, only 2 studies).
- **Caveat:** In the math worked-example meta-analysis, adding self-explanation prompts lowered the benefit (Barbieri 2023, again from summaries). The likely cause is that learners' explanations were often wrong and went uncorrected. Elaborative interrogation (plain "why is this true?" questions) rates only moderate utility (Dunlosky et al., 2013).
- **Why it works:** Producing the explanation forces you to tie the step to principles you already know.
- **How this app would use it:** Ask before the tutor explains. Use one or two prompts per node, on key steps only. The grader checks the explanation against the principle and corrects it.
- **Effort to add:** low.

**5. Count mastery only on a delayed, fresh problem**
- **Method:** Mark a node mastered only after the learner solves a new variant, without hints, in a later session.
- **Evidence:** Bloom's "two sigma" tutored groups were retested with new questions until they scored 80–90%. A mastery-learning group that got only this testing and feedback gained about 1.1 SD, roughly half of the two sigma (von Hippel, 2024). Realistic tutoring effects are 0.33–0.37 SD.
- **2026 trial:** A randomized trial with more than 6,000 middle-school students found that "three correct in a row" raised in-session results but did not, on its own, improve a test one week later (Oreopoulos et al., NBER). AI help combined with the mastery workflow did best. This is a working paper, not yet peer-reviewed.
- **Why it works:** Performance right after teaching overstates what stuck. A delay plus a new problem measures real learning.
- **How this app would use it:** Keep the quiz after each node, but treat a pass as "learned today." Promote the node to "mastered" only after a hint-free pass on a new variant at the next session; otherwise reteach the failing part. This also matches "successive relearning" (practice until correct, then again in later sessions; Rawson & Dunlosky, 2022).
- **Effort to add:** low.

**6. Problem first, then instruction (productive failure), for conceptual nodes**
- **Method:** Pose a problem the learner can attack with what they know but can't yet solve. Then teach, building on their attempt.
- **Evidence:** A meta-analysis of 53 studies found g = 0.36 for conceptual knowledge and transfer, but −0.03 for procedural skill (Sinha & Kapur, 2021). Designs that closely followed productive-failure principles reached up to 0.58.
- **Boundary conditions:** Instruction first did better for grades 2–5. Two small experiments found instruction first better on high-load material (Ashman et al., 2020). Unguided discovery loses to explicit instruction (d = −0.38; Alfieri et al., 2011).
- **Why it works:** The failed attempt brings up what you know and shows you the gap. The teaching that follows then answers a question you already have.
- **How this app would use it:** This is "how could I have discovered this?" turned into an activity. When a conceptual node's prerequisites are mastered, pose the motivating question for 10–15 minutes. Then derive the result by comparing the user's attempt with the standard one. For procedure-heavy or high-load nodes, or weak prerequisites, teach first with a worked example.
- **Effort to add:** low.

**7. Linked representations and small simulations**
- **Method:** Show the same idea as an equation, a diagram or graph, and sometimes an interactive model. Make the learner map between them.
- **Evidence:** A 2026 math meta-analysis found g = 0.29 for multiple vs single representations (15 studies, high variation; Liu et al.). A 2024 STEM meta-analysis found a third representation added little (g = 0.12) and helped only with support (Rexigel et al.). A meta-analysis of 59 K–12 STEM studies found simulations g = 0.62 vs none, and scaffolded simulations +0.49 over plain ones (SRI, 2014). A 2022 overview of reviews found that signaling, contiguity, coherence and segmenting all help (Noetel et al.).
- **Why it works:** Each representation exposes different structure, and linking them builds a richer mental model. Extra representations without support just add load.
- **How this app would use it:** Put a labeled plot or diagram next to each key equation, with labels on the figure itself. For engineering nodes, embed a slider widget (for example, vary R and C and watch the step response). Then ask a mapping question such as "which term flattens this curve?"
- **Effort to add:** medium to high (a widget per node).

### Already covered by the app's plan
- **Probe → DAG → teach:** The expertise reversal effect supports this. Adjustment: use each node's probe result to set the level of help (worked example vs straight problem), not just where to start.
- **85% target:** Wilson et al. derived 85% for binary classification learned by gradient descent. They say no formal theory yet covers human learning, and other noise assumptions move the optimum to 82% or 75%. Human data for vocabulary: scheduling reviews at 0.86–0.94 recall probability beat 0.40 and 0.98 (Eglington & Pavlik, 2020). Adjustment: aim for a band of 80–90% first-try, hint-free accuracy. Exclude probes and productive-failure problems, and track it for each facet.
- **FSRS:** Its default desired retention of 0.9 fits that band. Adjustments:
  - Review with new problem variants, not the same item, since memorizing one answer is not the skill.
  - In math, spacing is solid (g = 0.28), but plain retesting is not yet proven (g = 0.18, confidence interval crosses zero; Murray et al., 2025).
  - Testing transfers to new questions (d = 0.40) but only weakly for skills learned from worked examples, unless retrieval is elaborated (Pan & Rickard, 2018). So reviews should demand a full solution.
  - Always give feedback: it raised the classroom testing effect from g = 0.37 to 0.54 (Yang et al., 2021).
- **Confidence ratings:** Supported. High-confidence errors are corrected more readily (Metcalfe, 2017), and this held in a real college class (Carpenter et al., 2018). But in a classroom study, the advantage did not hold on new questions at semester's end (Corral & Carpenter). Adjustments:
  - After a confident error, explain where the reasoning failed; Metcalfe calls this analysis "crucial."
  - Schedule a new-variant check within days.
  - Show a calibration chart. Overconfidence predicts underachievement (Dunlosky & Rawson, 2012), and AI help inflates how much learners think they learned (Bastani et al., 2025; Kumar et al., 2023).
- **Socratic "why did you pick that?":** Supported; justifying your own choice gave g = 0.42 (Bisra). Keep it free text.
- **Model-graded free text:** Needed for self-explanation. Adjustment: always give the grader the reference solution and a rubric. ChatGPT-written hints failed quality checks on 32% of problems; running the model several times and keeping consistent answers ("self-consistency") cut that to near 0% for algebra and 13% for statistics (Pardos & Bhandari, 2024). Let the user dispute a grade.
- **Emotions:**
  - Confusion helps if it gets resolved. Learners who partly resolved their confusion beat those who stayed confused (D'Mello & Graesser).
  - Boredom is the most persistent and harmful state. Frustration was less harmful than expected (Baker et al., 2010).
  - The D'Mello–Graesser model of how emotions move from one to another often fails to replicate (Karumbaiah et al., 2018).
  - Adjustment: use simple rules. Hint after minutes of unresolved struggle; raise difficulty or switch to an application problem when the learner seems bored. Don't build a fixed emotion state machine.
- **Four facets:** Adjustment: test application and transfer directly with new contexts. Fluency does not imply transfer.
- **"Unconditional truths first" vs productive failure:** These conflict only for novices on high-load material; there, show the derivation as a worked example. Otherwise they fit: give axioms and definitions as givens, which lowers load, then let the learner attempt the derivation when ready.
- **Existing material instead of AI summaries:** Supported.
  - The Harvard AI tutor's success depended partly on high-quality instructional videos.
  - Using an AI to get explanations raised understanding, while using it in place of practice lowered understanding (Lehmann et al., 2025).
  - Adjustment: assign short, specific segments (section numbers, video timestamps), then follow each with a retrieval question.

### Avoid
- **Learning-styles matching:** Studies that test matching average d = 0.04 across 17 meta-analyses (Hattie & O'Leary, 2025). A 2024 meta-analysis called the benefits "too small and too infrequent" (Clinton-Lisell & Litzinger).
- **Rereading, highlighting or summaries as the main study activity:** Rated low utility (Dunlosky et al., 2013). Reading an AI summary is closest to rereading.
- **Unguided discovery:** d = −0.38 vs explicit instruction (Alfieri et al., 2011).
- **Answer-on-demand AI:** Students scored 17% lower once access was removed (Bastani et al., 2025).
- **"Three correct in a row" in one session as mastery:** It gave no delayed gain on its own (NBER, 2026).
- **Growth-mindset messages as a way to raise learning:** d = 0.02–0.05, not significant after correcting for bias (Macnamara & Burgoyne, 2022; other researchers dispute this). Fine as tone, but don't count on it.
- **Multiple-choice "why" prompts, or generic self-explanation prompts on every step without feedback:** No reliable benefit, or lower benefit (Bisra 2018; Barbieri 2023).

### AI tutor guardrails from recent trials
- **Hints, not answers.** Plain GPT-4 cut exam scores 17%. The version prompted to give hints and guide step by step removed that harm, but did not beat the no-AI group (Bastani et al., PNAS 2025). In another RCT, an AI that suggested guiding questions to human tutors raised student mastery by 4 percentage points (Tutor CoPilot, 2024).
- **Put the reference solution and common mistakes in the prompt.** Plain GPT-4 was correct only 51% of the time in the Bastani study. The Harvard tutor was given pre-written step-by-step solutions (Kestin et al., 2025).
- **Enforce the structure in code, not only in the prompt.** Kestin's team found a system prompt alone could not keep multi-part problems in order, so the platform walked students through one part at a time. In the NBER trial, AI helped only inside a workflow that turned mistakes into learning.
- **Learner attempts first, and help triggers after errors.** LLM explanations helped most when people tried the problem first (Kumar et al., 2023). Khanmigo users rarely asked for help after mistakes, and the effect was only 0.06–0.08 SD per year (Oreopoulos & Low, 2026). So the app should offer help on its own after an error.
- **Judge learning by delayed tests without the AI, not by how it feels.** Students with the hint-giving tutor felt they had done better but scored the same as the no-AI group (Bastani).
- **Two positive results:**
  - A Harvard physics RCT found 0.63 SD more learning with a structured AI tutor than with in-class active learning. It covered 194 students over two weeks in one course, with ceiling effects (Kestin et al., 2025).
  - A trial of Google's LearnLM tutor with every message checked by human tutors: 66.2% vs 60.7% on new problems for students helped by human tutors alone (165 students, 2025 preprint).

### Sources
- Wilson 2019 (85% rule): https://pmc.ncbi.nlm.nih.gov/articles/PMC6831579/
- Eglington & Pavlik 2020: https://pmc.ncbi.nlm.nih.gov/articles/PMC7567101/
- FSRS default 0.9: https://github.com/open-spaced-repetition/py-fsrs
- FSRS retention trade-off: https://github.com/open-spaced-repetition/fsrs4anki/wiki/The-optimal-retention
- Yang 2021: https://gwern.net/docs/spacedrepetition/2021-yang.pdf (DOI 10.1037/bul0000309)
- Pan & Rickard 2018: https://doi.org/10.1037/bul0000151
- Murray 2025: https://eprints.whiterose.ac.uk/id/eprint/229807
- van Gog & Sweller 2015 (background, testing effect and complex material): https://doi.org/10.1007/s10648-015-9310-x
- Rawson & Dunlosky 2022: https://doi.org/10.1177/09637214221100484
- Rohrer 2020, WWC review: https://ies.ed.gov/ncee/wwc/Study/88770
- Rohrer 2020, paper: https://www.gwern.net/doc/psychology/spaced-repetition/2019-rohrer.pdf
- Brunmair & Richter 2019 (accepted manuscript): https://www.psychologie.uni-wuerzburg.de/fileadmin/06020400/2019/Brunmair_Richter_in_press__2019_META-ANALYSIS_OF_INTERLEAVED_LEARNING.pdf
- Barbieri 2023 summary: https://www.learningscientists.org/blog/2024/1/25-1
- Barbieri 2023 summary: https://www.learningandthebrain.com/blog/?p=7218
- Barbieri 2022 poster: https://www.cehd.udel.edu/wp-content/uploads/2022/02/Worked-Examples-Meta-Poster_022722_SC.pdf
- Tetzlaff 2025: https://doi.org/10.1016/j.learninstruc.2025.102142
- Sweller 2019 (background, cognitive load theory): https://doi.org/10.1007/s10648-019-09465-5
- Bisra 2018: https://www.gwern.net/doc/psychology/spaced-repetition/2018-bisra.pdf
- Dunlosky 2013: https://doi.org/10.1177/1529100612453266
- Sinha & Kapur 2021: https://janfasen.nl/wp-content/uploads/2023/05/Sinha-and-Kapur-PS-I.pdf
- Ashman 2020: https://ifp.nyu.edu/2019/journal-article-abstracts/s10648-019-09500-5/
- Alfieri 2011: https://www.fisme.science.uu.nl/publicaties/literatuur/2011_alfieri_discovery_learning.pdf
- Metcalfe 2017: https://doi.org/10.1146/annurev-psych-010416-044022
- Carpenter 2018: https://pubmed.ncbi.nlm.nih.gov/29781391/
- Corral & Carpenter (summary): https://learningscientists.org/blog/2023/8/3-1
- Dunlosky & Rawson 2012, via Koriat commentary: https://newiipdm.haifa.ac.il/wp-content/uploads/2015/05/2012-Koriat-LI.pdf
- D'Mello confusion study: https://sciencedaily.com/releases/2012/06/120620103233.htm
- D'Mello confusion study (citation record): https://experts.colorado.edu/display/pubid_164503
- D'Mello & Graesser chapter: https://home.cs.colorado.edu/~mozer/Teaching/syllabi/TopicsInCognitiveScience/DMelloConfusion-Reprint.pdf
- Baker 2010: https://learninganalytics.upenn.edu/ryanbaker/BDRG-IJHCS-Final.pdf
- Karumbaiah 2018: https://library.apsce.net/index.php/ICCE/article/view/3622
- Kulik 1990 (background, mastery learning): https://doi.org/10.3102/00346543060002265
- von Hippel 2024: https://www.educationnext.org/wp-content/uploads/2024/03/ednext_XIV_2_von_hippel.pdf
- VanLehn 2011: https://doi.org/10.1080/00461520.2011.611369
- VanLehn effect-size table (slides): https://quality.mit.edu/files/2012/01/Quality-Symposium-Kurt-VanLehn.pdf
- NBER 2026: https://www.nber.org/papers/w35621
- Khanmigo 2026: https://edworkingpapers.com/ai26-1551
- Kestin 2025: https://pmc.ncbi.nlm.nih.gov/articles/PMC12179260/
- Bastani 2025: https://pmc.ncbi.nlm.nih.gov/articles/PMC12232635/
- LearnLM/Eedi trial: https://arxiv.org/abs/2512.23633
- Eedi scaled trial registration: https://www.socialscienceregistry.org/trials/18079
- Lehmann 2025: https://arxiv.org/abs/2409.09047v2
- Kumar 2023: https://scale.stanford.edu/ai/repository/math-education-large-language-models-peril-or-promise
- Tutor CoPilot: https://arxiv.org/abs/2410.03017
- Pardos & Bhandari 2024: https://doi.org/10.1371/journal.pone.0304013
- Liu 2026: https://guok.bnu.edu.cn/lwzz/c54b40c2f92b455e931eaf6069345bba.htm
- Rexigel 2024: https://doi.org/10.1007/s10648-024-09958-y
- SRI 2014: https://www.sri.com/wp-content/uploads/2022/04/simulations-for-stem-learning-brief.pdf
- Noetel 2022: https://doi.org/10.3102/00346543211052329
- Hattie & O'Leary 2025: https://doi.org/10.1007/s10648-025-10002-w
- Clinton-Lisell & Litzinger 2024: https://pmc.ncbi.nlm.nih.gov/articles/PMC11270031/
- Macnamara & Burgoyne 2022: https://doi.org/10.1037/bul0000352

### Claims the researcher could not fully verify

**Verified only through secondary sources:**
- Barbieri 2023: g = 0.48, the 55-study count, and the self-explanation finding come from two summaries; the paper is paywalled.
- Tetzlaff 2025: findings confirmed from the abstract only; full text not opened.
- D'Mello 2014: details come from a news release and the authors' book chapter, not the paper's abstract.
- Corral & Carpenter: findings come from a blog summary.
- Dunlosky & Rawson 2012: findings come from a commentary on the paper.
- Nickow 2020's 0.37 SD and Bloom's 80/90% thresholds come from von Hippel's article.
- Liu 2026: abstract read from a university-hosted page.

**Abstract-only:** Several claims rest on abstracts alone (pulled through Europe PMC and OpenAlex): Pan & Rickard, Metcalfe, Carpenter 2018, Stadler, Pardos, Noetel, Rexigel, Hattie, Macnamara, and Kulik.

**Not used because they could not be opened:** Kalyuga's rapid expertise assessment, the World Bank Nigeria trial, Rittle-Johnson 2017, and Kestin's supplementary system prompt.

**Reviewed but left out to keep the report short:** Stadler 2024 (LLM users felt less mental effort but reasoned worse) and Fan 2025 (ChatGPT improved essays without improving knowledge or transfer). Both point the same way as the "avoid answer-on-demand AI" point.

## Addendum: sources added for Draft 3 of the plan

These were found by web search on 2026-10-18 to support sections 7.9 and 7.10 of the plan. Each was checked against an abstract or summary only, not the full paper.

- **Loibl, Roll and Rummel 2017**, "Towards a Theory of When and How Problem Solving Followed by Instruction Supports Learning", *Educational Psychology Review* 29(4), 693–715. Problem solving before instruction helps only when the design uses contrasting cases or builds the teaching on students' own solutions. [Publication record at the Inter-University Computation Center](https://cris.iucc.ac.il/en/publications/towards-a-theory-of-when-and-how-problem-solving-followed-by-inst/)
- **Conceptual replication in Bayesian reasoning**: found no benefit for problem-first or for building on typical student solutions. The authors blame an attempt phase too shallow to expose gaps, and a mismatch between the students' solutions and those used in teaching. [Replication record at PH Freiburg](https://phfr.bsz-bw.de/frontdoor/index/index/docId/3543)
- **Adams, McLaren and others 2014**: 208 middle-school students who found and fixed errors in decimal solutions matched the problem-solving group right away, and scored higher a week later (d = 0.62). An earlier classroom study by Isotani and others (2011) with 255 students found no advantage. [Publication record at DFKI](https://dfki.de/web/forschung/projekte-publikationen/publikation/8598), [summary by The Learning Scientists](https://www.learningscientists.org/blog/2023/7/27)
- **Baker, Corbett and Koedinger 2004**, CHI 2004: in Cognitive Tutor classrooms, "gaming the system", meaning exploiting help and feedback to get answers, was more strongly tied to lower learning than any other off-task behaviour measured. [CHI 2004 paper hosted by RWTH Aachen](https://hci.rwth-aachen.de/materials/conferences/CHI2004/1p383.pdf)
