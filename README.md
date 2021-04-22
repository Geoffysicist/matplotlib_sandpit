# matplotlib_sandpit

from https://www.javaer101.com/en/article/3286817.html

1) Calling fig.canvas.draw() redraws everything. It's your bottleneck. In your case, you don't need to re-draw things like the axes boundaries, tick labels, etc.

2) In your case, there are a lot of subplots with a lot of tick labels. These take a long time to draw.

Both these can be fixed by using blitting.

To do blitting efficiently, you'll have to use backend-specific code. In practice, if you're really worried about smooth animations, you're usually embedding matplotlib plots in some sort of gui toolkit, anyway, so this isn't much of an issue.

However, without knowing a bit more about what you're doing, I can't help you there.

Nonetheless, there is a gui-neutral way of doing it that is still reasonably fast.

import matplotlib.pyplot as plt
import numpy as np
import time

x = np.arange(0, 2*np.pi, 0.1)
y = np.sin(x)

fig, axes = plt.subplots(nrows=6)

fig.show()

# We need to draw the canvas before we start animating...
fig.canvas.draw()

styles = ['r-', 'g-', 'y-', 'm-', 'k-', 'c-']
def plot(ax, style):
    return ax.plot(x, y, style, animated=True)[0]
lines = [plot(ax, style) for ax, style in zip(axes, styles)]

# Let's capture the background of the figure
backgrounds = [fig.canvas.copy_from_bbox(ax.bbox) for ax in axes]

tstart = time.time()
for i in xrange(1, 2000):
    items = enumerate(zip(lines, axes, backgrounds), start=1)
    for j, (line, ax, background) in items:
        fig.canvas.restore_region(background)
        line.set_ydata(np.sin(j*x + i/10.0))
        ax.draw_artist(line)
        fig.canvas.blit(ax.bbox)

print 'FPS:' , 2000/(time.time()-tstart)

This gives me ~200fps.

To make this a bit more convenient, there's an animations module in recent versions of matplotlib.

As an example:

import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

x = np.arange(0, 2*np.pi, 0.1)
y = np.sin(x)

fig, axes = plt.subplots(nrows=6)

styles = ['r-', 'g-', 'y-', 'm-', 'k-', 'c-']
def plot(ax, style):
    return ax.plot(x, y, style, animated=True)[0]
lines = [plot(ax, style) for ax, style in zip(axes, styles)]

def animate(i):
    for j, line in enumerate(lines, start=1):
        line.set_ydata(np.sin(j*x + i/10.0))
    return lines

# We'd normally specify a reasonable "interval" here...
ani = animation.FuncAnimation(fig, animate, xrange(1, 200), 
                              interval=0, blit=True)
plt.show()
