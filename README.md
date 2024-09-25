<h1>Euclidian TSP with introduced human error.</h1>

<h2>Description</h2>
The Travelling Salesman Problem (TSP) defines a situation where a ‘salesman’ needs to visit 𝑛
number of cities via the shortest route possible, starting and ending at the same city. This is normally found using heuristic algorithms, in this case the simulated annealing (SA) algorithm will be used to find the most optimal tour.

However, the travelling salesman problem seems to occur in a perfect world that does not account
for any problems or external factors that may occur, thus, this report will be exploring how adding
said factors can affect how the most optimal tour is found, if it ever is. This will be achieved via the introduction of stochasticity in the form of weather states and human error. While the simulated annealing algorithm is already an example of the use of Monte Carlo, the addition of further randomness will allow us to analyse how a problem like this set in the real world could turn out. Furthermore, adding an unpredictable factor such as weather that affects road length into the mix will extend the model in a more realistic way and allow us to determine whether heuristic algorithms such as simulated annealing are able to still reach the most optimal tour.
<br />


<h2>Languages and Utilities Used</h2>

- <b>Python</b> 
- <b>Jupyter</b>

<h2>Program walk-through:</h2>

- [Full report](https://github.com/cglandeira/Euclidian-TSP/blob/main/Ecludian%20TSP.pdf)
- [Marvov Chain code](https://github.com/cglandeira/Euclidian-TSP/blob/main/Markov%20Chains.ipynb)
- [Monte Carlo code](https://github.com/cglandeira/Euclidian-TSP/blob/main/Monte%20Carlo.ipynb)


<h2>Model Description:</h2>
The travelling salesman problem can be of any size but for most cases in this report, unless specified otherwise, we will be using 4 different cities numbered 0, 1, 2 and 3. With cost from once city to the other as follows:

<p>
  
|   | 0  | 1  | 2  | 3  |
|---|----|----|----|----|
| 0 | 0  | 10 | 15 | 20 |
| 1 | 10 | 0  | 35 | 25 |
| 2 | 15 | 35 | 0  | 30 |
| 3 | 20 | 25 | 30 | 0  |

</p>
<p>
  The ideal tour is then found to be [0, 1, 3,2] (with cost of 10 + 25 + 30 + 15 = 80), which can also be written in any other way in which the loop remains in the same order.
</p>

<p>
  While not many there are some assumptions:
  
- Each set of the 𝑛 number of cities is called a tour.
- While tours are written as [0, 1, 2, 3] it is assumed that the salesman starts and ends at the first city (so four distances are added up).
- The salesman does not repeat cities in a singular tour.
</p>
<p>
  As discussed previously this is the ‘simple’ version of the travelling salesman problem, which accounts for a static environment that will always remain the same (road costs will always remain the same) and does not consider human error. Our extended model, however, does take into account all these factors. So, there are a few more assumptions:

- The weather has three different states it can exists as:
    - 𝐹 – Fine with probability 𝑃(𝐹). In which road cost stays the same.
    - 𝑅 – Rainy with probability 𝑃(𝑅). In which certain roads (from any city to 1 and from any city to 2) not prepared for the rain get their cost multiplied by 1.5. Making the ideal tour in a rainy environment be [1, 3, 2, 0] -> 100.
    - 𝑆 – Snowy with probability 𝑃(𝑆). In which certain roads (from any city to 0 and from any city to 3) not prepared for the snow get their cost multiplied by 2. Making the ideal tour in a snowy environment be [3, 2, 0, 1] -> 120.
- Given the simulated annealing algorithm, whenever a new solution is processed there is a 50% chance of the solution being rejected due to human error (this will be applied with and without the weather states).
</p>
<p>
  In order to simulate the weather states, we’ll be using Markov Chains, and in turn transition matrices, which there are a few of. First, we’ll start with a world in which once it’s a Fine day the state will not change (not very realistic). Rain and Snow both have the same chance of staying the same than to transition into a different state. If transitioning, they have a lower chance of transitioning into a Fine day.
</p>
<p>
  So, the transition matrix is as follows:

|   | F   | R   | S   |
|---|-----|-----|-----|
| F | 1   | 0   | 0   |
| R | 0.15| 0.5 | 0.35|
| S | 0.15| 0.35| 0.5 |

</p>
<p>
  And in canonical form (Fine is an absorption state and the rest are transient.):

|   | S   | R   | F   |
|---|-----|-----|-----|
| S | 0.5 | 0.35| 0.15|
| R | 0.35| 0.5 | 0.15|
| F | 0   | 0   | 1   |
  
</p>
<p>
In the above matrix while there is a low chance of ending on a Fine day, once it is the model essentially becomes the same as the base model. So, in order to further test our extension, we will also make use of an ergodic transition matrix for a more realistic model, such as:

|   | F   | R   | S   |
|---|-----|-----|-----|
| F | 0.90| 0.05| 0.05|
| R | 0.15| 0.5 | 0.35|
| S | 0.15| 0.35| 0.5 |
  
</p>
<p>
  Where the chance of it being a fine day is still high, but now it can transition from Fine to any day, thus making it more real world like.
</p>
<p>
  As for the Monte Carlo implementation simulated annealing already has one, in which you start with a starting tour, perturb this solution, and if this new solution’s cost is less than the previous accept otherwise you accept if the exponent of the cost of the new solution minus the cost of the previous one over the current temperature is less than some random number. So:
</p>
<p>
  
```math
\text{Accept with probability } 
\begin{cases} 
1 & \text{if } f(\tilde{x}) > f(x_i) \\ 
\exp\left(\frac{f(\tilde{x}) - f(x_i)}{T}\right) & \text{otherwise} 
\end{cases}
```  
</p>
<p>
  However since we are adding a 50% chance of human error we will be first adding this twice (as humans can sometimes make a lot of mistakes) first if the cost of the new solution is better than the previous one, it also has to pass that 50-50 test, and if its not instead of the second equation a different random number is drawn and if its less than 0.5 it will be accepted. In this case, we will be using the following cities to simulate a larger model:

|   | 0    | 1    | 2    | 3    | 4    | 5    |
|---|------|------|------|------|------|------|
| 0 | 0    | 2053 | 1155 | 3017 | 1385 | 1456 |
| 1 | 2053 | 0    | 1080 | 3415 | 939  | 2876 |
| 2 | 1155 | 1080 | 0    | 3940 | 285  | 856  |
| 3 | 3017 | 3415 | 3940 | 0    | 3975 | 1984 |
| 4 | 1385 | 939  | 285  | 3975 | 0    | 3278 |
| 5 | 1456 | 2876 | 856  | 1984 | 3278 | 0    |
  
</p>
