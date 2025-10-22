# 🏗️ Scratch Card System Architecture

## Component Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                    Contest Platform                          │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┴───────────────────┐
        │                                       │
        ▼                                       ▼
┌───────────────────┐                  ┌──────────────────┐
│ Create Contest    │                  │ Contest View     │
│ Page              │                  │ Page             │
└───────────────────┘                  └──────────────────┘
        │                                       │
        │ Uses                                  │ Uses
        ▼                                       ▼
┌───────────────────┐                  ┌──────────────────┐
│ ScratchCardEditor │                  │ ThreeScratchCard │
│                   │                  │                  │
│ ┌───────────────┐ │                  │ ┌──────────────┐ │
│ │ Image Upload  │ │                  │ │ Three.js     │ │
│ │ Color Picker  │ │                  │ │ Scene        │ │
│ │ Sliders       │ │                  │ │              │ │
│ │ Toggles       │ │                  │ │ Canvas Mask  │ │
│ │ Dropdowns     │ │                  │ │              │ │
│ └───────────────┘ │                  │ │ Scratch      │ │
│         │         │                  │ │ Detection    │ │
│         ▼         │                  │ └──────────────┘ │
│ ┌───────────────┐ │                  └──────────────────┘
│ │ Preview       │ │
│ │ Component     │ │
│ └───────────────┘ │
└───────────────────┘
```

## Data Flow

```
User Action
    │
    ▼
┌─────────────────────────────────────┐
│ ScratchCardEditor                   │
│ - User changes settings             │
│ - Updates config state              │
└─────────────────────────────────────┘
    │
    │ onChange(config)
    ▼
┌─────────────────────────────────────┐
│ Parent Component (CreateContest)    │
│ - Stores config in state            │
│ - Saves to database on submit       │
└─────────────────────────────────────┘
    │
    │ Save to DB
    ▼
┌─────────────────────────────────────┐
│ Database                            │
│ - scratch_card_enabled: boolean     │
│ - scratch_card_config: JSON         │
└─────────────────────────────────────┘
    │
    │ Load contest
    ▼
┌─────────────────────────────────────┐
│ ContestView Component               │
│ - Fetches contest data              │
│ - Passes config to scratch card     │
└─────────────────────────────────────┘
    │
    │ config prop
    ▼
┌─────────────────────────────────────┐
│ ThreeScratchCard                    │
│ - Renders interactive card          │
│ - Handles scratch events            │
│ - Triggers callbacks                │
└─────────────────────────────────────┘
```

## Module Dependencies

```
┌──────────────────────┐
│ index.ts             │  ← Main entry point
│ (Exports all)        │
└──────────────────────┘
          │
          ├─────────────────────────────────────┐
          │                                     │
          ▼                                     ▼
┌──────────────────────┐              ┌──────────────────────┐
│ Components           │              │ Configuration        │
│                      │              │                      │
│ • ThreeScratchCard   │              │ • types.ts           │
│ • Preview            │              │ • scratchCardConfig  │
│ • Editor             │              │ • utils.ts           │
└──────────────────────┘              └──────────────────────┘
          │                                     │
          └─────────────┬───────────────────────┘
                        │
                        ▼
              ┌──────────────────────┐
              │ External Libraries   │
              │                      │
              │ • Three.js           │
              │ • React              │
              │ • Tailwind CSS       │
              └──────────────────────┘
```

## Component Interaction Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    ThreeScratchCard                          │
│                                                              │
│  ┌────────────────┐         ┌────────────────┐             │
│  │ Three.js       │         │ Canvas Mask    │             │
│  │ Renderer       │         │ (Scratch Layer)│             │
│  │                │         │                │             │
│  │ • Scene        │         │ • 2D Context   │             │
│  │ • Camera       │         │ • Draw Lines   │             │
│  │ • Mesh         │         │ • Calculate %  │             │
│  │ • Lighting     │         │                │             │
│  └────────────────┘         └────────────────┘             │
│         │                           │                       │
│         │                           │                       │
│         ▼                           ▼                       │
│  ┌──────────────────────────────────────────┐              │
│  │         Event Handlers                   │              │
│  │                                          │              │
│  │  • onMouseDown / onTouchStart           │              │
│  │  • onMouseMove / onTouchMove            │              │
│  │  • onMouseUp / onTouchEnd               │              │
│  └──────────────────────────────────────────┘              │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────────────────────────────────┐              │
│  │         State Management                 │              │
│  │                                          │              │
│  │  • isScratching                         │              │
│  │  • scratchedPercent                     │              │
│  │  • isRevealed                           │              │
│  └──────────────────────────────────────────┘              │
│         │                                                   │
│         ▼                                                   │
│  ┌──────────────────────────────────────────┐              │
│  │         Callbacks                        │              │
│  │                                          │              │
│  │  • onScratchProgress(percent)           │              │
│  │  • onReveal()                           │              │
│  └──────────────────────────────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

## State Management

```
┌─────────────────────────────────────────────────────────────┐
│                    Application State                         │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ Contest Data │   │ Scratch Card │   │ UI State     │
│              │   │ Config       │   │              │
│ • id         │   │              │   │ • loading    │
│ • name       │   │ • prizeImage │   │ • error      │
│ • enabled    │   │ • coverColor │   │ • revealed   │
│ • config     │   │ • brushSize  │   │ • progress   │
└──────────────┘   │ • threshold  │   └──────────────┘
                   │ • animations │
                   │ • effects    │
                   └──────────────┘
