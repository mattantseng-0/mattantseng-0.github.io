---
title: "Reservoir Computing & The Lorenz System"
layout: gridlay
date: 2025-01-01
sitemap: false
permalink: /projects/reservoir_computing_lorenz
thumbnail: "/images/project_images/rc_and_lorenz/lorenz_3d.png"
---

# Reservoir Computing & The Lorenz System

## Motivation

## Theory
### Reservoir Computing

The equation for the Leaky Integrator Echo State Network is: 

$$
x(t) = (1-a)x(t) + a\times \tanh(W_{in}u(t) + \hat{W}x(t-1))
$$

This equation can also be described by considering tha matrix dimensions: 

$$
[I][I\times R][R \times O] = [O]
$$

## The Echo State Property


### The Lorenz System

$$
\begin{split}
\frac{dx}{dt} & = \sigma (y-x) \\
\frac{dy}{dt} & = x(\rho - z) - y\\
\frac{dz}{dt} & = xy - \beta z\\
\end{split}
$$

The standard weights for the Lorenz System is shown in the following table.

| $\sigma$ | $\rho$ | $\beta$ |
| ------ | ------ | ------ |
| $10$ | $28$ | $\frac{8}{3}$ | 


## Methods
### Setting up the Python Environment

The package requirements are minimal for this project. The only packages necessary are `numpy`, `scipy` and `matplotlib`



```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import solve_ivp
```

### Modeling the Lorenz System

The next step in this project is to create our training data for the Lorenz system. In this section, we will use a Runge-Kutta-4 solver to simulate the Lorenz system. To start, create the following function whose input arguments correspond with the coeffecients of the Lorenz equations.

```python
def lorenz(t, vars, rho, sigma, beta):
    x, y, z = vars

    variable_vector = np.array([x, y, z, x*z, x*y])

    weights = np.array([[-sigma, sigma, 0, 0, 0],
                         [rho, -1, 0, -1, 0],
                         [0, 0, -beta, 0, 1]])

    # Perform matrix multiplication to obtain the Lorenz equations
    return weights@variable_vector

```

Next, define the three variables. You will notice that instead of using round numbers like $28$ and $10$ we use approximately those values. This is a little trick that we can play to make sure our simulation doesn't round as much.


```python
# These are the historical weights from the original Lorenz System 
rho = 27.999999
sigma = 9.999999
beta = 8/3
```

Now we need to define the duration, sampling rate and initial conditions for the system. We will use a random number to instantiate the $x$ initial condition. This will ensure that every time we run this simulation, we get a different time series. 

```python
t_start = 0
t_stop = 250

dt = 0.01

t_span = [t_start, t_stop]
t_eval = np.arange(t_start, t_stop, dt)
initial_conditions = [np.random.rand(), 1, 1]
```

Now that all of our parameters have been defined, all that is left to do is run the simulation and collect our data. For convenience, we will also take the solution and break it down into the appropriate variables.

```python
lorenz_soln = solve_ivp(lorenz, t_span, initial_conditions, t_eval= t_eval, args=(rho, sigma, beta))

time = lorenz_soln.t
x = lorenz_soln.y[0, :]
y = lorenz_soln.y[1, :]
z = lorenz_soln.y[2, :]

```

At this point, we have now created the time series data that we can use to train our LI-ESN model. Before we move on to creating the reservoir computing model, let's make some plots of our data to visualize this system.

We have already seen the 3D butterfly attractor for this system, so let's plot the time series for the $x$, $y$ and $z$ channels. One key observation about this time series plot is that the $x$ and $y$ channels exhibit antipodal symmetry.

<img src="../../images/project_images/rc_and_lorenz/lorenz_xyz_ts.png" alt="Pendulum with different damping behavior" width="80%"/>

### Data Preprocesing

Now that we have created the data set for the Lorenz system, we need to apply some processing operations before we can train on this data. 

#### Scaling

The first operation we need to perform on our data is to scale it to the range $[-1, +1]$. Recall that in the equation for the LI-ESN model we have a hyperbolic-tangent function. The following figure shows the plot of $y = tanh(x)$. Notice that for large $\|x\|$ the function is almost a constant. We want the inputs of our system to exist in the nonlinear region of the function and not on the near-constant tails.


<img src="../../images/project_images/rc_and_lorenz/tanh_plot.png" alt="tanh plot" width="80%"/>

The following code centers and scales the data from the lorenz system. 

