# Glitch Avatar System — Developer Documentation

This document provides comprehensive technical documentation for developers working with the Glitch avatar customization system. It covers the internal architecture, APIs, development workflows, and integration strategies.

## Table of Contents

- [Architecture Deep Dive](#architecture-deep-dive)
- [ActionScript API Reference](#actionscript-api-reference)
- [Asset Pipeline](#asset-pipeline)
- [Flash Development Workflow](#flash-development-workflow)
- [Performance Considerations](#performance-considerations)
- [Extending the System](#extending-the-system)
- [Integration Patterns](#integration-patterns)
- [Troubleshooting](#troubleshooting)
- [Migration Strategies](#migration-strategies)

## Architecture Deep Dive

### Core Class Hierarchy

```
Avatar (Main Controller)
├── ColorMatrix (Color Transformations)
├── AvatarContainer (Display Management)
├── Component Loaders (Dynamic Loading)
└── Rendering Pipeline (Layer Composition)
```

### Avatar.as Class Structure

The 973-line `Avatar.as` class is the heart of the system. Here's its key structure:

```actionscript
public class Avatar extends MovieClip {
    // Component type constants (lines 13-50)
    public const SHIRTTORSO:String = 'shirtTorso';
    public const SLEEVEUPPERCLOSE:String = 'sleeveUpperClose';
    // ... dozens more component definitions
    
    // Core rendering components
    private var skull_color:MovieClip;
    private var torso_color:MovieClip;
    private var avatarContainer_mc:MovieClip;
    
    // Position tracking for dynamic layout
    private var mouth_y:Number;
    private var nose_y:Number;
    private var ear_close_y:Number;
    // ... other positioning variables
    
    // Main initialization and rendering methods
    public function Avatar() { /* Constructor */ }
    private function makeColorMcsForBody():void { /* Body setup */ }
    public function initializeHead(config:Object):void { /* Avatar config */ }
    private function makeColorMCForSkin():MovieClip { /* Color handling */ }
}
```

#### Why This Design Matters

1. **Separation of Concerns**: Each constant represents a distinct render layer
2. **Dynamic Positioning**: Facial features can be adjusted in real-time
3. **Color Independence**: Skin/hair colors are applied via transforms, not asset variants
4. **Modular Loading**: Components can be loaded and unloaded dynamically

### Component Architecture

Each avatar component follows a strict structure:

```
Component (.fla/.swf)
├── Registration Point (0,0 aligned to avatar center)
├── Layer Structure
│   ├── Close Side (facing camera)
│   ├── Off Side (away from camera)  
│   └── Masks (for layering interactions)
├── Color Regions (for tinting)
└── Animation Frames (idle, walk, etc.)
```

#### Critical Design Principles

1. **Consistent Registration Points**: All components align to the same coordinate system
2. **Bilateral Rendering**: Both close and off-side views for 3/4 perspective
3. **Maskable Regions**: Components can hide/reveal parts of other components
4. **Animation Compatibility**: All components support the same animation set

### Rendering Pipeline

The avatar rendering follows this precise order:

```
1. Base Body (skull, torso) - Foundation layer
2. Undergarments - Closest to skin
3. Pants/Skirts - Lower body clothing
4. Shirts - Upper body base layer
5. Coats/Outerwear - Over shirts
6. Socks - Before shoes
7. Shoes - Footwear layer
8. Gloves - Hand coverings
9. Jewelry (rings, bracelets) - Accessories
10. Facial Features (nose, mouth) - Face details
11. Eyes - Critical for character identity
12. Hair - Major visual element
13. Hats/Headwear - Can hide hair
14. Special Effects - Temporary overlays
```

**Why This Order is Critical:**
- **Visual Realism**: Mimics how real clothing layers
- **Interaction Logic**: Upper layers can hide/modify lower layers
- **Performance**: Reduces overdraw in Flash renderer
- **Artist Workflow**: Intuitive for content creators

## ActionScript API Reference

### Core Avatar Methods

#### `initializeHead(config:Object):void`
Configures the avatar appearance from a configuration object.

```actionscript
// Example configuration
var config:Object = {
    "nose_scale": "1.002",          // Nose size multiplier
    "eyes": "01",                   // Eye component ID
    "ears_height": "0",             // Ear vertical offset
    "nose_height": "-0.03",         // Nose vertical offset  
    "hair": "basic",                // Hair component ID
    "skin_tint_color": "D4C159",    // Skin color (hex)
    "hair_tint_color": "696969",    // Hair color (hex)
    "pants": "A011",                // Pants component ID
    "ears": "none",                 // Ear component ID
    "mouth": "01",                  // Mouth component ID
    "eye_scale": "0.897",           // Eye size multiplier
    "shoes": "D011",                // Shoe component ID
    "nose": "none",                 // Nose component ID
    "eye_dist": "-1",               // Eye separation distance
    "skirt": "none",                // Skirt component ID
    "eye_height": "1.03",           // Eye vertical position
    "shirt": "none",                // Shirt component ID
    "dress": "none",                // Dress component ID
    "mouth_scale": "1.002",         // Mouth size multiplier
    "mouth_height": "0",            // Mouth vertical offset
    "ears_scale": "1.002",          // Ear size multiplier
    "necklace": "none",             // Necklace component ID
    "ring": "none",                 // Ring component ID
    "gloves": "none",               // Glove component ID
    "bracelet": "none",             // Bracelet component ID
    "hat": "Bear",                  // Hat component ID
    "ver": "07.02.10"              // Configuration version
};

avatar.initializeHead(config);
```

#### `makeColorMCForSkin(asset:DisplayObject, container:String, clipName:String):MovieClip`
Creates a colored version of a body part for skin tinting.

```actionscript
// Internal usage example
skull_color = makeColorMCForSkin(
    new sideSkull(),           // Asset class
    'sideHeadContainer_mc',    // Container path
    'sideSkullContainer_mc',   // Clip container name
    'sideSkull_mc'            // Clip name
);
```

### ColorMatrix API

The `ColorMatrix.as` class provides advanced color manipulation:

```actionscript
import ColorMatrix;

// Create color matrix for transformation
var matrix:ColorMatrix = new ColorMatrix();

// Apply color tint
matrix.adjustColor(brightness, contrast, saturation, hue);

// Apply to display object
var filter:ColorMatrixFilter = new ColorMatrixFilter(matrix);
displayObject.filters = [filter];
```

#### Common Color Operations

```actionscript
// Skin tone adjustment
function applySkinTone(target:DisplayObject, color:uint):void {
    var transform:ColorTransform = new ColorTransform();
    transform.color = color;
    target.transform.colorTransform = transform;
}

// Hair color with saturation boost
function applyHairColor(target:DisplayObject, color:uint, saturation:Number = 1.2):void {
    var matrix:ColorMatrix = new ColorMatrix();
    matrix.adjustSaturation(saturation);
    matrix.adjustBrightness(0.1);
    
    var transform:ColorTransform = new ColorTransform();
    transform.color = color;
    
    target.transform.colorTransform = transform;
    target.filters = [new ColorMatrixFilter(matrix)];
}
```

### Dynamic Component Loading

```actionscript
public class ComponentLoader extends EventDispatcher {
    private var loader:Loader;
    private var componentType:String;
    
    public function loadComponent(type:String, id:String, callback:Function):void {
        this.componentType = type;
        this.loader = new Loader();
        this.loader.contentLoaderInfo.addEventListener(Event.COMPLETE, onLoadComplete);
        
        var url:String = "assets/" + type + "/" + id + ".swf";
        this.loader.load(new URLRequest(url));
    }
    
    private function onLoadComplete(event:Event):void {
        var component:DisplayObject = loader.content;
        callback(componentType, component);
    }
}
```

## Asset Pipeline

### File Structure Requirements

Each component must follow this naming convention:

```
{category}_{id}.fla     // Flash source file
{category}_{id}.swf     // Compiled Flash movie
```

Examples:
- `ears_alien.fla` / `ears_alien.swf`
- `shirt_hawaiian.fla` / `shirt_hawaiian.swf`
- `nose_0001.fla` / `nose_0001.swf`

### Flash Asset Structure

Every `.fla` file must contain:

```
Timeline
├── Frame 1: Idle state
├── Frame 2-N: Animation frames (optional)
└── Assets
    ├── {component}_close (close-side view)
    ├── {component}_offside (off-side view)
    ├── Masks (interaction masks)
    └── Color regions (for tinting)
```

#### Registration Point Standards

All components must have their registration point at:
- **X**: 0 (centered horizontally)
- **Y**: 0 (aligned to avatar baseline)

This ensures consistent positioning across all components.

### JSFL Automation Script

The `AvatarAssets.jsfl` script automates asset processing:

```javascript
// Extract all library symbols with AS linkage
var doc = fl.getDocumentDOM();
var libItems = doc.library.items;
var classesString = 'var AvatarAssetsClasses = [';

for(var i = 0; i < libItems.length; i++) {
    if(libItems[i].linkageExportForAS) {
        classesString += '"' + libItems[i].linkageClassName + '",';
    }
}
```

**Purpose**: Generates ActionScript class lists for dynamic loading

### Build Process

1. **Asset Creation**: Artists create `.fla` files following structure guidelines
2. **Review**: Ensure registration points and naming comply with standards  
3. **Compile**: Export `.swf` files from Flash IDE
4. **Validation**: Test components with Avatar.as system
5. **Integration**: Add component IDs to game database
6. **Deployment**: Deploy assets to content delivery network

## Flash Development Workflow

### Setting Up Development Environment

1. **Adobe Flash CS3+**: Primary development tool
2. **ActionScript 3**: Programming language
3. **Flash Player 10+**: Runtime for testing
4. **Version Control**: Recommended for `.fla` file management

### Creating New Components

#### Step 1: Analyze Existing Component
```bash
# Open reference component
flash.exe existing_component.fla

# Study layer structure, naming conventions, registration points
```

#### Step 2: Create New Asset
```actionscript
// In Flash IDE
1. File > New > ActionScript 3.0 Class
2. Create artwork on appropriate layers
3. Set registration point to (0,0)
4. Name layers following convention:
   - {component}_close
   - {component}_offside
   - masks (if needed)
```

#### Step 3: Configure Library
```actionscript
// Right-click symbol in library
1. Properties > Advanced
2. Export for ActionScript: ✓
3. Class: {component}_{id}
4. Base class: flash.display.MovieClip
```

#### Step 4: Test Integration
```actionscript
// Create test configuration
var testConfig:Object = {
    "shirt": "your_new_component_id",
    // ... other standard options
};
avatar.initializeHead(testConfig);
```

### Animation Guidelines

All components should support these animation states:

```actionscript
// Frame labels (optional but recommended)
Frame 1: "idle"     // Default standing pose
Frame 10: "walk1"   // Walking animation frame 1  
Frame 20: "walk2"   // Walking animation frame 2
Frame 30: "gesture" // Special gesture animation
```

**Animation Principles:**
- **Consistency**: All components animate together
- **Looping**: Animations should loop seamlessly
- **Performance**: Keep frame counts minimal for performance
- **Flexibility**: Support both looping and one-shot animations

## Performance Considerations

### Memory Management

The avatar system must handle potentially hundreds of components:

```actionscript
// Component caching strategy
private var componentCache:Dictionary = new Dictionary();

public function getComponent(type:String, id:String):DisplayObject {
    var key:String = type + "_" + id;
    
    if (componentCache[key]) {
        return componentCache[key]; // Return cached version
    }
    
    // Load new component
    var component:DisplayObject = loadComponent(type, id);
    componentCache[key] = component;
    return component;
}

// Memory cleanup
public function clearUnusedComponents():void {
    for (var key:String in componentCache) {
        if (!isComponentInUse(key)) {
            delete componentCache[key];
        }
    }
}
```

### Rendering Optimization

```actionscript
// Layer visibility optimization
public function updateComponentVisibility():void {
    // Hide components that are completely covered by others
    if (hasCoat) {
        shirtComponent.visible = false; // Shirt hidden by coat
    }
    
    if (hasHat) {
        hairComponent.visible = false; // Hair hidden by hat
    }
}

// Batch color transformations
public function applyColorTransforms():void {
    // Apply all color changes in single frame to avoid multiple redraws
    var transforms:Array = [skinTransform, hairTransform];
    applyTransformsBatch(transforms);
}
```

### Loading Strategies

```actionscript
// Progressive loading for better UX
public function loadAvatarProgressive(config:Object):void {
    // Priority 1: Essential components (head, body)
    loadComponent("skull", config.skull, onEssentialLoaded);
    loadComponent("torso", config.torso, onEssentialLoaded);
    
    // Priority 2: Visible clothing
    loadComponent("shirt", config.shirt, onClothingLoaded);
    loadComponent("pants", config.pants, onClothingLoaded);
    
    // Priority 3: Accessories and details
    loadComponent("bracelet", config.bracelet, onAccessoryLoaded);
    loadComponent("necklace", config.necklace, onAccessoryLoaded);
}
```

## Extending the System

### Adding New Component Categories

To add a new component type (e.g., "wings"):

1. **Define Constants** in Avatar.as:
```actionscript
public const WINGCLOSE:String = 'wingClose';
public const WINGOFFSIDE:String = 'wingOffside';
```

2. **Add Rendering Logic**:
```actionscript
private function renderWings(config:Object):void {
    if (config.wings && config.wings != "none") {
        var wings:DisplayObject = loadComponent("wings", config.wings);
        // Position wings behind torso but in front of background
        insertComponentAtLayer(wings, WING_LAYER_INDEX);
    }
}
```

3. **Update Initialization**:
```actionscript
public function initializeHead(config:Object):void {
    // ... existing code ...
    renderWings(config);
}
```

4. **Create Asset Templates**:
```bash
# Create wings/ directory structure
mkdir assets/wings/
# Add wing component .fla/.swf files following naming convention
```

### Custom Color Systems

```actionscript
// Advanced color system with multiple tint regions
public class AdvancedColorSystem {
    private var primaryColors:Array = [];
    private var secondaryColors:Array = [];
    
    public function applyMultiToneColors(component:DisplayObject, 
                                       primary:uint, 
                                       secondary:uint):void {
        // Apply primary color to main regions
        var primaryTransform:ColorTransform = new ColorTransform();
        primaryTransform.color = primary;
        
        // Apply secondary color to accent regions  
        var secondaryTransform:ColorTransform = new ColorTransform();
        secondaryTransform.color = secondary;
        
        // Apply to different child clips
        component.getChildByName("primary").transform.colorTransform = primaryTransform;
        component.getChildByName("secondary").transform.colorTransform = secondaryTransform;
    }
}
```

### Animation Extensions

```actionscript
// Custom animation controller
public class AvatarAnimationController {
    private var avatar:Avatar;
    private var currentAnimation:String = "idle";
    
    public function playAnimation(animName:String, loop:Boolean = true):void {
        switch(animName) {
            case "dance":
                playDanceAnimation();
                break;
            case "wave":
                playWaveAnimation();
                break;
            case "jump":
                playJumpAnimation();
                break;
        }
    }
    
    private function playDanceAnimation():void {
        // Coordinate animation across all avatar components
        avatar.avatarContainer_mc.gotoAndPlay("dance");
        syncComponentAnimations("dance");
    }
}
```

## Integration Patterns

### Modern Web Integration

#### HTML5 Canvas Conversion
```javascript
class ModernAvatarRenderer {
    constructor(canvasElement) {
        this.canvas = canvasElement;
        this.ctx = this.canvas.getContext('2d');
        this.components = new Map();
        this.renderOrder = [
            'torso', 'pants', 'shirt', 'coat', 
            'head', 'hair', 'hat', 'accessories'
        ];
    }
    
    async loadComponent(category, id) {
        // Convert SWF to sprite sheets for web use
        const spriteData = await fetch(`/api/sprites/${category}/${id}`);
        const component = await spriteData.json();
        this.components.set(category, component);
    }
    
    render() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        
        for (const componentType of this.renderOrder) {
            const component = this.components.get(componentType);
            if (component) {
                this.renderComponent(component);
            }
        }
    }
    
    renderComponent(component) {
        // Apply color transformations
        if (component.tintColor) {
            this.ctx.globalCompositeOperation = 'multiply';
            this.ctx.fillStyle = component.tintColor;
        }
        
        // Draw component sprite
        this.ctx.drawImage(
            component.image,
            component.x, component.y,
            component.width, component.height
        );
        
        this.ctx.globalCompositeOperation = 'source-over';
    }
}
```

#### React Component Integration
```jsx
import React, { useState, useEffect } from 'react';

const AvatarCustomizer = () => {
    const [avatarConfig, setAvatarConfig] = useState({
        hair: 'basic',
        shirt: 'plain_tee',
        pants: 'plain_trousers'
    });
    
    const [availableComponents, setAvailableComponents] = useState({});
    
    useEffect(() => {
        // Load available components from avatar asset manifest
        fetch('/api/avatar/components')
            .then(res => res.json())
            .then(components => setAvailableComponents(components));
    }, []);
    
    const updateComponent = (category, componentId) => {
        setAvatarConfig(prev => ({
            ...prev,
            [category]: componentId
        }));
    };
    
    return (
        <div className="avatar-customizer">
            <div className="avatar-preview">
                <AvatarRenderer config={avatarConfig} />
            </div>
            
            <div className="component-selectors">
                {Object.entries(availableComponents).map(([category, components]) => (
                    <ComponentSelector
                        key={category}
                        category={category}
                        components={components}
                        selected={avatarConfig[category]}
                        onChange={(componentId) => updateComponent(category, componentId)}
                    />
                ))}
            </div>
        </div>
    );
};
```

### Game Engine Integration

#### Unity Integration
```csharp
using UnityEngine;
using System.Collections.Generic;

public class UnityAvatarSystem : MonoBehaviour {
    [System.Serializable]
    public class AvatarConfig {
        public string hair;
        public string shirt;
        public string pants;
        public Color skinColor = Color.white;
        public Color hairColor = Color.black;
    }
    
    [SerializeField] private AvatarConfig config;
    [SerializeField] private Transform avatarRoot;
    
    private Dictionary<string, GameObject> loadedComponents = new Dictionary<string, GameObject>();
    
    public void ApplyConfiguration(AvatarConfig newConfig) {
        config = newConfig;
        
        // Clear existing components
        ClearComponents();
        
        // Load new components in correct order
        LoadComponent("torso", "base");
        LoadComponent("pants", config.pants);
        LoadComponent("shirt", config.shirt);
        LoadComponent("hair", config.hair);
        
        // Apply colors
        ApplyColors();
    }
    
    private void LoadComponent(string category, string componentId) {
        if (string.IsNullOrEmpty(componentId) || componentId == "none") return;
        
        string resourcePath = $"AvatarComponents/{category}/{componentId}";
        GameObject prefab = Resources.Load<GameObject>(resourcePath);
        
        if (prefab != null) {
            GameObject instance = Instantiate(prefab, avatarRoot);
            loadedComponents[category] = instance;
        }
    }
    
    private void ApplyColors() {
        // Apply skin color to relevant components
        if (loadedComponents.ContainsKey("torso")) {
            var renderer = loadedComponents["torso"].GetComponent<SpriteRenderer>();
            renderer.color = config.skinColor;
        }
        
        // Apply hair color
        if (loadedComponents.ContainsKey("hair")) {
            var renderer = loadedComponents["hair"].GetComponent<SpriteRenderer>();
            renderer.color = config.hairColor;
        }
    }
}
```

#### Unreal Engine Integration
```cpp
// AvatarSystemComponent.h
#pragma once

#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "Engine/Texture2D.h"
#include "AvatarSystemComponent.generated.h"

USTRUCT(BlueprintType)
struct FAvatarConfiguration {
    GENERATED_BODY()
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FString Hair;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FString Shirt;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FString Pants;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FLinearColor SkinColor;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FLinearColor HairColor;
};

UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class YOURGAME_API UAvatarSystemComponent : public UActorComponent {
    GENERATED_BODY()

public:
    UAvatarSystemComponent();
    
    UFUNCTION(BlueprintCallable, Category = "Avatar")
    void ApplyConfiguration(const FAvatarConfiguration& Config);
    
    UFUNCTION(BlueprintCallable, Category = "Avatar")
    void LoadComponent(const FString& Category, const FString& ComponentId);

private:
    UPROPERTY(VisibleAnywhere, Category = "Avatar")
    TMap<FString, UTexture2D*> LoadedComponents;
    
    UPROPERTY(EditAnywhere, Category = "Avatar")
    FAvatarConfiguration CurrentConfig;
};
```

### Mobile Platform Adaptations

#### iOS/Android Considerations
```swift
// iOS Swift implementation
class MobileAvatarRenderer {
    private var spriteCache: [String: UIImage] = [:]
    private let renderQueue = DispatchQueue(label: "avatar.render")
    
    func loadAvatar(config: AvatarConfig, completion: @escaping (UIImage?) -> Void) {
        renderQueue.async {
            var layers: [UIImage] = []
            
            // Load components in render order
            let renderOrder = ["torso", "pants", "shirt", "hair"]
            
            for component in renderOrder {
                if let componentId = config[component],
                   let sprite = self.loadSprite(category: component, id: componentId) {
                    layers.append(sprite)
                }
            }
            
            // Composite layers into final image
            let finalImage = self.compositeImages(layers)
            
            DispatchQueue.main.async {
                completion(finalImage)
            }
        }
    }
    
    private func loadSprite(category: String, id: String) -> UIImage? {
        let cacheKey = "\(category)_\(id)"
        
        if let cached = spriteCache[cacheKey] {
            return cached
        }
        
        guard let path = Bundle.main.path(forResource: id, ofType: "png", inDirectory: category),
              let image = UIImage(contentsOfFile: path) else {
            return nil
        }
        
        spriteCache[cacheKey] = image
        return image
    }
}
```

## Troubleshooting

### Common Issues

#### Component Not Appearing
```actionscript
// Debug checklist:
1. Verify file path: assets/{category}/{id}.swf exists
2. Check naming convention: category_id format
3. Confirm registration point: (0,0) alignment
4. Test layer order: component might be behind others
5. Validate ActionScript linkage: class properly exported

// Debug code:
trace("Loading component:", category, id);
trace("File exists:", File.exists("assets/" + category + "/" + id + ".swf"));
trace("Component loaded:", component != null);
trace("Component visible:", component.visible);
```

#### Color Tinting Not Working
```actionscript
// Common causes:
1. Component doesn't have color regions defined
2. ColorTransform applied to wrong display object
3. Blend mode conflicts with color transformation
4. Component uses bitmap fills instead of vector fills

// Debug solution:
public function debugColorTransform(target:DisplayObject):void {
    trace("Target type:", getQualifiedClassName(target));
    trace("Transform exists:", target.transform.colorTransform != null);
    trace("Current color:", target.transform.colorTransform.color.toString(16));
    
    // Test with basic red tint
    var testTransform:ColorTransform = new ColorTransform();
    testTransform.color = 0xFF0000;
    target.transform.colorTransform = testTransform;
}
```

#### Performance Issues
```actionscript
// Performance monitoring
public class AvatarProfiler {
    private var renderStartTime:Number;
    private var componentLoadTimes:Dictionary = new Dictionary();
    
    public function startRender():void {
        renderStartTime = getTimer();
    }
    
    public function endRender():void {
        var renderTime:Number = getTimer() - renderStartTime;
        trace("Total render time:", renderTime, "ms");
        
        if (renderTime > 100) { // Flag slow renders
            trace("WARNING: Slow avatar render detected");
            optimizeRendering();
        }
    }
    
    private function optimizeRendering():void {
        // Disable unnecessary filters
        // Reduce animation frame rate
        // Cache frequently used components
    }
}
```

#### Layer Ordering Problems
```actionscript
// Layer debugging
public function debugLayers():void {
    trace("=== Avatar Layer Debug ===");
    
    for (var i:int = 0; i < avatarContainer_mc.numChildren; i++) {
        var child:DisplayObject = avatarContainer_mc.getChildAt(i);
        trace("Layer", i, ":", getQualifiedClassName(child), "visible:", child.visible);
    }
    
    trace("=== End Layer Debug ===");
}

// Fix layer ordering
public function fixLayerOrder():void {
    var correctOrder:Array = [
        "torso", "pants", "shirt", "coat",
        "skull", "nose", "mouth", "eyes", "hair", "hat"
    ];
    
    for (var i:int = 0; i < correctOrder.length; i++) {
        var component:DisplayObject = findComponentByType(correctOrder[i]);
        if (component && component.parent == avatarContainer_mc) {
            avatarContainer_mc.setChildIndex(component, i);
        }
    }
}
```

### Flash-Specific Issues

#### SWF Loading Failures
```actionscript
// Robust loading with error handling
public function loadComponentSafe(category:String, id:String, callback:Function):void {
    var loader:Loader = new Loader();
    var loaderContext:LoaderContext = new LoaderContext();
    loaderContext.allowCodeImport = true;
    
    loader.contentLoaderInfo.addEventListener(Event.COMPLETE, function(e:Event):void {
        callback(loader.content, null);
    });
    
    loader.contentLoaderInfo.addEventListener(IOErrorEvent.IO_ERROR, function(e:IOErrorEvent):void {
        trace("Failed to load:", category, id, "Error:", e.text);
        callback(null, e.text);
    });
    
    loader.contentLoaderInfo.addEventListener(SecurityErrorEvent.SECURITY_ERROR, function(e:SecurityErrorEvent):void {
        trace("Security error loading:", category, id, "Error:", e.text);
        callback(null, e.text);
    });
    
    var url:String = "assets/" + category + "/" + id + ".swf";
    loader.load(new URLRequest(url), loaderContext);
}
```

#### Cross-Domain Policy Issues
```xml
<!-- crossdomain.xml for asset hosting -->
<?xml version="1.0"?>
<cross-domain-policy>
    <allow-access-from domain="*.yourgame.com" />
    <allow-access-from domain="*.cdn.yourgame.com" />
    <allow-http-request-headers-from domain="*" headers="*" />
</cross-domain-policy>
```

## Migration Strategies

### From Flash to Modern Platforms

#### Phase 1: Asset Conversion
```bash
# Convert SWF to sprite sheets using tools like Swiffy or custom scripts
for file in assets/*/*.swf; do
    echo "Converting $file to sprite sheet..."
    swf2sprites "$file" "${file%.swf}.json"
done

# Generate metadata for each component
python generate_component_metadata.py
```

#### Phase 2: Logic Migration
```typescript
// TypeScript equivalent of Avatar.as core logic
interface AvatarConfig {
    hair?: string;
    shirt?: string;
    pants?: string;
    skinColor?: string;
    hairColor?: string;
    // ... other properties
}

class ModernAvatar {
    private components: Map<string, HTMLImageElement> = new Map();
    private container: HTMLElement;
    
    constructor(containerId: string) {
        this.container = document.getElementById(containerId)!;
    }
    
    async configure(config: AvatarConfig): Promise<void> {
        // Clear existing components
        this.container.innerHTML = '';
        
        // Load components in render order
        const renderOrder = ['torso', 'pants', 'shirt', 'hair'];
        
        for (const category of renderOrder) {
            const componentId = config[category as keyof AvatarConfig];
            if (componentId && componentId !== 'none') {
                await this.loadComponent(category, componentId);
            }
        }
        
        // Apply colors
        this.applyColors(config);
    }
    
    private async loadComponent(category: string, id: string): Promise<void> {
        const img = new Image();
        img.src = `/assets/${category}/${id}.png`;
        
        return new Promise((resolve, reject) => {
            img.onload = () => {
                this.components.set(category, img);
                this.container.appendChild(img);
                resolve();
            };
            img.onerror = reject;
        });
    }
    
    private applyColors(config: AvatarConfig): void {
        // Modern CSS filter-based color manipulation
        if (config.skinColor) {
            const torso = this.components.get('torso');
            if (torso) {
                torso.style.filter = `hue-rotate(${this.hexToHue(config.skinColor)}deg)`;
            }
        }
    }
    
    private hexToHue(hex: string): number {
        // Convert hex color to hue rotation value
        // Implementation depends on specific color mapping requirements
        return 0;
    }
}
```

#### Phase 3: Feature Parity
```javascript
// Ensure all Flash features are replicated
const featureMatrix = {
    'component_loading': '✅ Implemented',
    'color_tinting': '✅ Implemented', 
    'layer_ordering': '✅ Implemented',
    'animation_support': '⚠️  Partial - basic animations only',
    'dynamic_scaling': '❌ TODO',
    'performance_optimization': '⚠️  Partial - needs more work'
};

// Migration validation checklist
function validateMigration() {
    const tests = [
        () => testComponentLoading(),
        () => testColorTransforms(),
        () => testLayerOrdering(),
        () => testPerformance()
    ];
    
    tests.forEach((test, index) => {
        try {
            test();
            console.log(`✅ Test ${index + 1} passed`);
        } catch (error) {
            console.log(`❌ Test ${index + 1} failed:`, error);
        }
    });
}
```

### Backward Compatibility

```actionscript
// Maintain Flash version for legacy support
public class CompatibilityAvatar extends Avatar {
    private var modernBridge:ModernAvatarBridge;
    
    public function CompatibilityAvatar() {
        super();
        
        // Initialize bridge to modern system if available
        if (ExternalInterface.available) {
            modernBridge = new ModernAvatarBridge();
        }
    }
    
    override public function initializeHead(config:Object):void {
        super.initializeHead(config);
        
        // Sync with modern system
        if (modernBridge) {
            modernBridge.syncConfiguration(config);
        }
    }
}
```

This comprehensive developer documentation provides the technical depth needed to understand, modify, and extend the Glitch avatar system. Whether you're maintaining the original Flash implementation or migrating to modern platforms, this guide covers the essential architectural knowledge and practical implementation details.