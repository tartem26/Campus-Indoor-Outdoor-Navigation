# Campus Indoor-Outdoor Navigation
Room-to-room campus navigation that links building floor plans to OpenStreetMap for seamless indoor-to-outdoor routing. Implement an end-to-end computer vision pipeline to convert floor plans into routable maps and compute real-time paths with `A*` dynamic edges. Includes a simple Flask API, ReactJS demo, sample data, and a quickstart to run locally.
<img src="https://github.com/user-attachments/assets/44ef83c9-ddc5-49a3-a3f7-6680228800c5" />
<img src="https://github.com/user-attachments/assets/eb8405c6-afeb-48b4-9a3e-f3dd323e0f05" />

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
<img src="https://github.com/user-attachments/assets/4a4b94c2-f741-443d-b3d0-c7af0deb8afe" />

- **Crops floor plans** to the perimeter using OpenCV (scanline/contour extraction), removing legends/whitespace for efficiency.
- **Door detection** by `cv2.matchTemplate` with multi-angle rotations and scale grid search (e.g., `0.9/1.0/1.1`); non-maximum suppression via KD-tree to keep the best match per door.
- **Door removal:** converts door-arc pixels to free space to permit traversal.
  <img src="https://github.com/user-attachments/assets/d1d692ea-fdcf-4a9f-94c3-a6056265bd6c" />
  
- **Entrance mapping:** detects indoor perimeter (contours), centroid alignment, area scale, IoU rotation; moves entrance markers slightly inward for robust indoor routing.
  <img src="https://github.com/user-attachments/assets/7eca2a5a-2c91-4c80-a74e-beca2b555726" />
  <img src="https://github.com/user-attachments/assets/b8d9c8a0-8fdc-4bd7-97f3-22c68ccf32fb" />
  <img src="https://github.com/user-attachments/assets/fbfd3880-f6ad-48f7-8720-5db2be5b5b44" />

## Outdoor Graph & Weights
- **Data:** buildings, entrances, roads/intersections from OpenStreetMap (exported as OSM). Edge base weight = travel time.
- **Adjustments:**
    - **Mode:** walking vs. biking multipliers.
    - **Weather:** wind via dot product, where rain increases edge time.
    - **Safety:** proximity to emergency call stations.
    - **Crime:** historical heatmap multiplier.
      <img src="https://github.com/user-attachments/assets/ade961cf-39f0-40c2-8ac5-c52271d93a72" />
    - **Construction:** closures set edge weight to `∞`.
<img src="https://github.com/user-attachments/assets/7d737b7f-9f3e-4e61-8b62-d4b9974387fa" />
<img src="https://github.com/user-attachments/assets/6a3b670b-2359-4a6d-bb7c-8b8a5b96feed" />

## Traffic Model via Markov Chains
- Builds a transition matrix over intersections.
- Includes self-loops (some travelers stop) and a sink node for exits.
- Computes the stationary distribution and scales by live counter data to estimate per-edge flows used in weights.
<img src="https://github.com/user-attachments/assets/772a803a-4206-4f40-b2a5-c8fd7b63799a" />
<img src="https://github.com/user-attachments/assets/d347e4a5-5a77-4d3d-b7ef-eeb5fc91f7db" />
<img src="https://github.com/user-attachments/assets/c86dc3d6-71f4-4989-ae69-243ba6fa9213" />

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
> From the `demo` folder, launch with `python app.py` after installing all dependencies.

## Limitations & Next Steps
- Improve door detection efficiency and fix mis-placed entrances in OSM to reduce unreachable rooms.
- Expand to more buildings and generalize to other campuses.
- Refine real-time feeds and safety overlays.
