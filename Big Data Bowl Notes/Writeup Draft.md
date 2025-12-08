Title:
Evaluation of Receivers' Deep Ball In-Air Separation Ability

Subtitle:
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

**Executive Summary**
Deep passes are some of the most valuable offensive plays in football, yet teams still lack an effective way to quantify receivers' ability to adjust, track, and create catchable space once the ball is in the air. This project seeks to address this gap, developing methodology to quantify receivers' influence on completion probability when the ball is in the air.

Using frame-by-frame tracking data, I engineered features describing movement of receivers and defenders alongside their spatial relationships both between one and other and relative to the ball. With these, I trained a model designed to estimate the probability of a completion for each post-release frame of a play. In combining these estimates, I derived a metric called 'Net In-Air Completion Probability Change' (NCPC), which measures how much a receiver increases (or decreases) the chance of a completion while the ball is in flight, relative to the moment of release.

To support interpretation of this metric and its underlying modeling of completion probability, I built a two-paned play animation tool. For a given play, the tool illustrates player movement of all tracked players alongside the evolving completion probability curve estimated from the model, enabling analysts to visualize how motion related spatial dynamics impact catch probability while the ball is in the air.

Applied across the entirety of the 2023 regular season dataset, NCPC reveals meaningful and sometimes surprising differences in receivers' in-flight impact on deep target completion probability. The metric offers teams a new and unique data-driven way to evaluate skills including route adjustment, ball-tracking, and in-flight playmaking ability in one of football's highest-leverage play types.


**Motivation/Football Background**
Deep passes occupy a unique and influential space within modern offensive strategy. Despite their relative infrequency and lower completion probability, their explosive-yardage potential creates disproportional advantage in Expected Points Added (EPA) compared to other offensive play types. Because of this, defenses commit substantial resources to preventing their success, including safety depth, coverage rotations, and leverage techniques. The threat of the deep ball is one that shapes defensive behavior on every snap, making downfield attempts one of the most strategically impactful play types in the game.

Beyond quarterback accuracy and defensive structure, which are clearly impactful, receivers play a critical and often under-recognized role in determining the outcome of deep passes. Deep targets substantially test a receiver's ability to track the ball over long trajectories, adjust their routes mid-flight, manage leverage on defenders, and create substantial catch space, all at a high speed. The effectiveness of a receiver in these actions meaningfully impacts the likelihood of completion, yet these abilities are difficult to quantify with traditional statistics and modern separation metrics.

The ability to quantify a receiver's in-flight impact on completion probability offers direct strategic value to teams. For play callers, understanding which receivers are most talented at improving completion probability on deep attempts informs route design, matchup selection, and personnel usage. For front offices, it offers the ability to evaluate in-air ball tracking ability and playmaking, traits which complement route-running and separation metrics, supporting player evaluation and acquisition decisions.

Most existing tracking-based analytics focus on separation across the entire route or at the moment of the throw. While useful, these approaches emphasize early route movement and change-of-direction elements that are less prominent in deep vertical routes. By isolating player movement, spatial relationships, and separation dynamics once the ball is released, we can focus directly on the phase in which the completion of deep passes is decided. This perspective allows us to capture and quantify a receiver's ability to influence completion probability while the ball is in the air, a skill that is undervalued in current analytics yet fundamental to the success of downfield passing.

**Dataset and Feature Engineering**
The provided dataset for this project included frame-level tracking data for all offensive and defensive players in the 2023 NFL season. Pre-release frames contained full movement information, such as location, speed, direction, and orientation, while post-release frames lacked speed, direction, and orientation related fields. A play-level context file provided additional metadata, including pass depth and play outcome.

To model fluctuations in completion probability during ball flight, reconstructing post-release movement variables was essential. Frame-to-frame displacement vectors from player locations yielded instantaneous speed and direction, which were smoothed over a three-frame rolling window to estimate speed, acceleration, and direction, ensuring continuity across the point of release.

The tracking data was reformatted into a wide per-frame structure, where each row contained movement information for the targeted receiver and the four nearest defenders (both at release and per frame), alongside key spatial relationships, including receiver-defender and player-ball distances.