```python
x_scaling_factor = np.abs(np.max(x))
y_scaling_factor = np.abs(np.max(y))
z_scaling_factor = np.abs(np.max(z))

x_amp_shift = np.mean(x)
y_amp_shift = np.mean(y)
z_amp_shift = np.mean(z)

print("X Scaling: ", x_scaling_factor)
print("Y Scaling: ", y_scaling_factor)
print("Z Scaling: ", z_scaling_factor)

lorenz_data = np.copy(lorenz_soln.y)

lorenz_data[0, :] -= x_amp_shift
lorenz_data[1, :] -= y_amp_shift
lorenz_data[2, :] -= z_amp_shift

# Now apply the scaling factor to our data.
lorenz_data[0, :] /= x_scaling_factor
lorenz_data[1, :] /= y_scaling_factor
lorenz_data[2, :] /= z_scaling_factor
```

#### Training Inputs/Outputs

To determine how we want to format our training data, we need to consider what task we want the LI-ESN accomplish. For this article, we are going to have the input be the state of the system $X[n]$ and the ouput will be the predicted next step $X'[n+1]$. Creating a system that predicts the $X'[n+1]$ value will allow us to feed the output of the LI-ESN model back to the input and recursively run our model. 

Now that we know what we want our model to accomplish, we can create our input/output datasets that can be used for training. The following lines of Python will create the `lorenz_training_data` that corresponds with the $X[n]$ samples and the `lorenz_target_data` that corresponds with the $X[n+1]$ samples. By training the output of the LI-ESN with the $X[n+1]$ data, we can train it to predict these values for a given input.


```python
num_training_points = 25000
lorenz_subset = lorenz_data[:, :num_training_points]

lorenz_training_data = np.copy(lorenz_subset[:, :-1])

lorenz_target_data = np.copy(lorenz_subset[:, 1:])
```


#### Additive Noise

Now that the data has been scaled and centered to fit in the sensitive region of $\tanh$, and we have split our data into training inputs and outputs, the next step is to add noise to the training input data. The reason we want to add noise to the training inputs is to make our system more robust. When we train on noisy inputs, the LI-ESN model will learn how to accept some amount of variance in the input information. That way, when one of our inputs isn't quite what the model expects, it still knows how to generate a valid output. The following Python will add Guassian noise to the training input data. In this case, we are using noise with $\sigma = 0.05$ and $\mu = 0$.

```python

noise_magnitude = 0.05

training_noise = np.random.normal(0, noise_magnitude, np.shape(lorenz_training_data))

lorenz_training_data += training_noise

```

This is the end of our data preprocessing operations. We can create a plot that shows the input and output training data. Notice how this data is centered about $0$, the inputs are noisy and the outputs (targets) are clean.



<img src="../../images/project_images/rc_and_lorenz/training_inputs_outputs.png" alt="tanh plot" width="80%"/>


### Creating a Leaky Integrator Echo State Network in Python

Congratulations, you have now made it to the last major portion of this article. This is possibly the most important part because we are now going to take all of the data that we generated and processed and use it in our custom LI-ESN model.

#### Setting Up the Model Parameters

Before we get started, we need to define some constants that will be used for our LI-ESN model. 

- Reservoir Size:
    - The reservoir is an $M\times M$ matrix
- Input Size:
    - The input size is the dimension of the input data. In our case, we have three state variables $[x, y, z]$
- Spectral Radius:
    - The spectral radius of the reservoir helps ensure that we obtain the echo state property. We want the reservoir to operate "on the edge of chaos" to be the most effective. TODO: ADD CITATION
- Leaky Integrator Factor: 
    - The Leaky Integrator Factor helps match the time scale of our reservoir to the time scale of the system. We want the reservoir to eventually forget old inputs
- Transient:
    - We are going to instantiate our reservoir with random noise. The transient is used to wash out the effects of this random instantiation.

```python
reservoir_size = 1000
input_size = 3
spectral_radius = 1
leaky_integrator_factor = 0.5
sparsity = 0.5
transient = 100
```

#### Defining the Input and Reservoir Matrices


```python
# First: Define the input matrix
input_weights = np.random.uniform(low = -1, high = 1, size = (reservoir_size, input_size)) 


# Second: Define the reservoir matrix
reservoir_weights = np.random.normal(0, 1, (reservoir_size, reservoir_size))
```


