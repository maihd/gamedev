# GameDev Tips & Tricks
Examples code will C for sake of simple and porting to other languages.


## Philosophy
Iteration is King.


## Techniques
Iteration is King. Repeat it!

- C/C++ unity build
- Hot reload
- Save system
- Replay system
- Time-travel debugging
- Headless automated test (required Save system & Replay system)


## Entity managements
Iteration is King. Repeat again!

- Hot reload is a core feature that make iteration faster
- Hot reload required compilation is fast -> Unity build. C is even compile faster than C++.
- Hot reload have global state
- Hot reload required serialization
- Hot reload can be get helped from Save system & Replay System 

All thoses thing required a entity system and game state is simple and serialization. But thoses problems are solved in previous game era: Fat struct (a.k.a Large Arrays of Things).
```C
enum ThingKind // no need uint8_t or uint16_t, saved 2 or 3 bytes is not worth
{
    ThingKind_None,
    ThingKind_Player,
    ThingKind_Enemy_Goblin,
    ThingKind_Enemy_Orc,

    // If you do UI 
    ThingKind_UI_Button,
    ThingKind_UI_Text,
    ThingKind_UI_Slider,
};

enum ThingFlags // Fast turning on/off features/behaviours of Kind. Use uint64_t of BitSet if your required bigger set
{
    ThingFlags_None,
    // ThingFlags_Movable   = 1 << 0,
    // ThingFlags_AI_Input  = 1 << 1,
    // Blah blah
};

typedef struct ThingHandle
{
    int32_t index;
    int32_t generation;
} ThingHandle;

typedef struct alignas(64) Thing // Cacheline commonly have size of 64 byte
{
    // Header
    uint32_t kind;              // offset 0, in C, enum size is implementation-defined, so we use uint32_t instead
    uint32_t flags;             // offset 4, in C, enum size is implementation-defined, so we use uint32_t instead

    // Properties
    vec3 position;              // offset 8
    quat rotation;              // offset 24 (vec3 is simd)
    vec3 scale;                 // offset 40

    // Relationship
    ThingHandle parent;         // offset 56
    ThingHandle firstChild;     // offset 64
    ThingHandle lastChild;      // offset 72
    ThingHandle nextSibling;    // offset 80 
    ThingHandle prevSibling;    // offset 88

    // Padding, good for hot reloading, serialization
    char padding[160];          // offset 96
} Thing;

// You should control your thing size, 1KB is not too big
// Commonly 256 is big enough for most cases
static_assert(size_of(Thing) == 160, "Thing size is wrong");

// Things container, this is simple implementation, you can use HandleMap instead
#define MAX_THINGS 1024
typedef struct Things
{
    Thing       things[MAX_THINGS];
    bool        used[MAX_THINGS];
    int32_t     free[MAX_THINGS];
    int32_t     firstFree;
} Things;
```


## Gameplay loops
Jakub explain this better than me.
- https://jakubtomsu.github.io/posts/input_in_fixed_timestep/
- https://jakubtomsu.github.io/posts/fixed_timestep_without_interpolation/

Sample code:
```C
bool running = true;
while (running) 
{
    while (frameTime > fixedTimeStep) 
    {
        frameTime -= fixedTimeStep;

        SavePreviousGameState(&gameState, &prevGameState);
        UpdateGameState(&gameState, fixedTimeStep);
    }

    float alpha = clamp01(frameTime / fixedTimeStep);
    DrawGame(&gameState, &prevGameState, alpha);
}
```


## Tweening, Actions, Behaviours and AI
- Because we only have one data type of entity, doing tween can be fast, parallel. Easy to write custom functions.
- This is also true with actions, but in my opinion, we should use actions as set of functions that doing things on entities.
- Behaviours is just a combine of tweening and actions, can have flags to check if behaviour is active.
- AI is higher level, which have state machine, behaviours tree, complex actions, phrasing.


## Game design
Choose the game you like, and clone it. Just make sure make the game fun. The ideas will chase you.


## Memory allocations
- Should only allocating at startup, allocated big enough memory for requirements of each systems
- Entity management is a very first things you allocated in startup, allocate a max amount of entities you can have in game. Because entity is a very movable things, they can create/destroy frequently.
- Use temp allocator and frame allocator if memory is short live
- Below is about game engine memory allocations, sometime you dont need it
- Resources should use bucket allocator (like NaughtyDogs' frame bucket, Media Molecule SOPage)


## If you follows all of theses, you will earn
- Smooth gameplay
- Fast iteration for development game
- Continous developing current level without re-play the level from beginning (specially your game is RPG)
- Determined, good for multiplayer game
- Crash dump, debugging, auto testing are easy as eating cake


## Libraries
- https://github.com/calinou/awesome-gamedev
- https://github.com/Gforcex/OpenGraphic
- My tech stack:
    - SDL3 for platform layer
    - OpenGL in my old games
    - Raylib for something prototyping
    - ImGui for UI Editor
    - Simple retain UI system based on Things
    - Odin or a custom C++ runtime for programming languages. C for practice.
    - Box2D/Box3D for physics
    - stbs for some media loading stuffs
    - cgltf for loading models
    - Spall for profiler
    - RadDebugger for source code debugging
    - LDtk if I doing 2d tilemap games
    - And custom tools come with required of the game


## Project template
Coming soon.


## Mai previous experiences
- beef-gamedev: https://github.com/maihd/beef-gamedev
- raylib-games: https://github.com/maihd/raylib-games
- neonshooter: https://github.com/maihd/neonshooter
- pixel_adventure: https://github.com/maihd/pixel_adventure
- graphics-programming: https://github.com/maihd/graphics-programming
- yolo game programming: https://github.com/maihd/yolo


## Additional resources
- https://www.redblobgames.com/
- Proofs of Fat Struct that worked:
    - https://www.youtube.com/watch?v=1Gce4l5orts
    - https://www.youtube.com/watch?app=desktop&v=wo84LFzx5nI