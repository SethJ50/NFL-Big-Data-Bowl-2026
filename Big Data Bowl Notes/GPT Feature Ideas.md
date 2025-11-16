# ✅ 1. **Relative Movement Toward the Ball**

You already suggested:

### ✔ **Rate of distance change to the ball (receiver & defenders)**

Compute over sliding window (e.g., last 3 frames):

- `tgt_rec_ball_dist_change_3 = ball_dist[t] - ball_dist[t-3]`
    
- `def_1_ball_dist_change_3 = ...`
    

You can also compute:

### ✔ **Velocity toward the ball**

Project player velocity vector **onto the direction to the ball**:

`v_toward_ball = player_speed * cos(angle between velocity vector and vector to ball)`

This tells whether they’re **running toward**, **away**, or **sideways** relative to the ball.

---

# ✅ 2. **Receiver–Defender Relative Momentum**

You suggested comparing their momentum. You can formalize it as:

### ✔ **Relative momentum magnitude**

`relative_momentum_defN = (rec_speed * rec_mass) - (defN_speed * defN_mass)`

_(If you don’t have mass, just remove it — speed alone is fine)_

### ✔ **Directional momentum advantage**

Use velocity vectors and direction toward catch point:

`momentum_advantage_defN = v_rec · u_ball_dir  -  v_defN · u_ball_dir`

Where `u_ball_dir` is a unit vector from player → ball landing point.

Interpretation:

- Positive → receiver moving more effectively toward where the ball is going
    

---

# ✅ 3. **Closing Speed (Receiver–Defender)**

The most important single feature in matchup dynamics:

`closing_speed_defN = (defN_rec_dist[t-1] - defN_rec_dist[t]) / delta_t`

- Positive → defender closing in
    
- Negative → defender losing ground
    

Also compute:

- `max_closing_speed_last_5_frames`
    
- `closing_speed_slope` (is he accelerating toward receiver?)
    

---

# ✅ 4. **Receiver Separation Features**

### ✔ **Separation distance**

You already have `def_N_dist`.

Add:

- `min_separation_all_defenders`
    
- `mean_separation_3_closest_defenders`
    
- `separation_rankings` (e.g., def_1 is closest/2nd-closest)
    

### ✔ **Separation trend**

`separation_change_defN_3 = defN_dist[t] - defN_dist[t-3]`

---

# ✅ 5. **Competition Angles**

Angles influence catch probability enormously.

### ✔ **Defender Angle to the Receiver’s Path**

`angle_defN_to_rec_path = angle(v_defN, v_rec)`

If the defender is pursuing from behind or from a difficult angle, the catch likelihood increases.

### ✔ **Angular convergence toward the anticipated catch point**

Compute angle between defender’s velocity and the ball trajectory:

`angle_defN_ball_alignment = angle(v_defN, vector_to_landing_point)`

Small angle → defender is aligned and able to contest.

---

# ✅ 6. **Time-to-Arrive Features**

Time-to-point-of-arrival is a powerful interaction feature used in sports analytics.

### ✔ **Time to reach ball (assuming straight-line & constant speed)**

`t_rec_to_ball = tgt_rec_ball_dist / tgt_rec_s t_defN_to_ball = defN_ball_dist / defN_s`

Also use:

- `t_def1_to_ball - t_rec_to_ball`
    
- `min_t_defN_to_ball - t_rec_to_ball`
    

This approximates “who will get there first?”

---

# ✅ 7. **Space Contestation Features**

Great predictors when multiple defenders are nearby.

### ✔ **Crowdedness index**

`crowdedness = sum( 1/defN_dist for all defenders )`

Closer defenders penalize catch probability.

### ✔ **Defenders within a radius**

Count defenders within:

- 1 yard
    
- 2 yards
    
- 3 yards
    

These are great categorical features.

---

# ✅ 8. **Separation vs Ball Arrival Timing**

Interaction of two dynamics:

`separation_at_ball_arrival = defN_dist[t_ball_arrival]`

Or:

`trend_separation_until_ball = slope over last 5 frames`

---

# ✅ 9. **Route Geometry Based Features**

Receiver angle/acceleration relative to route and ball trajectory:

### ✔ **Receiver acceleration along route direction**

Capture ability to “burst” toward the ball:

`a_along_dir = rec_accel * cos(angle(rec_dir, route_dir))`

### ✔ **Acceleration toward ball**

`a_toward_ball = rec_accel * cos(angle(rec_dir, vector_to_ball))`

---

# ✅ 10. **Interaction Between Multiple Defenders**

Often catches are determined by **the combination of**:

- nearest defender
    
- “help” defender coming over the top
    

Consider:

### ✔ **Angle between two closest defenders (pincer angle)**

`def_pincer_angle = angle( vector_def1->rec , vector_def2->rec )`

Small angle → they’re converging from similar direction  
Large → bracket coverage (two angles attacking)

---

# ✅ 11. **Ball Trajectory Features**

Even simple ones help:

- Ball height (if available)
    
- Ball horizontal velocity
    
- Angle of descent toward catch point
    

But even in your data:

### ✔ **Horizontal ball distance from landing point**

(tells whether ball is arriving soon)

`ball_dist_change = tgt_rec_ball_dist[t] - tgt_rec_ball_dist[t-1]`

---

# 🧠 Feature Engineering Ranking (What Matters Most)

Based on NFL catch probability modeling literature:

1. **Receiver–defender separation**
    
2. **Closing speed**
    
3. **Time-to-arrive (TTA) difference**
    
4. **Angles of pursuit (alignment with ball)**
    
5. **Receiver acceleration burst**
    
6. **Defensive crowdedness**
    
7. **Receiver–defender relative momentum**
    
8. **Separation trend in last few frames**
    

You already have the core variables; the **differenced features** and **vector-angle features** typically give the biggest jumps in predictive power.