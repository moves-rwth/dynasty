# Feasibility
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
At the end of the output can be found the analysis result, which has one of the following two forms
```
Unsatisfiable!
```
or
```
Satisfiable!
using x3: 6, x4: 3, x5: 4, x6: 5
```
where in the second it is also specified which options were assigned to each hole and thus providing an instance of the program that correctly emulates fair six-sided die with fair coin (in case you get different result, there are four possible solutions of this particular example :alien:).
