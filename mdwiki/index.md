# Capacitor and inductor models

## Summary

- Transients of linear dynamical circuits (containing L or C) can be simulated by repeatedly solving an equivalent resistive DC circuit.
- Discrete-time models are derived from first principles and approximations of the exponential function.
- Using these models, we can build arbitrarily complex dynamic circuits.

## A discharging capacitor

At $t=0$, the switch is closed and the capacitor C with initial voltage $v_{c0}$ is discharged by the resistor $R$.

![](assets/20211108_222752_RC.jpg)

How can we determine the capacitor voltage $v_c(t)$?

> Differential equations solve problems like
> "Given $\dot{y}(t)$ and $y(t=0)$, find $ (t>0)$".

The *rate of change* of the capacitor voltage ("dot" for time derivative) is

$$
\dot{v}_c = \frac{d v_c}{d t}
$$

Or, using the relations of charge $Q$ and current $I$

$$
\begin{aligned}
\dot{v}_c &= \frac{\dot{Q}}{C} \\
\\
& = -\frac{i_c}{C} \\
\\
& = -\frac{v_c}{RC} \\
\\
& = -\frac{v_c}{\tau}
\end{aligned}
$$

The negative sign above indicates that discharging produces a negative rate of change, making $v_c$ smaller in magnitude.

> A device current flowing *into the positive pin* is counted positive. For the capacitor here: will increase its charge $Q$ and voltage $V$.

From

$$
\dot{v}_c = -\frac{v_c}{\tau}
$$

we know that at any time $t>0$ the rate of change of $v_c$ is proportional to $v_c$.
In other words, we are looking for a function of time which - up to a factor -  is identical to its own derivative.
The only function with this property is (except 0) the exponential function.

With

$$
v_c(t) = v_{c0}\exp(-t/\tau)
$$

we verify that indeed

$$
\dot{v}_c = -\frac{v_{c0}}{\tau}\exp(-t/\tau) = -\frac{v_c}{\tau}
$$

![](/RC_analytical.jpg)

If the rate of change remained at its initial value, the capacitor would be fully discharged after $\tau=RC$ seconds. This also means the initial slope is proportional to $V_0$.

> A *closed-form* solution for an *analytical* model is valid for all $t>0$.

For realistic circuits, we will not be able to provide analytical solutions. Also, the input signals usually cannot be described by closed formulas.

Since we are usually only intested in solutions at certain time points, we will look at *discrete-time* models. This will also greatly simplify the calculations.

## Approximations of $e^x$

