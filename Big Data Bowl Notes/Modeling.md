I want to create a model that calculates completion probability at every frame from ball release to the end of the play. With this, I want to do something with considering how certain offensive players increase the completion percentage when the ball is in the air, and the contrast for defensive players.

For this model, I have the following in mind for inputs:
- Distance from each player to ball spot
- Distance from targeted player to each defender
- Last N Frames Momentum
	- Momentum comparison (speed, direction) between targeted receiver and defender
	- Momentum comparison to where the ball is thrown (pythagorean theorem, speed component towards ball?)

#### Basic Features
- tgt_rec_ / def_N_
	- x, y
	- s: Speed (yds/sec)
	- a: acceleration (yds / sec^2)
	- o: orientation of player (deg)
	- dir: angle of player motion (deg)
	- dist: Distance from targeted recever (defenders only)
	- ball_dist: Distance from ball landing position
- ball_land_x / ball_land_y
- complete
- route_of_targeted_receiver

#### Advanced Features
- **closing_speed_to_ball**: How quickly a player's distance to the ball is changing (yds / sec)
	- `closing_speed_to_ball = ball_dist[t - 1] - ball_dist[t] / dt`
	- Note: Frame per every tenth of a second
- **v_rec / v_defN**: The 2D velocity vector components (vx, vy) for receiver and each defender
	- `theta = np.deg2rad(df['tgt_rec_dir'])`
	- `vx = speed * cos(theta)`
	- `vy = speed * sin(theta)`
- **u_ball_dir**: Unit vector pointing from player -> ball landing point.
	- `ux = ball_land_x - x`
	- `uy = ball_land_y - y`
	- `dist = np.hypot(ux, uy)`
	- `u_ball_x = ux / dist`
	- `u_ball_y = uy / dist`
- **v_toward_ball**: Signed component of a player's velocity toward the ball's landing point (yds/sec)
	- `ux = ball_land_x - tgt_rec_x`
	- `uy = ball_land_y - tgt_rec_y`
	- `dist = np.hypot(ux, uy)`
	- `v_toward_ball = (vx * ux + vy * uy) / dist`
- **momentum_advantage_defN**: How much more effectively the receiver is moving toward the ball than defender N (yds/sec)
	- `momentum_adv_defN = (v_rec * u_ball_dir) - (v_defN * u_ball_dir)`
- **closing_speed_defN**: Defender's rate of change of distance to the reciever (yds/sec)
	- `closing_speed_defN = (def_N_dist[t - 1] - def_N_dist[t] / dt`
- **min_separation_all_def**: Minimum of def_N_dist across all defenders at current frame (yds)
	- `min_separation = min(def_1_dist, ..., def_4_dist)`
- **mean_separation_closest_3_defs**: Average separation of 3 defenders closest to the reciever
	- `mean_sep_closest_3 = np.sort(dists, axis=1)[:, :3].mean(axis=1)`
- **angle_defN_to_rec_path**: Angle between defender N's motion vector and receiver's motion vector (deg)
	- `dot = df['def_1_vx']*df['tgt_rec_vx'] + df['def_1_vy']*df['tgt_rec_vy']`
	- `den = (np.hypot(df['def_1_vx'], df['def_1_vy']) * np.hypot(df['tgt_rec_vx'], df['tgt_rec_vy'])).clip(min=1e-8)`
	- `df['angle_def1_rec_deg'] = np.degrees(np.arccos((dot/den).clip(-1,1)))`
- **angle_defN_ball_alignment**: Angle between defender's motion vector and the vector from defender to the ball landing point
	- `dot = def_vx * u_def_x + def_vy * u_def_y`
	- `den = (np.hypot(def_vx, def_vy) * np.hypot(u_def_x, u_def_y)).clip(min=1e-8)`
	- `df['angle_def1_ball_deg'] = np.degrees(np.arccos((dot/den).clip(-1,1)))`
- **t_rec_to_ball**: Estimated time (seconds) for player to reach ball landing point assuming straight line, constant speed.
	- `eps = 1e-6`
	- `df['t_rec_to_ball'] = df['tgt_rec_ball_dist'] / df['tgt_rec_s'].clip(min=eps)`
- **min_t_def_minus_t_rec**: Difference between minimum est. arrival of defender to ball and est. arrival to ball of receiver
	- `min_t_def_to_ball = min(of all t_defN_to_ball)`
	- `min_t_def_minus_t_rec = min_t_def_to_ball - t_rec_to_ball`
- **crowdedness**: Quantifies how 'crowded' a receiver's area is
	- `eps = 1e-3`
	- `df['crowdedness'] = (1.0 / (df[[f"def_{i}_dist" for i in range(1,5)]] + eps)).sum(axis=1)`
- **a_toward_ball**: Component of receiver acceleration in the direction of the ball (yds/sec^2)
	- `ax = tgt_rec_a * cos(theta)`
	- `ay = tgt_rec_a * sin(theta)`
	- `df['tgt_rec_a_toward_ball'] = ax * df['u_ball_x'] + ay * df['u_ball_y']`
- **def_in_phase**: Quantifies alignment of a defender with a receiver. +1 indicates movement in same direction, 0 indicates orthogonal movement, -1 indicates opposite direction (out of phase)
	- `def_in_phase = cos(def_dir - rec_dir)`
- **def_pincer_angle**: Angle between vectors from receiver to the two closest defenders. Small angle = approaching from similar direction, large angle = bracketing from two sides
	- Longer Implementation


#### Final Model Features
- tgt_rec_
	- ball_dist
	- v_toward_ball
	- closing_speed_ball
	- a_toward_ball
	- t_to_ball
- def_closeN_
	- dist
	- ball_dist
	- v_toward_ball
	- closing_speed_ball
	- closing_speed_rec
	- t_to_ball
	- in_phase
	- momentum_adv
	- angle_rec_deg
	- angle_ball_deg
- min_separation_all_def
- mean_separation_closest_3_defs
- min_t_def_to_ball
- min_t_def_minus_t_rec
- crowdedness
- closest_2_def_angle_diff
- pass_length
- route_of_targeted_receiver
- *complete*























# Practical tips & engineering notes

- **Frame rate / dt**: If every frame is uniform (e.g., 10 Hz), you can use frames directly; otherwise compute `dt` from timestamps. Always include `dt` in rates for interpretability.
    
- **Smoothing / noise**: Speeds, accelerations, and directions are noisy. Use short rolling windows (3–5 frames) or an exponential smoother before derivative calculations.
    
- **Normalization**: Many features benefit from normalization: divide by ball distance, or use z-scoring per play or across dataset.
    
- **Missing defenders**: Some plays might have fewer than 4 defenders in your wide format (NaNs). Use `.clip` / `.fillna` and be careful when aggregating.
    
- **Conventions**: Confirm `dir` convention by comparing to `(dx,dy)` between frames. If mismatch, convert `dir` first.
    
- **Units**: keep consistent units (yards, seconds). Label features clearly (e.g., `_yards_per_sec`, `_deg`).
    
- **Windowed versions**: Create time-windowed variants:
    
    - `closing_speed_last_3_mean`
        
    - `v_toward_ball_rolling_std_5`
        
    - `momentum_advantage_ema_3`
        
- **Interactions**: Combine features into interaction terms or ratios that models may find useful:
    
    - `momentum_adv_def_min / min_separation_all_def`
        
    - `t_rec_to_ball / (min_t_def_to_ball + eps)`
        
- **Feature importance**: once a model is trained, check which of these actually help — many will be correlated.

##### EDA

Binary Output Variable: Completion / Incomplete







##### Expected Completion Percentage Model
