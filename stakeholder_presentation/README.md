# Stakeholder Presentation

**Will my campaign get funded? Predicting Kickstarter success before launch**

A 16-slide deck for a non-technical audience (about 10 minutes). It summarises the findings of the project notebook [04_kickstarter-success.ipynb](../04_kickstarter-success.ipynb).

| File / Link | Description |
| --- | --- |
| [stakeholder_presentation.pptx](stakeholder_presentation.pptx) | **The slides to present.** Opens in PowerPoint, Google Slides and Keynote; editable charts and speaker notes on every slide |
| [stakeholder_presentation.pdf](stakeholder_presentation.pdf) | The slides as PDF (read directly on GitHub, or present from any PDF viewer) |
| [Online version](https://claude.ai/artifact/X5ALuhDRr3fqYcgsgUDJqt) | Web version with presenter mode (opens for people it has been shared with) |

## How to Present

- **PowerPoint:** open `stakeholder_presentation.pptx`, then **Slide Show → From Beginning** (F5). Choose **Use Presenter View** to see the speaker notes on your own screen while the audience sees only the slides.
- **Google Slides:** upload the `.pptx` to Google Drive, open it with Google Slides, then **Slideshow → Presenter view**.
- **Keynote (Mac):** open the `.pptx` directly in Keynote.

All charts are native PowerPoint charts: right-click a chart and choose **Edit Data** to change the numbers.

## Slides

| # | Slide |
| --- | --- |
| 1 | Will my campaign get funded? |
| 2 | Our team |
| 3 | The problem: all or nothing |
| 4 | Our question |
| 5 | Smaller goals succeed far more often |
| 6 | Arts beat tech: Dance 66%, Technology 24% |
| 7 | The 30-day default and 60+ days do worst |
| 8 | The trap: data from the future |
| 9 | How we built it, step by step |
| 10 | The model separates risky from promising |
| 11 | What it gets right, and where it fails |
| 12 | Category and goal drive the prediction |
| 13 | Three things to decide before launch |
| 14 | The Campaign Checker |
| 15 | Limits and next steps |
| 16 | Thank you. Questions? |

## Speaker Script

What to say on each slide. The same text is stored as speaker notes in the PowerPoint file and the online version.

### 1. Will my campaign get funded?

Welcome. Today we answer one question that every Kickstarter creator asks before launch: will my campaign get funded? We studied 331,675 finished campaigns and built a model that estimates the chance of success using only what a creator knows before launch day.

### 2. Our team

We are a team of three. Kashif Kazmi is our project manager, who planned the milestones, ran our Kanban board and kept the team on track. Christian Balan is our design thinker, who framed the problem from the creator’s point of view and shaped the story you will hear. Furkan Yepette is our data scientist, who cleaned and explored the data and built the prediction model.

### 3. The problem: all or nothing

Kickstarter is all or nothing. If you miss your goal, even by one dollar, you get nothing. And most campaigns do miss: 6 out of 10 finished campaigns failed. Creators make their key decisions, like how much money to ask for and how long to run, before launch and without data. That is the gap we want to close.

### 4. Our question

Our question is simple: before launch, how likely is a campaign to succeed? We used 331,675 campaigns that finished as either successful or failed. The model gets the campaign plan as input: the goal, the category, the country, how long it runs, when it starts and the title. It returns a chance of success between 0 and 100 percent.

### 5. Smaller goals succeed far more often

Our first and strongest finding: the size of the goal. Campaigns asking for less than one thousand dollars succeed 55 percent of the time. Above fifty thousand dollars, only 15 percent succeed. In every category, successful projects asked for less than failed ones, typically half to two thirds as much. The goal is the biggest lever a creator controls.

### 6. Arts beat tech: Dance 66%, Technology 24%

Second finding: the category matters a lot. Dance, theater and comics projects succeed about two times out of three. Technology and journalism succeed only about one time in four. One reason: tech projects ask for much more money. The typical successful tech project asked for ten thousand dollars, a failed one twenty thousand. Arts projects often have a loyal community behind them.

### 7. The 30-day default and 60+ days do worst

Third finding: timing. Half of all campaigns simply keep the 30-day default, and only 38 percent of those succeed. Campaigns under four weeks and of 32 to 44 days do clearly better, and campaigns of 60 days or more do worst, with only one in four succeeding. This is probably about who chooses: creators who plan carefully also pick the length on purpose. It is a pattern, not proof of cause. Also, the year matters: in 2014 the number of campaigns jumped by 44 percent and success rates fell, probably because there was more competition and less selection. Since 2015 success rates have been recovering.

### 8. The trap: data from the future

A key step was spotting target leakage. The data also contains the money pledged and the number of backers. Those numbers already contain the answer: the rule “pledged at least the goal” matches the outcome in 99.997 percent of cases. But a creator does not know them before launch. A model that used them would look perfect in testing and be useless in real life, so we removed them.

### 9. How we built it, step by step

Here is our process in five steps. First we cleaned the data and removed the future information. Second, we locked away 20 percent of the campaigns as a test set, like a final exam the model never sees while learning. Third, we set a simple rule as the score to beat. Fourth, we compared three different kinds of models using cross-validation, which means we split the training data into five parts, train on four and check on the fifth, five times. Fifth, we tuned the best model and took the final exam once.

### 10. The model separates risky from promising

How good is the model? We tested it once, on 66,335 campaigns it had never seen. Our main score is called ROC-AUC. In simple words: if you show the model one successful and one failed campaign, how often does it give the successful one the higher chance? Guessing gets 0.5. The simple goal rule gets 0.63. Our model gets 0.77, so about three out of four pairs are ranked correctly. In practice: of the ten percent of campaigns the model rates most promising, 77 percent really succeeded. Of the ten percent it rates most risky, only 5 percent did.

### 11. What it gets right, and where it fails

Here is what the model gets right and wrong. It finds 83 percent of the campaigns that really succeeded. The price is false hope: about 45 percent of its “likely success” calls end up failing, because many failed campaigns look exactly like successful ones on paper. The good news: its percentages are honest. When it says 60 percent, about 60 out of 100 such campaigns really succeed. Its biggest mistakes are tiny board game campaigns that failed and huge technology or food campaigns that succeeded. What explains them, like product quality or an existing fan base, is not in our data.

### 12. Category and goal drive the prediction

Which information does the model use most? We tested this by scrambling one piece of information at a time and measuring how much worse the model gets. The sub-category, for example board games or indie rock, and the funding goal are by far the most important. The launch year and the campaign length follow. Details like the title length, the launch hour or the country matter only a little.

### 13. Three things to decide before launch

What does this mean for creators? Three decisions to make before launch. One: ask for less. Set the lowest goal that really funds the project and use stretch goals for the extras. Two: plan the length on purpose. Campaigns that kept the 30-day default did worse than most others, and 60-day campaigns did worst. Three: know your category. In hard categories like technology, food or fashion, build an audience before you launch. Remember, these are patterns from past campaigns, not guarantees.

### 14. The Campaign Checker

Our data product idea is the Campaign Checker. A creator enters the campaign plan and gets an honest probability of success. Then they can compare: how did similar campaigns with a lower goal or a different length do? The table shows the model's estimates for a US board game. At five thousand dollars and 21 days, 93 percent. At fifty thousand dollars and 60 days, only 27 percent. Kickstarter itself could also use it, for example to coach creators before launch. One caveat: these are patterns from the past, not promises.

### 15. Limits and next steps

Every model has limits. Ours finds patterns, not causes. It cannot see the product, the video, the rewards or the creator's fan base, which explains many of its mistakes. The data ends in early 2018, so the platform may have changed. And almost half of its “likely success” calls still fail. Next, we would add richer data like the creator's history and the campaign description, test the model on the newest campaigns, refresh the data, and build the Campaign Checker as a web tool while tracking its accuracy.

### 16. Thank you. Questions?

Thank you for listening. In one sentence: ask for less, plan your campaign length, know your category, and check your odds before you launch. We are happy to take your questions.
