# Power Apps Canvas Apps YAML Structure Guide

## Overview
This document provides comprehensive instructions for working with YAML code for Power Apps canvas apps based on the official Microsoft Power Apps YAML schema (v3.0).

**Official Schema Source**: https://raw.githubusercontent.com/microsoft/PowerApps-Tooling/refs/heads/master/schemas/pa-yaml/v3.0/pa.schema.yaml

## Root Structure
Every Power Apps YAML file follows this top-level structure:

```yaml
App:
  Properties:
    # App-level properties and formulas
    StartScreen: =Screen1

Screens:
  # Screen definitions

ComponentDefinitions:
  # Custom component definitions

DataSources:
  # Data source configurations

EditorState:
  # Editor metadata (screen order, etc.)
```

## 1. App Section
The `App` section defines application-level properties and configuration.

```yaml
App:
  Properties:
    StartScreen: =Screen1
    BackEnabled: =false
    # Other app properties with Power Fx formulas
```

### Key Points:
- Contains application-wide settings
- Properties use Power Fx formulas (prefixed with `=`)
- `StartScreen` property is commonly specified

## 2. Screens Section
Defines all screens in the application as an unordered map.

```yaml
Screens:
  Screen1:
    Properties:
      # Screen properties
    Children:
      - Label1:
          Control: Label
          Properties:
            Text: ="Hello World"
            X: =10
            Y: =10
      - Button1:
          Control: Button
          Properties:
            Text: ="Click Me"
            X: =10
            Y: =100
```

### Screen Structure:
- **Properties**: Screen-level properties and formulas
- **Children**: Array of controls on the screen (ordered by z-index)

### Control Definition Format:
```yaml
ControlName:
  Control: ControlType      # Required: Control type identifier
  Properties:
    PropertyName: =PowerFxFormula
  # Optional properties:
  Group: GroupName          # For organizing controls in Studio
  Variant: VariantName      # Control variant (affects default properties)
  MetadataKey: Key          # Metadata identifier for control
  Layout: LayoutName        # Layout configuration
  IsLocked: true/false      # Whether control is locked in editor
  Children: []              # For container controls (ordered by z-index)
```

### Control Versioning:
You can specify control versions using the `@` operator:
```yaml
MyButton:
  Control: Button@2.1.0     # Specific version
  Properties:
    Text: ="Click Me"

MyLabel:
  Control: Label            # Uses latest version by default
  Properties:
    Text: ="Hello World"
```

## 3. Control Types

### Standard Controls
Common first-party controls include:
- **Basic Controls**: `Label`, `Button`, `TextInput`, `HTMLText`
- **Input Controls**: `Slider`, `Toggle`, `Checkbox`, `Radio`, `Dropdown`, `Combobox`, `DatePicker`, `ListBox`
- **Display Controls**: `Image`, `Icon`, `Video`, `Audio`, `PDF viewer`, `Barcode scanner`
- **Layout Controls**: `Container`, `Rectangle`, `Circle`, `Gallery`, `DataTable`, `Form`
- **Chart Controls**: `Column chart`, `Line chart`, `Pie chart`
- **Advanced Controls**: `Timer`, `Camera`, `Microphone`, `Add picture`, `Import`, `Export`

### Container and Layout Controls
Special attention for container controls and their children:
```yaml
MyContainer:
  Control: Container
  Properties:
    Width: =300
    Height: =200
    Fill: =RGBA(240, 240, 240, 1)
  Children:
    - Label1:
        Control: Label
        Properties:
          Text: ="Inside Container"
          X: =10         # Relative to container
          Y: =10         # Relative to container
    - Button1:
        Control: Button
        Properties:
          Text: ="Container Button"
          X: =10
          Y: =50
```

### Custom Components
```yaml
MyCustomControl:
  Control: Component
  ComponentName: MyComponent
  Properties:
    X: =10
    Y: =10
    # Custom component properties
```

### Code Components (PCF)
```yaml
MyPCFControl:
  Control: CodeComponent
  ComponentName: publisherprefix_namespace.classname
  Properties:
    X: =10
    Y: =10
```

## 4. Component Definitions
Define reusable custom components:

```yaml
ComponentDefinitions:
  MyComponent:
    DefinitionType: CanvasComponent
    Description: "A reusable component"
    AllowCustomization: true
    AccessAppScope: false
    CustomProperties:
      InputText:
        PropertyKind: Input
        DataType: Text
        Description: "Input text property"
        Default: ="Default Value"
      OutputValue:
        PropertyKind: Output
        DataType: Number
        Description: "Output number value"
    Properties:
      Fill: =RGBA(255, 255, 255, 1)
      Height: =100
      Width: =200
    Children:
      - Label1:
          Control: Label
          Properties:
            Text: =Parent.InputText
```

### Custom Property Types:
- **Input**: Receives values from parent
- **Output**: Sends values to parent
- **InputFunction**: Function called by parent
- **OutputFunction**: Function defined in component
- **Event**: Triggers events to parent
- **Action**: Function with side effects

### Data Types:
- `Text`, `Number`, `Boolean`
- `DateAndTime`, `Color`, `Currency`
- `Record`, `Table`, `Image`
- `VideoOrAudio`, `Screen`

## 5. Data Sources
Configure data connections:

```yaml
DataSources:
  MyTable:
    Type: Table
    Parameters:
      TableLogicalName: account
  
  MyActions:
    Type: Actions
    ConnectorId: shared_office365users
    Parameters:
      # Additional connector parameters
```

### Data Source Types:
- **Table**: Dataverse tables or other tabular data
- **Actions**: Connector actions and flows

## 6. Editor State
Maintains editor organization:

```yaml
EditorState:
  ScreensOrder:
    - Screen1
    - Screen2
    - Screen3
  ComponentDefinitionsOrder:
    - MyComponent
    - AnotherComponent
```

## Power Fx Formula Guidelines

### Formula Syntax:
- All formulas must start with `=`
- Use Power Fx syntax for expressions
- Null values can be represented as `null` (without quotes)
- Examples:
  ```yaml
  Text: ="Hello World"
  X: =10
  Visible: =Toggle1.Value
  OnSelect: =Navigate(Screen2, ScreenTransition.Fade)
  OptionalProperty: null    # Represents no value
  ```

### Common Formula Patterns:
```yaml
# Static values
Text: ="Static Text"
X: =50
Visible: =true

# Control references
Text: =TextInput1.Text
Visible: =Toggle1.Value

# Parent references (for controls in containers/galleries)
Width: =Parent.Width - 20
Height: =Parent.TemplateHeight    # In gallery templates

# Functions
OnSelect: =Navigate(NextScreen, ScreenTransition.Slide)
Text: =Concatenate("Hello ", User().FullName)

# Conditional logic
Visible: =If(Toggle1.Value, true, false)
Fill: =If(Button1.Pressed, RGBA(255,0,0,1), RGBA(0,255,0,1))

# Data operations
Items: =Filter(DataSource, Status = "Active")
Text: =LookUp(Users, ID = 123).Name
```

### Z-Index and Control Ordering:
- Controls in the `Children` array are ordered by z-index
- First control in array = bottom layer (z-index 1)
- Last control in array = top layer (highest z-index)
- All controls use ascending order starting from 1

## Naming Conventions

### Entity Names:
- Screen names: Descriptive and unique
- Control names: TypeName + Number (e.g., `Button1`, `Label2`)
- Component names: PascalCase

### Property Names:
- Standard properties: Use exact casing from schema
- Custom properties: PascalCase recommended

## Best Practices

### 1. Structure Organization:
- Keep screens logically organized
- Group related controls using the `Group` property
- Use meaningful names for all entities

### 2. Formula Writing:
- Keep formulas readable and well-formatted
- Use comments in complex formulas when possible
- Avoid overly complex nested expressions

### 3. Component Design:
- Design components to be reusable
- Provide clear descriptions for custom properties
- Use appropriate property kinds (Input/Output)

### 4. Data Source Management:
- Use descriptive names for data sources
- Document connection requirements
- Keep data source configurations minimal

## Validation Rules

### Required Properties:
- All controls must have a `Control` property
- Component definitions must have `DefinitionType`
- Data sources must have `Type`

### Naming Patterns:
- Entity names: Minimum 1 character, alphanumeric
- Control type IDs: Follow pattern `^([A-Z][a-zA-Z0-9]*/)?[A-Z][a-zA-Z0-9]*(@\d+\.\d+\.\d+)?$`
- Code component names: Follow pattern `^([a-z][a-z0-9]{1,7})_([a-zA-Z0-9]\.)+[a-zA-Z0-9]+$`

## Common Issues and Solutions

### 1. Invalid Control Types:
- Ensure control types are spelled correctly
- Check for proper casing
- Verify control type is supported in schema

