---
layout: default
title: "Enhanced GPS Path Following Algorithm"
---

# Enhanced GPS Path Following Algorithm

In my pursuit to improve GPS reliability and overall path–following performance, I made modifications to the base `path.py` file used in the Donkey Car project. This page details the modifications, explains their impact, and illustrates how these improvements go beyond the original implementation.

---

## Overview

### Why These Changes?

- **Sequential Waypoint Tracking:**  
  The base implementation could inadvertently choose waypoints from earlier in the path, leading to erratic behavior. To resolve this, I introduced a state variable `self.last_index` that ensures the algorithm only considers future (i.e., higher sequence number) waypoints during cross–track error computation.

- **Wrap–Around Mechanism:**  
  On closed or looping tracks, the vehicle must eventually return to the start. I added a configurable parameter `self.wrap_threshold` so that when the vehicle nears the origin (i.e., within a specified distance), the waypoint index resets to zero. This allows for seamless looping and improved path continuity.

**General Impact:**  
These changes improve GPS reliability by enforcing a consistent, forward-moving selection of waypoints. This not only prevents erratic behavior but also ensures that the vehicle follows a smoother, more predictable path.

---

## Complete Modified `path.py` File

Below is the complete `path.py` file with all modifications clearly marked with comments (`# CHANGED` and `# NEW`):