Alter the matrices: 
```python
# Now we need to change the sparsity and spectral radius of our reservoir

# Start by changing the sparsity: 

# How big is our reservoir?
total_elements = np.size(reservoir_weights)

# use the given ratio to determine how many need to be zeros
number_to_zeros = int(sparsity*total_elements)

# now get the correct number of indicies
indices_to_set_to_zero = np.random.choice(total_elements, number_to_zeros, replace=False)

# flatten matrix into vector to make assignment easier
flat_matrix = reservoir_weights.flatten()

# set certain values to 0
flat_matrix[indices_to_set_to_zero] = 0

# turn vector back into matrix
reservoir_weights = flat_matrix.reshape(reservoir_weights.shape)

# Now change the spectral radius of the reservoir: 

# start by calculating the current spectral radius: 
current_spectral_radius = np.max(np.abs(np.linalg.eigvals(reservoir_weights)))


# Now divide through. This creates a matrix with a spectral radius of 1
reservoir_weights = reservoir_weights / current_spectral_radius

# multiply by the desired spectral radius

reservoir_weights = spectral_radius * reservoir_weights
```

#### Input Training Data

```python
# This is where we store our reservoir outputs
reservoir_states = np.zeros((np.shape(lorenz_training_data)[1], reservoir_size))

for i in range(1, np.shape(lorenz_training_data)[1]):
    previous_reservoir_state = reservoir_states[i-1, :]

    input_data = lorenz_training_data[:, i]

    # This is the equation for the leaky integrator reservoir computer

    # x(t) = (1-a)x(t-1) + a*tanh(W_in*u(t) + W_r*x(t-1))
    reservoir_states[i, :] = ((1-leaky_integrator_factor)*previous_reservoir_state 
                                + leaky_integrator_factor*np.tanh(input_weights@input_data 
                                + reservoir_weights@previous_reservoir_state))
                                
```



#### Taking Care of the Transient

Before we fit our model to the data, we need to remove the effects of our random weight instantiation. The following figure entitled _Reservoir States WITH Transient_, shows the output of the reservoir and illustrates the effect of the random instantiation. Notice that the output of the reservoir is different for the first $\approx 50$ indices. After this timeframe, the behavior settles down into a steady state behavior. If we were to include this transient in the training method, we would be negatively affected by our random instantiation. To resolve this issue, we truncate the reservoir outputs and, in this case, remove the first 100pts to ensure that the transient effects are removed (shown on the right).

<img src="../../images/project_images/rc_and_lorenz/reservoir_output_with_transient.png" style="display:inline-block; width:45%;"> <img src="../../images/project_images/rc_and_lorenz/reservoir_output_without_transient.png" style="display:inline-block; width:48%;">

#### Training the LI-ESN


```python
# Take the pseudo inverse of the reservoir states
S_cross = np.linalg.pinv(reservoir_states)


# Multiply the pseudo inverse with the target outputs to determine the readout weights 
readout_weights = (S_cross@lorenz_target_data.T).T
```

#### Generating an Output from the Model

In order to recursively generate an output from our model

```python
# For this example, let's graph the last point in our training data and use this as our starting point. 
# We could also grab any other point in our lorenz system.
# initial_conditions = lorenz_target_data[:, 1234]

initial_conditions = np.random.normal(0, 1, [3, ])


print(np.shape(initial_conditions))

num_steps_to_generate = 10000

output = np.zeros([num_steps_to_generate, input_size])

output[0, :] = initial_conditions

inference_reservoir_states = np.zeros((num_steps_to_generate, reservoir_size))
inference_reservoir_states[0, :] = reservoir_states[-1, :]



for i in np.arange(1, num_steps_to_generate):
    # take the predicted outputs and feed them back into the input 
    # Note that we put our initial conditions in the output vector, 
    # so for the first run, we will just pull the initial conditions 
    current_input = output[i-1, :] 

    previous_reservoir_state = inference_reservoir_states[i-1, :]
    

    # now step the reservoir forwards
    inference_reservoir_states[i, :] = ((1-leaky_integrator_factor)*previous_reservoir_state 
                                + leaky_integrator_factor*np.tanh(input_weights@current_input 
                                + reservoir_weights@previous_reservoir_state))



    # Use the state of the reservoir and the readout weights to get the   predicted output. 
    output[i, :] = (readout_weights@inference_reservoir_states[i, :])
    # print(predictions[i, :])
```



# THIS IS A TEST
{% raw %}
<pre> ```mermaid graph TD; A[Start] --> B{Is it working?}; B -- Yes --> C[Celebrate!]; B -- No --> D[Fix it]; ``` </pre>
{% endraw %}




