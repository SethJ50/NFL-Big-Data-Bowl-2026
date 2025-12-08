Title:
Tracking In-Air Receiver Impact on Deep Vertical Passes

Subtitle:
Quantifying receivers' ability to influence deep vertical pass completion probability when the ball is in the air.

This project investigates how effective receivers are at increasing completion probability from the time the ball is released to when it lands by analyzing spatial, geometric, and motion related features of a route progression.

This project investigates how effective receivers are at increasing the probability of a completion when the ball is in the air. By analyzing spatial, geometric, and motion related features for the targeted receiver and nearby defenders throughout the progression of the phase of a deep pass while the ball is in the air.

Writeup Skeleton:
- Introduction: the deep pass is an event of relatively high risk and high reward, and captivates the audience due to its perceived uncertainty. Despite this perception, the completion of a deep pass strongly relies on both the ability of the receiver to make a play on the ball, the defenders (both individually and as a group) to work to prevent such a play, and the dynamics of how the motion, geometry, and spatial relation of the receiver and such defenders interact. Though the entirety of a route matters with regard to the likelyhood of a receiver catching the ball, the subset of time in which the ball in the air tends to be perceived as most uncertain, ignoring the abilities of the receiver to tangibly increase the probability of a catch while the ball is in the air, alongside the ability of defenders and the defense as a whole to prevent such task. With a sight on quantifying the uncertainty of the phase of a deep pass when the ball is in the air, and a goal of understanding which of the NFL's receivers exhibited the best ability to increase the chances that the deep ball is caught, this project seeks to investigate how spatial, geometric, and motion related features over the progression of a route with the ball in the air impacts ultimate probability of completion, allowing the ranking of receivers ability in this area on a custom metric called 'Net In-Air Completion Probability Change'

**Phrasing Notes***:
Impacts
Route Adjustment, Ball Tracking, In-Air playmaking,

Features:
Player movement/motion, spatial relationships, separation dynamics, ball-relative tracking

Referring to the Metric:
Quantifies influence on completion probability during ball-flight window of downfield targets, how much a receiver increases/decreases chance of completion while ball is in flight relative to moment of release, in-flight impact on deep targets, 

### Executive Summary
Deep passes are some of the most valuable offensive plays in football, yet teams still lack an effective way to quantify receivers' ability to adjust, track, and create catchable space once the ball is in the air. This project seeks to address this gap, developing methodology to quantify receivers' influence on completion probability when the ball is in the air.

Using frame-by-frame tracking data, I engineered movement and spatial features describing receiver-defender interactions. With these, I trained a model to estimate completion probability for every post-release frame of a play, and derived a play-level metric called 'Net In-Air Completion Probability Change' (NCPC), which measures how much a receiver increases (or decreases) the chance of a completion while the ball is in flight, relative to the moment of release. To support its interpretation and its underlying completion probability modeling, I built a visualization tool that animates player movement alongside the evolving completion probability curve.

Applied across the entirety of the 2023 regular season dataset, NCPC reveals meaningful and sometimes surprising differences in receivers' in-flight impact on deep target completion probability. The metric offers teams a new and unique data-driven way to evaluate skills including route adjustment, ball-tracking, and in-flight playmaking ability in one of football's highest-leverage play types.

### Motivation and Football Background
Deep passes occupy a unique and influential space within modern offensive strategy. Despite their relative infrequency and lower completion probability, their explosive-yardage potential creates disproportional advantage in Expected Points Added (EPA) compared to other offensive play types. The threat of such plays causes defenses to commit substantial resources to prevent them, creating a clear advantage for offenses with a successful downfield passing game.

Beyond clear influence of quarterback accuracy and defensive structure, receivers play a critical role in determining the outcome of deep passes. Deep targets test a receiver's ability to track the ball, adjust their routes, manage leverage, and create space at high speeds. The impact of these traits is clear, but the ability to quantify them isn't straightforward. Existing tracking based analytics focus on separation across the entirety of the route, emphasizing change of direction elements that are less prominent in deep vertical routes. Changing focus to receiver-defender spatial dynamics allows us to isolate the phase in which the completion of deep passes is decided. This perspective allows us to quantify receivers' influence on completion probability while the ball is in the air, a skill undervalued in current analytics yet fundamental to the success of downfield passing.

Quantification of a receiver's in-flight impact on completion probability offers direct strategic value. For play callers, understanding which receivers are most talented at improving completion probability on deep attempts informs route design, matchup selection, and personnel usage. For front offices, it offers the ability to evaluate in-air ball tracking ability and playmaking, traits which complement route-running and separation metrics, supporting player evaluation and acquisition decisions. Together, these motivations establish the need for a metric that captures receiver skill when the ball is in the air in downfield passes.

