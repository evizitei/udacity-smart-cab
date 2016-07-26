# Smartcab Submission Report


## After Random Action implementation

#### QUESTION: Observe what you see with the agent's behavior as it takes random actions. Does the smartcab eventually make it to the destination? Are there any other interesting observations to note?

So far, the cab (as expected) moves about at random.  Sometimes it happens
upon a waypoint, and I imagine with sufficient running time and no obstacles
it will always _eventually_ hit the waypoint given infinite time,
but there's also no assurance that it will do so within any finite timeline
no matter how large you make it.

## After State Storage

#### QUESTION: What states have you identified that are appropriate for modeling the smartcab and environment? Why do you believe each of these states to be appropriate for this problem?

I believe the things that should go in local state for the agent should be bits
of data that are useful for deciding what to do next.  In addition to the desired
direction of travel from the planner, almost every input qualifies
with the exception of what any car to the right in the current intersection is
planning to do.

I can't think of a scenario in which we care about that.  If we want to go straight,
then if the light is green we can go, and if red we can't.  If we want to go right,
then on green we just go, on red we care what the car from the left wants to do.
If we want to go left, then on red we stop, and on green we care what the oncoming
car wants to do.  In no case do we care about the intentions of the car to the
right (unless that car is not a reliable rule follower, something we might
actually want to consider in a more realistic simulation).

We also don't really care about deadline, since regardless of how long is left on the clock
we still want to take the optimal action at each step (we would not want to make a car that
would break safety laws when in a hurry, this wouldn't be a lot better than a human
driver).

#### OPTIONAL: How many states in total exist for the smartcab in this environment? Does this number seem reasonable given that the goal of Q-Learning is to learn and make informed decisions about each state? Why or why not?

This seems like a combinations problem, though I don't know by heart the formulas
for arriving at a solution without thinking it through.  If we just care about
the intended direction of travel, then there are 3 states (since as long as
we aren't on the waypoint right now we will want to travel in _some_ direction).
We also care about the color of the light, which means we can be in 2 light states
(red or green) for each direction we might want to travel, 6 possible states.  Now
we add in the state of oncoming traffic, for which there are 4 states (often
there is no car there resulting in "None"); this is also independent which
means 4 options for each of the existing 6 states, or 24 possible states.  Then
finally we can account for the state of the car to the left, which can be in any
of 4 states, once again independent of any other data point, so multiply by 4 again
to result in 96 possible combinations.  This is a lot more than many of the simple
examples, but it's still tractable even on an underpowered piece of hardware
to keep track of state/action values for 96 possible states.  The number of
possible actions is 4 ("None" being an option if the safety circumstances of
the current environment prevent taking the desired action immediately), so that
means we have 96 states and 4 possible actions, or 384 state/action pairs.

## After Q-Learning stage 1

#### QUESTION: What changes do you notice in the agent's behavior when compared to the basic driving agent when random actions were always taken? Why is this behavior occurring?

At first the behavior is not very different at all, for at least the first episode or two.
Quickly the agent begins to manage to get to the waypoint before the deadline,
though at least for this iteration it's behavior is a bit strange (more on that
  in a minute).  Over time the agent also begins to drive more safely and legally,
  though not always.  This is occuring because it examines it's history of behavior
and what sort of rewards have occured when it's taken actions in given states before.
When deciding whether or not to go "forward" when the light is red, it notices
that it's been penalized for this before and that taking the "None" action in
this state is better, for example.

This is not totally as I'd expect, though.  For one thing, the agent appears
to prefer to keep driving rather than hold still; so if it can't drive forward like
it would prefer because of a red light, but it _can_ turn right, it seems
to do so even though it will get hit with a -0.5 reward.  I suspect this is because
I've set the discount rate for future rewards to be 0.8, and it's valuing the future
2.0 rewards it will get for driving back towards the waypoint too highly.  I think
I should reduce the discount rate signficiantly.

Additionally, it still takes illegal actions sometimes because I'm using an
exploration value of 10% and so even when it knows the right thing to do, 10%
of the time it takes a random action, often an unsafe or illegal one.  I think
I need to have some exploration path that doesn't just pick a random value; maybe
we start with a higher epsilon value and reduce it to 0 over time by reducing it
each time the exploration path is hit by a tiny amount.