```

## File Responsibilities

| File | Responsibility | Dependencies |
|------|---------------|--------------|
| **ThreeScratchCard.tsx** | Interactive scratch card rendering | Three.js, utils.ts, types.ts |
| **ThreeScratchCardPreview.tsx** | Lightweight preview display | types.ts |
| **ScratchCardEditor.tsx** | Configuration UI | Preview, types.ts, config.ts |
| **types.ts** | Type definitions | None |
| **scratchCardConfig.ts** | Default values & constants | types.ts |
| **utils.ts** | Helper functions | types.ts |
| **index.ts** | Public API exports | All components |
| **scratch-card.css** | Styles & animations | None |

## Rendering Pipeline

```
1. Component Mount
   │
   ├─→ Initialize Three.js Scene
   │   ├─→ Create Camera
   │   ├─→ Create Renderer
   │   ├─→ Load Prize Texture
   │   ├─→ Create Mesh with Material
   │   └─→ Add Lighting
   │
   ├─→ Initialize Scratch Canvas
   │   ├─→ Set Canvas Dimensions
   │   ├─→ Fill with Cover Color
   │   └─→ Overlay Cover Image (if any)
   │
   └─→ Start Animation Loop
       └─→ Render Scene (60 FPS)

2. User Interaction
   │
   ├─→ Touch/Mouse Down
   │   └─→ Set isScratching = true
   │
   ├─→ Touch/Mouse Move
   │   ├─→ Get Canvas Position
   │   ├─→ Draw Scratch Line
   │   ├─→ Calculate Percentage
   │   ├─→ Update State
   │   └─→ Check Threshold
   │
   └─→ Touch/Mouse Up
       └─→ Set isScratching = false

3. Reveal Trigger
   │
   ├─→ Clear Scratch Canvas
   ├─→ Trigger Animation
   ├─→ Trigger Effect
   ├─→ Call onReveal()
   └─→ Update UI
```

## Performance Optimization

```
┌─────────────────────────────────────────────────────────────┐
│                    Optimization Layers                       │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ Rendering    │   │ Event        │   │ Memory       │
│              │   │ Handling     │   │ Management   │
│ • RAF loop   │   │              │   │              │
│ • Conditional│   │ • Debounce   │   │ • Cleanup    │
│   updates    │   │ • Throttle   │   │ • Dispose    │
│ • GPU accel  │   │ • Passive    │   │ • GC hints   │
└──────────────┘   └──────────────┘   └──────────────┘
```

## Error Handling

```
Try/Catch Boundaries
    │
    ├─→ Image Loading
    │   ├─→ Fallback to placeholder
    │   └─→ Log error
    │
    ├─→ Three.js Initialization
    │   ├─→ Check WebGL support
    │   └─→ Graceful degradation
    │
    └─→ Canvas Operations
        ├─→ Validate context
        └─→ Safe defaults
```

## Extension Points

```
┌─────────────────────────────────────────────────────────────┐
│                    Customization Hooks                       │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ Callbacks    │   │ Config       │   │ Styling      │
│              │   │              │   │              │
│ • onReveal   │   │ • Colors     │   │ • CSS vars   │
│ • onProgress │   │ • Sizes      │   │ • Tailwind   │
│ • onReset    │   │ • Thresholds │   │ • Classes    │
└──────────────┘   └──────────────┘   └──────────────┘
```

## Testing Strategy

```
Unit Tests
    │
    ├─→ Utils Functions
    │   ├─→ calculateScratchPercentage()
    │   ├─→ drawScratchLine()
    │   └─→ getCanvasPoint()
    │
    ├─→ Config Validation
    │   └─→ DEFAULT_SCRATCH_CARD_CONFIG
    │
    └─→ Type Checking
        └─→ ScratchCardOptions

Integration Tests
    │
    ├─→ Component Rendering
    ├─→ User Interactions
    └─→ State Updates

E2E Tests
    │
    ├─→ Full Scratch Flow
    ├─→ Mobile Touch Events
    └─→ Reveal Animations
```

## Deployment Checklist

- [ ] Install dependencies (`npm install three @types/three`)
- [ ] Import CSS styles
- [ ] Update database schema
- [ ] Add to contest form
- [ ] Add to contest view
- [ ] Test on desktop
- [ ] Test on mobile
- [ ] Test animations
- [ ] Test callbacks
- [ ] Optimize images
- [ ] Configure CDN (optional)
- [ ] Enable gzip compression
- [ ] Monitor performance
- [ ] Set up analytics
- [ ] Deploy to production

---

**Architecture designed for scalability, maintainability, and performance.**
