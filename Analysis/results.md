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

Wanted to do the eyes like actual mouse, so 120 degrees horizontally, which is x radians.
Because...
Each eye done separately. 180 degress so pi. 

But even though that seemed to have been implemented correctly, they get confuse and instead continue going in a straight line and system prefers really fast ones so it seems multiple sensors that overlap causes issues in the system. that might be a design flaw in the simulation. 

same issue happepend with two same proximity sensors if there was an area of overlapping, so this implementaion seemed to be hard.

I done the same but no overlapping so two proximity sensors but same size/parameters and results:


After reducing the vision, seemed better/worse...

Big range/unsure mouse? Check...

b)
It used a This.InitFFN(4); feedforward which should be worse, did 4 layers. Same sensors, 
More Layers 20, 50. 
More layers seems to make it overfit and not actually better. 
They go in circles and wait a little bit, 1 second and then go to the nearest cheese, they dont seem optimal, and take longer time to trian to approximat ethe FNN results. 

But when trained on same hidden layers 4, it seems very good, still stops but seems fast and optimal. 

Tried reducting layers to 2 since tha twas a postivie trend it seems. 


4)
// The EvoMouse's fitness is the amount of cheese collected, divided by
	// the power usage, so a mouse is penalised for simply charging around
	// as fast as possible and randomly collecting cheese - it needs to find
	// its target carefully.
	virtual float GetFitness()const
	{
		return This.cheesesFound > 0 ? static_cast<float>(This.cheesesFound) / This.DistanceTravelled.as<float>() : 0;
	}

 Describe this mathematically and in plain words. 

The fitness function measures the efficiency of the EvoMouse in collecting cheese. It is defined as the number of cheeses collected divided by the total distance traveled. This encourages the mouse to find cheese efficiently rather than moving randomly and expending unnecessary energy.

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


I am changing the fitness function to...

The mice in the default sensors often collide with other mice which slows them down, i want to penalise that. 

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


5) 
One idea is to output how many cheese get collected per generation as a percentage of total cheese, so we can see what numbers of total cheeses each fitness function leads to.

Another more interestin gidea is to do a Measure Convergence Rate (step up from the simple idea) where we measure how many generations it takes to reach a performance threashold (80% of cheese collected). 
The faster we congerge, the better. 



Question 2:
Sa) the the behaviour of the system after 10, 50 and 100 generations and base the answers on that? see how it changes. 

Mouse behaviour changes....
