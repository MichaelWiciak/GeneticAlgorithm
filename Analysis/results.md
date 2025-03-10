Note that for latter exercises, I went back to the FFN and Angel Sensor approache back.

Question 1:

Part 1:
Both average and best fitness remain relatively stable over many generations (from ~300 onwards), so the algorithm has likely found an optimal solution or reached a local optimum.
That is what it looked like visually after testing it up to 2000 generations.
From ~190 onwards up to ~230 the GA for both average and best fitness was consistently improving. However, later on, there was no improvements observable up to 2000 generations to its fair to assume it has plateaued.
So the best graph will stop at 250 generations
There are some peaks of best fitness and as such the average increases for that specific point but a general trend is not observable.
Running more generations beyond convergence doesn’t provide meaningful improvements and just wastes resources.
Stopping when fitness stagnates ensures efficiency so its important to consider computational cost.

(spikes because sometimes they get lucky, a lot of cheese on the path of a mathematical curve they are following)

Part 2: Describe the behaviour of the mice as it evolves over evolutionary time.

Intially, the mice move in come kind of mathematical equation/curve way and are trying to collect cheese/yellow dots. However, they are just following a path and dont take into account the cheese/modify their pathing based on the nearest cheese. So they collect cheese only by chance.

As the generations progress, the mice get better at collecting cheese and the average fitness increases. This is because the mice are now actively looking for the cheese and trying to travel to the closest cheese relative to their position. There is now fierce competition between the mice to get to the cheese first, often many mice go for the same cheese because it is the closest to them.
The mice are now actively trying to get to the cheese and are not just moving randomly. They are now following a path to the cheese and are trying to get to the cheese, the speed of them is variable.

Part 3:
For both, why i am doing these experiments, whats my design, quantify the performance (graphs again) and decide if its worth it. what are the advantages and disadvantages of it?
Put in the seeds and parameter values.
Keep seed the same 1. !!!

a) so for this modify sensor in evo mouse/combine th etwo/ experiment with parameters.

\subsection{Reasoning and Design}

Wanted to change the sensors to be like sensors of an actual mouse (two eyes) as seen in The-mouse-and-human-visual-systems-share-basic-similarities-but-differ-in-complexity.png (load it in the latex, it is available). As seen in figure (the mouse vision png) Each eye has vision of around \( 140^\circ = \frac{7\pi}{9} \) radians. The mice have a smaller field of common vision (both eyes see it at the same time) comparred to humans so wanted a sensor where the mouse would prioritise the cheese in the middle of its vission (where both sensors overlap) and then go to the cheese. This is a more realistic approach to how a mouse would behave in real life. If only one sensor sees the cheese, it would go to that cheese. To achieve the same orientation of the eyes as in the figure, after trial and error -0.8 and 0.8 were the best values for the sensors (right and left respectively).

As such, the result looked like this (sensor1.png) for just the left sensor and (sensor2.png) for both sensors.

\subsection{Implementation}
To implement this, I added a `This.Add("Eyes", CombinedEyeSensor<Cheese>(2.44, 1000.0, 0.8));` a combined sensors in the initialisation of class EvoMouse in Mouse.cc.

In sensor.h, the sensor was defined as
`template <class T>
Sensor* CombinedEyeSensor(double scope, double range, double orientation)
{
// Create the left and right sensors with the specified parameters
Sensor* left = new BeamSensor(scope, range, Vector2D(0.0, 0.0), orientation);
Sensor\* right = new BeamSensor(scope, range, Vector2D(0.0, 0.0), -orientation);

    // Set the matching function for both sensors
    left->SetMatchingFunction(new MatchKindOf<T>);
    right->SetMatchingFunction(new MatchKindOf<T>);

    // Create the evaluation function combining both sensors
    SensorEvalFunction* evalFunc = new EvalNearestCombination(left, right, range);

    // Set the evaluation function for both sensors (assuming the sensors can share the same eval function)
    left->SetEvaluationFunction(evalFunc);
    right->SetEvaluationFunction(evalFunc);

    left->SetScalingFunction(new ScaleLinear(0.0, range, 1.0, 0.0));
    right->SetScalingFunction(new ScaleLinear(0.0, range, 1.0, 0.0));


    return right;

}`They used the`BeamSensor` which act similar to eyes (limited sight range and only at specific orientations).

