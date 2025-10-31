# Campus Indoor-Outdoor Navigation
Room-to-room campus navigation that links building floor plans to OpenStreetMap for seamless indoor-to-outdoor routing. Implement an end-to-end computer vision pipeline to convert floor plans into routable maps and compute real-time paths with `A*` dynamic edges. Includes a simple Flask API, ReactJS demo, sample data, and a quickstart to run locally.

## Overview
  - **Problem:** Existing apps don't route to specific rooms or choose the right entrance; thus, students struggle with building/room navigation.
  - **Solution:** Campus-specific web app with room-to-room navigation that accounts for transportation mode, weather, crime, construction, and traffic flows.
  - **Core methods:**
    - `A*` with Euclidean heuristic.
    - Indoor Computer Vision pipeline (crop → door detection/removal → entrance alignment).
    - Outdoor weights updated by live/contextual signals.
    - Traffic modeled via Markov chains and scaled to live counts.

## Key Features
  - **Outdoor `A*`:**
    - Time-weighted edges
    - Heuristic = Euclidean distance
    - Weights adjusted by transport mode
    - Weather (wind/rain)
      > Wind effect uses the dot product of travel and wind directions.
    - Crime proximity
    - Construction
  - **Indoor `A*`:**
    - Binary images (white = free, black = walls) with 4-connectivity.
    - Doors "opened" by detection/removal to enable room-level routing.
  - **Traffic modeling:**
    - Converts campus graph to Markov transition matrix.
    - Adds self-loops and an off-campus sink node for stability.
    - Computes the stationary distribution (eigenvector) and scales with live counter data.
  - **Entrance alignment:**
    - Aligns indoor/outdoor polygons by centroid.
    - Scales by area and rotates to maximize IoU, then projects entrances from the outdoor map to the floor plan.

## Indoor Computer Vision Pipeline
1. **Crops floor plans** to the perimeter using OpenCV (scanline/contour extraction), removing legends/whitespace for efficiency.
2. **Door detection** by `cv2.matchTemplate` with multi-angle rotations and scale grid search (e.g., `0.9/1.0/1.1`); non-maximum suppression via KD-tree to keep the best match per door.
3. **Door removal:** converts door-arc pixels to free space to permit traversal.
4. **Entrance mapping:** detects indoor perimeter (contours), centroid alignment, area scale, IoU rotation; moves entrance markers slightly inward for robust indoor routing.

## Outdoor Graph & Weights
- **Data:** buildings, entrances, roads/intersections from OpenStreetMap (exported as OSM). Edge base weight = travel time.
- **Adjustments:**
    - **Mode:** walking vs. biking multipliers.
    - **Weather:** wind via dot product, where rain increases edge time.
    - **Safety:** proximity to emergency call stations.
    - **Crime:** historical heatmap multiplier.
    - **Construction:** closures set edge weight to `∞`.

## Traffic Model via Markov Chains
- Builds a transition matrix over intersections.
- Includes self-loops (some travelers stop) and a sink node for exits.
- Computes the stationary distribution and scales by live counter data to estimate per-edge flows used in weights.

## System Architecture (High-Level)
- **Frontend:** campus map + floor-plan viewer, route visualization.
- **Backend:**
    - **Preprocessing service** for floor plans (crop, door detection/removal, entrance alignment).
    - **Graph services:** outdoor graph with dynamic weights, indoor raster-A* pathfinder, and global router to stitch indoor ↔ outdoor via mapped entrances.
- **Data:** OSM campus graph, floor-plan images, and weather/crime/construction/traffic feeds.

## How to Use
1. **Load campus:** outdoor graph (OSM) + preprocessed floor plans.
2. **Pick start & destination rooms (or buildings):** app selects nearest entrances and runs outdoor/indoor `A*`.
3. **Toggle filters:**
    - Choose bike/walk
    - Enable safety mode
    - Incorporate weather/traffic overlays for the route

## Limitations & Next Steps
- Improve door detection efficiency and fix mis-placed entrances in OSM to reduce unreachable rooms.
- Expand to more buildings and generalize to other campuses.
- Refine real-time feeds and safety overlays.
