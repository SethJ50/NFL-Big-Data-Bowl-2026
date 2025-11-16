#### wide_data
**Purpose**: This data contains general play information and ball land information, alongside information for the targeted receiver and the 4 closest defenders to him. This will be used to develop a final data frame with model features.
**Variables**:
- game_id
- play_id
- frame_id
- to_release: Indicator of if frame was 'before' or 'after' release
- {tgt_rec_ / def_1_ / def_2_ / def_3_ / def_4_}
	- x
	- y
	- s: Speed (yards / second)
	- a: acceleration in (yards / second^2)
	- o: orientation of player (deg)
	- dir: angle of player motion (deg)
	- dist: Distance from targeted receiver
	- ball_dist: Distance from ball landing position
- ball_land_x
- ball_land_y
- *complete*: 1 if pass completed, 0 otherwise
- pass_result
- play_description
- quarter
- game_clock
- down
- yards_to_go
- yardline_side
- yardline_number
- home_team_abbr
- visitor_team_abbr
- possession_team
- pre_snap_home_score
- pre_snap_visitor_score
- pass_length
- offense_formation
- receiver_alignment
- route_of_targeted_receiver
- pass_location_type
- team_coverage_man_zone
- team_coverage_type
- yards_gained

#### model_data
**Purpose**: Features and output (completion) to train models on
**Variables:**
- {def_N_}
	- dist: Distance of N'th defender from targeted receiver
	- ball_distance: Distance from ball landing position of N'th defender
- {tgt_rec_}
	- ball_dist: Distance from receiver to ball landing
	- Momentum Quantifiers:
		- in direction & speed towards ball
		- directional & speed interaction with defenders
- *complete*: 1 if pass completed, 0 otherwise
- 