### Data and Feature Engineering
The provided dataset included frame-level tracking data for all offensive and defensive players in the 2023 NFL season. Pre-release frames contained full movement information, including location, speed, direction, and orientation, while post-release frames lacked speed, direction, and orientation related fields. A play-level context file provided additional metadata, including pass depth and play outcome.

To model fluctuations in completion probability during ball flight, reconstructing post-release movement variables was essential. Frame-to-frame displacement vectors from player locations yielded instantaneous speed and direction, which were smoothed over a three-frame rolling window to estimate speed, acceleration, and direction, ensuring continuity across the point of release.

The tracking data was reformatted into a wide per-frame structure, where each row contained movement information for the targeted receiver and the four nearest defenders (both at release and per frame), alongside key spatial relationships, including receiver-defender and player-ball distances.

Advanced features were then created in three categories: (1) player motion features such as velocity toward the ball, angle of attack, and estimated time to arrival; (2) defender pursuit features including closing speed and defender-ball angle of attack; and (3) interaction features capturing relative advantage, such as separation, crowdedness, and time-to-ball differentials. A temporal feature, 'flight normalized time', is created to represent the position of each frame within the ball-flight window.

We evaluated feature behavior by plotting their average trajectories over flight normalized time separately for completions versus incompletions, seeking to identify which features best distinguished play outcome at different periods of ball-flight. Receiver-defender distance, crowdedness, defender closing speed, receiver ball distance advantage, and time-to-ball advantage exhibited the strongest separation, suggesting meaningful use for the completion probability model.

Finally, we performed pairwise correlation analysis to prevent feature duplication. Defender angle to receiver and defender in phase angle were found to be nearly perfectly correlated, so in phase angle was excluded. With this modification, we finalized our feature set and split the data into training and test sets for model development.

### Modeling Methodology:
Using these engineered features, our goal was not solely classification of a completions, but developing evolving, fame-by-frame estimates of the probability that a pass would ultimately be completed. Each post-release frame was treated as an individual training instance labeled with the play's completion outcome. Rather than a sequence-based approach, temporal context was preserved using the flight normalized time feature, which encodes a frame's position within the ball in flight window.

Given the nonlinear, interaction-dependent nature of our feature set, we fit an XGBoost classifier to enable the capturing of such complex spatial relationships. Incorporating flight normalized time as a feature enabled the model to learn how spatial cues' impact evolves as a play progresses. A randomized train test split of observations ensured generalizable probability estimates.

A key challenge was preventing the classifier from memorizing the final outcome of the play. Because every post-release frame shares the same completion label for a given play, an unconstrained model tends to assign near-certain probabilities early in ball flight, ignoring meaningful spatial variation. To counter this, we applied regulation by limiting tree depth and the number of estimators, while using a low learning rate, resulting in heightened sensitivity to frame-by-frame spatial changes rather than play-level outcome memorization. This produces smooth and evolving signals of completion probability reflecting evolving in-play receiver-defender dynamics.

After training, the model was applied across the dataset, generating estimated completion probability signals for every post-release frame. The trajectory of estimated probabilities fluctuate in response to provided movement and spatial features, remaining smooth enough for integration while sensitive to changes in player movement, leverage, and separation.

Beyond frame-by-frame estimates, we quantify a receiver's influence at play level with the 'Net In-Air Completion Probability Change' metric. Using the estimated completion-probability at release as a baseline, we treat the curve from release to ball arrival as continuous over flight normalized time, and the area above or below the baseline is computed. Areas above the baseline indicate positive contributions and below indicate negative, capturing the receiver's cumulative impact on the play's completion probability net of the expectation at release.

Overall, the model produces frame-by-frame completion probabilities and the NCPC metric, providing a quantitative measure of receiver impact throughout a play. Aggregated across all plays, NCPC allows ranking of receivers by their ability to increase completion probability in-flight. While not a standalone measure of overall performance, it offers a consistent lens for comparing situational impact across deep pass plays.

### Results
In the media gallery, we provide the top and bottom receivers in average Net In-Air Completion Probability Change (NCPC) on vertical deep passes in 2023. Among the top performers, we see expected names such as Nico Collins, Tyreek Hill, and Ja'Marr Chase, known for their downfield talents. This list also includes several "gadget" receivers like Cedrick Wilson, Rondale Moore, and Christian Kirk. Though these receivers aren't known as all-around receivers, they share traits including speed and change of direction that make them effective deep threats warranting this positive recognition.

Among the lowest NCPC values on vertical routes, we observe players including Michael Thomas, DeVante Parker, and Rashod Bateman. Each of these receivers are generally not viewed as elusive downfield threats, so their lower NCPC values aligns with intuition, reflecting limited ability to create separation and leverage.