For visual purposes, both sensors cant be displayed at the same time but both sensors are included in the evaluation fuction and the evaluation function will include data from both sensors when making a decision.

The evaluation function is defined as follows in sensorfunctions.h:
`class EvalNearestCombination : public SensorEvalFunction {
	public:
		EvalNearestCombination(Sensor* leftSensor, Sensor* rightSensor, double range)
			: leftSensor(leftSensor), rightSensor(rightSensor), range(range),
			  nearestSoFar(range), bestCandidate(NULL) {}
	
		virtual ~EvalNearestCombination() {}
	
		virtual void Reset() {
			bestCandidate = NULL;
			nearestSoFar = range;
			bestCandidateVec = Vector2D(0, 0);
		}
	
		virtual void operator()(WorldObject* obj, const Vector2D& loc) {
			double leftDistance = (leftSensor->GetLocation() - loc).GetLength();
			double rightDistance = (rightSensor->GetLocation() - loc).GetLength();
	
			// Check if the object is within the range of both sensors (common vision)
			bool visibleToLeft = leftDistance <= range;
			bool visibleToRight = rightDistance <= range;
	
			if (visibleToLeft && visibleToRight) {
				// Prioritize objects visible to both sensors (common vision)
				if (leftDistance < nearestSoFar && rightDistance < nearestSoFar) {
					nearestSoFar = std::min(leftDistance, rightDistance);
					bestCandidate = obj;
					bestCandidateVec = loc;
				}
			} else {
				// If not visible to both, consider the nearest object seen by either sensor
				if (visibleToLeft && leftDistance < nearestSoFar) {
					nearestSoFar = leftDistance;
					bestCandidate = obj;
					bestCandidateVec = loc;
				} else if (visibleToRight && rightDistance < nearestSoFar) {
					nearestSoFar = rightDistance;
					bestCandidate = obj;
					bestCandidateVec = loc;
				}
			}
		}
	
		virtual double GetOutput() const {
			return nearestSoFar;
		}
	
	private:
		Sensor* leftSensor;
		Sensor* rightSensor;
		double range;
		double nearestSoFar;
		WorldObject* bestCandidate;
		Vector2D bestCandidateVec;
	};`

The evaluation function prioritises objects visible to both sensors (common vision) and if not visible to both, considers the nearest object seen by either sensor. The output of the evaluation function is the distance to the nearest object.

\subsection{Performance}

As seen in (question3a.png), the performance is pretty bad and there doesnt seem to be any consistent improvement over the initial setup.

For the final generation simulated I got
`Generation:    818   Average fitness: 0.00140647   Best fitness: 0.00706239`

But even though my design seemed to have been implemented correctly, the mice get instead continue going in a straight line as fast as possible and because of their speed, it makes it very difficult to turn. If there is a unique cheese in sight of left sensor and another uniuque cheese in sight of right sensor, because of the speed of the mice, the mice widdle left and right until they pass both the cheeses and nothing gets picked up and as such the system prefers really fast ones as they might occasianlly randomly bump into the cheese. So it seems multiple sensors that overlap causes issues in the system. that might be a design flaw in the simulation or requires a more robust evaluation function.

Same issue happepend with two same `ProximitySensor` if there was an area of overlapping (so it used the default evaluation function of `EvalNearest`). Increasing the range from the default of `200.0` to `1000.0` did not help.

Testing this with two `ProximitySensor` but no overlapping, so two proximity sensors but same size/parameters and results, it was the same. So I decided to swtich to just 1 proximity sensor with
`ProximitySensor<Cheese>(1.5, 2000.0, 0)`
But it results in (question3a2.png), which reflects the same results of my combinedSensor.

\subsection{Advantages and disadvantages}

Using the beamSensor seems to make the program much slower than when using the `NearestAngleSensor`.
The mouse no longer make sharp u-turns and instead proritieses going forward, which makes the simulation more realisic as the mouse doesnt have a hidden sense where each cheese is unless it sees it.
However, the mice are still not optimal in their pathing and prefer going fast and straight, which might not be ideal what a mouse would do in a real environment.

The beamSensor seems clearly inferior to the `NearestAngleSensor` in terms of performance and efficiency, so it is not worth here unless evaluation function is improved.

b)

