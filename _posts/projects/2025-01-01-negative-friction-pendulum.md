---
title: "Negative Friction Pendulum"
layout: gridlay
date: 2025-01-01
sitemap: false
permalink: /projects/negative_friction_pendulum
---

# Negative Friction Pendulum

## Theory

### Simple Pendulum Motion
When a pendulum is at rest, the tension of the rod and the weight of the pendulum are in direct opposition resulting in a static system (Fig. \ref{fig:static_pendulum}). When, however, the pendulum is raised to one side and then released, the weight of the pendulum creates a moment about the origin of the pendulum resulting in angular motion (Fig. \ref{fig:raised_pendulum}). 

\begin{figure}[!h]
    \begin{minipage}[t]{0.45\linewidth}
        \centering
        \begin{tikzpicture}[baseline = (current bounding box.north)]
            % Draw the fixed point
            
            \draw[thick] (0,0) -- (0.2,0.4) -- (-0.2,0.4) -- cycle;
            \filldraw[black] (0,0) circle (2pt)  node[left] {o};
            \draw[->] (-0.2,-0.1) arc (200:330:0.25) node[right] {$\tau_{applied}$};
            % Draw the pendulum rod
            \draw[dashed] (0,0) -- (0,-3);
             \draw[<->] (-0.5, 0) -- node[left] {L} (-0.5, -3);
            
            % Draw the pendulum mass (circle)
            \filldraw[fill=gray] (0,-3) circle (8pt);
            
            % Draw force vectors
            \draw[->, thick] (0,-3) -- (0,-2) node[above, right] {$T$};
            \draw[->, thick] (0,-3) -- (0,-4) node[below, right] {$mg$};
            \filldraw[black] (0,-3) circle (2pt);
            
        \end{tikzpicture}
        \caption{Pendulum at rest} 
        \label{fig:static_pendulum}

    \end{minipage} \hfill
    \begin{minipage}[t]{0.45\linewidth}
        \centering
        \begin{tikzpicture}[baseline = (current bounding box.north)]
            % Draw the fixed point
            \draw[thick] (0,0) -- (0.2,0.4) -- (-0.2,0.4) -- cycle;
            \filldraw[black] (0,0) circle (2pt)  node[left] {o};
            \draw[->] (-0.2,-0.1) arc (200:330:0.25) node[right] {$\tau_{applied}$};
            % Draw the pendulum rod
            \draw[dashed] (0,0) -- (2,-2);
            
            % Draw the pendulum mass (circle)
            \filldraw[fill=gray] (2,-2) circle (8pt);
           
            
            % Draw force vectors
            \draw[->, thick] (2,-2) -- (2-0.707,-2+0.707) node[above, right] {$T$};
            \draw[->, dashed] (2,-2) -- (2,-3) node[below] {$mg$};
            \draw[->, thick] (2,-2) -- (2+0.707,-2-0.707);
            \draw[->, thick] (2,-2) -- (2 - 0.707,-2 - 0.707);

            \filldraw[black] (2,-2) circle (2pt);

            \draw (0, -1) arc (270:315:1) node[below left, yshift = -0.2cm ] {$\theta$};
            \draw[dashed] (0, 0) -- (0, -1.5);

            \draw (0, -4.2) -- (0, -4.2);

            
        \end{tikzpicture}
        \caption{Raised Pendulum with Component Vectors}
        \label{fig:raised_pendulum}

    \end{minipage}
\end{figure}



Mathematically, we can describe this behavior with Newton's Second Law (Eq. \ref{eq:angular_acceleration})

$$
    \tau = I\ddot{\theta}
    \label{eq:angular_acceleration}
$$

\begin{center}
\textit{I = mass moment of inertia, $\ddot{\theta}$ = angular acceleration}
\end{center}

Using (Eq. \ref{eq:angular_acceleration}) and Fig. \ref{fig:raised_pendulum}, we can determine the sum of moments about the origin. While there is a term for $\tau_{applied}$ in the equation, we will ignore it for now and assume it is zero.

$$
    \sum \tau_o = mgLsin(\theta) + \tau_{applied} = I\ddot{\theta}
    \label{eq:sum_moments}