To examine NCPC more closely, we consider two representative plays, one positive and one negative. First is a 48-yard touchdown by Rondale Moore, the second ranked receiver in average NCPC on vertical routes in 2023 (see animated visualization in the media gallery). At the moment of release, two defenders appear to bracket Moore downfield, with a model estimate of 25% completion probability at this moment. However, with no safety help over the top, Moore accelerates vertically, creating extensive separation from both defenders. The completion probability sharply increases to 80% before arrival, resulting in an NCPC of +0.277, one of the strongest values observed on vertical targets this season.

In contrast, Michael Thomas, who ranked last among all receivers with 5 or more deep vertical targets, illustrates a negative example. Thomas is targeted on a go route up the left sideline, where the model assigns a completion probability of 35% at release, but as the trailing defender closes in, Thomas fails to generate separation or leverage despite an arguably under-thrown ball. The completion probability trends down to around 20% and the ball is deflected. Thomas earns an NCPC of -0.093, accurately reflecting his inability to create downfield separation.

Further, analysis of the distributions (see media gallery) of NCPC values follows intuition. For completions, NCPC values range from -0.2 to 0.35, with most values clustered between 0.0 and 0.2. This makes sense intuitively, as completed deep passes generally require some sort of gain in receiver advantage. Incompletions, in contrast, range in NCPC value from -0.2 to 0.2 centered just below zero, reflecting the fact that many unsuccessful deep passes involve little to no positive gain for receivers, often even involving decline as defenders close space.

Overall, these results illustrate that NCPC offers a coherent and interpretable measure of how receivers shape completion probability during the ball's flight. Despite this, the metric's behavior and interpretation is bounded by several limitations noted in the following section.

### Limitations
Several limitations surround the NCPC metric. First, because the data only includes player movement information, important factors to deep pass completion outcomes, such as receiver catch radius, jumping ability, and physical contact during the route aren't captured nor factored into the metric. Another significant limitation is the metric's lack of isolation to the targeted receiver's ability. Completion outcomes depend heavily on factors like  ball placement and defensive ability, yet the NCPC metric doesn't control for them. Finally, exploratory analysis of the metric revealed a sensitivity to deviation from typical route paths. For example, in off-schedule situations where quarterbacks scramble and the receiver breaks off their route, the closing in of defenders results in a significant reduction in completion probability that unfairly penalizes the receiver. Future work may better identify and remove such plays to ensure NCPC is evaluated only on true vertical routes. Beyond these, other limitations likely remain considering the scope and complexity of modeling deep ball completion outcomes.

### Conclusion
Despite their volatility, downfield passes hold great value in modern offensive football, existing as one of the most efficient ways to move the ball due to their explosive upside. While many factors go into their successful completion, the ability of a receiver to track the ball, adjust their route, and create space during flight is difficult to quantify. Using detailed movement, separation, and spatial interaction features, this work models completion probability over the  in-air portion of deep passes and summarizes a receivers cumulative impact through the Net In-Air Completion Probability Change (NCPC) metric. NCPC measures how much a receiver increases (or decreases) the likelihood of a completion relative to the probability at ball release.

Given the importance of the deep passing game, NCPC provides play callers, front offices, and analysts a descriptive metric for evaluating receiver impact and optimizing personnel and play-calling decisions for one of football's highest-reward play types.


Use Cases:
Examples: Scouting - identify receivers skilled at deep ball tracking / separation, Game Planning - choose deep target matchups where receivers historically gain probability, Analytics Staff - pair metric with QB ball placement skills



- Football Interpretation & Use Cases
	- ?

Limitations
Add: I assume receiver is free to optimize path, sometime route constrained, physical contact not captured....

doesn't have anything to do with ball placement, in air skill, etc

very defense and qb dependent, non isolation

weird finding: off schedule plays can very much penalize you... a.j. brown - 2023122501, 3436


- Limitations & Future Work
	- Where the ball is placed and defensive influence on probability of completion.
	- More optimal paths for a wide receiver to take to increase probability of completion.
	- Temporal approach that more acurrately takes into account the progression of the feature values over time without a sole completion label for each frame.
- Conclusion




Rondale Moore: 2023111904, 136, https://www.youtube.com/watch?v=d-9SgldS3J8 0:33
Michael Thomas: 2023101900, 1277, https://www.youtube.com/watch?v=tJq8LO78XuM 3:42

Michael Thomas Incompletion
Week 7 2023: Jaguars @ Saints
NFL YouTube Play: https://www.youtube.com/watch?v=tJq8LO78XuM 3:42