```python
#!/usr/bin/env python3
"""
path.py

This module contains various classes for recording, saving, and using a
path for a Donkey Car. In particular, the CTE class has been modified to use
a sequential approach when following a recorded path. The modifications
include:
  - Keeping track of a last (largest) waypoint index so that only future points are
    considered in the cross-track error computation.
  - When the vehicle nears the origin (within a configurable threshold), the index is reset.
  
All changes to support sequential path following and robust wrap–around
are marked with comments.
"""

import pickle
import math
import logging
import pathlib
import numpy
from PIL import Image, ImageDraw

from donkeycar.parts.transform import PIDController
from donkeycar.utils import norm_deg, dist, deg2rad, arr_to_img, is_number_type

# ------------------------------
# Path Recording Classes
# ------------------------------

class AbstractPath:
    def __init__(self, min_dist=1.):
        self.path = []  # list of (x, y) tuples
        self.min_dist = min_dist
        self.x = math.inf
        self.y = math.inf

    def run(self, recording, x, y):
        if recording:
            d = dist(x, y, self.x, self.y)
            if d > self.min_dist:
                logging.info(f"path point ({x},{y})")
                self.path.append((x, y))
                self.x = x
                self.y = y
        return self.path

    def length(self):
        return len(self.path)

    def is_empty(self):
        return 0 == self.length()

    def is_loaded(self):
        return not self.is_empty()

    def get_xy(self):
        return self.path

    def reset(self):
        self.path = []
        return True

    def save(self, filename):
        return False

    def load(self, filename):
        return False


class CsvPath(AbstractPath):
    def __init__(self, min_dist=1.):
        super().__init__(min_dist)

    def save(self, filename):
        if self.length() > 0:
            with open(filename, 'w') as outfile:
                for (x, y) in self.path:
                    outfile.write(f"{x}, {y}\n")
            return True
        else:
            return False

    def load(self, filename):
        path = pathlib.Path(filename)
        if path.is_file():
            with open(filename, "r") as infile:
                self.path = []
                for line in infile:
                    xy = [float(i.strip()) for i in line.strip().split(sep=",")]
                    self.path.append((xy[0], xy[1]))
            return True
        else:
            logging.info(f"File '{filename}' does not exist")
            return False

        self.recording = False


class CsvThrottlePath(AbstractPath):
    def __init__(self, min_dist: float = 1.0) -> None:
        super().__init__(min_dist)
        self.throttles = []

    def run(self, recording: bool, x: float, y: float, throttle: float) -> tuple:
        if recording:
            d = dist(x, y, self.x, self.y)
            if d > self.min_dist:
                logging.info(f"path point: ({x},{y}) throttle: {throttle}")
                self.path.append((x, y))
                self.throttles.append(throttle)
                self.x = x
                self.y = y
        return self.path, self.throttles

    def reset(self) -> bool:
        super().reset()
        self.throttles = []
        return True

    def save(self, filename: str) -> bool:
        if self.length() > 0:
            with open(filename, 'w') as outfile:
                for (x, y), v in zip(self.path, self.throttles):
                    outfile.write(f"{x}, {y}, {v}\n")
            return True
        else:
            return False

    def load(self, filename: str) -> bool:
        path = pathlib.Path(filename)
        if path.is_file():
            with open(filename, "r") as infile:
                self.path = []
                for line in infile:
                    xy = [float(i.strip()) for i in line.strip().split(sep=",")]
                    self.path.append((xy[0], xy[1]))
                    self.throttles.append(xy[2])
            return True
        else:
            logging.warning(f"File '{filename}' does not exist")
            return False


class RosPath(AbstractPath):
    def __init__(self, min_dist=1.):
        super().__init__(min_dist)  # CHANGED: Fixed incorrect superclass call

    def save(self, filename):
        outfile = open(filename, 'wb')
        pickle.dump(self.path, outfile)
        return True

    def load(self, filename):
        infile = open(filename, 'rb')
        self.path = pickle.load(infile)
        self.recording = False
        return True

# ------------------------------
# Image and Origin Helpers
# ------------------------------

class PImage(object):
    def __init__(self, resolution=(500, 500), color="white", clear_each_frame=False):
        self.resolution = resolution
        self.color = color
        self.img = Image.new('RGB', resolution, color=color)
        self.clear_each_frame = clear_each_frame

    def run(self):
        if self.clear_each_frame:
            self.img = Image.new('RGB', self.resolution, color=self.color)
        return self.img


class OriginOffset(object):
    '''
    Use this to set the car back to the origin without restarting it.
    '''
    def __init__(self, debug=False):
        self.debug = debug
        self.ox = 0.0
        self.oy = 0.0
        self.last_x = 0.0
        self.last_y = 0.0
        self.reset = None

    def run(self, x, y, closest_pt):
        """
        Translate the current position by the stored origin.
        If a reset is requested, the origin is updated with the current position.
        """
        if is_number_type(x) and is_number_type(y):
            # If reset flag is set, update origin to current position.
            if self.reset:
                self.ox = x
                self.oy = y

            self.last_x = x
            self.last_y = y
        else:
            logging.debug("OriginOffset ignoring non-number")

        # Compute translated position.
        pos = (0, 0)
        if self.last_x is not None and self.last_y is not None and self.ox is not None and self.oy is not None:
            pos = (self.last_x - self.ox, self.last_y - self.oy)
        if self.debug:
            logging.info(f"pos/x = {pos[0]}, pos/y = {pos[1]}")

        # If resetting, clear the starting search index for CTE algorithm.
        if self.reset:
            if self.debug:
                logging.info(f"cte/closest_pt = {closest_pt} -> None")
            closest_pt = None

        # Clear the reset flag.
        self.reset = False

        return pos[0], pos[1], closest_pt

    def set_origin(self, x, y):
        logging.info(f"Resetting origin to ({x}, {y})")
        self.ox = x
        self.oy = y

    def reset_origin(self):
        """
        Reset the origin with the next incoming value.
        """
        self.ox = None
        self.oy = None
        self.reset = True

    def init_to_last(self):
        self.set_origin(self.last_x, self.last_y)

# ------------------------------
# Plotting Helpers
# ------------------------------

class PathPlot(object):
    '''
    Draw a path on an image.
    '''
    def __init__(self, scale=1.0, offset=(0., 0.0)):
        self.scale = scale
        self.offset = offset

    def plot_line(self, sx, sy, ex, ey, draw, color):
        """
        Draw a line between two points on the image.
        """
        draw.line((sx, sy, ex, ey), fill=color, width=1)

    def run(self, img, path):
        if type(img) is numpy.ndarray:
            stacked_img = numpy.stack((img,)*3, axis=-1)
            img = arr_to_img(stacked_img)

        if path:
            draw = ImageDraw.Draw(img)
            color = (255, 0, 0)
            for iP in range(0, len(path) - 1):
                ax, ay = path[iP]
                bx, by = path[iP + 1]
                # Note: y increases going north (thus inverted scaling).
                self.plot_line(ax * self.scale + self.offset[0],
                               ay * -self.scale + self.offset[1],
                               bx * self.scale + self.offset[0],
                               by * -self.scale + self.offset[1],
                               draw,
                               color)
        return img


class PlotCircle(object):
    '''
    Draw a circle on an image.
    '''
    def __init__(self, scale=1.0, offset=(0., 0.0), radius=4, color=(0, 255, 0)):
        self.scale = scale
        self.offset = offset
        self.radius = radius
        self.color = color

    def plot_circle(self, x, y, rad, draw, color, width=1):
        """
        Draw an ellipse (circle) at the specified position.
        """
        sx = x - rad
        sy = y - rad
        ex = x + rad
        ey = y + rad
        draw.ellipse([(sx, sy), (ex, ey)], fill=color)

    def run(self, img, x, y):
        draw = ImageDraw.Draw(img)
        self.plot_circle(x * self.scale + self.offset[0],
                         y * -self.scale + self.offset[1],  # Note: y is inverted.
                         self.radius,
                         draw, 
                         self.color)
        return img

# ------------------------------
# Modified CTE (Cross Track Error) Class
# ------------------------------
# CHANGED: The CTE class below has been modified to enforce sequential
# waypoint following by keeping track of the last index (self.last_index)
# and applying a wrap–around threshold (self.wrap_threshold) so that only points
# with increasing sequence numbers are considered until the vehicle nears the origin.

from donkeycar.la import Line3D, Vec3

class CTE(object):

    def __init__(self, look_ahead=1, look_behind=1, num_pts=None, wrap_threshold=2.0) -> None:
        self.num_pts = num_pts
        self.look_ahead = look_ahead
        self.look_behind = look_behind
        # NEW: Track the last visited index to enforce sequential progression.
        self.last_index = 0  
        # NEW: Set the wrap threshold distance (in same units as GPS)
        self.wrap_threshold = wrap_threshold  

    def nearest_pt(self, path, x, y, from_pt=None, num_pts=None):
        """
        CHANGED: Modified to only consider waypoints starting from the last_index.
        When the search reaches the end, if the vehicle is close enough to the origin
        (within wrap_threshold), the index is wrapped to 0.
        """
        # Use last_index if from_pt is not provided.
        if from_pt is None:
            from_pt = self.last_index
        if num_pts is None:
            num_pts = len(path) - from_pt  # only consider remaining points

        if num_pts < 0:
            logging.error("num_pts must not be negative.")
            return None, None, None

        min_pt = None
        min_dist = float('inf')
        min_index = from_pt

        # Only iterate over indices from last_index onward.
        for i in range(from_pt, len(path)):
            p = path[i]
            d = dist(p[0], p[1], x, y)
            if d < min_dist:
                min_pt = p
                min_dist = d
                min_index = i

        # If we are at the last point, check whether we should wrap around.
        if min_index == len(path) - 1:
            origin_dist = dist(path[0][0], path[0][1], x, y)
            if origin_dist < self.wrap_threshold:
                logging.info(f"Wrapping around: distance to origin {origin_dist:.2f} < threshold {self.wrap_threshold}")
                min_index = 0

        self.last_index = min_index  # update the sequential progress
        return min_pt, min_index, min_dist

    def nearest_waypoints(self, path, x, y, look_ahead=None, look_behind=None, from_pt=None, num_pts=None):
        """
        CHANGED: Use the new nearest_pt method and then select sequential neighbors.
        """
        if look_ahead is None:
            look_ahead = self.look_ahead
        if look_behind is None:
            look_behind = self.look_behind

        nearest, i, _ = self.nearest_pt(path, x, y, from_pt, num_pts)
        # Use sequential indices without modulo arithmetic.
        index_a = max(i - look_behind, 0)
        index_b = min(i + look_ahead, len(path) - 1)
        return path[index_a], nearest, path[index_b], i

    def nearest_track(self, path, x, y, look_ahead=None, look_behind=None, from_pt=0, num_pts=None):
        """
        Returns the segment (a, b) of the path used to compute the error.
        """
        a, nearest, b, i = self.nearest_waypoints(path, x, y, look_ahead, look_behind, from_pt, num_pts)
        if a is None or b is None:
            logging.error("Unable to determine a valid track segment.")
            return None, None, None
        return (a, b, i)

    def run(self, path, x, y, from_pt=None):
        """
        Compute the cross–track error (CTE) from the current position to the path.
        """
        cte = 0.0
        a, b, i = self.nearest_track(path, x, y, 
                                     look_ahead=self.look_ahead, look_behind=self.look_behind, 
                                     from_pt=from_pt, num_pts=self.num_pts)
        
        if a and b:
            logging.info(f"Nearest segment: ({a[0]}, {a[1]}) to ({b[0]}, {b[1]}) for position ({x}, {y})")
            # Create vectors for the two endpoints (assuming a flat plane with y=0 in 3D)
            a_v = Vec3(a[0], 0.0, a[1])
            b_v = Vec3(b[0], 0.0, b[1])
            p_v = Vec3(x, 0.0, y)
            line = Line3D(a_v, b_v)
            err = line.vector_to(p_v)
            sign = 1.0
            cp = line.dir.cross(err.normalized())
            if cp.y > 0.0:
                sign = -1.0
            cte = err.mag() * sign            
        else:
            logging.info(f"No valid nearest point found for ({x},{y})")
        return cte, i

# ------------------------------
# PID Pilot
# ------------------------------

class PID_Pilot(object):

    def __init__(self,
                 pid: PIDController,
                 throttle: float,
                 use_constant_throttle: bool = False,
                 min_throttle: float = None) -> None:
        self.pid = pid
        self.throttle = throttle
        self.use_constant_throttle = use_constant_throttle
        self.variable_speed_multiplier = 1.0
        self.min_throttle = min_throttle if min_throttle is not None else throttle

    def run(self, cte: float, throttles: list, closest_pt_idx: int) -> tuple:
        steer = self.pid.run(cte)
        if self.use_constant_throttle or throttles is None or closest_pt_idx is None:
            throttle = self.throttle
        elif throttles[closest_pt_idx] * self.variable_speed_multiplier < self.min_throttle:
            throttle = self.min_throttle
        else:
            throttle = throttles[closest_pt_idx] * self.variable_speed_multiplier
        logging.info(f"CTE: {cte} steer: {steer} throttle: {throttle}")
        return steer, throttle
```