\subsection{Reasoning and Design}
Currently, the simulation is using `This.InitFFN(4)` meaning a feedforward network with 4 hidden layers. The simulation allows for the usage of Deep Neural Networks (DNN) which in theory ... (add details here and how that is relevant to the mouse example).
\subsection{Implementation}
To implement this I changed the initialisation of the EvoMouse in Mouse.cc to `This.InitDNN(x);` which initialises a DNN with x hidden layers. The number of inputs and outputs are the same as the FFN, so the DNN has 2 inputs and 2 outputs which the program defaults to if they are not specified.
To allow for this, I had to change from which class `EvoMouse` inherits from: `class EvoMouse : public  EvoDNNAnimat`
and change the `OnCollision` function to include `EvoDNNAnimat::OnCollision(obj)` instead.
\subsection{Performance}
Having tested the DNN with 2,4,20 hidden layers, it is clear that 2 layers did the best job. It is clear that more layers seems to make it overfit and not actually better. It will still approximate the results of inly using 2 layers but after a lot more generations. So it is not worth it to use more layers than 2.
(q32layer.png) (q34layer.png) (q320layer.png).

\subsection{Advantages and disadvantages}

The network takes longer to train that the FFN. DNN also seem to make the mouse stop sometimes to reasess the situation, which is not ideal, it slows it down, but less layers reduce this.
They go in circles and wait a little bit, 1 second and then go to the nearest cheese, they dont seem optimal, and take longer time to trian to approximat ethe same FNN results.
The origianl FFN seemed to have make it easier to and quicker to train and have smoother paths.

However the main advantage is that the DNN achieves better efficiency (higher fitness) than the FFN, so it is worth it to use the DNN over the FFN if you are willing to train for longer. We approxiame 0.1 with 2 hidden layers in DNN while in FNN, we get to ~0.008 maxium.

4.

The fitness function found in `mouse.cc` `GetFitness()` measures the efficiency of the EvoMouse in collecting cheese. It is defined as the number of cheeses collected divided by the total distance traveled. This encourages the mouse to find cheese efficiently rather than moving randomly and expending unnecessary energy.

In LaTeX:

\[
F = \begin{cases}
\frac{C}{D}, & C > 0 \\
0, & C = 0
\end{cases}
\]

where:

- \( F \) is the fitness score,
- \( C \) is the number of cheeses collected,
- \( D \) is the total distance traveled.

The mice in the default sensors often collide with other mice which slows them down, i want to penalise that. Also, i want to encourage them to find cheese quickly so they would optimise their speed if needed, to reach the cheese as fast as possible, not just blidnly getting it.

### 1. **Efficiency with Time Factor**

- Encourage the mouse to collect cheese quickly by incorporating time into the function.
- **Fitness function:**  
  \[
  F = \frac{C}{D \cdot T}
  \]
- \( T \) is the total simulation time. This penalizes slow mice.

### 3. **Avoiding Obstacles**

- Penalize mice that collide with obstacles while still rewarding cheese collection.
- **Fitness function:**  
  \[
  F = \frac{C}{D + O}
  \]
- \( O \) is the number of obstacle collisions.

I decided to combine both into positive feedback and negative feedback, postivei for fidning it quickly and negative for obstacles.

\[
F = \frac{C}{D \cdot T} \times \frac{1}{1 + O}
\]

where:

- \( F \) is the fitness score.
- \( C \) is the number of cheeses collected.
- \( D \) is the total distance traveled.
- \( T \) is the total simulation time.
- \( O \) is the number of obstacle collisions.

The result of performance:
There are a lot less crashes, the nodes slow down when close to another mice, there is still the same competetivness where they sprint to the cheese but they actively change their speed to not crash and try to get cheese as fast as possible, allowing for their speed to be very variable, which is a good improvement.

As can be seen in graphs (fitness.png), the fitness function deteriorates the performance of the mice as their fitness is lower than the default fitness function.

5.  One idea is to output how many cheese get collected per generation as a percentage of total cheese, so we can see what numbers of total cheeses each fitness function leads to.

Another more interestin gidea is to do a Measure Convergence Rate (step up from the simple idea) where we measure how many generations it takes to reach a performance threashold (80% of cheese collected).
The faster we congerge, the better.

This allows for an objective comparison of the fitness functions and their performance in terms of convergence rate, as what we care about, i assume, that as much cheese as possible in the environment is collected.

Implementation:
equation, code etc.

Note: can include those metrics into graphs?, cant.

However...

