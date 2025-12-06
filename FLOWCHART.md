# Paint Application Flowchart

This document contains flowcharts depicting the architecture and flow of a Paint application.

## 1. Application Startup Flow

```mermaid
flowchart TD
    Start([Application Start]) --> Init[Initialize Application]
    Init --> LoadConfig[Load Configuration Settings]
    LoadConfig --> CreateUI[Create User Interface]
    CreateUI --> InitCanvas[Initialize Canvas]
    InitCanvas --> SetDefaults[Set Default Tool & Color]
    SetDefaults --> RegisterEvents[Register Event Handlers]
    RegisterEvents --> Ready([Application Ready])
```

## 2. Main Application Architecture

```mermaid
flowchart TD
    User([User]) --> UI[User Interface Layer]
    UI --> ToolBar[Tool Bar]
    UI --> ColorPicker[Color Picker]
    UI --> Canvas[Drawing Canvas]
    UI --> MenuBar[Menu Bar]
    
    ToolBar --> ToolManager[Tool Manager]
    ColorPicker --> ColorManager[Color Manager]
    Canvas --> DrawingEngine[Drawing Engine]
    MenuBar --> FileManager[File Manager]
    
    ToolManager --> Brush[Brush Tool]
    ToolManager --> Pencil[Pencil Tool]
    ToolManager --> Eraser[Eraser Tool]
    ToolManager --> Line[Line Tool]
    ToolManager --> Rectangle[Rectangle Tool]
    ToolManager --> Circle[Circle Tool]
    ToolManager --> Fill[Fill Tool]
    
    DrawingEngine --> CanvasState[Canvas State Manager]
    CanvasState --> UndoRedo[Undo/Redo Stack]
    
    FileManager --> Save[Save Image]
    FileManager --> Load[Load Image]
    FileManager --> Export[Export Image]
```

## 3. Drawing Tool Selection Flow

```mermaid
flowchart TD
    Start([User Clicks Tool]) --> CheckTool{Which Tool?}
    CheckTool -->|Brush| SetBrush[Set Current Tool = Brush]
    CheckTool -->|Pencil| SetPencil[Set Current Tool = Pencil]
    CheckTool -->|Eraser| SetEraser[Set Current Tool = Eraser]
    CheckTool -->|Line| SetLine[Set Current Tool = Line]
    CheckTool -->|Rectangle| SetRect[Set Current Tool = Rectangle]
    CheckTool -->|Circle| SetCircle[Set Current Tool = Circle]
    CheckTool -->|Fill| SetFill[Set Current Tool = Fill]
    
    SetBrush --> UpdateCursor[Update Mouse Cursor]
    SetPencil --> UpdateCursor
    SetEraser --> UpdateCursor
    SetLine --> UpdateCursor
    SetRect --> UpdateCursor
    SetCircle --> UpdateCursor
    SetFill --> UpdateCursor
    
    UpdateCursor --> ShowOptions[Show Tool Options Panel]
    ShowOptions --> End([Ready to Draw])
```

## 4. Drawing Event Handling Flow

```mermaid
flowchart TD
    Start([Mouse Event on Canvas]) --> EventType{Event Type?}
    
    EventType -->|Mouse Down| MouseDown[Store Start Position]
    MouseDown --> SetDrawing[Set isDrawing = true]
    SetDrawing --> InitShape[Initialize Shape/Stroke]
    
    EventType -->|Mouse Move| CheckDrawing{Is Drawing?}
    CheckDrawing -->|Yes| CheckTool{Current Tool?}
    CheckDrawing -->|No| SkipMove[Skip]
    
    CheckTool -->|Brush/Pencil| DrawStroke[Draw Continuous Stroke]
    CheckTool -->|Eraser| EraseStroke[Erase at Position]
    CheckTool -->|Line/Rect/Circle| DrawPreview[Draw Preview Shape]
    CheckTool -->|Fill| SkipMove2[Skip]
    
    DrawStroke --> UpdateCanvas[Update Canvas Display]
    EraseStroke --> UpdateCanvas
    DrawPreview --> UpdateCanvas
    
    EventType -->|Mouse Up| FinalizeShape[Finalize Shape/Stroke]
    FinalizeShape --> AddToHistory[Add to Undo History]
    AddToHistory --> SetNotDrawing[Set isDrawing = false]
    SetNotDrawing --> UpdateCanvas
    
    UpdateCanvas --> End([Wait for Next Event])
    SkipMove --> End
    SkipMove2 --> End
```

## 5. Color Selection Flow

```mermaid
flowchart TD
    Start([User Selects Color]) --> Method{Selection Method?}
    
    Method -->|Color Picker| OpenPicker[Open Color Picker Dialog]
    OpenPicker --> SelectColor[User Selects Color]
    SelectColor --> ValidateColor[Validate Color Value]
    
    Method -->|Palette| ClickPalette[Click Palette Color]
    ClickPalette --> ValidateColor
    
    Method -->|Custom RGB| EnterRGB[Enter RGB Values]
    EnterRGB --> ValidateColor
    
    Method -->|Eyedropper| ActivateEyedropper[Activate Eyedropper Tool]
    ActivateEyedropper --> ClickCanvas[Click on Canvas]
    ClickCanvas --> SampleColor[Sample Color at Position]
    SampleColor --> ValidateColor
    
    ValidateColor --> UpdateCurrent[Update Current Color]
    UpdateCurrent --> UpdateUI[Update Color Display in UI]
    UpdateUI --> End([Color Selected])
```

## 6. File Operations Flow

