# Glitch Avatars — Comprehensive Avatar Customization System

<a href="http://www.glitch.com">Glitch</a> was a browser-based MMO created by 
<a href="http://tinyspeck.com">Tiny Speck</a>. This repository contains the
complete art assets, source code, and Flash files for the game's highly sophisticated 
avatar customization system—one of the most comprehensive player personalization 
systems ever created for an online game.

## Table of Contents

- [Why This System is Significant](#why-this-system-is-significant)
- [System Architecture Overview](#system-architecture-overview)
- [Repository Structure](#repository-structure)
- [Creating Your Own Avatar](#creating-your-own-avatar)
- [Integration in Game Development](#integration-in-game-development)
- [Technical Implementation](#technical-implementation)
- [Getting Started](#getting-started)
- [Developer Resources](#developer-resources)
- [License](#license)

## Why This System is Significant

The Glitch avatar system represents a breakthrough in MMO character customization for several critical reasons:

### **Unprecedented Customization Depth**
Unlike typical games with 10-20 appearance options, Glitch offered thousands of combinations across:
- **Facial Features**: 5 categories (ears, eyes, hair, mouth, nose) with dozens of options each
- **Wardrobe System**: 8 clothing categories with hundreds of unique items
- **Layered Rendering**: Complex multi-layer system supporting coats over shirts, accessories, and more
- **Color Customization**: Dynamic skin and hair tinting with real-time color transformations

### **Modular Design Philosophy**
Each avatar component is completely modular and interchangeable, allowing for:
- **Infinite Scalability**: New items can be added without breaking existing combinations
- **Performance Optimization**: Only required assets are loaded per character
- **Memory Efficiency**: Shared base components reduce memory footprint
- **Content Pipeline**: Artists can work on components independently

### **Advanced Technical Implementation**
The system demonstrates sophisticated game development techniques:
- **Layered Animation System**: Characters maintain consistent animations across all customizations
- **Dynamic Asset Loading**: Real-time loading and compositing of character components
- **Color Matrix Transformations**: Advanced shader-like color manipulation in Flash
- **Scalable Architecture**: Supports both 2D sprite generation and real-time rendering

## System Architecture Overview

The avatar system operates on a three-tier architecture:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Base Avatar   │────│  Vanity System  │────│ Wardrobe System │
│   (Foundation)  │    │ (Facial Features)│    │   (Clothing)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │  Avatar Engine  │
                    │  (Compositor)   │
                    └─────────────────┘
```

### **Why This Architecture Matters in Game Development:**

1. **Separation of Concerns**: Each system handles distinct aspects of character appearance
2. **Independent Scaling**: Teams can work on clothing, faces, and core systems simultaneously  
3. **Performance Control**: Systems can be optimized independently based on usage patterns
4. **Content Management**: Artists and developers have clear ownership boundaries
5. **Quality Assurance**: Testing can be compartmentalized by system

## Repository Structure

### `/base_avatar/` - Core Avatar Engine
**Significance**: The foundation that makes all customization possible

- **`Avatar.as`** (973 lines): Main rendering engine that composites all avatar components
  - *Why it matters*: Handles complex layering logic that ensures clothing renders correctly
  - *Game dev impact*: Reference implementation for character rendering in 2D games
  
- **`Avatar.fla`**: Flash source file containing the base character rig and animations
  - *Why it matters*: Defines the skeletal structure that all customizations must conform to
  - *Game dev impact*: Template for creating consistent character proportions
  
- **`ColorMatrix.as`**: Advanced color transformation system
  - *Why it matters*: Enables real-time skin/hair color changes without pre-rendered variants
  - *Game dev impact*: Technique applicable to any game requiring dynamic recoloring

### `/vanity/` - Facial Customization System
**Significance**: Provides character identity and personality expression

Each subdirectory represents a facial feature category:

- **`/ears/`**: 16+ ear variations (pointed, alien, buddha, fish, etc.)
  - *Why it matters*: Enables species diversity and fantasy character creation
  - *Game dev impact*: Shows how to support non-human character types
  
- **`/eyes/`**: 40+ eye styles (cyclops, alien, goth, rainbow, etc.)
  - *Why it matters*: Primary method of character personality expression
  - *Game dev impact*: Demonstrates animation techniques for facial features
  
- **`/hair/`**: Dozens of hairstyles with color variation support
  - *Why it matters*: Most visible customization option, critical for player identity
  - *Game dev impact*: Reference for implementing dynamic hair color systems
  
- **`/mouth/`**: Various mouth shapes and expressions
  - *Why it matters*: Affects character personality and emotion representation
  - *Game dev impact*: Foundation for facial animation and expression systems
  
- **`/nose/`**: Multiple nose shapes and accessories (piercings, etc.)
  - *Why it matters*: Subtle but important for character distinctiveness
  - *Game dev impact*: Shows attention to detail in character customization

### `/wardrobe/` - Clothing and Accessories System
**Significance**: Enables role-playing, self-expression, and game progression rewards

- **`/shirt/`**: 100+ shirt variations
  - *Significance*: Foundation garment that affects how other clothing layers render
  - *Game dev impact*: Demonstrates complex layering systems in 2D games
  
- **`/coat/`**: Outerwear that renders over shirts
  - *Significance*: Shows advanced layering where one item affects visibility of others
  - *Game dev impact*: Reference for implementing equipment that modifies other equipment
  
- **`/pants/` & `/skirt/`**: Lower body clothing with gender flexibility
  - *Significance*: Supports diverse character presentations and cultural expression
  - *Game dev impact*: Shows inclusive character design practices
  
- **`/shoes/`**: Footwear that completes outfits
  - *Significance*: Often overlooked but critical for complete character presentation
  - *Game dev impact*: Demonstrates attention to complete character design
  
- **`/hat/`**: Headwear that can hide or modify hair appearance
  - *Significance*: Shows complex interaction between different customization systems
  - *Game dev impact*: Reference for items that affect multiple character features
  
- **`/bracelet/`**: Accessories that add character detail
  - *Significance*: Small details that contribute to overall character richness
  - *Game dev impact*: Shows how micro-customizations enhance player investment

## Creating Your Own Avatar

### Prerequisites
- Adobe Flash CS3 or later (for editing .fla files)
- Basic understanding of Flash/ActionScript development
- Image editing software (optional, for creating new assets)

### Step-by-Step Avatar Creation Process

#### Method 1: Using Existing Components (Beginner)

1. **Choose Your Base Features**:
   ```
   Navigate to /vanity/ folders and select:
   - One file from /ears/ (e.g., ears_0001.swf)
   - One file from /eyes/ (e.g., eyes_01.swf)  
   - One file from /hair/ (e.g., hair_basic.swf)
   - One file from /mouth/ (e.g., mouth_01.swf)
   - One file from /nose/ (e.g., nose_0001.swf)
   ```

2. **Select Wardrobe Items**:
   ```
   Choose clothing from /wardrobe/ folders:
   - /shirt/ - Base clothing layer
   - /pants/ or /skirt/ - Lower body
   - /shoes/ - Footwear
   - /hat/ (optional) - Headwear
   - Accessories from /bracelet/, /coat/, etc.
   ```

3. **Configure the Avatar** (using Avatar.as):
   ```actionscript
   // Example avatar configuration
   var avatarConfig = {
       "ears": "ears_0001",
       "eyes": "eyes_01", 
       "hair": "hair_basic",
       "mouth": "mouth_01",
       "nose": "nose_0001",
       "shirt": "plain_tee_light",
       "pants": "plain_trousers",
       "shoes": "basic_shoes",
       "skin_tint_color": "D4C159",
       "hair_tint_color": "696969"
   };
   ```

4. **Load and Render**:
   The Avatar.as class will automatically composite these components into a complete character.

#### Method 2: Creating Custom Components (Advanced)

1. **Analyze Existing Components**:
   - Open any .fla file from the vanity or wardrobe folders
   - Study the layer structure and naming conventions
   - Note the registration points and animation frames

2. **Create New Component**:
   - Use existing .fla as template
   - Replace artwork while maintaining structure
   - Ensure proper naming of layers and symbols
   - Test registration points align with base avatar

3. **Export and Test**:
   - Export as .swf with identical naming convention
   - Test integration with Avatar.as system
   - Verify layering works correctly with other components

### Advanced Customization Techniques

#### Color Tinting System
The ColorMatrix.as system allows dynamic recoloring:
```actionscript
// Apply custom skin tone
var skinColor = new ColorTransform();
skinColor.color = 0xD4C159; // Hex color value
avatar.applySkinTint(skinColor);
```

#### Animation Integration
All components support the base animation system:
- Idle stance animation
- Walking animation  
- Gesture animations
- Expression changes

#### Layering Rules
Understanding the rendering order is crucial:
1. Base body (skull, torso)
2. Undergarments
3. Pants/skirts
4. Shirts
5. Coats/outerwear
6. Accessories (bracelets, etc.)
7. Facial features
8. Hair
9. Hats/headwear

## Integration in Game Development

### Why Use This System in Your Game

1. **Player Retention**: Deep customization increases player investment and time spent in-game
2. **Monetization**: Cosmetic items provide non-pay-to-win revenue opportunities  
3. **Community Building**: Unique avatars foster player identity and social interaction
4. **Content Pipeline**: Modular system allows for continuous content updates
5. **Performance**: Efficient rendering system suitable for MMO environments

### Integration Strategies

#### For 2D Games
- Use the Flash rendering system directly
- Export components as sprite sheets
- Implement the layering logic in your game engine
- Adapt the color tinting system for your platform

#### For 3D Games  
- Use 2D avatars as UI elements (profiles, chat, etc.)
- Convert components to textures for 3D character customization
- Adapt the modular design philosophy to 3D assets
- Use the color system for material property variations

#### For Web Games
- Direct Flash implementation (legacy support)
- Convert to HTML5 Canvas or WebGL
- Use as reference for CSS-based avatar systems
- Implement in JavaScript game frameworks

### Adaptation Examples

```javascript
// Example: Converting to HTML5 Canvas
class HTMLAvatarRenderer {
    constructor(canvasId) {
        this.canvas = document.getElementById(canvasId);
        this.ctx = this.canvas.getContext('2d');
        this.components = {};
    }
    
    loadComponent(type, filename) {
        // Load image equivalent of .swf file
        const img = new Image();
        img.src = `assets/${type}/${filename}.png`;
        this.components[type] = img;
    }
    
    render() {
        // Implement layering logic from Avatar.as
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        
        // Render in correct order
        const renderOrder = ['torso', 'pants', 'shirt', 'head', 'hair', 'hat'];
        renderOrder.forEach(component => {
            if (this.components[component]) {
                this.ctx.drawImage(this.components[component], 0, 0);
            }
        });
    }
}
```

## Technical Implementation

### Core Technologies
- **Adobe Flash/ActionScript 3**: Primary development platform
- **Flash Player**: Runtime environment
- **JSFL (Flash JavaScript)**: Asset pipeline automation
- **SWF Format**: Compiled asset format

### Key Technical Concepts

#### Dynamic Asset Loading
```actionscript
// Example from Avatar.as
public function loadComponent(type:String, id:String):void {
    var loader:Loader = new Loader();
    var url:String = "assets/" + type + "/" + id + ".swf";
    loader.load(new URLRequest(url));
    loader.contentLoaderInfo.addEventListener(Event.COMPLETE, onComponentLoaded);
}
```

#### Layered Rendering System
The avatar uses complex layer management:
- Close-side body parts (facing camera)
- Off-side body parts (away from camera)  
- Dynamic masking for clothing interactions
- Z-depth sorting for proper layering

#### Color Transformation
Advanced color manipulation without asset duplication:
```actionscript
// Skin tinting implementation
public function applySkinTint(color:uint):void {
    var colorTransform:ColorTransform = new ColorTransform();
    colorTransform.color = color;
    skull_color.transform.colorTransform = colorTransform;
    torso_color.transform.colorTransform = colorTransform;
}
```

## Getting Started

### For Game Developers
1. **Study the Architecture**: Review Avatar.as to understand the rendering pipeline
2. **Experiment with Components**: Load different combinations to see the system in action
3. **Analyze Asset Structure**: Open .fla files to understand the component format
4. **Plan Your Adaptation**: Decide how to integrate with your game engine
5. **Start Small**: Begin with a subset of components for proof of concept

### For Content Creators
1. **Install Flash Tools**: Get Adobe Flash for editing .fla files
2. **Study Templates**: Open existing components to understand the format
3. **Create Test Assets**: Make simple modifications to existing components
4. **Test Integration**: Verify your assets work with the Avatar.as system
5. **Build Library**: Create a collection of custom components

### For Researchers/Students
1. **Analyze Code Structure**: Study Avatar.as for software architecture patterns
2. **Research Flash Technology**: Understand the historical context of Flash in games
3. **Study Modular Design**: Learn from the component-based architecture
4. **Explore Color Theory**: Examine the ColorMatrix implementation
5. **Compare to Modern Systems**: Contrast with current avatar customization approaches

## Developer Resources

- **[README-DEV.md](./README-DEV.md)**: Comprehensive technical documentation
- **[glitch-client](http://github.com/tinyspeck/glitch-client)**: Original game client implementation
- **Flash Documentation**: Adobe ActionScript 3 reference materials
- **Community Forums**: Game development communities for Flash/2D game development

## License

All files are provided by Tiny Speck under the 
<a href="http://creativecommons.org/publicdomain/zero/1.0/legalcode">Creative
Commons CC0 1.0 Universal License</a>. This is a broadly permissive "No Rights 
Reserved" license — you may do what you please with what we've provided. Our 
intention is to dedicate these works to the public domain and make them freely 
available to all, without restriction.

All files are provided AS-IS. Tiny Speck cannot provide any support to help you 
bring these assets into your own projects. Many of these files are not 
structured in a standard, straightforward way, and they may take a bit of 
your time and work to understand.

Note: the Glitch logo and trademark are *not* among the things we are making 
available under this license. Only items in the files explicitly included 
herein are covered.

There is no obligation to link or credit the works, but if you do, please link 
to <a href="http://glitchthegame.com">glitchthegame.com</a>, our permanent 
"retirement" site for the game and these assets. Of course, links/shoutouts to 
Tiny Speck (<a href="http://tinyspeck.com">tinyspeck.com</a>) and/or Slack 
(<a href="http://slack.com">slack.com</a>) are appreciated.

### Exceptions to the CC0 1.0 Universal License

This repository includes an adaptation of the ColorMatrix classes by 
Quasimondo and Grant Skinner. Our license does not cover their work.

---

## Contributing

Documentation contributions are welcome! If you figure something out that you think others could learn from, write up a quick how-to document and submit it to us as a pull request. Share your knowledge!

**Areas where contributions are especially valuable:**
- Modern game engine integration examples
- HTML5/WebGL conversion techniques  
- Mobile platform adaptations
- Performance optimization strategies
- Additional language bindings 
