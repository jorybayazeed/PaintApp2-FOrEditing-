# Paint Application - Design Documentation

## Table of Contents
1. [Design Principles](#design-principles)
2. [Component Interactions](#component-interactions)
3. [Data Flow](#data-flow)
4. [Event System](#event-system)
5. [State Management](#state-management)
6. [Plugin Architecture](#plugin-architecture)

## Design Principles

The Paint Application is designed with the following principles:

### 1. Separation of Concerns
- **UI Layer**: Handles rendering and user interactions
- **Logic Layer**: Contains business logic and algorithms
- **Data Layer**: Manages data persistence and state

### 2. Single Responsibility
- Each class/module has one clear responsibility
- Tools are self-contained and independently testable
- Services are focused on specific domains

### 3. Open/Closed Principle
- Open for extension (new tools, filters, exporters)
- Closed for modification (core functionality is stable)

### 4. Dependency Inversion
- High-level modules don't depend on low-level modules
- Both depend on abstractions (interfaces)

## Component Interactions

### Drawing Operation Sequence

```mermaid
sequenceDiagram
    participant User
    participant UI
    participant Controller
    participant ToolManager
    participant Tool
    participant DrawEngine
    participant Canvas
    
    User->>UI: Click on canvas
    UI->>Controller: mouseDown event
    Controller->>ToolManager: getActiveTool()
    ToolManager-->>Controller: currentTool
    Controller->>Tool: onMouseDown(event)
    Tool->>DrawEngine: beginStroke(position, settings)
    DrawEngine->>Canvas: updatePixels(data)
    Canvas-->>UI: render update
    
    User->>UI: Drag mouse
    UI->>Controller: mouseMove event
    Controller->>Tool: onMouseMove(event)
    Tool->>DrawEngine: continueStroke(position)
    DrawEngine->>Canvas: updatePixels(data)
    Canvas-->>UI: render update
    
    User->>UI: Release mouse
    UI->>Controller: mouseUp event
    Controller->>Tool: onMouseUp(event)
    Tool->>DrawEngine: endStroke()
    DrawEngine->>Canvas: finalizeStroke()
    Controller->>StateManager: saveState(canvas)
    StateManager-->>Controller: state saved
    Canvas-->>UI: final render
```

### File Save Operation

```mermaid
sequenceDiagram
    participant User
    participant MenuBar
    participant Controller
    participant FileService
    participant Canvas
    participant FileSystem
    
    User->>MenuBar: Click "Save"
    MenuBar->>Controller: saveCommand()
    Controller->>FileService: save(currentPath)
    
    alt No current path
        FileService->>User: Show "Save As" dialog
        User-->>FileService: Select path and format
    end
    
    FileService->>Canvas: getImageData()
    Canvas-->>FileService: imageData
    FileService->>FileService: encode(imageData, format)
    FileService->>FileSystem: writeFile(path, data)
    FileSystem-->>FileService: success/error
    
    alt Save successful
        FileService-->>Controller: success
        Controller->>User: Show success message
    else Save failed
        FileService-->>Controller: error
        Controller->>User: Show error dialog
    end
```

### Undo/Redo Operation

```mermaid
sequenceDiagram
    participant User
    participant UI
    participant Controller
    participant StateManager
    participant Canvas
    
    User->>UI: Press Ctrl+Z (Undo)
    UI->>Controller: undoCommand()
    Controller->>StateManager: undo()
    
    StateManager->>StateManager: Check undo stack
    alt Undo available
        StateManager->>StateManager: Pop from undo stack
        StateManager->>StateManager: Push current to redo stack
        StateManager->>Canvas: restoreState(previousState)
        Canvas->>Canvas: Apply state
        Canvas-->>UI: Update display
        StateManager-->>Controller: success
    else No undo available
        StateManager-->>Controller: no action
    end
    
    Controller->>UI: Update undo/redo buttons
```

### Color Selection Flow

```mermaid
sequenceDiagram
    participant User
    participant ColorPicker
    participant ColorManager
    participant ToolManager
    participant Tool
    
    User->>ColorPicker: Select color
    ColorPicker->>ColorManager: setForegroundColor(color)
    ColorManager->>ColorManager: Validate color
    ColorManager->>ColorManager: Add to recent colors
    ColorManager-->>ColorPicker: Color updated
    ColorPicker->>UI: Update color display
    
    ColorManager->>ToolManager: getCurrentTool()
    ToolManager-->>ColorManager: activeTool
    ColorManager->>Tool: updateColor(color)
    Tool->>Tool: Apply new color to settings
```

## Data Flow

### Application Startup Data Flow

```mermaid
graph TD
    A[Application Start] --> B[Load Configuration]
    B --> C{Config Valid?}
    C -->|Yes| D[Apply Settings]
    C -->|No| E[Use Defaults]
    D --> F[Initialize Components]
    E --> F
    F --> G[Create Canvas]
    F --> H[Initialize Tools]
    F --> I[Setup UI]
    G --> J[Application Ready]
    H --> J
    I --> J
    J --> K[Wait for User Input]
```

### Drawing Data Flow

```mermaid
graph LR
    A[User Input] --> B[Event Handler]
    B --> C[Tool Manager]
    C --> D[Active Tool]
    D --> E[Drawing Engine]
    E --> F[Canvas Buffer]
    F --> G[Render Pipeline]
    G --> H[Display]
    
    E -.-> I[State Manager]
    I -.-> J[History Stack]
```

### File Operations Data Flow

```mermaid
graph TD
    A[File Operation Request] --> B{Operation Type?}
    B -->|Open| C[File Dialog]
    B -->|Save| D[File Dialog if needed]
    B -->|Export| E[Export Options Dialog]
    
    C --> F[Read File]
    F --> G[Decode Image]
    G --> H[Validate]
    H --> I{Valid?}
    I -->|Yes| J[Load to Canvas]
    I -->|No| K[Show Error]
    
    D --> L[Get Canvas Data]
    L --> M[Encode Image]
    M --> N[Write File]
    N --> O{Success?}
    O -->|Yes| P[Update Status]
    O -->|No| Q[Show Error]
    
    E --> R[Select Format]
    R --> S[Get Export Settings]
    S --> L
```

## Event System

### Event Types and Handlers

```mermaid
graph TB
    subgraph "Mouse Events"
        E1[mousedown]
        E2[mousemove]
        E3[mouseup]
        E4[click]
        E5[dblclick]
        E6[wheel]
    end
    
    subgraph "Keyboard Events"
        K1[keydown]
        K2[keyup]
        K3[keypress]
    end
    
    subgraph "Touch Events"
        T1[touchstart]
        T2[touchmove]
        T3[touchend]
    end
    
    subgraph "Application Events"
        A1[toolchange]
        A2[colorchange]
        A3[canvasresize]
        A4[statechange]
    end
    
    E1 --> Handler[Event Handler]
    E2 --> Handler
    E3 --> Handler
    E4 --> Handler
    E5 --> Handler
    E6 --> Handler
    K1 --> Handler
    K2 --> Handler
    K3 --> Handler
    T1 --> Handler
    T2 --> Handler
    T3 --> Handler
    A1 --> Handler
    A2 --> Handler
    A3 --> Handler
    A4 --> Handler
    
    Handler --> Dispatcher[Event Dispatcher]
    Dispatcher --> Listeners[Event Listeners]
```

### Event Propagation

```javascript
// Event lifecycle
{
  "phase": "capture",      // Capture phase (top-down)
  "phase": "target",       // Target phase (at element)
  "phase": "bubble",       // Bubble phase (bottom-up)
  "defaultPrevented": false,
  "propagationStopped": false
}
```

### Custom Event System

```javascript
// Event emitter pattern
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
  }
  
  emit(event, data) {
    if (!this.events[event]) return;
    this.events[event].forEach(listener => {
      listener(data);
    });
  }
  
  off(event, listener) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter(l => l !== listener);
  }
}
```

## State Management

### Application State Structure

```javascript
{
  // Canvas state
  "canvas": {
    "width": 800,
    "height": 600,
    "zoom": 1.0,
    "pan": { "x": 0, "y": 0 },
    "backgroundColor": "#FFFFFF"
  },
  
  // Tool state
  "tool": {
    "current": "brush",
    "settings": {
      "brush": { "size": 10, "opacity": 1.0, "color": "#000000" },
      "eraser": { "size": 20 },
      "shape": { "type": "rectangle", "filled": false }
    }
  },
  
  // UI state
  "ui": {
    "showGrid": false,
    "showRulers": true,
    "theme": "light",
    "sidebarVisible": true,
    "toolOptionsVisible": true
  },
  
  // File state
  "file": {
    "path": null,
    "modified": false,
    "format": "png"
  },
  
  // History state
  "history": {
    "canUndo": false,
    "canRedo": false,
    "undoStackSize": 0,
    "redoStackSize": 0
  }
}
```

### State Transitions

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Drawing: mouseDown
    Drawing --> Drawing: mouseMove
    Drawing --> Idle: mouseUp
    
    Idle --> SelectingTool: toolButton click
    SelectingTool --> Idle: tool selected
    
    Idle --> SelectingColor: colorPicker open
    SelectingColor --> Idle: color selected
    
    Idle --> FileOperation: file menu click
    FileOperation --> Idle: operation complete
    FileOperation --> Idle: operation cancelled
    
    Idle --> Modified: canvas changed
    Modified --> Idle: saved
    Modified --> Idle: discarded
```

### State Persistence

```mermaid
graph TD
    A[Application State] --> B{Save Trigger?}
    B -->|Manual Save| C[User Clicks Save]
    B -->|Auto Save| D[Timer Trigger]
    B -->|On Exit| E[Window Close]
    
    C --> F[Serialize State]
    D --> F
    E --> F
    
    F --> G[Compress if needed]
    G --> H[Store to Disk]
    H --> I{Storage Type?}
    I -->|Settings| J[Config File]
    I -->|Project| K[Project File]
    I -->|Temp| L[Temp Directory]
```

## Plugin Architecture

### Plugin System Design

```mermaid
graph TB
    subgraph "Core Application"
        A[Plugin Manager]
        B[Plugin Registry]
        C[Plugin Loader]
    end
    
    subgraph "Plugin Interface"
        D[IPlugin]
        E[ITool]
        F[IFilter]
        G[IExporter]
    end
    
    subgraph "Plugins"
        H[Custom Brush Plugin]
        I[Blur Filter Plugin]
        J[SVG Exporter Plugin]
    end
    
    A --> B
    A --> C
    C --> D
    D -.implements.- E
    D -.implements.- F
    D -.implements.- G
    
    H -.implements.- E
    I -.implements.- F
    J -.implements.- G
    
    C -.loads.- H
    C -.loads.- I
    C -.loads.- J
```

### Plugin Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Discovered: Plugin found
    Discovered --> Loaded: Load plugin
    Loaded --> Validated: Validate plugin
    Validated --> Initialized: Initialize
    Initialized --> Active: Activate
    Active --> Suspended: Deactivate
    Suspended --> Active: Reactivate
    Active --> Unloaded: Unload
    Suspended --> Unloaded: Unload
    Unloaded --> [*]
```

### Plugin API Example

```javascript
// Plugin interface
class IPlugin {
  getName() { throw new Error("Not implemented"); }
  getVersion() { throw new Error("Not implemented"); }
  initialize(context) { throw new Error("Not implemented"); }
  shutdown() { throw new Error("Not implemented"); }
}

// Tool plugin interface
class IToolPlugin extends IPlugin {
  getIcon() { throw new Error("Not implemented"); }
  onActivate() { throw new Error("Not implemented"); }
  onDeactivate() { throw new Error("Not implemented"); }
  onMouseDown(event) { throw new Error("Not implemented"); }
  onMouseMove(event) { throw new Error("Not implemented"); }
  onMouseUp(event) { throw new Error("Not implemented"); }
}

// Example custom tool plugin
class SprayPaintTool extends IToolPlugin {
  getName() { return "Spray Paint"; }
  getVersion() { return "1.0.0"; }
  
  initialize(context) {
    this.canvas = context.canvas;
    this.settings = {
      density: 50,
      radius: 20,
      color: "#000000"
    };
  }
  
  onMouseMove(event) {
    if (!event.isDrawing) return;
    
    // Spray paint logic
    const { x, y } = event.position;
    for (let i = 0; i < this.settings.density; i++) {
      const angle = Math.random() * Math.PI * 2;
      const distance = Math.random() * this.settings.radius;
      const px = x + Math.cos(angle) * distance;
      const py = y + Math.sin(angle) * distance;
      this.canvas.drawPixel(px, py, this.settings.color);
    }
  }
}
```

## Tool System Design

### Tool Abstraction

```mermaid
classDiagram
    class ITool {
        <<interface>>
        +getName() String
        +getIcon() Image
        +getCursor() Cursor
        +activate()
        +deactivate()
        +onMouseDown(event)
        +onMouseMove(event)
        +onMouseUp(event)
        +getSettings() ToolSettings
        +setSettings(settings)
    }
    
    class AbstractTool {
        <<abstract>>
        #name: String
        #icon: Image
        #settings: ToolSettings
        +activate()
        +deactivate()
        #drawPreview()
        #finalizeDraw()
    }
    
    class StrokeTool {
        <<abstract>>
        #points: Point[]
        #smoothing: float
        +onMouseMove(event)
        #interpolatePoints()
        #applySmoothing()
    }
    
    class ShapeTool {
        <<abstract>>
        #startPoint: Point
        #endPoint: Point
        #previewShape: Shape
        +onMouseDown(event)
        +onMouseMove(event)
        +onMouseUp(event)
    }
    
    class BrushTool {
        -brushTexture: Image
        -opacity: float
        +onMouseMove(event)
        -applyBrush(position)
    }
    
    class LineTool {
        -antialiasing: bool
        +onMouseUp(event)
        -drawLine(start, end)
    }
    
    class RectangleTool {
        -filled: bool
        -cornerRadius: float
        +onMouseUp(event)
        -drawRectangle(start, end)
    }
    
    ITool <|.. AbstractTool
    AbstractTool <|-- StrokeTool
    AbstractTool <|-- ShapeTool
    StrokeTool <|-- BrushTool
    ShapeTool <|-- LineTool
    ShapeTool <|-- RectangleTool
```

## Drawing Engine Architecture

### Rendering Pipeline

```mermaid
graph TB
    A[Draw Command] --> B{Layer Enabled?}
    B -->|No| Z[Skip]
    B -->|Yes| C[Apply Tool Transform]
    
    C --> D[Rasterize]
    D --> E[Apply Opacity]
    E --> F[Apply Blend Mode]
    
    F --> G[Update Dirty Region]
    G --> H{Optimization Level?}
    
    H -->|Full| I[Render All Layers]
    H -->|Partial| J[Render Dirty Regions]
    
    I --> K[Composite Layers]
    J --> K
    
    K --> L[Apply Canvas Transform]
    L --> M[Display Buffer]
    M --> N[Screen]
```

### Layer Blending

```mermaid
graph LR
    A[Layer N] --> B[Blend Mode]
    C[Layer N-1] --> B
    B --> D[Blended Result]
    
    E[Blend Modes] -.-> B
    
    subgraph "Blend Modes"
        E --> F[Normal]
        E --> G[Multiply]
        E --> H[Screen]
        E --> I[Overlay]
        E --> J[Add]
        E --> K[Subtract]
    end
```

## Performance Optimization Strategies

### Dirty Rectangle Optimization

```javascript
class DirtyRectManager {
  constructor() {
    this.dirtyRegions = [];
  }
  
  addDirtyRect(x, y, width, height) {
    this.dirtyRegions.push({ x, y, width, height });
  }
  
  mergeDirtyRects() {
    // Merge overlapping rectangles
    // Return minimal set of rectangles to redraw
  }
  
  clearDirtyRects() {
    this.dirtyRegions = [];
  }
}
```

### Canvas Tiling for Large Images

```mermaid
graph TB
    A[Large Canvas] --> B[Divide into Tiles]
    B --> C[Tile 1]
    B --> D[Tile 2]
    B --> E[Tile 3]
    B --> F[Tile N]
    
    G[Viewport] -.-> H{Which Tiles Visible?}
    H --> I[Load Visible Tiles]
    H --> J[Unload Hidden Tiles]
    
    I --> K[Render Visible Tiles]
    K --> L[Display]
```

### Brush Stroke Optimization

```javascript
class OptimizedStroke {
  constructor() {
    this.points = [];
    this.minDistance = 2; // Minimum pixel distance between points
  }
  
  addPoint(x, y, pressure = 1.0) {
    if (this.points.length === 0) {
      this.points.push({ x, y, pressure });
      return;
    }
    
    const last = this.points[this.points.length - 1];
    const distance = Math.sqrt((x - last.x) ** 2 + (y - last.y) ** 2);
    
    if (distance >= this.minDistance) {
      this.points.push({ x, y, pressure });
    }
  }
  
  simplify() {
    // Douglas-Peucker algorithm or similar
    // Reduce number of points while maintaining shape
  }
}
```

## Testing Strategy

### Unit Testing Structure

```mermaid
graph TB
    A[Test Suite] --> B[Tool Tests]
    A --> C[Drawing Engine Tests]
    A --> D[File Service Tests]
    A --> E[State Manager Tests]
    
    B --> B1[Brush Tool Tests]
    B --> B2[Shape Tool Tests]
    B --> B3[Eraser Tool Tests]
    
    C --> C1[Rendering Tests]
    C --> C2[Blending Tests]
    C --> C3[Transform Tests]
    
    D --> D1[Load Tests]
    D --> D2[Save Tests]
    D --> D3[Export Tests]
    
    E --> E1[Undo Tests]
    E --> E2[Redo Tests]
    E --> E3[State Persistence Tests]
```

### Integration Testing

```mermaid
sequenceDiagram
    participant Test
    participant App
    participant Tool
    participant Canvas
    
    Test->>App: Initialize()
    Test->>Tool: Select Brush
    Test->>Canvas: Simulate mouseDown(100, 100)
    Test->>Canvas: Simulate mouseMove(150, 150)
    Test->>Canvas: Simulate mouseUp(150, 150)
    Test->>Canvas: getPixel(125, 125)
    Canvas-->>Test: pixel color
    Test->>Test: Assert pixel is not white
```

## Error Handling

### Error Hierarchy

```mermaid
graph TB
    A[Error] --> B[ApplicationError]
    
    B --> C[FileError]
    C --> C1[FileNotFoundError]
    C --> C2[FileFormatError]
    C --> C3[FilePermissionError]
    
    B --> D[CanvasError]
    D --> D1[InvalidDimensionsError]
    D --> D2[OutOfMemoryError]
    
    B --> E[ToolError]
    E --> E1[ToolNotFoundError]
    E --> E2[InvalidToolSettings]
    
    B --> F[StateError]
    F --> F1[CorruptedStateError]
    F --> F2[HistoryLimitError]
```

### Error Recovery Strategy

```javascript
class ErrorHandler {
  handle(error) {
    console.error(error);
    
    if (error instanceof FileNotFoundError) {
      this.showDialog("File not found. Please select a valid file.");
    } else if (error instanceof OutOfMemoryError) {
      this.releaseMemory();
      this.showDialog("Out of memory. Try reducing canvas size.");
    } else if (error instanceof CorruptedStateError) {
      this.recoverFromAutosave();
    } else {
      this.showDialog("An unexpected error occurred. Please restart the application.");
      this.logError(error);
    }
  }
  
  recoverFromAutosave() {
    const autosave = this.loadAutosave();
    if (autosave) {
      this.loadState(autosave);
      this.showDialog("Recovered from autosave.");
    }
  }
}
```

## Accessibility Features

### Keyboard Navigation

```javascript
const shortcuts = {
  // File operations
  'Ctrl+N': 'new',
  'Ctrl+O': 'open',
  'Ctrl+S': 'save',
  'Ctrl+Shift+S': 'saveAs',
  
  // Edit operations
  'Ctrl+Z': 'undo',
  'Ctrl+Y': 'redo',
  'Ctrl+Shift+Z': 'redo',
  'Ctrl+C': 'copy',
  'Ctrl+V': 'paste',
  
  // Tool selection
  'B': 'brush',
  'P': 'pencil',
  'E': 'eraser',
  'L': 'line',
  'R': 'rectangle',
  'C': 'circle',
  
  // View
  'Ctrl++': 'zoomIn',
  'Ctrl+-': 'zoomOut',
  'Ctrl+0': 'zoomReset',
  'Ctrl+G': 'toggleGrid',
};
```

### Screen Reader Support

```javascript
class AccessibilityManager {
  announceToolChange(toolName) {
    this.announce(`${toolName} tool selected`);
  }
  
  announceAction(action) {
    this.announce(action);
  }
  
  announce(message) {
    // Use ARIA live regions
    const liveRegion = document.getElementById('aria-live-region');
    liveRegion.textContent = message;
  }
  
  provideAlternativeText() {
    // Provide alt text for UI elements
    // Describe canvas content when possible
  }
}
```

## Internationalization (i18n)

### Translation Structure

```javascript
const translations = {
  en: {
    menu: {
      file: 'File',
      edit: 'Edit',
      view: 'View',
      help: 'Help'
    },
    tools: {
      brush: 'Brush',
      pencil: 'Pencil',
      eraser: 'Eraser',
      line: 'Line',
      rectangle: 'Rectangle'
    },
    messages: {
      saveSuccess: 'File saved successfully',
      saveError: 'Error saving file',
      unsavedChanges: 'You have unsaved changes. Do you want to save?'
    }
  },
  es: {
    menu: {
      file: 'Archivo',
      edit: 'Editar',
      view: 'Ver',
      help: 'Ayuda'
    },
    // ... Spanish translations
  }
  // ... other languages
};
```

This design documentation provides comprehensive coverage of the Paint Application's design, including component interactions, data flow, event systems, state management, and various architectural patterns that support extensibility and maintainability.