Advanced features were then created in three categories: (1) player motion features such as velocity toward the ball, angle of attack, and estimated time to arrival; (2) defender pursuit and leverage features, including closing speed and defender-ball angle of attack; and (3) interaction features capturing relative advantage, such as separation, crowdedness, and time-to-ball differentials. A temporal feature, 'flight normalized time', is created to represent the position of each frame within the ball-flight window.

We evaluated feature behavior by plotting their average trajectories over flight normalized time, separately for completions versus incompletions, to identify which feature values best distinguished completion outcomes and when in the in-flight phase this separation emerged. Receiver-defender distance, crowdedness, defender closing speed, receiver ball distance advantage, and time-to-ball advantage exhibited the strongest separation, suggesting meaningful use for the completion probability model.

Finally, we performed pairwise correlation analysis to prevent feature duplication. Defender angle to receiver and defender in phase angle were found to be nearly perfectly correlated, so in phase angle was excluded. With this modification, we finalized our feature set and split the data into training and test sets for model development.


**Methodology:**
Using these engineered features, our goal was not conventional binary classification of a completion, but evolving, fame-by-frame estimates of the probability that a pass would ultimately be completed. Each post-release frame was treated as an individual training instance labeled with the play's ultimate completion outcome. Rather than a sequence-based approach, temporal context was preserved using the flight normalized time feature, which encodes a frame's position within the ball in flight window.

Given the nonlinear, interaction-dependent nature of our feature set, we fit an XGBoost classifier to enable the capturing of such complex spatial relationships. Incorporating flight normalized time as a feature enabled the model to learn how spatial cues' impact evolves as a play progresses. A randomized train test split of observations ensured generalizable probability estimates.

A key challenge was preventing the classifier from memorizing the final outcome of the play. Because every post-release frame shares the same completion label, an unconstrained model tends to assign near-certain probabilities early in ball flight, ignoring meaningful spatial variation. To counter this, we applied regulation by limiting tree depth and the number of estimators, while using a low learning rate, resulting in heightened sensitivity to frame-by-frame spatial changes rather than play-level outcome memorization. This produces smooth and evolving signals of completion probability reflecting evolving in-play receiver-defender dynamics.

After training, the model was applied across the dataset, generating estimated completion probability signals for every post-release frame. The trajectory of estimated probabilities fluctuate in response to provided movement and spatial features, remaining smooth enough for integration while sensitive to changes in player movement, leverage, and separation.

Beyond frame-by-frame estimates, we quantify a receiver's influence at play level with the 'Net In-Air Completion Probability Change' metric. Using the estimated completion-probability at release as a baseline, we treat the curve from release to ball arrival as continuous over flight normalized time, and the area above or below the baseline is computed. Areas above the baseline indicate positive contributions and below indicate negative, capturing the receiver's cumulative impact on the play's completion probability net of the expectation at release.

Overall, the model produces frame-by-frame completion probabilities and the NCPC metric, providing a quantitative measure of receiver impact throughout a play. Aggregated across all plays, NCPC allows ranking of receivers by their ability to increase completion probability in-flight. While not a standalone measure of overall performance, it offers a consistent lens for comparing situational impact across deep pass plays.

**Results**
In the media gallery, we provide the top and bottom receivers in average Net In-Air Completion Probability Change (NCPC) on vertical deep passes in 2023. Among the top performers, we see expected names such as Nico Collins, Keenan Allen, Tyreek Hill, and Ja'Marr Chase, known for their downfield route-running and physical talents. This list also includes several "gadget" receivers like Cedrick Wilson, Rondale Moore, Christian Kirk, and Jameson Williams. Though these receivers weren't known as all-around receivers in 2023, they share traits including speed and change of direction that make them effective deep threats well-suited to generate positive opportunities downfield.

Among the lowest NCPC values on vertical routes, we observe players including Michael Thomas, Darnell Mooney, Rashod Bateman, and DeVante Parker. Receivers like Thomas, Bateman, and Parker are generally not viewed as elusive downfield threats, so their lower NCPC values aligns with intuition, reflecting limited ability to create separation and leverage. Mooney's appearance near the bottom of this list, however, highlights a limitation of the NCPC modeling which will be explored further below.

