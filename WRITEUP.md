# Net In-Air Completion Probability Change (NCPC)

*Quantifying receiver impact on deep passes while the ball is in the air — NFL Big Data Bowl 2026*

---

## Executive Summary

Deep passes are some of the most valuable offensive plays in football, yet teams still lack an effective way to quantify receivers' ability to adjust, track, and create catchable space once the ball is in the air. This project addresses that gap by developing a methodology to quantify receivers' influence on completion probability while the ball is in flight.

Using frame-by-frame tracking data, I engineered movement and spatial features describing receiver-defender interactions. With these, I trained a model to estimate completion probability for every post-release frame of a play, and derived a play-level metric called **Net In-Air Completion Probability Change (NCPC)**. NCPC measures how much a receiver increases (or decreases) the chance of a completion while the ball is in flight, relative to the moment of release. To support its interpretation and the underlying completion probability model, I built a visualization tool that animates player movement alongside the evolving completion probability curve.

Applied across the entire 2023 regular season, NCPC reveals meaningful and sometimes surprising differences in receivers' in-flight impact on deep target completion probability. The metric gives teams a new, data-driven way to evaluate route adjustment, ball-tracking, and in-flight playmaking in one of football's highest-leverage play types.

---

## Motivation and Football Background

Deep passes occupy a unique and influential space within modern offensive strategy. Despite their relative infrequency and lower completion probability, their explosive-yardage potential creates a disproportionate advantage in Expected Points Added (EPA) compared to other play types. The threat of such plays forces defenses to commit substantial resources to prevent them, giving offenses with a successful downfield passing game a clear edge.

Beyond the influence of quarterback accuracy and defensive structure, receivers play a critical role in the outcome of deep passes. Deep targets test a receiver's ability to track the ball, adjust routes, manage leverage, and create space at high speeds. The impact of these traits is clear, but quantifying them isn't straightforward. Existing tracking-based analytics focus on separation across the entire route, emphasizing change-of-direction elements that are less prominent on deep vertical routes.

Shifting the focus to receiver-defender spatial dynamics isolates the phase in which deep-pass completions are decided. This perspective quantifies receivers' influence on completion probability while the ball is in the air, a skill undervalued in current analytics yet fundamental to downfield passing success.

Such a metric offers direct strategic value:

- **Play callers:** knowing which receivers are best at improving completion probability on deep attempts informs route design, matchup selection, and personnel usage.
- **Front offices:** it evaluates in-air ball tracking and playmaking, traits that complement route-running and separation metrics and support player evaluation and acquisition decisions.

Together, these motivations establish the need for a metric that captures receiver skill when the ball is in the air on downfield passes.

---

## Data and Feature Engineering

### Data

The provided dataset included frame-level tracking data for all offensive and defensive players in the 2023 NFL season. Pre-release frames contained full movement information (location, speed, direction, and orientation), while post-release frames lacked speed, direction, and orientation fields. A play-level context file provided additional metadata, including pass depth and play outcome.

### Reconstructing post-release movement

To model fluctuations in completion probability during ball flight, reconstructing post-release movement variables was essential. Frame-to-frame displacement vectors from player locations yielded instantaneous speed and direction, which were smoothed over a three-frame rolling window to estimate speed, acceleration, and direction, ensuring continuity across the point of release.

### Data structure

The tracking data was reformatted into a wide per-frame structure. Each row contains movement information for the targeted receiver and the four nearest defenders (both at release and per frame), alongside key spatial relationships such as receiver-defender and player-ball distances.

### Engineered features

Advanced features were created in three categories:

1. **Player motion:** velocity toward the ball, angle of attack, and estimated time to arrival.
2. **Defender pursuit:** closing speed and defender-ball angle of attack.
3. **Interaction:** relative advantage measures such as separation, crowdedness, and time-to-ball differentials.

A temporal feature, **flight normalized time**, represents the position of each frame within the ball-flight window.

### Feature evaluation

We plotted average feature trajectories over flight normalized time, separately for completions and incompletions, to identify which features best distinguished play outcome at different stages of ball flight. The strongest separation came from:

- Receiver-defender distance
- Crowdedness
- Defender closing speed
- Receiver ball-distance advantage
- Time-to-ball advantage

This suggested meaningful use for the completion probability model.

Finally, pairwise correlation analysis was used to prevent feature duplication. Defender angle to receiver and defender in-phase angle were nearly perfectly correlated, so in-phase angle was excluded. With this modification, the feature set was finalized and the data split into training and test sets for model development.

---

## Modeling Methodology

### Frame-level completion probability

The goal was not solely to classify completions, but to develop evolving, frame-by-frame estimates of the probability that a pass would ultimately be completed. Each post-release frame was treated as an individual training instance labeled with the play's completion outcome. Rather than using a sequence-based approach, temporal context was preserved through the flight normalized time feature, which encodes a frame's position within the in-flight window.

Given the nonlinear, interaction-dependent nature of the feature set, I fit an **XGBoost classifier** to capture complex spatial relationships. Including flight normalized time as a feature let the model learn how the impact of spatial cues evolves as a play progresses. A randomized train-test split of observations ensured generalizable probability estimates.