### 2. Formula Errors:
- All formulas must start with `=`
- Use proper Power Fx syntax
- Check for correct property references

### 3. Structure Validation:
- Maintain proper YAML indentation
- Ensure required properties are present
- Follow the schema structure exactly

### 4. Custom Component Issues:
- Verify `ComponentName` matches definition
- Ensure custom properties are properly defined
- Check property kinds are appropriate
- Validate component library references if using external components

### 5. Performance Considerations:
- Avoid deeply nested formulas in YAML
- Use efficient data source queries
- Consider delegable formulas for large datasets
- Minimize complex calculations in frequently updated properties

## Advanced Topics

### 1. Component Library Integration:
```yaml
ComponentDefinitions:
  MyLibraryComponent:
    DefinitionType: CanvasComponent
    AllowCustomization: true
    ComponentLibraryUniqueName: "pub_MyComponentLibrary"
    # Component definition details
```

### 2. Responsive Design Considerations:
- Use `Parent.Width` and `Parent.Height` for responsive sizing
- Consider container-based layouts for complex UIs
- Use formulas for dynamic positioning and sizing

### 3. Gallery Templates:
```yaml
MyGallery:
  Control: Gallery
  Properties:
    Items: =DataSource
    TemplateSize: =100
  Children:
    - GalleryTemplate:  # Template for each gallery item
        Children:
          - TitleLabel:
              Control: Label
              Properties:
                Text: =ThisItem.Title
                Width: =Parent.TemplateWidth - 20
```

### 4. Form Controls and Data Cards:
```yaml
MyForm:
  Control: Form
  Properties:
    DataSource: =DataSource
    DefaultMode: =FormMode.New
  Children:
    - DataCard1:
        Control: DataCard
        Properties:
          DataField: ="Title"
        Children:
          - DataCardValue1:
              Control: TextInput
              Properties:
                Default: =Parent.Default
```

### 5. Error Handling in Formulas:
```yaml
Properties:
  Text: =IfError(LookUp(DataSource, ID = 123).Name, "Not Found")
  Visible: =!IsError(DataSource)
  OnSelect: =IfError(
    Navigate(DetailScreen, ScreenTransition.Cover),
    Notify("Navigation failed", NotificationType.Error)
  )
```

## Advanced Topics

### 1. Component Library Integration:
```yaml
ComponentDefinitions:
  MyLibraryComponent:
    DefinitionType: CanvasComponent
    AllowCustomization: true
    ComponentLibraryUniqueName: "pub_MyComponentLibrary"
    # Component definition details
```

### 2. Responsive Design Considerations:
- Use `Parent.Width` and `Parent.Height` for responsive sizing
- Consider container-based layouts for complex UIs
- Use formulas for dynamic positioning and sizing

### 3. Gallery Templates:
```yaml
MyGallery:
  Control: Gallery
  Properties:
    Items: =DataSource
    TemplateSize: =100
  Children:
    - GalleryTemplate:  # Template for each gallery item
        Children:
          - TitleLabel:
              Control: Label
              Properties:
                Text: =ThisItem.Title
                Width: =Parent.TemplateWidth - 20
```

### 4. Form Controls and Data Cards:
```yaml
MyForm:
  Control: Form
  Properties:
    DataSource: =DataSource
    DefaultMode: =FormMode.New
  Children:
    - DataCard1:
        Control: DataCard
        Properties:
          DataField: ="Title"
        Children:
          - DataCardValue1:
              Control: TextInput
              Properties:
                Default: =Parent.Default
```

### 5. Error Handling in Formulas:
```yaml
Properties:
  Text: =IfError(LookUp(DataSource, ID = 123).Name, "Not Found")
  Visible: =!IsError(DataSource)
  OnSelect: =IfError(
    Navigate(DetailScreen, ScreenTransition.Cover),
    Notify("Navigation failed", NotificationType.Error)
  )
```
- Use the schema URL for validation: `http://powerapps.com/schemas/pa-yaml/v3.0/pa.schema`
- Ensure all required properties are present
- Follow the exact structure defined in the schema
- Test formulas for Power Fx compliance

## Additional Resources
- Official Schema: https://raw.githubusercontent.com/microsoft/PowerApps-Tooling/refs/heads/master/schemas/pa-yaml/v3.0/pa.schema.yaml
- Power Fx Documentation: Official Microsoft Power Fx reference
- Power Apps Documentation: Microsoft Learn Power Apps section
