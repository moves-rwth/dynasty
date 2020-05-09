This file provides a more detailed introduction to using Dynasty. Before reading it, the very first overview of the tool, including instructions how to install it, can be obtained in [README](https://github.com/moves-rwth/dynasty/blob/master/README.md).

# Feasibility Analysis

Given a sketch, i.e., a probabilistic program with holes, and a specification of properties that the complete program should have, the goal of feasibility analysis is to find an instantiation of the holes such that the induced program satisfies the desired properties. All methods presented here except the evolutionary algorithm are complete, i.e., if the there is no feasible instantiation, the algorithms eventually report so.

## Emulation of a six-sided die

Let us consider an emulation of a fair six-sided die with a fair coin. This of course can be achieved when the coin is tossed repeatedly until the certain final state is reached. We encode the problem using our PRISM-based sketching language into three files, which form compulsory input for the tool and are introduced below.

TODO: picture of the algorithm

#### Template file (*.templ)

Template or also sketch is the main part of a problem specification. It encodes the whole logic of an algorithm and contains holes to be filled with suitable option values. Below the variable **s** represents a state of the emulation process and **d** the generated number on the die. Starting in state 0 the coin is tossed with 50% chance to pick one of the states 1 or 2. For either of these states the next step is going to be synthesized as represented by undefined constants x3-6. As soon as state 7 is reached the emulation ends and when this happens the value of variable **d** is set with the generated number on die.
```
dtmc

const int x3;
const int x4;
const int x5;
const int x6;

module die
        // local state
        s : [0..7] init 0;
        // value of the dice
        d : [0..6] init 0;

        [] s=0 -> 0.5 : (s'=1) + 0.5 : (s'=2);
        [] s=1 -> 0.5 : (s'=x3) + 0.5 : (s'=x4);
        [] s=2 -> 0.5 : (s'=x5) + 0.5 : (s'=x6);
        [] s=3 -> 0.5 : (s'=1) + 0.5 : (s'=7) & (d'=1);
        [] s=4 -> 0.5 : (s'=7) & (d'=3) + 0.5 : (s'=7) & (d'=2);
        [] s=5 -> 0.5 : (s'=2) + 0.5 : (s'=7) & (d'=4);
        [] s=6 -> 0.5 : (s'=7) & (d'=6) + 0.5 : (s'=7) & (d'=5);
        [] s=7 -> 1: (s'=7);
endmodule
```

Notice that one has to be careful about potentially ill-formed sketches. The checks performed are not necessarily sufficient.

#### Options for holes (*.allowed)

In this file we define final sets of possible values for each hole (undefined constant in the template). Each line contains a hole name followed by assignable values separated by semicolon.
```
x3;0;1;2;3;4;5;6;7
x4;0;1;2;3;4;5;6;7
x5;0;1;2;3;4;5;6;7
x6;0;1;2;3;4;5;6;7
```
#### Properties (*.properties)

This file contains specification of reachability properties in this case and generally also of expected rewards. The properties below ensure fairness of the six-sided die emulated by the algorithm.
```
P>= 0.16 [F s=7 & d=1]  
P>= 0.16 [F s=7 & d=2]  
P>= 0.16 [F s=7 & d=3]  
P>= 0.16 [F s=7 & d=4]  
P>= 0.16 [F s=7 & d=5]  
P>= 0.16 [F s=7 & d=6]
```

### Running the example

The program can be run by specifying the project directory and necessary files in the command line or using the configuration file. The introduced example has multiple properties and thus we can only use evolutionary search (ea) or CEGIS (cegis), which support this.

**All options specified on command line:**
```
python3 dynasty.py  --project examples/die/  --sketch die.templ  --allowed die.allowed  --properties die.properties ea
```
**All options specified in [configuration file](https://github.com/moves-rwth/dynasty/tree/master/examples/die/die.cfg):**
```
python3 dynasty.py --config examples/die/die.cfg
```
**Command line options override the specification in configuration file:**
```
python3 dynasty.py --config examples/die/die.cfg cegis
```
#### The result
At the end of the output can be found the analysis result
```
Satisfiable!
using x3: 6, x4: 3, x5: 4, x6: 5
```
that contains the assignment to each hole and thus providing an instance of the program that correctly emulates fair six-sided die with fair coin (in case you get different result, there are four possible solutions of this particular example :alien:).

If we tweak a bit the probability for instance *P>= ~~0.16~~ [F s=7 & d=6]* to *P>= 0.20 [F s=7 & d=6]*, the analysis result is
```
Unsatisfiable!
```
as there is no existing instance of the sketch that would satisfy the objectives.

## Grid (POMDP)

This example models a 4x4 grid with a goal state where we want to synthesize finite state controller that will reach the goal state in the least expected time.

### Running the example

All undefined constants have to be specified either as holes in _*.allowed_ file or directly using command line option --constants. The latter is very common, since the constants can be also used as thresholds in property specification (*.properties). Constant CMAX denotes the upper bound for the number of steps to the target and the rest are thresholds used in the property files. We can run this example with cegis, cegar(lift) and evolutionary algorithm (TODO: other types of synthesis alg.).

**Synthetize the controler that will reach the target within 10 steps with probability equal or higher than 0.77:**
```
python3 dynasty.py --config examples/grid/4x4grid_sl.cfg  --constants CMAX=11,T_EXP=0.0,T_SLOW=0.0,T_FAST=0.77 cegis
```

In contrast with cegis and cegar, evolutionary algorithm is incomplete and it uses timeout value of 30 seconds as default, which can be changed by option '--timeout'.

**Running the evolution with one second timeout for the synthesis:**
```
python3 dynasty.py --config examples/grid/4x4grid_sl.cfg  --timeout 1  --constants CMAX=11,T_EXP=0.0,T_SLOW=0.0,T_FAST=0.77 ea
```

**Synthesize the controller that will reach the target in expected number of steps less than 15.5:**
```
python3 dynasty.py --config examples/grid/4x4grid_sl.cfg  --properties reward.properties --constants CMAX=400,T_EXP=15.5,T_FAST=0.0,T_SLOW=0.0 lift
```
If we check expected rewards, we implicitly have to assume that the probability to reach the target set is one (TODO: cegis article link). This implicit assumption can be made explicit with option '--check-prerequisites':
```
python3 dynasty.py --config examples/grid/4x4grid_sl.cfg  --check-prerequisites  --properties reward.properties --constants CMAX=400,T_EXP=15.5,T_FAST=0.0,T_SLOW=0.0 cegis
```


# Partitioning

This problem is also known as threshold synthesis. It aims to partition the set of instantiations into a set of accepting instantiations, i.e., instantiations that satisfy the property at hand, and rejecting instantiations, i.e., instantiations that do not satisfy the property at hand. Methods cegis and evolutionary search do not support this this type of synthesis.

**A usage example of partitioning on grid (POMDP) benchmark mentioned in 'Feasibility' section:**
```
python3 dynasty.py --config examples/grid/4x4grid_sl.cfg  --partitioning  --constants CMAX=11,T_EXP=0.0,T_SLOW=0.0,T_FAST=0.77 lift
```
**Result depicts all instances (option combinations) that satisfy the property:**
```
Subfamilies above: 
[HoleOptions{M01: [2],M11: [0],M21: [1],P01: [3],P11: [3],P21: [2]}, HoleOptions{M01: [2],M11: [0],M21: [1],P01: [3],P11: [2],P21: [3]}, HoleOptions{M01: [1],M11: [2],M21: [0],P01: [3],P11: [3],P21: [2]}, HoleOptions{M01: [1],M11: [2],M21: [0],P01: [3],P11: [2],P21: [3]}]
```

Notice that '--partitioning' cannot be combined with '--optimality'.


# Optimality

Optimal feasibility analysis is enabled by adding an optimality criterion. An optimality criterion consists of a property, a direction, and a relative tolerance, written in a file:

```
P=? [ F (o=2 & c<=5) ]
max
relative 0.0
```

This describes that the probability described by the first line should be maximized among all feasible options. By increasing the relative tolerance, we relax this hard constraint and only require that the obtained instantiation is at least (1-tolerance) times global maximum.

By passing such a criterion, the tool automatically switches to optimal feasibility, except the evolutionary search, where the optimized property is selected from the *.properties file. The order of the property is specified via option '--optimize-one'.

## Dynamic power manager (DPM)

DPM refers to strategies that attempt to make power mode changing decisions based on the information about their usage pattern available at runtime. The objective is to minimize power consumption, while minimizing the effect on performance. The model consists of a Service Requester (SR), a Service Provider (SP), a Service Request Queue (SRQ), and the power manager (PM). The SR models the arrival of requests and the SRQ corresponds to a (finite) queue in which the requests that cannot immediately be served are stored. The SP is the resource which services requests. It can have several states of operation with varying service rates (the time taken to service requests). The PM is a controller that observes the system and issues commands to the SP which correspond instruct it to change its current state.

TODO: image

Description above was adopted from the web sites of [PRISM](https://www.prismmodelchecker.org/casestudies/power.php).

TODO: running example - coming soon