### Preventing outcome memorization

A key challenge was preventing the classifier from memorizing the final outcome of the play. Because every post-release frame shares the same completion label for a given play, an unconstrained model tends to assign near-certain probabilities early in ball flight, ignoring meaningful spatial variation. To counter this, I applied regularization by limiting tree depth, number of estimators, and learning rate. This heightens sensitivity to frame-by-frame spatial changes rather than play-level outcome memorization, producing smooth, evolving probability signals that reflect in-play receiver-defender dynamics.

After training, the model was applied across the dataset to generate completion probability estimates for every post-release frame. These trajectories respond to changes in player movement, leverage, and separation while remaining smooth enough for integration.

### The NCPC metric

Beyond frame-by-frame estimates, receiver influence at the play level is quantified with **Net In-Air Completion Probability Change (NCPC)**:

1. Take the estimated completion probability at release as the baseline.
2. Treat the curve from release to ball arrival as continuous over flight normalized time.
3. Compute the area above or below the baseline. Area above indicates a positive contribution; area below indicates a negative one.

This captures the receiver's cumulative impact on the play's completion probability, net of the expectation at release.

Overall, the model produces frame-by-frame completion probabilities and the NCPC metric, a quantitative measure of receiver impact throughout a play. Aggregated across all plays, NCPC ranks receivers by their ability to increase completion probability in flight. While not a standalone measure of overall performance, it offers a consistent lens for comparing situational impact across deep pass plays.

---

## Results

### Leaderboard

The media gallery shows the top and bottom receivers in average NCPC on vertical deep passes in 2023.

- **Top performers:** expected names such as Nico Collins, Tyreek Hill, and Ja'Marr Chase, known for their downfield talents. The list also includes several "gadget" receivers like Cedrick Wilson, Rondale Moore, and Christian Kirk. Though not known as all-around receivers, they share speed and change-of-direction traits that make them effective deep threats.
- **Lowest values:** Michael Thomas, DeVante Parker, and Rashod Bateman. None of these receivers is generally viewed as an elusive downfield threat, so their low NCPC values align with intuition, reflecting limited ability to create separation and leverage.

### Case studies

To examine NCPC more closely, consider two representative plays, one positive and one negative.

**Positive: Rondale Moore, 48-yard touchdown (NCPC = +0.277)**
Moore ranked second in average NCPC on vertical routes in 2023 (see the animated visualization in the media gallery). At release, two defenders bracket Moore downfield, and the model estimates a 25% completion probability. With no safety help over the top, Moore accelerates vertically and creates extensive separation from both defenders. Completion probability climbs sharply to 80% before arrival, giving an NCPC of +0.277, one of the strongest values observed on a vertical target this season.

**Negative: Michael Thomas, deflected go route (NCPC = -0.093)**
Thomas ranked last among all receivers with 5 or more deep vertical targets. He is targeted on a go route up the left sideline, where the model assigns a 35% completion probability at release. As the trailing defender closes in, Thomas fails to generate separation or leverage despite an arguably underthrown ball. Completion probability trends down to around 20% and the ball is deflected. His NCPC of -0.093 reflects his inability to create downfield separation.

### Distributions

The distributions of NCPC values (see media gallery) also follow intuition:

- **Completions:** NCPC ranges from -0.2 to 0.35, with most values clustered between 0.0 and 0.2. This makes sense, as completed deep passes generally require some gain in receiver advantage.
- **Incompletions:** NCPC ranges from -0.2 to 0.2, centered just below zero. Many unsuccessful deep passes involve little to no positive gain for the receiver, often even a decline as defenders close space.

Overall, these results show that NCPC is a coherent and interpretable measure of how receivers shape completion probability during the ball's flight. The metric's behavior and interpretation are bounded by several limitations, described below.

---

## Limitations

- **Movement-only data.** Factors that matter for deep pass outcomes, such as catch radius, jumping ability, and physical contact during the route, aren't captured in the metric.
- **No isolation of the receiver.** Completion outcomes depend heavily on ball placement and defensive ability, which NCPC doesn't control for.
- **Sensitivity to off-script routes.** In off-schedule situations, such as a quarterback scramble where the receiver breaks off the route, defenders closing in cause a significant drop in completion probability that unfairly penalizes the receiver. Future work could identify and remove such plays so NCPC is evaluated only on true vertical routes.

Other limitations likely remain, given the scope and complexity of modeling deep-ball completion outcomes.

---

## Conclusion

Despite their volatility, downfield passes hold great value in modern offensive football, existing as one of the most efficient ways to move the ball thanks to their explosive upside. While many factors go into a successful completion, a receiver's ability to track the ball, adjust their route, and create space during flight is difficult to quantify.

Using detailed movement, separation, and spatial interaction features, this work models completion probability over the in-air portion of deep passes and summarizes a receiver's cumulative impact through NCPC, which measures how much a receiver increases (or decreases) the likelihood of a completion relative to the probability at ball release.

Given the importance of the deep passing game, NCPC gives play callers, front offices, and analysts a descriptive metric for evaluating receiver impact and optimizing personnel and play-calling decisions for one of football's highest-reward play types.
