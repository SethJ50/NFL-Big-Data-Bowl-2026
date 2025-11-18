#### Data Viz Features
- Game Metadata
	- game_id
	- home_team_abbr
	- visitor_team_abbr
	- possession_team
- Play Metadata
	- play_id
	- play_description
	- quarter
	- game_clock
	- down
	- yards_to_go
	- yardline_side
	- yardline_number
	- possession_team
	- pre_snap_home_score
	- pre_snap_visitor_score
- Play Data
	- pass_length
	- pass_result
	- complete
	- offense_formation
	- receiver_alignment
	- ball_land_x, ball_land_y
	- route_of_targeted_receiver
	- pass_location_type
	- team_coverage_man_zone
	- team_coverage_type
	- yards_gained
- Frame Data
	- frame_id
	- to_release
	- frames_till_event
	- flight_normalized_time
- tgt_rec_
	- name
	- x, y
	- s, a, o, dir
	- ball_dist
	- v_toward_ball
	- closing_speed_ball
	- t_to_ball
	- a_toward_ball
- def_N_ (also def_closeN_ for ranked)
	- name
	- x, y
	- s, a, o, dir
	- dist
	- ball_dist
	- v_toward_ball
	- closing_speed_ball
	- closing_speed_ball_adv
	- momentum_adv
	- closing_speed_rec
	- angle_rec_deg
	- angle_ball_deg
	- t_to_ball
	- in_phase
- Team Defense
	- min_separation_all_def
	- mean_separation_closest_3_defs
	- min_t_def_to_ball
	- min_t_def_minus_t_rec
	- crowdedness
	- closest_2_def_angle_diff

Idea: I have written out some thoughts on each feature. I would be interested in creating viz's / EDA metrics for each of these to address my thoughts, and write out some narrative around it... A lot of these may involve adding back some more features, and may lead to ideas for other features...

route_of_targeted_receiver
GO        52406
POST      23024
CORNER    17944
CROSS     12176
IN         5550
HITCH      4554
OUT        2968
WHEEL      1168
SLANT       636
FLAT        474
SCREEN      110
#### By Feature
**dist**: How close is this defender to the receiver? The lower this is for the closest defender, avg for closest 2, 3 rec, the better. (Averages essentially result in a crowdedness metric)
- Most meaningful in regards to crowdedness... closest defender distance, and some sort of average for crowdedness
- Should be able to plot average value of this over time buckets and see some separation especially toward end of route

**ball_dist**: Straight forward, is the receiver, or the defenders, closer to where the ball is landing? *Should we add an advantage metric here?*
- Will be meaningful to include for receiver, as well as analyzing a closest defender to the ball, potentially an average distance of defenders to ball as a ball-crowdedness metric. Might also be useful to add in an advantage quantification between closest defender to ball and receiver.
- Plot ball_dist of closest defender over time, aggregated ball distance for N defenders over time, and advantage over time.

**v_toward_ball**: For a receiver, should check this out on change of direction routes to analyze the breaking point on the route. I think that from a modeling perspective, this is important. *Consider adding advantage between this and the closest defender to the receiver, and this and the closest receiver to the ball*
- Individually for receivers or defenders, this is pretty much meaningless... the best thing to come from this is purely on an advantage level... See below

**momentum_adv:** This is the difference between each defenders velo towards the ball and defenders... *If we take def_close1_ for this, we satisfy the advantage between this and the closest defender to the receiver... we need to also create this for closest defender to ball, but in a way this is encapsulated in the t_to_ball*
- Can use closest defender to receiver of this to quantify advantage of movement to ball.
- Should also create one of these for the closest defender to the ball versus the receiver, though this is kind of encapsulated in t_to_ball

**closing_speed_ball**: How quickly is the distance between the player and the ball closing? While v_toward_ball is predictive and direction based, closing_speed_ball is outcome driven: it captures the actual effect of the entire movement and ball flight. (Ex: A defender may only be angled slightly toward the ball, but he may be moving fast still toward the ball and the distance may be shrinking rapidly... this metric captures this)
- May want to create an advantage metric for this for analysis...

