# Pixel Fighter

A local two-player fighting game built with Python and Pygame.

## Overview

Pixel Fighter is a prototype fighting game that I developed to practice
object-oriented programming, interactive game loops, collision detection,
character-state management, and frame-based sprite animation.

The game supports two players on the same keyboard and includes movement,
jumping, dashing, attacks, health tracking, animation states, and character
facing logic.

## Features

- Two-player local keyboard controls
- Walking and jumping
- Directional dashing with a rechargeable dash meter
- Multiple attacks
- Attack hitboxes and collision detection
- Health and damage tracking
- Character-facing behavior
- Frame-based sprite animations
- Individual timing values for different animation states
- Death, hurt, idle, running, jumping, dashing, and attacking states

## Project Structure

- `main.py` — game initialization, asset loading, interface drawing, and main loop
- `fighter_class.py` — fighter movement, combat, animation, collision, and state logic
- `requirements.txt` — Python package requirements
- `Samurai_2_Title_Screen.png` — project title artwork

## Controls

### Player 1

- Move: `A` and `D`
- Jump: `W`
- Attack: `R` and `T`
- Dash: Left Shift

### Player 2

- Move: Left and Right Arrow
- Jump: Up Arrow
- Attack: `M` and `N`
- Dash: Right Shift

## Running the Project

Install Python 3 and the required dependency:

```bash
pip install -r requirements.txt