```mermaid
flowchart TD
    Start([User Menu Action]) --> Action{Which Action?}
    
    Action -->|New| CheckUnsaved{Unsaved Changes?}
    CheckUnsaved -->|Yes| PromptSave[Prompt to Save]
    PromptSave --> UserChoice{User Choice?}
    UserChoice -->|Save| DoSave[Execute Save]
    UserChoice -->|Don't Save| ClearCanvas[Clear Canvas]
    UserChoice -->|Cancel| End([Action Cancelled])
    CheckUnsaved -->|No| ClearCanvas
    DoSave --> ClearCanvas
    ClearCanvas --> ResetState[Reset Canvas State]
    ResetState --> End2([New Canvas Ready])
    
    Action -->|Open| CheckUnsaved2{Unsaved Changes?}
    CheckUnsaved2 -->|Yes| PromptSave2[Prompt to Save]
    CheckUnsaved2 -->|No| OpenDialog[Open File Dialog]
    PromptSave2 --> UserChoice2{User Choice?}
    UserChoice2 -->|Save| DoSave2[Execute Save]
    UserChoice2 -->|Don't Save| OpenDialog
    UserChoice2 -->|Cancel| End
    DoSave2 --> OpenDialog
    OpenDialog --> SelectFile[User Selects File]
    SelectFile --> LoadFile[Load Image File]
    LoadFile --> ValidateFile{Valid Image?}
    ValidateFile -->|Yes| DisplayImage[Display on Canvas]
    ValidateFile -->|No| ShowError[Show Error Message]
    ShowError --> End
    DisplayImage --> End2
    
    Action -->|Save| CheckFilename{Has Filename?}
    CheckFilename -->|Yes| SaveToFile[Save to Existing File]
    CheckFilename -->|No| SaveAsDialog[Show Save As Dialog]
    SaveAsDialog --> UserEnterName[User Enters Filename]
    UserEnterName --> SaveToFile
    SaveToFile --> ShowSuccess[Show Success Message]
    ShowSuccess --> End2
    
    Action -->|Save As| SaveAsDialog
    
    Action -->|Export| ExportDialog[Show Export Dialog]
    ExportDialog --> SelectFormat[Select Export Format]
    SelectFormat --> SelectLocation[Select Save Location]
    SelectLocation --> ExportFile[Export to File]
    ExportFile --> ShowSuccess
```

## 7. Undo/Redo Flow

```mermaid
flowchart TD
    Start([User Action]) --> Action{Which Action?}
    
    Action -->|Undo| CheckUndoStack{Undo Stack Empty?}
    CheckUndoStack -->|Yes| Disabled[Action Disabled]
    CheckUndoStack -->|No| PopUndo[Pop from Undo Stack]
    PopUndo --> PushRedo[Push Current State to Redo Stack]
    PushRedo --> RestoreState[Restore Previous State]
    RestoreState --> UpdateCanvas[Update Canvas Display]
    UpdateCanvas --> End([Action Complete])
    
    Action -->|Redo| CheckRedoStack{Redo Stack Empty?}
    CheckRedoStack -->|Yes| Disabled
    CheckRedoStack -->|No| PopRedo[Pop from Redo Stack]
    PopRedo --> PushUndo[Push Current State to Undo Stack]
    PushUndo --> RestoreState
    
    Action -->|New Drawing| ClearRedo[Clear Redo Stack]
    ClearRedo --> AddToUndo[Add Current State to Undo Stack]
    AddToUndo --> End
    
    Disabled --> End
```

## 8. Canvas State Management

```mermaid
flowchart TD
    Start([Canvas Operation]) --> Operation{Operation Type?}
    
    Operation -->|Draw| CaptureState[Capture Current Canvas State]
    CaptureState --> PerformDraw[Perform Drawing Operation]
    PerformDraw --> SaveToHistory[Save State to History]
    SaveToHistory --> UpdateView[Update Canvas View]
    
    Operation -->|Undo| RetrievePrevious[Retrieve Previous State]
    RetrievePrevious --> ApplyState[Apply State to Canvas]
    ApplyState --> UpdateView
    
    Operation -->|Redo| RetrieveNext[Retrieve Next State]
    RetrieveNext --> ApplyState
    
    Operation -->|Clear| ConfirmClear{Confirm Action?}
    ConfirmClear -->|Yes| ResetCanvas[Reset Canvas to Blank]
    ConfirmClear -->|No| Cancel[Cancel Operation]
    ResetCanvas --> SaveToHistory
    
    Operation -->|Resize| GetNewSize[Get New Dimensions]
    GetNewSize --> ValidateSize{Valid Size?}
    ValidateSize -->|Yes| ResizeCanvas[Resize Canvas]
    ValidateSize -->|No| ShowError[Show Error Message]
    ResizeCanvas --> PreserveContent[Preserve Existing Content]
    PreserveContent --> UpdateView
    ShowError --> Cancel
    
    UpdateView --> End([Operation Complete])
    Cancel --> End
```

## Summary

This flowchart documentation provides a comprehensive overview of a Paint application's architecture and workflows, including:

1. **Application Startup** - How the application initializes
2. **Main Architecture** - Overall structure and component relationships
3. **Tool Selection** - How users switch between drawing tools
4. **Drawing Events** - How mouse events are handled during drawing
5. **Color Selection** - Different methods for choosing colors
6. **File Operations** - Save, load, and export functionality
7. **Undo/Redo** - History management for user actions
8. **Canvas State** - Managing the drawing canvas state

These diagrams can be used as a reference for implementing or understanding the Paint application's functionality.