6. collective behaviour:
   Collective behabiour refers to the phenomenn where individual agents who are independend and each have
   simple rules they follow,through simple interation exibit complex,coordinates behaviour on a group level. The behaviour of the agentsi isn't explicity programmed or directed by any central authority. As such, to count something as collective behaviour, we need to have local interaction between agents,decentralised control, emergence of complex patterns,adaptation.

   The mice in the simulation attempt to get as much cheese as possible and since there is fix amount of cheese, there is competition for which mice will survive until the next generation. The mice often collide with each other and steal the cheese from different mices that are also going for the same cheese. They will take the most efficient paths to reach the cheese but if a different mice takes their cheese before it arrives, then the mice recalculates for which cheese it will go for but it doesnt take into account other mice.

   The mice do not satisfy many of the requireemtns of collective behavoiur. The mice do not interact with one another and have no way to leave any communication like pheromones behind. They seem to be not aware of the other mice's location or for what target cheese each is going for. The mice dont have a central authority and act independetly, so they satisfy this.There is adaptation over the generations of the mice, they do get better at finding the most optimal paths to get the cheese.However, there is no emergence, there are no interactions between the mice and they dont form any complex patterns as other mice are not taken into account. However, whe nyou do change the fitness function to what I had in Question 4, the mice try to avoid collisions and slow down if they see a mouse in the trajectory, so they do adapt to the environment and the other mice.

Question 2:

7. it is just a plots. with justrificaations

8. Describe the behaviour of the agents. Is this an example of co-evolution? Argue. You should
   explain your reasons and evidence your answer (for co-evolution or for its absence) based on the
   behaviour of the agents after different numbers of generations.

At 10 generations, prey standing still and the predators randomly getting to them, but eventyally, the mouse run away and predators start to chase at 50 (prey arent the best and often go so fast that they crash into nearby wolfs) but at 100 and beyond, they run away from a predaror but stay still so they can stay away as far as possible from any predators, so they into account not only the predator that is chaging them but also other predators around it.

For this example, since it is prey and predator, this could be an example of Competitive co-evolution. The two populations are in competition and the success of one population depends on outcompeteing the other (they are competeing for limited resources of each other). However, for that to be true, they both have to evolve in response to each other’s changes in a way that drives the adaptation of both. This makes a cycle of adaptation.

The prey is definitely adapting and improving but the predator seems to be the same over time, just chases the prey in a straight line. So it is not a clear example of co-evolution as the predator is not adapting to the prey's changes.

9. coevolution in graph ->

Judging from the fitness plots for the predator and prey simulations (Figure \ref{fig:predator} and \ref{fig:prey}), there is clear evidence that the prey population is improving consistently over time. In all three simulations, the prey's fitness continues to increase, and the plots show a convergence to similar values. This indicates that the prey are adapting and becoming more effective at avoiding the predators.

However, when we look at the predators over time, the fitness appears to be decreasing. The average fitness for the predators shows a noticeable decline, with fitness spikes becoming lower and lower as the generations progress. This suggests that the predators are not evolving in response to the prey's adaptations. There are no sudden improvements or changes in the predator’s strategy that would indicate adaptation to the prey’s behavior.

Therefore, this does not present a clear example of co-evolution.

10. intelligent behavuiour??

In Intelligence without Representation by Rodney Brooks, intelligence is defined as the ability to achieve goals in the world. Brooks argues that intelligence does not require high-level symbolic representations or abstract thinking, but instead, it emerges through direct interaction with the environment.

In a predator/prey situation, Brooks suggests that both the predator and the prey must have special characterisitcs.
The predator must have the ability to track and capture the prey, which involves sensing, reacting to the environment, and adapting behavior to ensure successful hunting.
The prey must be capable of sensing danger and evading capture, requiring an ability to detect the predator and take evasive actions. Both the predator and the prey must exhibit adaptive behavior for the agent's behaviour to be considered intelligent.

Notes: 10 both predator and prey are going in a circle, not catching each other.
Note: 50 the predators sometimes go towards tehh mouse, but not all of them, the rest sometimes go and sometimes stay in the same place, in a circle, like it is partolling. The prey tryto travel fast in straight lines or in circles to avoid prey, some seems to purposely avoiding the predators if they get too close.

Note: 100:
All prey are running away from the predators and all predators are chasing the mice, the prey just does circles and since the prey is as fast as the predators, the predarors never catch up unless a special where a different predator appears in its path, the prey doesnt react in time and gets caught. the predators are just following thep rey in a straight line to the nearest mice they see.
