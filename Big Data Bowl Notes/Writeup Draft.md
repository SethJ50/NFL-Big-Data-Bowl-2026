Title:
Evaluation of Receivers' Deep Ball In-Air Separation Ability

Subtitle:
This project investigates how effective receivers are at increasing completion probability from the time the ball is released to when it lands by analyzing spatial, geometric, and motion related features of a route progression.

This project investigates how effective receivers are at increasing the probability of a completion when the ball is in the air. By analyzing spatial, geometric, and motion related features for the targeted receiver and nearby defenders throughout the progression of the phase of a deep pass while the ball is in the air.

Writeup Skeleton:
- Introduction: the deep pass is an event of relatively high risk and high reward, and captivates the audience due to its perceived uncertainty. Despite this perception, the completion of a deep pass strongly relies on both the ability of the receiver to make a play on the ball, the defenders (both individually and as a group) to work to prevent such a play, and the dynamics of how the motion, geometry, and spatial relation of the receiver and such defenders interact. Though the entirety of a route matters with regard to the likelyhood of a receiver catching the ball, the subset of time in which the ball in the air tends to be perceived as most uncertain, ignoring the abilities of the receiver to tangibly increase the probability of a catch while the ball is in the air, alongside the ability of defenders and the defense as a whole to prevent such task. With a sight on quantifying the uncertainty of the phase of a deep pass when the ball is in the air, and a goal of understanding which of the NFL's receivers exhibited the best ability to increase the chances that the deep ball is caught, this project seeks to investigate how spatial, geometric, and motion related features over the progression of a route with the ball in the air impacts ultimate probability of completion, allowing the ranking of receivers ability in this area on a custom metric called 'Net In-Air Completion Probability Change'

**Executive Summary**
Focuses: What problem coaches have, what you built, one surprising actionable insight, what tool/viz you deliver





- Executive Summary
	- Problem Focused On: Receiver's ability to increase completion probability while ball is in the air on deep passes.
	- Why it matters?: (not a strong argument yet) Analyzing receivers ability to increase completion percentage while the ball is in the air indicates a key skill/ability that is useful to know for coaches, scouts, front offices to assess performance, optimize gameplans, scout opponents, ...
	- Core Metric: 'Net In-Air Completion Probability Change' uses estimates of ultimate completion probability at each frame of a route while the ball is in the air, and creates a net integration based sum of how much total completion probability was added (or lost) while the ball is in the air until it lands/is caught
	- Key Insights / Outcomes: ? We were able to rate receivers on this metric over the course of the entirety of the dataset, resulting in rankings that included expected players as well as surprises. We could further dive into case studies of particular players / plays and and visualize what the model sees with regards to the features and understand how spatial and motion related characteristics of route development throughout the ball in air period increase completion probability.
	- Visualization or Tool Built: I created an animated visualization which shows the motion of the players over the course of time, with a plot underneath showing the estimated completion probability at each frame while the ball is in the air. This completion probability is also encoded as color and size of the receivers marker on the field animation. This allows for visual analysis of model results, ..., etc.

**Motivation/Football Background**
Add concrete football Motivations: Struggle to quantify ball tracking ability, receivers who gain separation post-throw are extremely valuable, deep targets account for disproportionate EPA, route adjustment versus assigned route distinctions, ability to gain leverage on DBs






- Motivation / Football Background
	- Why ball in-air time matters: Ball in air time matters with regards to a downfield completion due to the fact that deep passes feature the longest amount of time while the ball is in the air, and rely heavily on how skilled and able the receiver is to track the ball, and use their athletic talents like speed, quickness, and cutting to create pockets for themselves to catch the ball. It is clear that the defense and ball placement of the quarterback impacts this as well, but by viewing this window in time with a sight solely on the receiver, we can create a consolidated metric that combines their athletic talents with ball tracking and adjustment of route.
	- What coaches currently evaluate or struggle to quantify?: I am not really sure, maybe they need to better understand which receivers are able to best use their talents to increase deep ball probability, and use this to adjust personel decisions, acquisitions, and use of receivers. Focusing deep targets on players with this skill is well advised.
	- Gaps in current analytics: I think that a lot of the separation modeling in current analytics has to do with short passes and those with significant breaks, which is certainly meaningful to quantify players route running. My approach is unique in 2 ways: 1) The metric quantifies a combination of athletic ability, ball tracking, and route manipulation (in accordance with what the qb sees) and 2) We attempt to model, at each frame, the probability of ultimate completion, for highly volatile passes, giving a view into the change of these dynamics over the course of the ball in air period.

Dataset and Feature Engineering
Fix: Move 60-70% of feature list to Appendix, in main text summarize categories with 2-3 examples each (receiver features, defender features, interaction features)



