# Project Description: Adaptive RL 3D Character Intelligence

*Working title. Budget: free tools only. Hardware: RTX 4060 (8 GB VRAM).*

## 1. Overview

This project aims to build a highly optimized intelligence that controls a 3D physics-simulated character and makes dynamic, context-aware decisions. Instead of traditional game AI (behavior trees, state machines, scripted animations), the character learns how to move and how to act through reinforcement learning (RL), so its behavior emerges from experience rather than hand-written rules.

The core idea: the same character should react differently to even tiny changes in its situation, because it is responding to what it perceives, not following a fixed script.

## 2. Purpose and Goals

**Primary goal:** a character that moves like a human and decides like a person in its role.

**Specific goals:**

- Natural, physically believable locomotion: walking, running, jumping, vaulting, crouching, climbing, recovering from stumbles.
- Robustness: the character adapts to pushes, uneven terrain, different obstacles, and changes in body parameters.
- Perception: the character senses and interprets its environment instead of reading perfect world data.
- Role-based decision making: behavior depends on who the character is (a police officer, a pregnant civilian, a bystander) and what is happening around it.
- Efficiency: runs in real time on consumer hardware and can be deployed in a game engine.

**Non-goals (for now):** photorealistic rendering, multiplayer networking, commercial release, human-level general intelligence.

## 3. What Makes This Different From Traditional Game AI

| Traditional game AI | This project |
| --- | --- |
| Pre-made animations blended together | Physics-driven motion produced by a learned controller |
| Hand-written rules and triggers | Behavior learned from rewards and experience |
| Same input gives same scripted output | Small changes in the scene can produce different responses |
| Perfect knowledge of the world | Decisions based on perceived information |
| Fixed personality | Role and personality that change how the character acts |

## 4. System Architecture

The intelligence is split into three layers. This is the most important design decision, because training everything as one giant model is far too hard. Each layer is trained separately, then connected.

### Layer 1: Motor control (the body)

A low-level policy that turns commands into joint movements. It knows how to balance, step, jump, vault, crouch and get up. It is trained with RL and uses motion capture as a style reference so movement looks human.

- Input: body state (joint angles, velocities, orientation), plus a command such as "move toward this point at this speed" or "crouch".
- Output: target joint positions (PD control).

### Layer 2: Skill library

Reusable behaviors built on Layer 1: walk, run, sprint, jump, vault, climb, crouch-walk, take cover, turn and face a target, reach for an item, flee, push away.

### Layer 3: Perception and decision making (the mind)

A high-level system that observes the environment, understands the situation, and chooses which skill to use and where to direct it.

- Perception: starts with simulated senses (ray casts, nearby object lists, sound events), later moves to first-person camera vision.
- Decision making: a high-level policy conditioned on the character's role, goals and emotional state (for example fear level).

## 5. Development Phases

### Phase 1: Foundations

- Set up MuJoCo and a standard humanoid.
- Train basic walking with PPO to confirm the pipeline works.
- Set up GitHub for version control.

### Phase 2: Motion capture and imitation

- Import free motion capture data (CMU Mocap, LAFAN1) and retarget it to the character's skeleton.
- Train the character to imitate clips (DeepMimic-style tracking).
- Note: mocap does not make the model "understand" what should happen. It acts as a demonstration the reward function compares against, so the character learns to move in a human way.

### Phase 3: Skill learning

- Train walk, run, jump, vault, crouch and get-up behaviors.
- Move to adversarial style learning (AMP) so one controller can combine many motions without clip-by-clip tracking.
- Add robustness: random pushes, terrain variation, mass and friction randomization.

### Phase 4: Environment awareness

- Build test environments with obstacles, ledges, doors and moving agents.
- Give the character perception: height map or ray casts first, then egocentric depth or camera images.
- Train navigation and obstacle handling (vault over a wall, duck under a beam).

### Phase 5: Role-based decision making

- Create scenario environments, for example:
  - **Police officer** witnesses a violent crime: take cover, assess threat, call for backup, approach or hold position depending on risk.
  - **Pregnant civilian** sees an attacker nearby: increase distance, call for help, draw pepper spray, protect abdomen, flee toward crowds.
  - **Bystander** with no training: freeze, flee, or help based on personality traits.
