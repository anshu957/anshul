---
layout: post
title: Visualizing Chaos on 2D Torus 
date: 2021-01-14 00:28:30
description: using python's matplotlib library, let's try to visualize chaos on 2D torus
categories: visualization, python 
comments: true
---


{% highlight py  %}
import cycler
from matplotlib.pyplot import autoscale
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
from matplotlib import animation
from scipy.integrate import solve_ivp
from mpl_toolkits.mplot3d import Axes3D
import matplotlib
from mayavi import mlab
{% endhighlight %}

{% highlight py  %}
plt.style.use('default')
K = 1.4
N_IC = 200
{% endhighlight %}


{% highlight py  %}
def standard_map(state):
    q, p = state
    p_next = (p + K*np.sin(q))  % (2*np.pi)
    q_next = (q + p_next) % (2*np.pi)
    return np.array([q_next, p_next])
def get_orbit(state, trun):
    orbit = []
    for i in range(trun):
        next_state = standard_map(state)
        orbit.append(next_state)
        state = next_state
    return np.array(orbit)


orbits = []
for i in range(N_IC):
    initial_condition = 15*np.random.random(2)
    orbits.append(get_orbit(initial_condition, 1000))

plt.style.use('dark_background')
fig = plt.figure(figsize=(12, 10))
ax1 = fig.add_subplot(111)
#color = plt.cm.gist_rainbow(np.linspace(0, 1, N_IC))
#matplotlib.rcParams['axes.prop_cycle'] = cycler.cycler('color', color)

for i in range(N_IC):
    ax1.plot(orbits[i][:, 0], orbits[i][:, 1], '.', markersize=0.5)
{% endhighlight %}
---------
##### The phase-space trajectories from various initial conditons (represented by different colors) on a cartesian plane (x-y) looks like: 
 <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/phasespace_traj_flat.png' | relative_url }}" alt="" title="example image"/>


{% highlight py  %}
fig = plt.figure(figsize=(12, 10))
ax = fig.add_subplot(111, projection='3d', autoscale_on=False,
                     xlim=(-60, 60), ylim=(-60, 60), zlim=(-30, 30))
ax.view_init(50, 10)
#theta, phi = np.meshgrid(orbits[1][:, 0], orbits[1][:, 1])
angle = np.linspace(0, 2*np.pi, 50)
r, R = 25, 70.

theta1, phi1 = np.meshgrid(angle, angle)
X1 = (R + r * np.cos(phi1)) * np.cos(theta1)
Y1 = (R + r * np.cos(phi1)) * np.sin(theta1)
Z1 = r * np.sin(phi1)
ax.plot_surface(X1, Y1, Z1, color='white', alpha=0.03, rstride=3, cstride=3)
color = plt.cm.rainbow(np.linspace(0, 1, 100))
matplotlib.rcParams['axes.prop_cycle'] = cycler.cycler('color', color)

for i in range(N_IC):
    phi, theta = orbits[i][:, 0], orbits[i][:, 1]
    X = (R + r * np.cos(phi)) * np.cos(theta)
    Y = (R + r * np.cos(phi)) * np.sin(theta)
    Z = r * np.sin(phi)

    ax.plot(X, Y, Z, '.', markersize=0.7)



ax.set_axis_off()

#plt.savefig('beautifulTorus.pdf')
{% endhighlight %}

##### The same phase space trajectories (as above) but now projected onto a 2-torus: 
 <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/phasespace_traj_torus.png' | relative_url }}" alt="" title="example image"/>


{% highlight py  %}
fig = plt.figure(figsize=(12, 10))
ax = fig.add_subplot(111, projection='3d', autoscale_on=False,
                     xlim=(-60, 60), ylim=(-60, 60), zlim=(-30, 40))
ax.view_init(52, 10)
#theta, phi = np.meshgrid(orbits[1][:, 0], orbits[1][:, 1])
angle = np.linspace(0, 2*np.pi, 50)
r, R = 25, 60.

theta1, phi1 = np.meshgrid(angle, angle)
X1 = (R + r * np.cos(phi1)) * np.cos(theta1)
Y1 = (R + r * np.cos(phi1)) * np.sin(theta1)
Z1 = r * np.sin(phi1)
ax.plot_surface(X1, Y1, Z1, color='white', alpha=0.12, rstride=3, cstride=3)


def update_frames(frame):
    for i in range(N_IC):
        phi, theta = orbits[i][:, 0], orbits[i][:, 1]
        X = (R + r * np.cos(phi)) * np.cos(theta)
        Y = (R + r * np.cos(phi)) * np.sin(theta)
        Z = r * np.sin(phi+2*frame)

    ln.set_data(data.y[0:2, :4*frame])
    #ax.plot(X, Y, Z, '.', markersize=0.5)

    ln.set_3d_properties(data.y[2, :4*frame])
    ax.view_init(elev=5., azim=frame*0.3)

    return ln,



line_anim = FuncAnimation(fig, update_frames, frames=int(0.25*len(
    data.y[::2].T)), blit=True, interval=5)
line_anim.save('RotationLorenz.gif', writer=animation.PillowWriter(fps=30))
{% endhighlight %}    
    