$$ 
\begin{center}
\textit{$\theta$ = angle of pendulum, m = mass of pendulum, g = gravity, L = length of pendulum, $\tau_{applied}$ = external torque applied to pendulum}
\end{center}

Based on (Eq. \ref{eq:sum_moments}) we can show the behavior of a simple pendulum (Fig. \ref{fig:Undamped_motion}). Notice that without friction, the perturbed pendulum will swing forever.

\begin{figure}[!h]
    \centering
    \includegraphics[width = \linewidth]{undamped_motion.png}
    \caption{Undamped Pendulum Motion}
    \label{fig:Undamped_motion}
\end{figure}

% needed in second column of first page if using \IEEEpubid
%\IEEEpubidadjcol



### Friction on the Pendulum

In the real world world where friction exists, an additional element in terms of angular velocity ($\dot{\theta}$) must be included. The following equation (Eq. \ref{eq:sum_moments_friction}) can be used to calculate the motion of a pendulum with friction. 

\newpage
$$
    mgLsin(\theta) - \beta \dot{\theta} + \tau_{applied} = I\ddot{\theta}
    \label{eq:sum_moments_friction}
$$
\begin{center}
    \textit{$\beta$ = friction coeffecient}
\end{center}

Using this equation, we can start to categorize the behavior of the pendulum based on the magnitude of the coeffecient of friction, $\beta$.

\begin{figure}[!h]
    \centering
    \includegraphics[width = \linewidth]{damping_behavior.png}
    \caption{Undamped, Underdamped, Overdamped Behavior}
    \label{fig:damping_behavior}
\end{figure}

Based on Fig. \ref{fig:damping_behavior} we can see that increasing the friction causes damping behavior in the pendulum. A larger coefficient of friction causes the pendulum to converge to the fixed point faster. If a positive coefficient of friction results in decaying behavior, it's easy to see how a negative coefficient of friction should result in growth in the pendulum's swing. This concept is illustrated in Fig. \ref{fig:negative_friction} by using a coefficient of friction with a negative sign. For the switched pendulum, negative friction creates the stretching mechanism. 


\begin{figure}[!h]
    \centering
    \includegraphics[width = \linewidth]{negative_friction.png}
    \caption{Pendulum Behavior with Negative Friction}
    \label{fig:negative_friction}
\end{figure}


It should be noted that in Fig. \ref{fig:negative_friction} when the pendulum crosses $\theta = \pi$, the growth explodes exponentially. $\theta = \pi$ corresponds with the pendulum going over the top, at which point it begins spinning uncontrollably.

## Real World Implementation

### Motor Characteristics

In previous sections, we have created a theoretical math model for the switched pendulum. While the derivation of the complete electromechanical math model is outside the scope of this paper, it is necessary to prove that it is physically possible to create a pendulum with negative friction. 

\subsection{Motor Characteristics}
In our switched pendulum we will use a DC motor to make the connection between the electrical and mechanical components of our system. The DC motor will allow us to alter the friction of the system and apply an external moment to shift the fixed point away from zero. 

When the stator of the motor is rotated, the motion of the stator moving by the coils of the motor results in an induced voltage called back emf. The back emf will be the the mechanism that allows us to increase the friction of the system. Based on information found in the datasheets of most DC motors, the following equation can be used to determine the magnitude of the back emf \cite{noauthor_permanent_nodate}: 

\begin{equation}
    V_b(t) = K_B \frac{d\theta}{dt}
    \label{eq:back_emf}
\end{equation}
\begin{center}
    \textit{$V_b$ = back voltage, $K_b$ = motor back emf constant}
\end{center}

Next, we know that supplying current to the motor causes rotation. The motor torque constant can be used to calculate the torque being applied to the pendulum as a function of current \cite{noauthor_permanent_nodate}. 

\begin{equation}
    \tau_{motor} = K_m*i
    \label{eq:motor_torque}
\end{equation}
\begin{center}
    \textit{$K_m$ = motor torque constant, $i$ = current}
\end{center}

While not explicitly used in this paper, Eq. \ref{eq:motor_torque} will be used to determine the necessary current supply to set the fixed points of the switched pendulum.



### Negative Resistors and Negative Friction

The equation for rotational power is: 