To examine NCPC more closely, we consider two representative plays, one positive and one negative. For each, videos of the animation of player movement and completion probability curve are included in the media gallery. First is a 48-yard touchdown by Rondale Moore, the second ranked receiver in average NCPC on vertical routes in 2023. At the moment of release, two defenders appear to bracket Moore downfield, with a model estimate of 25% completion probability at this moment. However, with no safety help over the top, Moore accelerates vertically, creating extensive separation from both defenders. The completion probability sharply increases to 80% before arrival, resulting in an NCPC of +0.277, one of the strongest values observed on vertical targets this season.

In contrast, Michael Thomas, who ranked last among all receivers with 5 or more deep vertical targets, illustrates a negative example. In Week 7, Derek Carr targets Thomas on a go route up the left sideline. At time of release, the model assigns a completion probability of 35%, but as the trailing defender closes in, Thomas fails to generate separation or leverage. The completion probability trends down to around 20%, and Thomas earns an NCPC of -0.093, with the ball ultimately being deflected. Though Carr's ball placement could have been better, Thomas' inability to create separation downfield is represented clearly through the NCPC value.

We also include distribution density curves of NCPC values for vertical deep passes in the media gallery. For completions, NCPC values roughly range from -0.2 to 0.35, with most values clustered between 0.0 and 0.2. This makes sense intuitively, as completed deep passes generally require some sort of gain in receiver advantage. Incompletions, in contrast, range in NCPC value from -0.2 to 0.2 centered just below zero, reflecting the fact that many unsuccessful deep passes involve little to no positive gain for receivers, often even involving decline as defenders close space.

Overall, the results of NCPC illustrate its strength in quantification of a receiver's ability to influence completion probability through features the model was trained on, including separation, leverage, pursuit, and movement dynamics. Though other traits, such as catch radius, vertical jumping, strength, and hand skills impact receivers abilities on deep ball targets, NCPC isolates movement and space creation components of downfield receiving. These components allow NCPC to highlight valuable traits for personnel evaluation and game-planning, including ball tracking, route adjustment, and the ability to create space during the ball's flight. 

**Limitations**
Despite the insightful findings and outcomes brought about throughout the development of the NCPC metric, several limitations showed their face. The primary limitation was the components of the data included. Since the dataset featured only movement related data for players, key factors to deep ball ability, including physical features of the receiver like catch radius and jumping ability and physical contact throughout the route were not incorporated in modeling. Another key limitation was the lack of isolation of the receivers talents. Clearly, more factors are at play contributing to the completion outcome of a deep pass, including specifically the ball placement of the quarterback and the ability of surrounding defenders. Finally, a key limitation was found in some highly negative NCPC values. In some scenarios marked 'go' routes, off schedule plays, where the quarterback may have been scrambling, require the receiver to change path, resulting often in big decreases in completion probabilities as defensive backs close in, unrightfully penalizing the receiver. Future work may identify and remove these sorts of plays, sticking only to vertical routes ran on their intended path. These limitations are just three of the more impactful ones identified, but many more limitations could likely be identified with the approach taken for the NCPC metric.

**Conclusion**
Despite their volatility, downfield passes hold great value in modern offensive football, existing as one of the most efficient ways to move the ball due to their explosive upside. Though many factors go into their successful completion, the ability of a receiver to track the ball, adjust their route, and create space to catch the deep ball is often one that overlooked due to its difficulty to quantify. By using advanced player movement, separation, and spatial interaction features, I modeled completion probability over the course of the in-air period of deep passes, and used the cumulative gain or loss of such probability relative to that at the moment the ball was released to create a metric called 'Net In-Air Completion Probability Change'. This metric seeks to quantify how much a receiver increases or decreases the chance of a completion in ball flight. With the clear value of the deep pass in modern NFL offense, play callers, front offices, and analysts alike can look to NCPC to make personnel decisions, evaluate performance, and optimize play-calling to maximize the efficiency of one of the highest reward plays in football.


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