Simple and rational polynomials provide such useful [approximations of the exponential function](https://en.wikipedia.org/wiki/Pad%C3%A9_table).

For small $x$:

$$
\begin{aligned}
\exp(x) & \approx 1 + x \\
       \\ 
       & \approx \frac{1}{1-x} \\
       \\
       & \approx \frac{1+x/2}{1-x/2}\\
\end{aligned}
$$

As we will see below, the denominator gets multiplied to the other side and becomes a linear operator.

![](assets/20211108_223429_exp_approx.PNG)

To improve the approximation for larger $x$, we could further increase the order of the polynomials, but that would make the problem again nonlinear.

> In transient circuit simulation, x in the exponential above is usually proportional to the time step. So instead of one big time step, more smaller steps are taken. The solver uses the result of the previous step as initial value for the following step.

Depending on the type of our approximation, we distinguish 3 different *integrators* or *stepping methods*:

## Forward Euler (FE) method

With the analytic solution for $v_c$, basic relations for charge and current

$$
\boxed{
\begin{aligned}

v_c &= v_c(t) = v_{c0}\exp(-t/\tau)
\\
\\
\dot{v}_c &= -\frac{v_c}{\tau}= -\frac{v_c}{RC} = -\frac{i_c}{C}
\end{aligned}
}
$$

and the approximation

$$
\exp(x) \approx 1 + x
$$

we can calculate the new capacitor voltage at time $t + h$ as

$$
\begin{aligned}
v_c(n+1) & = v_c(n) (1 - h/\tau) 
\\
\\
&= v_c(n) + h \dot{v}_c(n)
\\
\\
&= v_c(n) + \frac{h}{C}i_c(n)
\end{aligned}
$$

Thus the FE equivalent circuit of a discrete-time capacitor model is a voltage source whose value can be calculated from previous (known) values only.

> If the new value depends only on old values and time derivatives, the method is called *explicit*, otherwise *implicit*.

A drawback of FE as *first order method* is its low acccuracy, requiring many small time steps for a good approximation.

> A *stiff* circuit (or differential equation) contains very different time constants, often caused by very small (parasitic) capacitors or inductors.

Since FE only uses the rate of change at the start of the timestep, it has issues with stiff circuits. FE acts on fast transients which quickly die out and become irrelevant. Growing over- and undershoots then lead to instable solutions.

## Backward Euler (BE) method

Using the approximation

$$
\exp(x) \approx \frac{1}{1 - x}
$$

we can calculate the new capacitor voltage (multiply $1+h/\tau$ to the other side) as

$$
\begin{aligned}
v_c(n+1) & = v_c(n) \frac{1}{1 + h/\tau} 
\\ 
\\
& = h \dot{v}_c(n+1) + v_c(n)
\\ 
\\
& = \frac{h}{C}i_c(n+1) + v_c(n)
\end{aligned}
$$

Remark that the BE solution depends on the derivative of the unknown new voltage: it is therefore an *implicit* method.

> The BE model of a capacitor is thus represented as a voltage source in series with a resistor, or [Thevenin](https://en.wikipedia.org/wiki/Th%C3%A9venin%27s_theorem) equivalent.
> The series V-R adds an intermediate node, for circuit simulation we therefore prefer the parallel I-G or [Norton](https://en.wikipedia.org/wiki/Norton%27s_theorem) equivalent.
> Thevenin or Norton equivalents of dynamical components are also called *companion models*.

Or, for the BE Norton equivalent circuit

$$
\begin{aligned}
i_c(n+1) & = \frac{C}{h}v_c(n+1) - \frac{C}{h}v_c(n)
\\ 
\\
& = gn_c v_c(n+1) + in_c(n)
\end{aligned}
$$

as resistance $1/gn_c$, parallel to history current source $in_c$

$$
\begin{aligned}
gn_c &= \frac{C}{h}
\\
\\
in_c(n) &= -\frac{C}{h}v_c(n)
\\
\\
&= -gn_c v_c(n)
\end{aligned}
$$

We see that the history current $in_c$ needs to be recalculated at every step, but conductance $gn_c$ only after a change of time step ($C$ assumed constant).

BE is still a first-order method, but is stiffly stable. One drawback is that BE acts like a lowpass: when simulating oscillators it often underestimates the real amplitude.

## Trapezoidal (TR) method

Using the approximation

$$
\exp(x) \approx \frac{1 + x/2}{1 - x/2}
$$

we can calculate the new capacitor voltage as

$$
\begin{aligned}
v_c(n+1) & = v_c(n) \frac{1 - h/(2\tau)}{1 + h/(2\tau)}  
\\ 
\\
& = h \frac{\dot{v}_c(n+1) + \dot{v}_c(n)}{2} + v_c(n)
\\ 
\\
& = \frac{h}{2C}i_c(n+1) + \frac{h}{2C}i_c(n) + v_c(n)
\end{aligned}
$$

Or, for the TR Norton equivalent circuit

$$
\begin{aligned}
i_c(n+1) & = \frac{2C}{h}v_c(n+1) - \frac{2C}{h}v_c(n) -i_c(n)
\\ 
\\
& = gn_c v_c(n+1) + in_c(n)
\end{aligned}
$$

as resistance $1/gn_c$, parallel to history current source $in_c$

$$
\begin{aligned}
gn_c &= \frac{2C}{h}
\\
\\
in_c(n) &= -\frac{2C}{h}v_c(n) -i_c(n)
\\
\\
&= -gn_c v_c(n) -i_c(n)
\end{aligned}
$$

TR is a *second-order* *implicit* method and allows, compared to FE and BE, larger time steps. It is also stiffly stable and does not show unwanted damping of oscilations.

> After abrupt changes of input signals or topology (hard switching), TR can generate oscillation artifacts. As a remedy, one can temporally switch to BE for one or two steps.


---

In the pictures below, $i_n$ is the device current at step $n$, not to be confused with $in$, the Norton current source.

## Norton equivalent circuits

### [Capacitor](https://docs.lib.purdue.edu/cgi/viewcontent.cgi?article=1301&context=ecetr)

![](assets/20211108_223859_Norton_cap.PNG)

### [Inductor](https://docs.lib.purdue.edu/cgi/viewcontent.cgi?article=1301&context=ecetr)

![](assets/20211108_223925_Norton_ind.PNG)

# Nonlinear components

## <<<<<<<<<< tbd >>>>>>>>>>

> Transient simulation of nonlinear (containing diodes, transistors) circuits is thus reduced to iteratively solving (2-5x per time step) an equivalent circuit containing linear resistors and sources only.