$$
    P = \tau \omega = \tau \frac{d\theta}{dt}
$$
\begin{center}
    \textit{P = power, $\tau$ = torque, $\omega$ = angular velocity}
\end{center}

The equation for electrical power is: 

$$
    P = iV
$$

Ohm's law states: 

$$
    V = iR
$$

If we consider a motor/generator as a converter between mechanical power to electrical power, then we can say for an ideal generator: 

$$
    P_{mechanical} = P_{electrical}
$$

$$
    \tau_{friction} \frac{d\theta}{dt} = iV
$$

Use Ohm's law to relate the resistor to the applied torque:

$$
    \tau_{friction} \frac{d\theta}{dt} = i^2R
    \label{eq:Mpower_to_Epower}
$$

To maintain equilibrium, when R is increased, the $\tau_{friction}$ must also be increased to maintain the same rotational velocity: it's harder to spin the motor when the resistance is greater. 

With Eq. \ref{eq:Mpower_to_Epower}, we can see that attaching a positive resistor to the motor results in an \textit{increased} amount of friction, and attaching a negative resistor to the motor would \textit{decrease} the friction. While there is no passive component that behaves as a negative resistor, the negative impedance converter \cite{horowitz_art_2021} (Fig. \ref{fig:negative_impedance_converter}) is an active circuit that is effectively a negative resistor. With the negative impedance converter, voltage potential created by the motor's back emf results in current being supplied to the motor 
\footnote{The concept of negative resistance and negative friction is sometimes slippery to understand because it's not a naturally occurring phenomenon. The reader may find the following example helpful as a way of visualizing the behavior of a negative resistor. The mechanical analog to a resistor is a spring. As outlined in Eq. \ref{eq:spring}, compressing a spring results in an opposing force proportional to the spring's modulus of elasticity: the spring resists changes in length.   

$$
    F = K\Delta X
    \label{eq:spring}
$$

Having a negative spring in the real world, would be a spring with a \textit{negative} modulus of elasticity. Functionally, this would mean that a person attempting to compress the spring would have their hand pulled by the spring and the spring would compress even farther: the negative spring would assist in its own compression.}.

To solve for the resistance of the negative impedance converter, assume $v_{in} = v_2$.


$$
    v_1 = v_{in} - R_1i_{in}
$$

$$
    v_{in} = v_1(\frac{R_3}{R_2 + R_3})
$$


$$
    v_{in} = (v_{in} - R_1i_{in})(\frac{R_3}{R_2 + R_3})
$$

$$
    v_{in}(1- \frac{R_3}{R_2 + R_3}) = -\frac{R_3R_1i_{in}}{R_2 + R_3}
$$

$$
    v_{in} = -\frac{R_3R_1}{R_2}i_{in}
$$

Notice that this takes the form of Ohms law, but the resistance term has a negative sign. 


\begin{figure}[!h]
\begin{minipage}[t]{\linewidth}
\centering
    \begin{circuitikz}[scale = 0.65, transform shape, american, baseline = (current bounding box.north)]

        \draw(0, 0) node[op amp, noinv input up] (opamp) {};
        \node[ground] at (-2.44, -5) (ground) {};
        \draw (opamp.+) to[short] ++(-1.25,0) to[short, *-] ++(0, 2.5) to[R, l^=$R_1$] ++(4,0) to[short] ++(0, -3);
        \draw (opamp.-) -- ++(-1.25,0) -- ++(0,-2) to[R, l_=$R_3$] (ground);
        \draw(opamp.out) to[short] ++ (0.4, 0);
        \draw (1.55,0) to[short,*-] ++(0,-2.5) to[R, l^=$R_2$, -*] ++(-4,0);
        \draw (-4, 0.5) to[short] (-2.5, 0.5);
        \node at (-3.5, 0.75){$v_{in}$};
        \node at (-2.75, -0.75){$v_{2}$};
        \node at (2, 0){$v_1$};
        \node at (-3, 0){$i_{in}$};
        \draw[-latex] (-2.75,0) -- (-2,0);
        
    \end{circuitikz}
    \caption{Circuit Diagram for Negative Resistor}
    \label{fig:negative_impedance_converter}

\end{minipage}
\end{figure}