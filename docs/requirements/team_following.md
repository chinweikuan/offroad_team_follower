# Team Following: Requirements and Acceptance Criteria

Follow a beacon-carrying leader through off-road terrain while keeping clear of the other team members.

- Status: Draft
- Last updated: 2026-11-09 (UTC+8)
- Authors: CWK (requirements and design decisions), Claude (formatting and translation)

## 1. Use Case

The robot carries heavy loads for a group of people in natural, off-road environments such as wilderness and mountain forests. It follows a leader who carries a beacon, and it uses its onboard sensors to keep a safe distance from the other team members.

**Why a robot**: once the technology matures, a robot could share the load when a team enters hazardous areas, or let an expedition carry more equipment.

## 2. Team

- **Size**: 1 leader and 3 team members (fixed).
- **Leader**:
  - Walks at a normal off-road pace of about 1.5–3.0 km/h (0.4–0.8 m/s)
  - Sometimes stops
  - Sometimes backtracks, up to about 15 m
- **Team members**:
  - Walk roughly in single file, but spread out locally
  - May overtake one another or the robot
  - May stop to rest
  - May cross paths with the robot
  - Never get more than 30 m from the leader
- **Team rule**: the distance between the leader and the last team member never exceeds 30 m. When the last member stops, the leader stops and waits. This rule belongs to the scripted team behavior, not to the robot.
- **Problems that occur in practice**: the robot may block a team member's path, or move in ways that are hard to read, which disrupts the team members' own decisions.

## 3. What the Robot Knows (Sensing)

| Information | Source | Student (deployed policy) | Privileged teacher |
|---|---|---|---|
| Leader range and bearing | Beacon (assumed perfect, no dropouts) | ✅ | ✅ |
| Which detection is the leader | Match the beacon bearing against camera detections | ✅ | ✅ |
| Team members in view | Camera (60° field of view); individual identity not needed | ✅ | ✅ |
| Team members out of view | Simulator ground truth | ❌ | ✅ |
| Team members' intent (where they are heading) | Simulation script | ❌ | ✅ |
| Terrain | Student: the robot's own terrain map. Teacher: the perfect terrain map | Partial | ✅ |

## 4. Behavior Rules

### 4.1 Priority

When rules conflict, resolve them in this order:

**Safety distance > Not blocking > Team position > Distance to leader**

### 4.2 Rules

| Situation | Robot behavior |
|---|---|
| Moving in the same direction as the leader | Stay 10–20 m from the leader |
| Leader stops (not backtracking) | If already within 10–20 m, stop in place. If farther than 20 m, close to 20 m, then stop |
| Leader backtracks toward the robot | Wait in place, staying at least 1.5 m from the leader |
| Team position | Stay in the middle of the team: at least one person is behind the robot (resting people count) |
| Team members at the back stop to rest | Keep following the leader |
| Distance to anyone | At least 1.5 m |
| A team member approaches the robot while heading toward the leader | Yield and let them pass |
| A team member approaches the robot but is not heading toward the leader | Back away to keep distance, or stop and let them walk around |
| Forbidden | Passing between two team members who are close together |
| Forbidden | Blocking a team member's path |

### 4.3 Definitions

| Term | Definition |
|---|---|
| Moving in the same direction | The leader is moving and is not walking back toward the robot |
| Close together | Two team members less than 3 m apart |
| Heading toward the leader | The angle between a member's direction of motion and the direction from that member to the leader is under 45° |
| Yield | Move aside to 1.5 m from the team member |
| Blocking | The robot is within 1 m of a team member's predicted path over the next 3 s (constant-velocity prediction) |

## 5. Metrics and Acceptance Criteria

### 5.1 Principles

- All metrics are computed from **simulator ground truth**, sampled every 0.1 s, never from the robot's own perception.
- **Hard constraints use absolute thresholds**: safety, passing between team members, and tipping over must meet the threshold no matter what the comparison is.
- **All other metrics are relative**: they are compared against the baseline system.
- Relative thresholds are set only after the baseline has been measured. Thresholds marked "provisional" are reviewed after the first measurements.