- Dataset & Data Engineering
	- We were provided input data, which featured player tracking and player movement for all players prior to release, as well as output data, which was that for after the ball was release. First, since output data doesn't include key player movement tracking features like speed, direction, and acceleration, we had to use player motion to create smoothed estimates of these values based on past frames. 
	- Next, the goal was to create wide data, where for each frame of each play, we had the following information: x, y, s, a, o, dir, and ball_dist for targeted receivers, the closest 4 defenders at the time of ball release, and the closest 4 defenders for that frame, and also for defenders, we had their distance to the receiver at that frame.
	- Finally, we created a multitude of advanced player tracking features that took into account receiver/defender motion and interaction of such motion. The following features where created:
		- Targeted Receiver: v_toward_ball, closing_speed_ball, t_to_ball, ball_facing_angle, sideline_dist, turn_angle_smoothed, a_toward_ball
		- Defenders (def_N_*, def_closeN_*): v_toward_ball, closing_speed_ball, closing_speed_ball_adv, momentum_adv, closing_speed_rec, angle_rec_deg, angle_ball_deg, t_to_ball, in_phase
		- Interaction features:
			- min_separation_all_def, mean_separation_closest_3_defs, min_ball_dist_all_def,ball_dist_advantage, min_t_def_to_ball, min_t_def_minus_t_rec, crowdedness, closest_2_def_angle_diff
		- Additionally, we created a feature, flight normalized time, which is a value in the range 0 to 1 representing where the frame lies in the set of post release frames for a play - intended to incorporate how close to attempted catch the frame was to induce a temporal nature into modeling. Also a route_group (VERTICAL, INTERMEDIATE, QUICK, and BACKFIELD)
	- We calculated correlation for these, and did some EDA in visualizing and comparing a smoothed average / distribution curve for values of each metric for complete versus incomplete passes to get an idea of the 'separation' each metric induces in predictability of completion
	- We grabbed our features and split into a training and test set to be used for modeling

Methodology:
Fix: Judges don't need internal reasoning, they need the effect: why regularization produces usable probability curves, why smoother probabilities matter for frame by frame metric (replace long descriptions with crisp, high level narrative and put tuning details into Appendix)



- Methodology
	- I fit a xgboost classifier to the data, with all of the features for each frame with the label as complete. Because the label is binary and reflects the end of play completion status, with the goal of producing a probablistic signal over time of per frame completion probability as the ball is in the air, we intentionally regularized this model by limiting its number of estimators, max tree depth, and a low learning rate. As compared to a more complex model, who would learn a great deal of confidence, even in early frames of a play, whether or not that play resulted in an ultimate completion or not, regularization allows us a method to use the features derived to understand signals in the dataset related to confidence of completion / incompletion, while still considering the position of the frame within the ball flight with flight normalized time. With this manual regularization, we are able to produce stable and well behaved probability trajectories that fluctuate over time and serve as adequate estimates for the flucuation of probability of ultimate completion for each frame progression while the ball is in the air. The output of this model, when fit back to every frame in the dataset (not just training frames), is the models estimated ultimate completion probability per frame of each play.
	- Using completion probability estimates at each frame of a play, we sought to create a composite metric that quantified the amount by which the receiver increased their probability of completion over the course of the ball in air period. The concept here was clear - start with what the first frame after release predicted the completion probability as. Any frame after at which the completion probability is greater than the ball release probability should be positive contribution, and any frame with comp prob less than the ball release probability should be negative. We could create a 'curve', which linearly attaches each of the (frame, completion prob) pairs, and create this metric by calculating the 'area under the curve' using the completion probability at ball release as the 'zero line'. Any area above the line was added as 'positive', and underneath negative, resulting in an integration based metric that we termed 'Net In-Air Completion Probability Change', as it encompasses the 'net', or entirety of the ball in air period, with the baseline being the probability at ball release, valuing to 'change' of that completion probability as the metric.
	- For each play, we can now analyze individual frames completion probability values, as well as quantify for a play, a value as a signal of how much the receiver increased completion probability over the course of the play.

Results
Need to: explain why these players score high/low through examples, connect metric to football intuition, show how probability trajectories look... build one strong figure - distribution of ncpc by route group, case study of probability evolution over time with animation



- Results
	- Receiver Rankings: Discussion about best and worst rankings for vertical routes?
		- Household Names: Nico Collins, Brandon Aiyuk, Keenan Allen, Godwin, Tyreek, Waddle, Chase, Evans
		- Surprises: Ced Wilson, Rondale Moore, Dalton Kincaid, Kylen Granson, Christian Kirk, Westbrook-Ikhine, D.J. Chark
		- Worst:
			- Surprises: A.J. Brown
	- Case Study(s): Good: Christian Kirk, Bad: A.J. Brown

Use Cases:
Examples: Scouting - identify receivers skilled at deep ball tracking / separation, Game Planning - choose deep target matchups where receivers historically gain probability, Analytics Staff - pair metric with QB ball placement skills



- Football Interpretation & Use Cases
	- ?

Limitations
Add: I assume receiver is free to optimize path, sometime route constrained, physical contact not captured....



- Limitations & Future Work
	- Where the ball is placed and defensive influence on probability of completion.
	- More optimal paths for a wide receiver to take to increase probability of completion.
	- Temporal approach that more acurrately takes into account the progression of the feature values over time without a sole completion label for each frame.
- Conclusion


