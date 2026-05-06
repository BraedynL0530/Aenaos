1. Modular Vision-Language Chain
Vision Backbone (RT-DETR-L): ~100M parameters. Real-time, NMS-free "Eyes" using Global Self-Attention. Extracts Object Queries and Feature Maps.(COCO likely wont work to train for find an alternative, UPDATE: im thinking LVIS) using owl right now, i read somewhere that you can actually use owl to train rtdetr may experiment later.
Linear Projector Bridge: ~5M parameters. Converts raw NN features into Visual Tokens that are "language-ready."
Context Processor (44M Summarizer): Multi-head classifier that adapts visual tokens into a structured descriptor stream – entity + attributes + action_state + spatial + causal link. Interprets the "what" and "how" before sending to the Brain.
2. Efficiency & Delta Logic
Inter-Frame Cache (Component 3): Shared, stateful hub using a descriptor-based protocol (no IDs, no iter). Stores last 30 frames of entity descriptors. Allows Eyes and Brain to fetch and edit a single "truth" about a scene.
Global Motion Compensation (GMC): Uses Optical Flow to detect camera panning. If camera moves, cache coordinates shift; if only an object moves, a Delta-Trigger wakes up the Summarizer.
Delta Trigger: Only sends an update to the Brain when an entity's action_state or spatial relationship changes.
3. The Brain & Learning Loop
Smart Brain (Secondary LLM): 500M–1.9B parameters (e.g., Phi-3-mini, Llama 3.2 1B). Receives descriptor stream via gRPC. Maintains Episodic Context Memory (last ~10 seconds of descriptors) to link events across time.
Real-Time Refinement: Brain can send requests back to the Eyes to "focus" attention on specific entities (e.g., "focus on cup"), refining attention in the Shared Cache.
World Model (Lightweight Physics): <10M parameters. Predicts next action_state based on current state + causal link. Compares prediction to actual delta; if mismatch, triggers refinement.
4. Hardware & Scaling Specifications
Component
Parameters
Memory Estimate
RT-DETR-L
100M
~134 MB
Linear Projector
5M
~20 MB
Summarizer (44M)
44M
~60 MB
Vision Stack Total
149M
~200 MB
Brain (SLM)
500M–1.9B
~1–4 GB (can run on separate edge device)

Performance: Entire vision stack fits in 200MB, optimized for near-instant execution on mobile chips or edge GPUs. Brain can be on same device (if 4GB+ RAM) or cloud/remote edge.
5. Execution Flow
Input: Live video stream (WebSocket).
Detection: RT-DETR(for now im using owl-vit) identifies entities, attributes, boxes, motion deltas.
Delta Check: GMC + Delta Trigger – only proceed if action_state or spatial changed.
Projection: Linear Projector → visual tokens.
Summarizer Inference: 44M model produces descriptor (entity, action_state, causal, spatial, confidence).
Cache Update: Store descriptor in shared cache (last 30 frames). Compute diff from previous state.
Output: Structured descriptor diff sent via gRPC to Brain.
Brain Reasoning: SLM performs task execution, can request refinement.(MANN or RAGG since i cant update brain for learning)

Phases (For Grant Proposal)
Phase 1 (Months 1–9) – Core Descriptor Stream & Cache
Train RT-DETR on MS COCO (object detection + spatial)
Train Summarizer action_state head on Kinetics-700
Train causal link head on CLEVRER (synthetic physics)
Implement cache + diff engine + delta trigger
Integrate with a small Brain (e.g., Llama 3.2 1B) running on CPU/edge
Deliverable: Working prototype that answers "what caused X?" on tabletop videos (10-second clips). Operates at 5–10 FPS on a Jetson Orin.
Success metrics:
Action state classification accuracy ≥85%
Causal link accuracy (agent, verb, patient) ≥80%
Brain's answer matches ground truth on 100 held-out CLEVRER clips
Vision stack <200MB, total system <2GB
Phase 2 (Months 10–18) – Online Adaptation & Episodic Memory
Add lightweight online learning head (e.g., prototypical networks) to update descriptor embeddings from Brain feedback.
Extend episodic memory to minutes (using vector database).
Train on real-world egocentric videos (e.g., EPIC-KITCHENS) for more complex causal chains.
Deliverable: System that improves over time on long-duration tasks (e.g., "what did the hand do 30 seconds ago?").
Phase 3 (Months 19–24) – Deployment & Scaling
Quantize to INT8, target sub-100MB vision stack.
Optimize Brain to 500M parameters (e.g., MobileLLM).
Deploy on a smartphone or drone with real-time (15+ FPS) performance.
Deliverable: Open-source SDK and benchmark for edge video reasoning.