### 5.2 Metrics

| Area | Metric | How it is measured | Type | Threshold |
|---|---|---|---|---|
| Task | Fraction of same-direction travel time spent 10–20 m from the leader | Counted only during same-direction travel | Relative | No worse than baseline |
| Task | Fraction of time with at least one person behind the robot | Order by distance traveled along the route; resting people count | Relative | No worse than baseline |
| Task | Correct-stop rate when the leader stops | Within 5 s of the leader stopping, the robot has stopped within 20 m of the leader | Relative | No worse than baseline |
| Safety | Minimum distance to anyone | Robot-caused approaches only (see note 1) | Absolute | ≥ 0.5 m |
| Safety | Total time closer than 1.5 m, per run | Robot-caused approaches only (see note 1) | Absolute | ≤ 5 s (provisional) |
| Social | Passes between two team members | The robot's path crosses the line between two members less than 3 m apart | Absolute | 0 |
| Social | Blocking episodes longer than 3 s | Blocking as defined in 4.3; each continuous stretch over 3 s counts once | Relative | No worse than baseline |
| Terrain | Tip-overs | Body tilt exceeds 60° | Absolute | 0 |
| Terrain | Stuck events | The leader is moving and the robot is not in a stop-and-wait state, but the robot moves less than 0.1 m in 10 s | Relative | No worse than baseline |
| Smoothness | RMS jerk | Second time derivative of linear velocity | Relative | Set after first measurements |
| Smoothness | Steering reversals per minute | Sign changes of angular velocity where its magnitude exceeds 0.1 rad/s | Relative | Set after first measurements |

**Note 1**: a robot-caused approach is one where the robot moves toward that person faster than 0.05 m/s. A team member walking up to a stationary robot is not a violation.

### 5.3 Overall Acceptance

- Every absolute threshold is met in every scenario.
- No relative metric is worse than the baseline, within its confidence interval.
- At least one or two metrics are clearly better than the baseline.

## 6. Test Scenarios

| ID | Name | Use |
|---|---|---|
| S1 | Normal travel | Training and testing |
| S2 | Leader stops | Training and testing |
| S3 | Leader backtracks | Training and testing |
| S4 | One member overtakes | Training and testing |
| S5 | Last member rests | Training and testing |
| S6 | Member ahead of the robot rests | Training and testing |
| S7 | Two members overtake at once | **Testing only** |
| S8 | Unseen terrain | **Testing only** |

- **Generalization**: testing-only scenarios are combinations or variations of situations seen in training. S7 tests generalization in behavior, and S8 tests generalization in terrain.
- **The training/testing split is frozen before any experiment** and is not changed afterward.
- **Runs**: start with 10 runs per scenario per method and compute confidence intervals. If the intervals are too wide to separate the methods, add runs.
- **Methods compared**:
  1. Baseline: a classical system with the same sensors as the student
  2. Privileged teacher
  3. Policy 1: behavior cloning with DAgger
  4. Residual RL on top of Policy 1
- **Estimated cost**: 8 scenarios × 10 runs × 4 methods = 320 runs. At about 4 minutes per run, that is about 21 hours.
- **Map**: at 0.6 m/s for 4 minutes, the leader's route is about 150 m long. The map must be large enough for that. Simulation speed with CPU rendering on a map this size must be checked before implementation.

## 7. Hardest Cases

The leader or team members do something that was not anticipated in training.

## 8. Out of Scope and Assumptions

- A physical robot (Unitree Go2) and physical beacon hardware (UWB)
- Quadruped locomotion: a differential-drive stand-in is used
- Team members follow scripts and do not react to the robot
- No walking animation
- The beacon is assumed perfect, with no noise and no dropouts
- No weather or lighting changes

All of these will be listed in the final limitations section.

## 9. Open Items

- [x] Tip-overs: absolute, 0
- [x] Passing between team members: absolute, 0
- [ ] Review "total time closer than 1.5 m ≤ 5 s" after the first measurements
- [ ] Set relative thresholds after the baseline has been measured