**closing_speed_ball_adv**: Advantage difference between window closing speed toward ball of receiver versus a defender.

**closing_speed_receiver:** also review this

**t_to_ball**: This is kind of noisy, especially for defenders. I think in essence, this combines the speed that a defender is moving with the distance that they are away from the ball. Kind of sounds more interesting the more you think about it, but not in the way I originally expected. *This is interesting in an advantage perspective in min_t_def_minus_t_rec, but we probably need a new way to plot / do EDA*
- I don't think that looking at the closest defender to receiver value of this is that interesting... rather, I think just the metrics we have for comparing the minimum of this to the minimum of receiver is interesting...

**a_toward_ball:** Contemplating throwing this out just because I don't know what advantage it gives us over other stuff...
- Not really sure if this will be useful

**angle_rec_deg**: Angle difference with receiver in their motion. This is almost like angle of attack by defensive players... *This could be another one interesting to rank by closest to the ball as well*
- Evaluate how this interacts with completion or not by route type
- It may also be interesting to add a feature for this for the closest defender to the ball...

**angle_ball_deg**: This is kind of like how far the defender is off of a direct path to the ball... This one can be an interesting one to quantify misdirection in routes as well...
- Check this out for misdirection type routes... it would be cool to evaluate this on a completed misdirection route and see if we see anything there...

**in_phase**: This quantifies how aligned the degree of motion is between defender and receiver... investigate how this differs from angle_rec_deg
- Figure out if this is any different than angle_rec_ball

**min_separation_all_def**: A good aggregation of defense for a receiver's separation
- This should be interesting to plot over time... should be higher for completions

**mean_separation_closest_3_defs**: A good aggregation for group crowdedness
- Should also be higher for completions, might be worthy of replacing crowdedness with this (or 2 defenders...)

**min_t_def_to_ball**: This combines distance and speed towards ball for a defender, indicating ball pressure
- Not sure how much this does individually, but used in combination with the receiver (below)...

**min_t_def_minus_t_rec**: gives a great indication on advantage of receiver or defender on both motion to and distance from ball
- Love this for advantage of receiver both in direction to ball and speed of movement, especially for misdirection routes, there should be some visible separation between completion/not... the thing is that the time variable is noisy by design, we might want to plot this differently than a smoothed line plot

**crowdedness**: Another group defense crowdedness metric, maybe not as good as others...
- This is okay, but having all 4 defenders and such a simplistic calculation seems a little off...

**closest_2_def_angle_diff**: Interesting metric with regards to how 2 defenders attack the receiver... think this probably gets a lot more meaningful closer to ball catch attempt...
- Again, more meaningful toward ball catch attempt, and also the impact of this will very likely depend on the route type... some routes it might not even matter...
#### Viz Ideas
- Completion vs. Incompletion Smoothed & Banded Curve Plots
- Distribution Comparisons
	- Complete vs. Incomplete boxplots/density curves over binned course of route? (~10 bins?)
- Correlation Analysis
	- Correlation matrices of frame level features against complete
	- Probably need to make N buckets and thus N different correlation matrices / heatmaps
- Separability Analysis
	- Rank features by how different they are at different points along the route for complete vs. incomplete
- Feature Interactions
	- Explore meaningful interactions (rec velo toward ball vs. def closing speed, momentum adv vs. separation, crowdedness vs. pass result...)
- Aggregated play level summaries
	- mean, max, min, of a stat for a whole play
	- I thought this could be interesting in saying that if there is a big receiver momentum advantage (max) over the course of the play, it could be impactful on completion percentage
- Temporal Feature Transformations
	- Avg value over last N% of ball flight (as an example)
- Create plots that align with the animation
	- Plot a stat over time for defenders and or receiver





Separate Data by: Complete, WR Route Type

- Variable vs. Frame (Rec. and Defenders)
	- ball_dist
	- v_toward_ball
	- closing_speed_ball
	- t_to_ball
- By Route Type - Complete vs. Incomplete for Defenders
	- dist
	- momentum_adv
	- closing_speed_rec
	- angle_rec_deg
	- angle_ball_deg
	- in_phase

