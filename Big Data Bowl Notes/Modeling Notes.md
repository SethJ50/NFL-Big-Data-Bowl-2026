#### What is XGBClassifier?
- Simply the XGBoost model considered for binary classification, answering P(y = 1 | features)

#### Overtuned Model Notes
- Giving the model the same label for every frame means the model is being trained to learn: "Given these features, how likely is it that the overall play ended in a completion?"
- When the model is deep and strong, it learns near perfect separation between completed play versus incomplete play, even in early frames
- The issue is a fundamental mismatch between what the label represents and what the model is being asked to predict
- Label is not frame-level, so model learns play outcome, not frame outcome

#### Why a weak / regularized model avoids collapsing to 0/1?
- A shallow or highly regularized model cannot perfectly separate completion frames from incompletion frames so it outputs intermediate probabilities since it is more uncertain - which feels more realistic frame-by-frame
- Just because it seams realistic doesn't mean the model understands frame level uncertainty, its just because the model is being prevented from fully learning the data structure, we are seeing a side effect, not true temporal reasoning

#### How does flight_normalized_time help?
- With features like flight_normalized_time, distance_to_ball, and tgt_rec_t_to_ball, the model can learn something like "Even if the receiver has great separation early, the chance of completion is still not as high as when separation is the same at frame 0.9"
- This isn't 'temporal sequencing', it is 'feature conditioning'
- Model can learn rules like:
	- Early in the route, if far from ball, thats okay, separation isn't decisive
	- Late in the route, far from ball is bad and separation matters much more
- These time-based and spatial features don't fix the label mismatch, but the help the model output smooth-ish time-conditioned probabilities even though the label is constant across frames

#### Model Learning: Theoretical
- Model learns P(Play was completed | features in this frame) rather than P(completion happens at this frame | features in this frame)
- Fluctuations you see are not true completion probabilities, but moreso time-conditioned signals of "how much this frame resembles a frame from a completed play"

#### Model Feasibility
- Approach is reasonable if you accept that the model is producing "A per frame completion-likelyhood indicator based on how similar that frame's geometry looks to typical completed play frames"
- This is useful for visualization, understanding separation influence, building intuition, or feeding another model later...
- Is NOT a literal per-frame probability of completion

#### How flight_normalized_time helps theoretically
- Model can learn a phase-conditioned mapping like P(completion | features, time) which is why your probabilities look more realistic early and sharpen late