- Condition behavior on a role profile (training, equipment, risk tolerance, physical limits).
- Evaluate with scenario tests that change small details (distance, lighting, weapon visible or hidden, crowd size) and check that behavior changes sensibly.

### Phase 6: Optimization and deployment

- Shrink and quantize the policies, export to ONNX.
- Run in Godot or Unity Personal, driving a ragdoll or articulated character in real time.
- Profile and optimize inference speed.

## 6. How Dynamic Decision Making Could Work

Two realistic approaches for Layer 3, which can also be combined:

1. **Pure RL high-level policy.** Train a policy that outputs skill choices and targets, with rewards tied to role goals (stay safe, protect others, complete duties). Most "learned," but needs carefully designed scenarios and rewards, and can take a long time to train.
2. **Language or vision model as the planner.** A small free local model (for example a quantized 7B to 8B model through Ollama or llama.cpp) reads a description of the scene and the character's role, then outputs a high-level intent ("back away and call for help") that Layer 2 executes. This gives rich, flexible reactions much faster, at the cost of slower decision speed. Plan decisions at about 1 to 5 times per second while the motor layer runs at 30 to 60 Hz or more.

A practical path is to begin with option 2 to get convincing behavior early, then replace parts with trained RL policies where speed and responsiveness matter.

## 7. Tools and Technology (all free)

| Purpose | Tool |
| --- | --- |
| Physics simulation | MuJoCo, with MJX / MuJoCo Playground for GPU speed |
| RL algorithms | Stable-Baselines3 (PPO), or Brax / RL-Games for GPU-parallel training |
| Motion capture data | CMU Mocap Database, LAFAN1, AMASS (non-commercial, requires registration) |
| Language | Python, PyTorch or JAX |
| Local LLM (optional) | Ollama or llama.cpp |
| Game engine for deployment | Godot or Unity Personal |
| Model export | ONNX |
| Version control | Git and GitHub |

**Hardware notes (RTX 4060, 8 GB VRAM):** enough for MuJoCo/MJX training with thousands of parallel humanoids. Vision-based training and a local language model running at the same time will compete for VRAM, so train and run them separately. On Windows, use WSL2 for the best JAX GPU support.

## 8. Key Challenges and Risks

- **Reward design.** Characters often exploit the reward in unintended ways. Expect many iterations.
- **Compounding complexity.** Learning movement and high-level judgment together is very hard, which is why the layers are separated.
- **Perception from pixels.** Vision-based RL is much more expensive than state-based RL. Start with simple structured senses.
- **Sim-to-game gap.** A policy trained in MuJoCo may behave differently in Godot or Unity physics. Plan to test early, or run MuJoCo as the runtime.
- **Evaluating "realistic" decisions.** Behavior in social or crisis scenarios has no single right answer. Define scenario checklists and test variations to measure consistency and sensible reactions.
- **Scope.** The full vision is a long-term research-level project. Each phase should produce something working on its own.
- **Content considerations.** Violent scenarios should stay in simulation with abstract or stylized visuals, and the system should be designed for games, research and training simulations.

## 9. Success Criteria

- Phase 3: a single controller performs walk, run, jump, vault and crouch and recovers from pushes at least 90% of the time.
- Phase 4: the character crosses a randomly generated obstacle course using only its own perception.
- Phase 5: given the same scenario with small changes, the character shows meaningfully different but role-appropriate behavior, and two different roles respond differently to the same event.
- Phase 6: runs in real time (60 FPS) on the RTX 4060 inside a game engine.

## 10. Open Questions

- Final target engine: Godot, Unity, or staying in MuJoCo?
- Is the character humanoid-only, or should the system handle different body types?
- How realistic should the "fear" and emotion modeling be?
- How much of Layer 3 should be learned versus language-model driven?
- Is the end goal a game, a research demo, or a training simulation?

## 11. Immediate Next Steps

1. Create the GitHub repository and add this document as the README.
2. Install MuJoCo and train the stock humanoid to walk with PPO.
3. Download a few CMU mocap clips and test retargeting to the humanoid.
4. Pick a first imitation target: a single walk cycle.
