# Power Apps Canvas Apps YAML Structure Guide

## Overview
This document provides comprehensive instructions for working with YAML code for Power Apps canvas apps based on the official Microsoft Power Apps YAML schema (v3.0) and Power Fx documentation.

**Official Schema Source**: https://raw.githubusercontent.com/microsoft/PowerApps-Tooling/refs/heads/master/schemas/pa-yaml/v3.0/pa.schema.yaml

## Power Fx Design Principles
Power Fx is the formula language used throughout Power Apps canvas apps. It follows these core principles:

### Design Principles
- **Simple**: Uses familiar concepts from Excel formulas
- **Excel Consistency**: Aligns with Excel formula syntax and behavior
- **Declarative**: Describes what you want, not how to achieve it
- **Functional**: Avoids side effects; most functions are pure
- **Composition**: Complex logic built by combining simpler functions
- **Strongly Typed**: Type system ensures data integrity
- **Integrated**: Works seamlessly across the Power Platform

### Language Philosophy
Power Fx promotes:
- Low-code development through familiar Excel-like formulas
- Automatic recalculation when dependencies change
- Type safety with compile-time checking
- Functional programming patterns

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

## Power Apps Source Code Management

### Accessing Source Code Files:
Power Apps YAML files can be obtained through several methods:

1. **Power Platform CLI**:
   ```powershell
   # List canvas apps in environment
   pac canvas list
   
   # Download and extract YAML files
   pac canvas download --name "MyApp" --extract-to-directory "C:\path\to\destination"
   ```

2. **Manual Extraction from .msapp**:
   ```powershell
   # Extract .msapp file using PowerShell
   Expand-Archive -Path "C:\path\to\yourFile.msapp" -DestinationPath "C:\path\to\destination"
   ```

3. **Dataverse Git Integration**: Direct access to source files without .msapp files

### File Structure in .msapp:
- `\src\App.pa.yaml` - Represents the main App configuration
- `\src\[ScreenName].pa.yaml` - One file for each screen
- `\src\Component\[ComponentName].pa.yaml` - Component definitions

**Important Notes**:
- Only files in the `\src` folder are intended for source control
- .pa.yaml files are **read-only** and for review purposes only
- External editing, merging, and conflict resolution isn't supported
- JSON files in .msapp aren't stable for source control

### Schema Version Evolution:
1. **Experimental Format** (*.fx.yaml): No longer in development
2. **Early Preview**: Temporary format, no longer in use
3. **Source Code** (*.pa.yaml): Current active format with version control support

## Power Fx Formula Reference

### Formula Categories:

#### **Functions**: Take parameters, perform operations, return values
```yaml
Properties:
  Text: =Concatenate("Hello ", User().FullName)
  X: =Sum(10, 20, 30)
  Items: =Filter(DataSource, Status = "Active")
```

#### **Signals**: Return environment information (no parameters)
```yaml
Properties:
  Text: =Location.Latitude & ", " & Location.Longitude
  Visible: =Connection.Connected
  Color: =If(Acceleration.X > 5, Color.Red, Color.Blue)
```

#### **Enumerations**: Predefined constant values
```yaml
Properties:
  Fill: =Color.Blue
  Transition: =ScreenTransition.Fade
  Align: =Align.Center
```

#### **Named Operators**: Access container information
```yaml
Properties:
  Text: =ThisItem.Title        # In galleries
  Width: =Parent.Width - 20    # In containers
  Height: =Self.Height / 2     # Self-reference
```

### Essential Power Fx Functions for YAML:

#### **Navigation & App Control**:
```yaml
OnSelect: =Navigate(NextScreen, ScreenTransition.Cover)
OnSelect: =Back()
OnSelect: =Exit()
OnSelect: =Launch("https://example.com")
```

#### **Data Operations**:
```yaml
Items: =Filter(DataSource, Category = "Active")
Text: =LookUp(Users, ID = 123).Name
OnSelect: =Patch(DataSource, ThisItem, {Status: "Complete"})
OnSelect: =Collect(LocalCollection, {Name: TextInput1.Text})
```

#### **Conditional Logic**:
```yaml
Visible: =If(Toggle1.Value, true, false)
Text: =Switch(Status, "New", "🆕", "Complete", "✅", "❓")
Fill: =If(Value < 0, Color.Red, Color.Green)
```

#### **Text Manipulation**:
```yaml
Text: =Concatenate("Hello ", User().FullName)
Text: =Upper(TextInput1.Text)
Text: =Substitute(Label1.Text, "old", "new")
Text: =Left(Title, 10) & "..."
```

#### **Mathematical Operations**:
```yaml
Text: =Sum(Sales[Amount])
Text: =Average(Ratings[Score])
Text: =Round(Calculation, 2)
Text: =Max(Values[Number])
```

#### **Date & Time Functions**:
```yaml
Text: =Text(Now(), "mm/dd/yyyy")
Text: =DateDiff(StartDate, EndDate, Days)
Text: =Text(Today(), "dddd, mmmm dd, yyyy")
Visible: =IsToday(DueDate)
```

### Formula Syntax Guidelines:

#### **Basic Syntax Rules**:
- All formulas start with `=`
- No preceding `+` or `=` sign (unlike Excel)
- Double quotes for text strings: `="Hello World"`
- Property references: `ControlName.PropertyName`
- Comments not supported in YAML context

#### **Formula Elements**:
```yaml
# Literal values
Text: ="Static Text"
X: =42
Visible: =true

# Control property references
Text: =TextInput1.Text
Visible: =Checkbox1.Value

# Function calls
Text: =Upper(TextInput1.Text)
Items: =Sort(DataSource, Title)

# Complex expressions
Text: =If(IsBlank(TextInput1.Text), "Enter text", Upper(TextInput1.Text))
```

#### **Behavior vs. Property Formulas**:
```yaml
# Property formulas (calculate values)
Properties:
  Text: =Concatenate("Hello ", User().FullName)
  Visible: =Toggle1.Value

# Behavior formulas (perform actions - use semicolon for multiple actions)
Properties:
  OnSelect: =Set(MyVar, true); Navigate(NextScreen); Notify("Done!")
```

### Advanced Formula Patterns:

#### **Working with Collections**:
```yaml
Properties:
  Items: =Filter(MyCollection, Status = "Active")
  OnSelect: =ClearCollect(MyCollection, DataSource)
  OnSelect: =Collect(MyCollection, {Name: "New Item", Status: "Active"})
```

#### **Error Handling**:
```yaml
Properties:
  Text: =IfError(Value(TextInput1.Text), 0)
  OnSelect: =IfError(
    Patch(DataSource, ThisItem, {Field: Value}),
    Notify("Error updating record", NotificationType.Error)
  )
```

#### **Dynamic Property Setting**:
```yaml
Properties:
  Fill: =ColorValue("#" & HexInput.Text)
  Height: =Parent.Height * (Slider1.Value / 100)
  X: =If(Alignment = "Center", (Parent.Width - Self.Width) / 2, 0)
```

## Working with Formulas Best Practices

### Formula Organization:
- Break complex formulas into smaller, readable parts
- Use variables to store intermediate calculations
- Comment complex logic using descriptive control names
- Group related calculations together

### Performance Optimization:
- Use delegation-friendly functions when working with large datasets
- Avoid nested function calls in frequently updated properties
- Use collections for complex data transformations
- Minimize calls to external data sources

## Power Fx Operators and Identifiers

### Operator Categories:

#### **Property Access**:
```yaml
Properties:
  Text: =Slider1.Value        # Control property access
  Fill: =Color.Red           # Enumeration access
```

#### **Arithmetic Operators**:
```yaml
Properties:
  Text: =Value1 + Value2     # Addition
  Text: =Value1 - Value2     # Subtraction
  Text: =Value1 * Value2     # Multiplication
  Text: =Value1 / Value2     # Division
  Text: =Value1 ^ Value2     # Exponentiation
  Text: =Value1 % 100        # Percentage
```

#### **Comparison Operators**:
```yaml
Properties:
  Visible: =Price = 100      # Equal to
  Visible: =Price > 100      # Greater than
  Visible: =Price >= 100     # Greater than or equal
  Visible: =Price < 100      # Less than
  Visible: =Price <= 100     # Less than or equal
  Visible: =Price <> 100     # Not equal to
```

#### **Logical Operators**:
```yaml
Properties:
  Visible: =Price < 100 And Slider1.Value = 20    # Logical AND
  Visible: =Price < 100 Or Slider1.Value = 20     # Logical OR
  Visible: =Not(Price < 100)                      # Logical NOT
```

#### **String Concatenation**:
```yaml
Properties:
  Text: ="Hello" & " " & "World"    # String concatenation
```

#### **Membership Operators**:
```yaml
Properties:
  Visible: ="Windows" in ProductName          # Case-insensitive search
  Visible: ="Windows" exactin ProductName     # Case-sensitive search
  Visible: =Gallery1.Selected in SavedItems   # Collection membership
```

### Record Scope Operators:

#### **ThisItem and ThisRecord**:
```yaml
# In Gallery control
Properties:
  Text: =ThisItem.FirstName & " " & ThisItem.LastName

# In ForAll function
Properties:
  Items: =ForAll(Employees, ThisRecord.FirstName)
```

#### **As Operator**:
```yaml
# Gallery with named record scope
Properties:
  Items: =Employees As Employee
  Text: =Employee.FirstName & " " & Employee.LastName

# ForAll with named scope
Properties:
  Items: =ForAll(Employees As Emp, Emp.FirstName)
```

#### **Self and Parent Operators**:
```yaml
Properties:
  Fill: =Self.BorderColor    # Reference current control
  Width: =Parent.Width / 2   # Reference parent container
```

### Disambiguation Operator:

#### **Field Disambiguation**:
```yaml
Properties:
  # Access outer scope when names conflict
  Text: =MyTable[@FieldName]
  
  # Access global variables in record scope
  Text: =[@MyGlobalVar]
```

### Identifier Names:

#### **Special Characters in Names**:
```yaml
Properties:
  Text: =SimpleName                      # No quotes needed
  Text: ='Name with spaces'              # Single quotes for spaces
  Text: ='Name with "double" quotes'     # Escape with single quotes
  Text: ='Name with ''single'' quotes'   # Double single quotes to escape
```

## Power Fx Table Operations

### Table Structure:

#### **Record Definition**:
```yaml
Properties:
  # Inline record creation
  Items: ={Name: "John", Age: 30, Active: true}
  
  # Nested records
  Items: ={
    Person: {FirstName: "John", LastName: "Doe"},
    Status: "Active"
  }
```

#### **Table Creation**:
```yaml
Properties:
  # Using Table function
  Items: =Table(
    {Name: "Product A", Price: 10.99},
    {Name: "Product B", Price: 15.99}
  )
  
  # Single-column table with brackets
  Items: =["Option 1", "Option 2", "Option 3"]
```

### Table Functions:

#### **Filtering and Sorting**:
```yaml
Properties:
  Items: =Filter(Products, Price > 10)
  Items: =Sort(Products, Name, Ascending)
  Items: =SortByColumns(Products, "Price", Descending, "Name", Ascending)
```

#### **Data Transformation**:
```yaml
Properties:
  Items: =AddColumns(Products, "Total", Price * Quantity)
  Items: =RenameColumns(Products, "Price", "Cost")
  Items: =ShowColumns(Products, "Name", "Price")
  Items: =DropColumns(Products, "InternalID")
```

#### **Aggregation**:
```yaml
Properties:
  Text: =Sum(Products, Price)
  Text: =Average(Products, Rating)
  Text: =Max(Products, Price)
  Text: =Min(Products, Price)
  Text: =CountRows(Products)
```

#### **Record Scope Functions**:
```yaml
Properties:
  Items: =ForAll(
    Products,
    {
      Name: Upper(ThisRecord.Name),
      PriceCategory: If(ThisRecord.Price > 100, "Premium", "Standard")
    }
  )
```

### Single-Column Table Operations:

```yaml
Properties:
  # Extract single column
  Items: =ShowColumns(Products, "Name")
  Items: =Products.Name    # Shorthand syntax
  
  # Work with extracted column
  Text: =Concat(Products.Name, ", ")
```

## Power Fx Variables

### Variable Types:

#### **Global Variables**:
```yaml
Properties:
  # Set global variable
  OnSelect: =Set(MyGlobalVar, "Hello World")
  
  # Use global variable
  Text: =MyGlobalVar
```

#### **Context Variables** (Power Apps only):
```yaml
Properties:
  # Set context variable
  OnSelect: =UpdateContext({LocalVar: "Screen Specific"})
  
  # Navigation with context
  OnSelect: =Navigate(NextScreen, None, {PassedValue: 42})
```

#### **Collections**:
```yaml
Properties:
  # Create/replace collection
  OnSelect: =ClearCollect(MyCollection, DataSource)
  
  # Add to collection
  OnSelect: =Collect(MyCollection, {Name: "New Item"})
  
  # Use collection
  Items: =MyCollection
  Text: =First(MyCollection).Name
```

### Variable Best Practices:

#### **When to Use Variables**:
- State that can't be calculated from other controls
- Values that need to persist across user actions
- Performance optimization for expensive calculations
- Passing data between screens

#### **Variable Lifetime**:
- Global variables: App lifetime
- Context variables: Screen lifetime
- Collections: App lifetime (can be saved locally)

### Variable Examples:

#### **Running Total Pattern**:
```yaml
# Add button
Properties:
  OnSelect: =Set(RunningTotal, RunningTotal + Value(TextInput1.Text))

# Clear button  
Properties:
  OnSelect: =Set(RunningTotal, 0)

# Display
Properties:
  Text: =RunningTotal
```

#### **Collection Management**:
```yaml
# Load data
Properties:
  OnSelect: =ClearCollect(LocalData, RemoteDataSource)

# Filter and display
Properties:
  Items: =Filter(LocalData, Status = "Active")

# Add new item
Properties:
  OnSelect: =Collect(LocalData, {
    Name: NameInput.Text,
    Status: "Active",
    Created: Now()
  })
```

## Power Fx Data Types Reference

Power Fx supports a comprehensive type system with both primitive and compound data types. All data types can have a *blank* value (equivalent to null in databases).

### Data Type Overview

| Data Type | Description | Examples |
|-----------|-------------|----------|
| **Boolean** | True or false value | `=true`, `=false` |
| **Color** | Color specification with alpha channel | `=Color.Red`, `=RGBA(255, 128, 0, 0.5)` |
| **Currency** | Floating-point number with currency formatting | `=123`, `=4.56` |
| **Date** | Date without time in user's timezone | `=Date(2019, 5, 16)` |
| **DateTime** | Date with time in user's timezone | `=DateTimeValue("May 16, 2019 1:23:09 PM")` |
| **GUID** | Globally Unique Identifier | `=GUID()`, `=GUID("123e4567-...")` |
| **Hyperlink** | Text string containing hyperlink | `="https://powerapps.microsoft.com"` |
| **Image** | URI to image resource | `=MyImage`, `="https://example.com/logo.jpg"` |
| **Media** | URI to video/audio resource | `=MyVideo`, `="https://example.com/intro.mp4"` |
| **Number** | IEEE 754 double-precision floating-point | `=123`, `=-4.567`, `=8.903e121` |
| **Option Set** | Choice from predefined options | `=ThisItem.OrderStatus` |
| **Record** | Collection of named fields | `={Name: "John", Age: 30}` |
| **Record Reference** | Reference to record in entity | `=First(Accounts).Owner` |
| **Table** | Collection of records | `=Table({Name: "John"}, {Name: "Jane"})` |
| **Text** | Unicode text string | `="Hello, World"` |
| **Time** | Time without date in user's timezone | `=Time(11, 23, 45)` |
| **Two Option** | Boolean choice with labels | `=ThisItem.Taxable` |

### Blank Values

All data types support *blank* values (no value):

```yaml
Properties:
  # Set blank value
  OnSelect: =Set(MyVar, Blank())
  
  # Test for blank
  Visible: =Not(IsBlank(TextInput1.Text))
  
  # Replace blank with default
  Text: =Coalesce(OptionalField, "Default Value")
```

### Text, Hyperlink, Image, and Media Types

#### **Text String Handling**:
```yaml
Properties:
  # Embedded text with quotes
  Text: ="Jane said ""Hello, World!"""
  
  # Unicode text support
  Text: ="こんにちは世界"
  
  # Multi-line text
  Text: ="Line 1" & Char(10) & "Line 2"
```

#### **Image Resources**:
```yaml
Properties:
  # App resource image
  Image: =MyImageResource
  
  # Web URL image
  Image: ="https://example.com/logo.jpg"
  
  # Data URI image
  Image: ="data:image/png;base64,iVBORw0KGgo..."
  
  # Camera captured image
  Image: =Camera1.Photo
```

#### **Media Resources**:
```yaml
Properties:
  # App resource media
  Media: =MyVideoResource
  
  # Web URL media
  Media: ="https://example.com/video.mp4"
  
  # Database stored media
  Media: ="appres://datasources/Media/table/..."
```

#### **URI Reference Patterns**:
```yaml
Properties:
  # App resources
  Image: =MyLogo    # Returns "appres://blobmanager/..."
  
  # External URLs
  Image: ="https://northwindtraders.com/logo.jpg"
  
  # Database attachments
  Image: ="appres://datasources/Contacts/table/..."
```

### Number and Currency Types

#### **Numeric Range and Precision**:
- **Range**: -1.79769 × 10^308 to 1.79769 × 10^308
- **Integer precision**: -9,007,199,254,740,991 to 9,007,199,254,740,991
- **Floating-point**: IEEE 754 double-precision standard

```yaml
Properties:
  # Integer values
  Text: =123
  
  # Decimal values
  Text: =45.67
  
  # Scientific notation
  Text: =1.23e10
  
  # Currency formatting (same as Number)
  Text: =Text(123.45, "[$-en-US]$#,##0.00")
```

#### **Floating-Point Considerations**:
```yaml
Properties:
  # Floating-point approximation issues
  Text: =(55 / 100 * 100) - 55    # May not equal exactly 0
  
  # Use Round for comparisons
  Visible: =Round((55 / 100 * 100) - 55, 10) = 0
  
  # Large integers as text for precision
  Text: ="9007199254740992"    # Beyond safe integer range
```

### Date, Time, and DateTime Types

#### **Time Zone Handling**:

**User Local vs Time Zone Independent**:
```yaml
Properties:
  # User local (adjusts for timezone)
  Text: =Text(Now(), "mm/dd/yyyy hh:mm AM/PM")
  
  # Time zone independent (same everywhere)
  Text: =Text(DateTimeValue("2019-05-19T04:00:00"), "mm/dd/yyyy hh:mm AM/PM")
```

#### **Date/Time Construction**:
```yaml
Properties:
  # Date construction
  Text: =Date(2024, 12, 25)    # Christmas 2024
  
  # Time construction
  Text: =Time(14, 30, 0)       # 2:30 PM
  
  # DateTime construction
  Text: =DateTimeValue("December 25, 2024 2:30 PM")
  
  # Current date/time
  Text: =Now()                 # Current date and time
  Text: =Today()               # Current date at midnight
```

#### **Date/Time Calculations**:
```yaml
Properties:
  # Add/subtract time periods
  Text: =DateAdd(Today(), 30, Days)
  Text: =DateAdd(Now(), -2, Hours)
  
  # Calculate differences
  Text: =DateDiff(StartDate, EndDate, Days)
  Text: =DateDiff(StartTime, EndTime, Minutes)
  
  # Time zone conversions
  Text: =DateAdd(UTCTime, TimeZoneInformation().Bias, Minutes)
```

#### **Unix Time Conversion**:
```yaml
Properties:
  # Convert Unix timestamp to DateTime
  Text: =Text(DateAdd(Date(1970,1,1), UnixTimestamp, Seconds), "mm/dd/yyyy")
  
  # Convert DateTime to Unix timestamp
  Text: =RoundDown(Value(MyDateTime) / 1000, 0)
  
  # Handle milliseconds vs seconds
  Text: =Text(1000000000 * 1000, DateTimeFormat.UTC)
```

#### **SQL Server Time Handling**:
```yaml
Properties:
  # Parse ISO 8601 duration format
  Text: =With(
    Match("PT2H1M39S", "PT(?:(?<hours>\d+)H)?(?:(?<minutes>\d+)M)?(?:(?<seconds>\d+)S)?"),
    Time(Value(hours), Value(minutes), Value(seconds))
  )
```

### Boolean and Two Option Types

#### **Boolean Operations**:
```yaml
Properties:
  # Direct boolean values
  Visible: =true
  Visible: =false
  
  # Boolean from expressions
  Visible: =TextInput1.Text <> ""
  Visible: =CountRows(MyCollection) > 0
  
  # Boolean in conditions
  Text: =If(MyBoolean, "Yes", "No")
```

#### **Two Option Data Type**:
```yaml
Properties:
  # Two option comparison
  Visible: =ThisItem.TaxStatus = TaxStatus.Taxable
  
  # Two option as boolean
  Visible: =ThisItem.Taxable    # Direct boolean usage
  
  # Two option labels
  Text: =If(ThisItem.Taxable, "Taxable", "Non-Taxable")
```

### Option Sets

#### **Option Set Usage**:
```yaml
Properties:
  # Option set comparison
  Visible: =ThisItem.OrderStatus = OrderStatus.Active
  
  # Option set with entity scope
  Visible: =ThisItem.Status = 'Status (Accounts)'.Active
  
  # Option set in Switch
  Text: =Switch(
    ThisItem.Priority,
    Priority.High, "🔴 High",
    Priority.Medium, "🟡 Medium", 
    Priority.Low, "🟢 Low",
    "Unknown"
  )
```

#### **Global vs Local Option Sets**:
```yaml
Properties:
  # Global option set (shared across entities)
  Fill: =Switch(
    ThisItem.Priority,
    Priority.High, Color.Red,
    Priority.Medium, Color.Orange,
    Priority.Low, Color.Green,
    Color.Gray
  )
  
  # Local option set (entity-specific)
  Text: =Switch(
    ThisItem.Status,
    'OrderStatus (Orders)'.New, "📝 New Order",
    'OrderStatus (Orders)'.Shipped, "🚚 Shipped",
    'OrderStatus (Orders)'.Delivered, "✅ Delivered",
    "Unknown Status"
  )
```

### Record and Table Types

#### **Record Construction**:
```yaml
Properties:
  # Simple record
  Items: ={Name: "John Doe", Age: 30, Active: true}
  
  # Nested records
  Items: ={
    Person: {FirstName: "John", LastName: "Doe"},
    Contact: {Email: "john@example.com", Phone: "555-1234"},
    Metadata: {Created: Now(), Modified: Now()}
  }
  
  # Record from form
  OnSelect: =Patch(DataSource, Defaults(DataSource), {
    Name: NameInput.Text,
    Email: EmailInput.Text,
    Status: StatusDropdown.Selected.Value
  })
```

#### **Table Construction**:
```yaml
Properties:
  # Table function
  Items: =Table(
    {Name: "Product A", Price: 10.99, InStock: true},
    {Name: "Product B", Price: 15.99, InStock: false},
    {Name: "Product C", Price: 8.50, InStock: true}
  )
  
  # Single-column table (bracket notation)
  Items: =["Option 1", "Option 2", "Option 3"]
  
  # Table from collection
  Items: =MyCollection
```

#### **Record and Table Access**:
```yaml
Properties:
  # Access record fields
  Text: =ThisItem.Name
  Text: =First(MyTable).Email
  
  # Access nested record fields
  Text: =ThisItem.Person.FirstName
  Text: =MyRecord.Contact.Email
  
  # Table column extraction
  Items: =MyTable.Name    # Single-column table of names
  Items: =ShowColumns(MyTable, "Name", "Email")
```

### GUID Type

#### **GUID Operations**:
```yaml
Properties:
  # Generate new GUID
  Text: =GUID()
  
  # Parse GUID string
  Text: =GUID("123e4567-e89b-12d3-a456-426655440000")
  
  # Compare GUIDs
  Visible: =ThisItem.ID = GUID("123e4567-e89b-12d3-a456-426655440000")
  
  # GUID in data operations
  OnSelect: =Patch(DataSource, LookUp(DataSource, ID = MyGUID), {Status: "Updated"})
```

### Color Type

#### **Color Specifications**:
```yaml
Properties:
  # Named colors
  Fill: =Color.Red
  Fill: =Color.Blue
  Fill: =Color.Transparent
  
  # RGB colors
  Fill: =RGBA(255, 128, 0, 1)      # Orange, fully opaque
  Fill: =RGBA(255, 0, 0, 0.5)      # Red, 50% transparent
  
  # Hex colors
  Fill: =ColorValue("#FF8000")      # Orange
  Fill: =ColorValue("#" & HexInput.Text)  # Dynamic color
  
  # HSV colors
  Fill: =HSVA(120, 100, 100, 1)    # Pure green
```

#### **Color Calculations**:
```yaml
Properties:
  # Color blending
  Fill: =ColorFade(Color.Blue, 0.5)    # 50% faded blue
  
  # Conditional colors
  Fill: =If(Value > 100, Color.Green, Color.Red)
  
  # Color from data
  Fill: =Switch(
    ThisItem.Category,
    "Important", Color.Red,
    "Warning", Color.Orange,
    "Normal", Color.Blue,
    Color.Gray
  )
```

### Type Conversion and Validation

#### **Type Conversion Functions**:
```yaml
Properties:
  # Text conversions
  Text: =Text(123.45, "#,##0.00")        # Number to text
  Text: =Text(Now(), "mm/dd/yyyy")       # Date to text
  Text: =Text(Color.Red)                 # Color to hex text
  
  # Number conversions
  Text: =Value("123.45")                 # Text to number
  Text: =Value(TextInput1.Text)          # Input to number
  
  # Date conversions
  Text: =DateValue("12/25/2024")         # Text to date
  Text: =TimeValue("2:30 PM")            # Text to time
  Text: =DateTimeValue("12/25/2024 2:30 PM")  # Text to datetime
  
  # Boolean conversions
  Visible: =Boolean("true")              # Text to boolean
  Text: =Text(true)                      # Boolean to text
```

#### **Type Checking Functions**:
```yaml
Properties:
  # Check for blank/empty
  Visible: =Not(IsBlank(OptionalField))
  Visible: =Not(IsEmpty(MyCollection))
  
  # Check for errors
  Visible: =Not(IsError(Value(TextInput1.Text)))
  Text: =IfError(Value(TextInput1.Text), "Invalid number")
  
  # Check for numeric
  Visible: =IsNumeric(TextInput1.Text)
  
  # Type-safe operations
  Text: =If(IsBlank(OptionalText), "Default", OptionalText)
```

### Data Type Best Practices

#### **Memory and Performance**:
```yaml
Properties:
  # Efficient data loading
  Items: =Filter(LargeDataSource, Status = "Active")    # Server-side filtering
  
  # Avoid large media in memory
  OnSelect: =Set(TempImage, Camera1.Photo); UploadToDatabase(TempImage); Set(TempImage, Blank())
  
  # Use delegation-friendly operations
  Items: =Sort(Filter(DataSource, Active), Name)        # Delegable
  # Avoid: Sort(DataSource, If(Active, Name, ""))       # Not delegable
```

#### **Data Validation Patterns**:
```yaml
Properties:
  # Input validation
  Visible: =And(
    Not(IsBlank(NameInput.Text)),
    IsNumeric(AgeInput.Text),
    IsMatch(EmailInput.Text, Email)
  )
  
  # Data type safety
  Text: =Coalesce(
    Text(LookUp(DataSource, ID = MyID).Name),
    "Name not found"
  )
  
  # Error handling with types
  OnSelect: =IfError(
    Set(MyNumber, Value(TextInput1.Text)),
    UpdateContext({ValidationError: "Please enter a valid number"})
  )
```

#### **Cross-Platform Considerations**:
```yaml
Properties:
  # Timezone-aware date handling
  Text: =Text(
    DateAdd(UTCDateTime, TimeZoneInformation().Bias, Minutes),
    "mm/dd/yyyy hh:mm AM/PM"
  )
  
  # Large integer handling
  Text: =Text(LargeIntegerField, "0")     # Store as text for precision
  
  # Currency considerations
  Text: =Text(CurrencyAmount, "[$-en-US]$#,##0.00")    # Locale-specific formatting
```

## Power Fx Enhanced Connectors Reference

Enhanced connectors enable Power Platform to connect to tabular data in external data sources through a RESTful protocol based on OData. This comprehensive reference covers the enhanced connector protocol implementation.

### Overview

Enhanced connectors provide dynamic connectivity to external tabular data sources, unlike action connectors which invoke fixed OpenAPI operations. They implement a self-describing protocol with rich metadata endpoints for optimal design-time experiences.

#### **Key Components**:
1. **Metadata**: Describes the metadata of target data sources
2. **Transpiler**: Converts OData requests to native data source format  
3. **CRUD Operations**: Create, Read, Update, Delete operations

#### **Core Concepts**:

**Resource Hierarchy**:
```yaml
# Tabular data structure
Dataset: Collection of tables
├── Table: Has rows and columns containing data
    └── Item: Represents a row in the table
```

**Namespace Examples**:
| Connector | Dataset | Table |
|-----------|---------|-------|
| SQL | Database | Table |
| SharePoint | Site | List |
| Dataverse | Environment | Entity |

### Enhanced Connector Protocol

#### **Authentication Pattern**:
Enhanced connectors are "pass-through" - they receive auth tokens configured during connection creation:

```yaml
# Auth token types supported:
- API Key authentication
- OAuth bearer tokens  
- Property bag (base64 encoded JSON with multiple properties)
- Custom authentication schemes
```

#### **OData Query Support**:
Connectors support OData query operations:
- `$filter` - Filter records based on conditions
- `$select` - Select specific columns
- `$orderby` / `$sort` - Sort records
- `$top` - Limit number of records
- `$skip` - Skip records for pagination
- `$count` - Include total count

### Power Fx Integration and Delegation

#### **Delegation Mechanism**:
Power Fx translates expressions into efficient server-side operations:

```yaml
Properties:
  # Power Fx expression
  Items: =Sort(Filter(MyTable, Score > 100), Age)
  
  # Translates to OData query:
  # GET /datasets/default/tables/myTable512/items?$filter=new_score+gt+100&$sort=new_age
```

#### **Display Names vs Logical Names**:
```yaml
Properties:
  # Power Fx uses display names in expressions
  Items: =Filter(Employees, 'Full Name' = "John Doe")
  
  # Connector receives logical names in queries  
  # GET /items?$filter=emp_fullname+eq+'John Doe'
```

#### **Delegation Detection**:
```yaml
Properties:
  # Delegable operations (executed server-side)
  Items: =Filter(LargeTable, Status = "Active")    # Efficient
  
  # Non-delegable operations (may download all records)
  Items: =Filter(LargeTable, Len(Description) > 100)  # Warning issued
```

### REST API Endpoints

#### **Dataset Discovery**:

**GET /datasets**:
```yaml
# Returns available datasets
Response:
  value:
    - name: "default"
      displayName: "Main Database"
    - name: "archive"  
      displayName: "Archive Database"
```

**GET /$metadata.json/datasets**:
```yaml
# Connector-wide metadata
Response:
  tabular:
    source: "mru"
    displayName: "site"
    urlEncoding: "double"
    tableDisplayName: "list"
    tablePluralName: "lists"
  blob:
    source: "mru"
    displayName: "site"
    urlEncoding: "double"
```

#### **Table Discovery**:

**GET /datasets/{datasetName}/tables**:
```yaml
# Returns available tables in dataset
Response:
  value:
    - Name: "4bd37916-0026-4726-94e8-5a0cbc8e476a"
      DisplayName: "Documents"
    - Name: "5266fcd9-45ef-4b8f-8014-5d5c397db6f0"
      DisplayName: "MyTestList"
```

#### **Table Metadata**:

**GET /$metadata.json/datasets/{datasetName}/tables/{tableName}**:

```yaml
# Complete table metadata including:
Response:
  name: "logical_table_name"
  title: "Display Table Name"  
  x-ms-permission: "read-write"  # read-write | read-only | null
  x-ms-capabilities:
    sortRestrictions:
      sortable: true
      unsortableProperties: ["calculated_field"]
      ascendingOnlyProperties: ["created_date"]
    filterRestrictions:
      filterable: true
      nonFilterableProperties: ["blob_field"]
    selectRestrictions:
      selectable: true
    isDelegable: true
    filterFunctionSupport: ["eq", "ne", "gt", "ge", "lt", "le", "and", "or"]
    serverPagingOptions: ["top", "skiptoken"]
    isOnlyServerPagable: false
    supportsRecordPermission: true
  schema:
    type: "object"
    properties:
      field1:
        type: "string"
        x-ms-keyType: "primary"
        x-ms-keyOrder: 0
        x-ms-permission: "read-only"
        x-ms-sort: "asc,desc"
        x-ms-visibility: "none"
        x-ms-capabilities:
          filterFunctions: ["eq", "ne", "contains", "startswith"]
      field2:
        type: "number"
        x-ms-permission: "read-write"
        x-ms-capabilities:
          filterFunctions: ["eq", "ne", "gt", "ge", "lt", "le"]
```

### Table Capabilities Reference

#### **Table-Level Capabilities**:

**Sort Restrictions**:
```yaml
x-ms-capabilities:
  sortRestrictions:
    sortable: true                    # Boolean - table supports sorting
    unsortableProperties:             # Array - properties that can't be sorted
      - "calculated_field"
      - "blob_field"
    ascendingOnlyProperties:          # Array - properties supporting only ASC
      - "created_date"
```

**Filter Restrictions**:
```yaml
x-ms-capabilities:
  filterRestrictions:
    filterable: true                  # Boolean - table supports filtering
    nonFilterableProperties:          # Array - properties that can't be filtered
      - "attachment_field"
      - "complex_object"
    requiredProperties:               # Array - properties required in filters
      - "tenant_id"
```

**Select Restrictions**:
```yaml
x-ms-capabilities:
  selectRestrictions:
    selectable: true                  # Boolean - supports column selection
```

**Pagination Options**:
```yaml
x-ms-capabilities:
  serverPagingOptions:                # Array - supported paging methods
    - "top"                          # Supports $top parameter
    - "skiptoken"                    # Supports continuation tokens
  isOnlyServerPagable: false          # Boolean - requires server paging
  isPageable: true                    # Boolean - supports any paging
```

**Delegation Support**:
```yaml
x-ms-capabilities:
  isDelegable: true                   # Boolean - supports some delegation
  filterFunctionSupport:              # Array - supported filter functions
    - "eq"        # Equal
    - "ne"        # Not equal  
    - "gt"        # Greater than
    - "ge"        # Greater than or equal
    - "lt"        # Less than
    - "le"        # Less than or equal
    - "and"       # Logical AND
    - "or"        # Logical OR
    - "contains"  # String contains
    - "startswith" # String starts with
    - "endswith"  # String ends with
    - "not"       # Logical NOT
    - "null"      # Is null/blank
```

#### **Column-Level Capabilities**:

**Primary Key Configuration**:
```yaml
properties:
  id_field:
    type: "string"
    x-ms-keyType: "primary"           # Defines primary key
    x-ms-keyOrder: 0                  # Order in composite key
    x-ms-permission: "read-only"      # Usually read-only for generated keys
```

**Column Permissions**:
```yaml
properties:
  editable_field:
    x-ms-permission: "read-write"     # Can be updated
  calculated_field:
    x-ms-permission: "read-only"      # Cannot be updated
  standard_field:
    x-ms-permission: null             # Default read-write
```

**Sort Support**:
```yaml
properties:
  sortable_field:
    x-ms-sort: "asc,desc"            # Supports both directions
  asc_only_field:
    x-ms-sort: "asc"                 # Ascending only
  no_sort_field:
    x-ms-sort: "none"                # No sorting support
```

**Visibility Control**:
```yaml
properties:
  visible_field:
    x-ms-visibility: "none"          # Always visible
  advanced_field:
    x-ms-visibility: "advanced"      # In advanced view only
  internal_field:
    x-ms-visibility: "internal"      # Hidden from end users
```

**Filter Functions per Column**:
```yaml
properties:
  text_field:
    x-ms-capabilities:
      filterFunctions:                # Supported operations for this column
        - "eq"
        - "ne" 
        - "contains"
        - "startswith"
        - "endswith"
  number_field:
    x-ms-capabilities:
      filterFunctions:
        - "eq"
        - "ne"
        - "gt"
        - "ge"
        - "lt" 
        - "le"
```

### Option Sets and Enumerations

#### **Option Set Definition**:
```yaml
properties:
  status_field:
    type: "string"
    enum:                             # Logical values
      - "active"
      - "inactive"
      - "pending"
    x-ms-enum:
      name: "StatusOptions"           # Option set name
    x-ms-enum-values:                 # Display names
      - value: "active"
        displayName: "Active"
      - value: "inactive" 
        displayName: "Inactive"
      - value: "pending"
        displayName: "Pending Review"
```

#### **Using Option Sets in Power Fx**:
```yaml
Properties:
  # Reference by logical value
  Visible: =ThisItem.status_field = "active"
  
  # Reference by option set enumeration (preferred)
  Visible: =ThisItem.status_field = StatusOptions.Active
  
  # Switch based on option set
  Fill: =Switch(
    ThisItem.status_field,
    StatusOptions.Active, Color.Green,
    StatusOptions.Inactive, Color.Gray,
    StatusOptions.Pending, Color.Orange,
    Color.Red
  )
```

### Media and Attachment Handling

#### **Image/Media Column Definition**:
```yaml
properties:
  image_field:
    type: "string"
    format: "uri"                     # URI format
    x-ms-media-kind: "image"          # image | video | audio
    x-ms-media-base-url: "https://connector.blob.endpoint"
    x-ms-media-default-folder-path: "/uploads"
    x-ms-media-upload-format: "UploadFile"
  
  attachment_field:
    type: "string" 
    format: "byte"                    # Base64 encoded
    x-ms-media-kind: null             # Generic blob
```

#### **Using Media in Power Fx**:
```yaml
Properties:
  # Display image from connector
  Image: =ThisItem.image_field
  
  # Display video/audio
  Media: =ThisItem.video_field
  
  # Upload captured image
  OnSelect: =Patch(DataSource, ThisItem, {
    image_field: Camera1.Photo
  })
```

### Runtime CRUD Operations

#### **Create Record (POST)**:
```yaml
# POST /datasets/{dataset}/tables/{table}/items
# Content-Type: application/json

Request Body:
  name: "New Record"
  description: "Record description"
  status: "active"
  
Response:
  "@odata.etag": "W/\"12345\""
  id: "newly-created-id"
  name: "New Record"
  # ... other fields
```

#### **Read Records (GET)**:
```yaml
# GET /datasets/{dataset}/tables/{table}/items?$filter=status eq 'active'&$orderby=name&$top=50

Response:
  "@odata.context": "metadata_context"
  "@odata.count": 150                # When $count=true
  value:
    - "@odata.etag": "W/\"12345\""
      id: "record-1"
      name: "First Record"
    - "@odata.etag": "W/\"12346\""
      id: "record-2" 
      name: "Second Record"
```

#### **Update Record (PATCH)**:
```yaml
# PATCH /datasets/{dataset}/tables/{table}/items/{id}
# If-Match: W/"12345"
# Content-Type: application/json

Request Body:
  name: "Updated Name"
  status: "inactive"
  
Response:
  "@odata.etag": "W/\"12347\""      # New etag
  # ... updated record
```

#### **Delete Record (DELETE)**:
```yaml
# DELETE /datasets/{dataset}/tables/{table}/items/{id}
# If-Match: W/"12345"

Response: 204 No Content
```

### ETag and Concurrency Control

#### **Optimistic Concurrency Pattern**:
```yaml
# Enhanced connectors use ETags for concurrency control

Headers in Requests:
  If-Match: "W/\"12345\""           # For PATCH/DELETE operations
  If-None-Match: "*"                # For POST operations (create if not exists)

Headers in Responses:
  ETag: "W/\"12347\""               # New version after update

Error Response (Concurrency Conflict):
  Status: 412 Precondition Failed
  Body:
    message: "The record has been modified by another user"
    RequestUri: "/datasets/default/tables/accounts/items/123"
```

#### **ETag Handling in Power Fx**:
```yaml
Properties:
  # Automatic ETag handling in Power Apps
  OnSelect: =Patch(DataSource, ThisItem, {Status: "Updated"})
  
  # Manual ETag handling for conflict resolution
  OnSelect: =IfError(
    Patch(DataSource, ThisItem, {Status: "Updated"}),
    If(
      FirstError.Kind = ErrorKind.Conflict,
      Notify("Record was modified by another user. Please refresh and try again."),
      Notify("Update failed: " & FirstError.Message)
    )
  )
```

### Display Format Configuration

#### **Column Ordering Control**:
```yaml
# Table-level display format
x-ms-displayFormat:
  propertiesDisplayOrder:             # Gallery view on web
    - "title"
    - "description"
    - "status"
  propertiesCompactDisplayOrder:      # Gallery view on mobile
    - "title"
    - "status"
  propertiesTabularDisplayOrder:      # Grid/table view
    - "id"
    - "title"
    - "created_date"
    - "status"
```

#### **Column-Level Display Format**:
```yaml
properties:
  person_field:
    type: "object"
    x-ms-displayFormat:
      titleProperty: "fullName"        # Primary display field
      subtitleProperty: "email"        # Secondary display field  
      thumbnailProperty: "avatar"      # Image field for thumbnails
```

### Advanced Connector Features

#### **Dynamic Values Support**:
```yaml
properties:
  category_field:
    type: "string"
    x-ms-dynamic-values:
      builtInOperation: "categories"   # Dynamic value provider
      parameters:
        dataset: "default"
      value-path: "id"                # Value property
      value-title: "name"             # Display property
```

#### **Navigation Properties**:
```yaml
properties:
  related_record:
    x-ms-navigation:
      targetTable: "related_table"
      referentialConstraints:
        - property: "foreign_key_id"
          referencedProperty: "id"
```

#### **SharePoint-Specific Extensions**:
```yaml
properties:
  sharepoint_field:
    x-ms-sp:                          # SharePoint-specific capabilities
      choiceFormat: "dropdown"
      listId: "list-guid"
    x-ms-enable-selects: true         # Enable SPO ECS feature
```

### Error Handling and Status Codes

#### **Standard HTTP Status Codes**:
```yaml
Success Responses:
  200: "OK - Success"
  201: "Created - New record created"
  204: "No Content - Delete successful"

Error Responses:
  400: "Bad Request - Invalid request format"
  401: "Unauthorized - Authentication failed"
  404: "Not Found - Resource not found"
  409: "Conflict - Data conflict"
  412: "Precondition Failed - ETag mismatch"
  500: "Internal Server Error - Server error"
```

#### **Error Response Format**:
```yaml
Error Response Body:
  message: "Detailed error description"
  RequestUri: "/datasets/default/tables/accounts/items"
  details:                           # Optional additional details
    - code: "INVALID_FILTER"
      message: "The filter expression is not valid"
```

#### **Error Handling in Power Fx**:
```yaml
Properties:
  # Handle specific connector errors
  OnSelect: =IfError(
    Patch(DataSource, ThisItem, FormData),
    Switch(
      FirstError.Kind,
      ErrorKind.Network, Notify("Network error. Please check connection."),
      ErrorKind.Conflict, Notify("Record was modified. Please refresh."),
      ErrorKind.Permission, Notify("You don't have permission to update this record."),
      Notify("Update failed: " & FirstError.Message)
    )
  )
```

### Connector Implementation Best Practices

#### **Performance Optimization**:
```yaml
# Implement efficient delegation
- Support $select to reduce payload size
- Implement $filter for server-side filtering  
- Use $top for pagination
- Support $orderby for server-side sorting
- Minimize round trips with batch operations

# Example efficient query
GET /items?$select=id,name,status&$filter=status eq 'active'&$orderby=name&$top=100
```

#### **Security Considerations**:
```yaml
# Authentication best practices
- Validate auth tokens on every request
- Implement proper authorization checks
- Use HTTPS for all communications
- Sanitize all input parameters
- Implement rate limiting

# Example auth validation
Headers:
  Authorization: "Bearer {token}"
  X-API-Key: "{api-key}"
```

#### **Metadata Design Guidelines**:
```yaml
# Provide comprehensive capabilities metadata
- Define precise filter function support per column
- Set appropriate visibility levels
- Configure proper sort restrictions
- Implement logical vs display name mapping
- Provide meaningful error messages
```

### Testing Enhanced Connectors

#### **Power Fx Testing Patterns**:
```yaml
Properties:
  # Test basic connectivity
  Items: =DataSource
  
  # Test filtering capability
  Items: =Filter(DataSource, Status = "Active")
  
  # Test sorting capability  
  Items: =Sort(DataSource, Name, Ascending)
  
  # Test delegation warning detection
  Items: =Filter(DataSource, Len(Description) > 100)  # Should show warning
  
  # Test CRUD operations
  OnSelect: =Collect(DataSource, {Name: "Test", Status: "Active"})
  OnSelect: =Patch(DataSource, First(DataSource), {Status: "Updated"})
  OnSelect: =Remove(DataSource, Last(DataSource))
```

#### **Validation Checklist**:
```yaml
Metadata Validation:
  ✓ All required endpoints implemented
  ✓ Capabilities accurately reflect server support
  ✓ Column metadata includes proper types and restrictions
  ✓ Display names provided for user-friendly experience
  ✓ Primary keys properly configured

Runtime Validation:
  ✓ OData queries correctly parsed and executed
  ✓ Error responses follow standard format
  ✓ ETags implemented for concurrency control
  ✓ Pagination works with large datasets
  ✓ Security/auth properly enforced
```

### Integration with Power Apps YAML

#### **Data Source Configuration**:
```yaml
DataSources:
  MyEnhancedConnector:
    Type: Table
    ConnectorId: "custom_enhanced_connector" 
    Parameters:
      DatasetName: "default"
      TableName: "main_table"
      ConnectionId: "connection-guid"
```

#### **Using Enhanced Connector Data**:
```yaml
Properties:
  # Gallery bound to enhanced connector
  Items: =MyEnhancedConnector
  
  # Filtered and sorted data
  Items: =Sort(Filter(MyEnhancedConnector, Status = "Active"), Name)
  
  # Form connected to enhanced connector
  DataSource: =MyEnhancedConnector
  DefaultMode: =FormMode.New
  
  # Update record through enhanced connector
  OnSelect: =Patch(MyEnhancedConnector, Gallery1.Selected, {
    Name: TextInput1.Text,
    Status: Dropdown1.Selected.Value
  })
```

## Power Fx Expression Grammar Reference

Power Fx is based on formulas that bind a name to an expression. This comprehensive grammar reference defines the complete syntax for Power Fx expressions, covering lexical analysis, identifiers, operators, and expression structure.

### Grammar Overview

Power Fx expressions follow a formal grammar structure with:
- **Lexical Level**: Whitespace, comments, and tokens
- **Syntactic Level**: How tokens combine to form expressions
- **Semantic Level**: Type checking and evaluation

#### **Grammar Conventions**:
- Non-terminal symbols: *Italic type*
- Terminal symbols: `Fixed-width font`
- Optional elements: *Element*<sub>opt</sub>
- Alternatives: Listed on separate lines or with "one of"

### Lexical Analysis

#### **Expression Unit Structure**:
```yaml
# Complete expression unit
ExpressionUnit:
  ExpressionElements(opt)

ExpressionElements:
  ExpressionElement
  ExpressionElement ExpressionElements(opt)

ExpressionElement:
  Whitespace
  Comment  
  Token
```

Only **Token** elements are significant in the syntactic grammar.

#### **Whitespace Definition**:
```yaml
# Whitespace characters supported
Whitespace: (any of)
  - Unicode Space separator (class Zs)
  - Unicode Line separator (class Zl)
  - Unicode Paragraph separator (class Zp)
  - Horizontal tab character (U+0009)
  - Line feed character (U+000A)
  - Vertical tab character (U+000B)
  - Form feed character (U+000C)
  - Carriage return character (U+000D)
  - Next line character (U+0085)
```

#### **Comment Syntax**:

**Single-Line Comments**:
```yaml
Properties:
  # Single-line comments start with //
  Text: ="Hello World"    // This is a comment
  
  # Comments extend to end of line
  OnSelect: =Navigate(Screen2)  // Navigate to next screen
```

**Delimited Comments**:
```yaml
Properties:
  /* Multi-line comment
     can span multiple lines */
  Text: ="Hello World"
  
  OnSelect: =/* Inline comment */ Navigate(Screen2)
```

**Comment Grammar**:
```yaml
Comment:
  DelimitedComment
  SingleLineComment

SingleLineComment:
  "//" SingleLineCommentCharacters(opt)

DelimitedComment:
  "/*" DelimitedCommentCharacters(opt) "*/"
```

**Important Notes**:
- Comments do not nest
- `/*` and `*/` have no special meaning within single-line comments
- `//` and `/*` have no special meaning within delimited comments
- Comments are not processed within text literals

### Literals

#### **Literal Types**:
```yaml
Literal:
  LogicalLiteral
  NumberLiteral
  TextLiteral
```

#### **Logical Literals**:
```yaml
Properties:
  # Boolean values
  Visible: =true
  Visible: =false
  
  # Used in conditions
  Text: =If(true, "Yes", "No")
```

**Grammar**:
```yaml
LogicalLiteral: (one of)
  true
  false
```

#### **Number Literals**:
```yaml
Properties:
  # Integer numbers
  Text: =123
  Text: =0
  
  # Decimal numbers
  Text: =123.45
  Text: =0.5
  Text: =.5         # Leading zero optional
  
  # Scientific notation
  Text: =1.23e10    # 1.23 × 10^10
  Text: =1e5        # 1 × 10^5
  Text: =2.5E-3     # 2.5 × 10^-3
```

**Grammar**:
```yaml
NumberLiteral:
  DecimalDigits ExponentPart(opt)
  DecimalDigits DecimalSeparator DecimalDigits(opt) ExponentPart(opt)
  DecimalSeparator DecimalDigits ExponentPart(opt)

DecimalDigits:
  DecimalDigit
  DecimalDigits DecimalDigit

DecimalDigit: (one of)
  0 1 2 3 4 5 6 7 8 9

ExponentPart:
  ExponentIndicator Sign(opt) DecimalDigits

ExponentIndicator: (one of)
  e E

Sign: (one of)
  + -
```

#### **Text Literals**:
```yaml
Properties:
  # Simple text
  Text: ="Hello World"
  
  # Empty text
  Text: =""
  
  # Text with embedded quotes
  Text: ="She said ""Hello"" to me"    # She said "Hello" to me
  
  # Text with special characters
  Text: ="Line 1" & Char(10) & "Line 2"
  
  # Unicode text
  Text: ="こんにちは世界"
```

**Grammar**:
```yaml
TextLiteral:
  '"' TextLiteralCharacters(opt) '"'

TextLiteralCharacters:
  TextLiteralCharacter TextLiteralCharacters(opt)

TextLiteralCharacter:
  TextCharacterNoDoubleQuote
  DoubleQuoteEscapeSequence

TextCharacterNoDoubleQuote:
  any Unicode code point except double quote

DoubleQuoteEscapeSequence:
  '""'  # Two consecutive double quotes
```

### Identifiers

#### **Identifier Types**:
```yaml
Properties:
  # Regular identifiers
  Text: =SimpleName
  Text: =Control1
  Text: =MyVariable
  
  # Single-quoted identifiers (for special characters)
  Text: ='Name with spaces'
  Text: ='Name with "quotes"'
  Text: ='Field@Email.com'
```

#### **Regular Identifiers**:
```yaml
Identifier:
  IdentifierName but not Operator or ContextKeyword

IdentifierName:
  IdentifierStartCharacter IdentifierContinueCharacters(opt)
  "'" SingleQuotedIdentifier "'"

IdentifierStartCharacter:
  LetterCharacter
  "_"

IdentifierContinueCharacter:
  IdentifierStartCharacter
  DecimalDigitCharacter
  ConnectingCharacter
  CombiningCharacter
  FormattingCharacter
```

#### **Character Classes**:
```yaml
LetterCharacter:
  # Unicode categories
  - Uppercase letter (Lu)
  - Lowercase letter (Ll)
  - Titlecase letter (Lt)
  - Letter modifier (Lm)
  - Letter other (Lo)
  - Number letter (Nl)

DecimalDigitCharacter:
  # Unicode category Decimal digit (Nd)
  
ConnectingCharacter:
  # Unicode category Connector punctuation (Pc)
  
CombiningCharacter:
  # Unicode categories
  - Non-spacing mark (Mn)
  - Spacing combining mark (Mc)
  
FormattingCharacter:
  # Unicode category Format (Cf)
```

#### **Single-Quoted Identifiers**:
```yaml
Properties:
  # Identifiers with spaces
  Text: ='Full Name'
  Text: ='Customer ID'
  
  # Identifiers with special characters
  Text: ='Field@Email.com'
  Text: ='Price (USD)'
  
  # Identifiers with quotes (escaped)
  Text: ='Name with ''quotes'''    # Name with 'quotes'
```

**Grammar**:
```yaml
SingleQuotedIdentifier:
  SingleQuotedIdentifierCharacters

SingleQuotedIdentifierCharacters:
  SingleQuotedIdentifierCharacter SingleQuotedIdentifierCharacters(opt)

SingleQuotedIdentifierCharacter:
  TextCharactersNoSingleQuote
  SingleQuoteEscapeSequence

TextCharactersNoSingleQuote:
  any Unicode character except ' (U+0027)

SingleQuoteEscapeSequence:
  "''"  # Two consecutive single quotes
```

#### **Disambiguated Identifiers**:
```yaml
Properties:
  # Table column disambiguation
  Text: =Employees[@Name]         # Column Name from Employees table
  Text: =Orders[@CustomerID]      # Column CustomerID from Orders table
  
  # Global identifier (accessing outer scope)
  Text: =[@MyGlobalVariable]      # Global variable when in record scope
```

**Grammar**:
```yaml
DisambiguatedIdentifier:
  TableColumnIdentifier
  GlobalIdentifier

TableColumnIdentifier:
  Identifier "[@" Identifier "]"

GlobalIdentifier:
  "[@" Identifier "]"
```

#### **Context Keywords**:
```yaml
Properties:
  # Context-sensitive keywords
  Text: =Parent.Width              # Reference to parent container
  Text: =Self.Fill                 # Reference to current control
  Text: =ThisItem.Name             # Current item in gallery/collection
  Text: =ThisRecord.Status         # Current record in ForAll
```

**Grammar**:
```yaml
ContextKeyword: (one of)
  Parent
  Self
  ThisItem
  ThisRecord
```

#### **Case Sensitivity**:
- Power Fx identifiers are case-sensitive
- Authoring tools auto-correct to proper case
- Function names, operators, and keywords follow specific casing

### Separators

#### **Locale-Dependent Separators**:
```yaml
# Decimal separator (locale-dependent)
DecimalSeparator:
  "."    # Dot for English and most locales (1.23)
  ","    # Comma for some European locales (1,23)

# List separator (depends on decimal separator)
ListSeparator:
  ","    # Comma when decimal separator is dot
  ";"    # Semicolon when decimal separator is comma

# Chaining separator (depends on decimal separator)
ChainingSeparator:
  ";"    # Semicolon when decimal separator is dot
  ";;"   # Double semicolon when decimal separator is comma
```

#### **Usage Examples**:
```yaml
Properties:
  # Function arguments (using list separator)
  Text: =Concatenate("Hello", " ", "World")    # English locale
  Text: =Concatenate("Hello"; " "; "World")    # Some European locales
  
  # Chained expressions (using chaining separator)
  OnSelect: =Set(Var1, 10); Set(Var2, 20); Navigate(Screen2)     # English
  OnSelect: =Set(Var1, 10);; Set(Var2, 20);; Navigate(Screen2)   # European
  
  # Decimal numbers (using decimal separator)
  Text: =1.23    # English locale
  Text: =1,23    # Some European locales
```

### Operators

#### **Operator Categories**:
```yaml
Operator:
  BinaryOperator
  BinaryOperatorRequiresWhitespace
  PrefixOperator
  PrefixOperatorRequiresWhitespace
  PostfixOperator
```

#### **Binary Operators**:
```yaml
Properties:
  # Comparison operators
  Visible: =Value1 = Value2         # Equal
  Visible: =Value1 <> Value2        # Not equal
  Visible: =Value1 < Value2         # Less than
  Visible: =Value1 <= Value2        # Less than or equal
  Visible: =Value1 > Value2         # Greater than
  Visible: =Value1 >= Value2        # Greater than or equal
  
  # Arithmetic operators
  Text: =Value1 + Value2            # Addition
  Text: =Value1 - Value2            # Subtraction
  Text: =Value1 * Value2            # Multiplication
  Text: =Value1 / Value2            # Division
  Text: =Value1 ^ Value2            # Exponentiation
  
  # String concatenation
  Text: ="Hello" & " " & "World"    # Concatenation
  
  # Logical operators (symbols)
  Visible: =Condition1 && Condition2    # AND
  Visible: =Condition1 || Condition2    # OR
  
  # Membership operators
  Visible: ="text" in StringField       # Case-insensitive contains
  Visible: ="TEXT" exactin StringField  # Case-sensitive contains
```

**Grammar**:
```yaml
BinaryOperator: (one of)
  = < <= > >= <>     # Comparison
  + - * / ^          # Arithmetic
  &                  # String concatenation
  && ||              # Logical
  in exactin         # Membership
```

#### **Binary Operators Requiring Whitespace**:
```yaml
Properties:
  # Logical operators (keywords)
  Visible: =Condition1 And Condition2   # Must have whitespace after And
  Visible: =Condition1 Or Condition2    # Must have whitespace after Or
```

**Grammar**:
```yaml
BinaryOperatorRequiresWhitespace:
  "And" Whitespace
  "Or" Whitespace
```

#### **Prefix Operators**:
```yaml
Properties:
  # Logical NOT (symbol)
  Visible: =!IsBlank(TextInput1.Text)
  
  # Logical NOT (keyword - requires whitespace)
  Visible: =Not IsBlank(TextInput1.Text)
```

**Grammar**:
```yaml
PrefixOperator:
  "!"

PrefixOperatorRequiresWhitespace:
  "Not" Whitespace
```

#### **Postfix Operators**:
```yaml
Properties:
  # Percentage operator
  Text: =50%              # Equivalent to 0.5
  Text: =Value * 25%      # 25 percent of Value
```

**Grammar**:
```yaml
PostfixOperator:
  "%"
```

#### **Reference Operators**:
```yaml
Properties:
  # Dot notation (standard property access)
  Text: =Control1.Text
  Text: =Parent.Width
  
  # Bang notation (identifier with special characters)
  Text: =Table1!'Field Name'
  Text: =Collection1!'Email@Address'
```

**Grammar**:
```yaml
ReferenceOperator: (one of)
  "."    # Dot notation
  "!"    # Bang notation
```

### Object References

#### **Reference Structure**:
```yaml
Properties:
  # Simple references
  Text: =MyVariable
  Text: =Control1
  
  # Chained references
  Text: =Control1.Text
  Text: =Parent.Width
  Text: =Gallery1.Selected.Name
  
  # Complex references
  Text: =Collection1!'Field Name'.SubField
```

**Grammar**:
```yaml
Reference:
  BaseReference
  BaseReference ReferenceOperator ReferenceList

BaseReference:
  Identifier
  DisambiguatedIdentifier
  ContextKeyword

ReferenceList:
  Identifier
  Identifier ReferenceOperator ReferenceList
```

### Inline Data Structures

#### **Inline Records**:
```yaml
Properties:
  # Simple record
  Items: ={Name: "John", Age: 30}
  
  # Record with expressions
  Items: ={
    Name: TextInput1.Text,
    Created: Now(),
    IsActive: Toggle1.Value
  }
  
  # Empty record
  Items: ={}
```

**Grammar**:
```yaml
InlineRecord:
  "{" InlineRecordList(opt) "}"

InlineRecordList:
  Identifier ":" Expression
  Identifier ":" Expression ListSeparator InlineRecordList
```

#### **Inline Tables**:
```yaml
Properties:
  # Simple table
  Items: =["Option 1", "Option 2", "Option 3"]
  
  # Table with expressions
  Items: =[
    TextInput1.Text,
    TextInput2.Text,
    "Static Value"
  ]
  
  # Empty table
  Items: =[]
```

**Grammar**:
```yaml
InlineTable:
  "[" InlineTableList(opt) "]"

InlineTableList:
  Expression
  Expression ListSeparator InlineTableList
```

### Function Calls

#### **Function Call Syntax**:
```yaml
Properties:
  # Simple function call
  Text: =Upper("hello")
  Text: =Now()
  
  # Function with multiple arguments
  Text: =Concatenate("Hello", " ", "World")
  
  # Nested function calls
  Text: =Upper(Left(TextInput1.Text, 10))
  
  # Namespace qualified functions
  Text: =Math.Round(3.14159, 2)
```

**Grammar**:
```yaml
FunctionCall:
  FunctionIdentifier "(" FunctionArguments(opt) ")"

FunctionIdentifier:
  Identifier
  Identifier "." FunctionIdentifier

FunctionArguments:
  ChainedExpression
  ChainedExpression ListSeparator FunctionArguments
```

### Expression Structure

#### **Expression Types**:
```yaml
Expression:
  Literal                                    # 123, "text", true
  Reference                                 # Control1.Text
  InlineRecord                             # {Name: "John"}
  InlineTable                              # ["A", "B", "C"]
  FunctionCall                             # Upper("text")
  "(" Expression ")"                       # Parenthesized expression
  PrefixOperator Expression                # !condition
  Expression PostfixOperator               # value%
  Expression BinaryOperator Expression     # a + b
```

#### **Chained Expressions**:
```yaml
Properties:
  # Multiple statements separated by chaining separator
  OnSelect: =
    Set(Var1, 10);
    Set(Var2, 20);
    Navigate(Screen2)
  
  # Each statement is an expression
  OnSelect: =
    UpdateContext({Loading: true});
    ClearCollect(Data, DataSource);
    UpdateContext({Loading: false})
```

**Grammar**:
```yaml
ChainedExpression:
  Expression
  Expression ChainingSeparator ChainedExpression(opt)
```

### Operator Precedence

#### **Precedence Levels** (highest to lowest):
```yaml
1. Primary expressions:
   - Literals: 123, "text", true
   - Identifiers: MyVar, Control1
   - Parentheses: (expression)
   - Function calls: Function(args)

2. Postfix operators:
   - Percentage: value%

3. Member access:
   - Dot notation: object.property
   - Bang notation: table!'field name'

4. Prefix operators:
   - Logical NOT: !condition, Not condition

5. Multiplicative:
   - Multiplication: a * b
   - Division: a / b

6. Additive:
   - Addition: a + b
   - Subtraction: a - b

7. Exponentiation:
   - Power: a ^ b

8. String concatenation:
   - Concatenation: "a" & "b"

9. Relational:
   - Comparison: a < b, a <= b, a > b, a >= b

10. Equality:
    - Equal: a = b
    - Not equal: a <> b

11. Membership:
    - Contains: "text" in field
    - Exact contains: "TEXT" exactin field

12. Logical AND:
    - Symbol: condition1 && condition2
    - Keyword: condition1 And condition2

13. Logical OR:
    - Symbol: condition1 || condition2
    - Keyword: condition1 Or condition2
```

#### **Precedence Examples**:
```yaml
Properties:
  # Arithmetic precedence
  Text: =2 + 3 * 4          # Result: 14 (not 20)
  Text: =(2 + 3) * 4        # Result: 20
  
  # Comparison and logical precedence
  Visible: =Value1 > 10 And Value2 < 20    # (Value1 > 10) And (Value2 < 20)
  
  # String concatenation precedence
  Text: ="Value: " & Value1 + Value2       # "Value: " & (Value1 + Value2)
  Text: =("Value: " & Value1) + Value2     # Explicit grouping
```

### Grammar Validation Examples

#### **Valid Expressions**:
```yaml
Properties:
  # Simple literals
  Text: =123
  Text: ="Hello World"
  Text: =true
  
  # Complex expressions
  Text: =If(
    Toggle1.Value,
    "Option A",
    "Option B"
  )
  
  # Record and table literals
  Items: ={Name: "John", Age: 30, Active: true}
  Items: =["Red", "Green", "Blue"]
  
  # Function calls with various argument patterns
  Text: =Concatenate("Hello", " ", TextInput1.Text)
  Items: =Sort(Filter(DataSource, Active = true), Name)
  
  # Complex chained expressions
  OnSelect: =
    UpdateContext({IsLoading: true});
    ClearCollect(TempData, 
      AddColumns(
        Filter(DataSource, Status = "Active"),
        "DisplayText", Name & " (" & ID & ")"
      )
    );
    UpdateContext({IsLoading: false});
    Navigate(ResultScreen)
```

#### **Grammar Error Examples**:
```yaml
Properties:
  # Missing quotes in text literal
  Text: =Hello World           # Error: Invalid identifier
  
  # Unmatched parentheses
  Text: =Upper("hello"          # Error: Missing closing parenthesis
  
  # Invalid operator usage
  Text: =10 ++ 20              # Error: Invalid operator
  
  # Missing expression after operator
  Text: =10 +                  # Error: Incomplete expression
  
  # Invalid function syntax
  Text: =Upper)                # Error: Missing opening parenthesis
```

### Advanced Grammar Features

#### **Whitespace Handling**:
```yaml
Properties:
  # Whitespace is generally optional around operators
  Text: =10+20                 # Valid
  Text: =10 + 20              # Valid (preferred for readability)
  
  # Some operators require whitespace
  Visible: =Value1AndValue2    # Error: Invalid identifier
  Visible: =Value1 And Value2  # Valid: And requires whitespace
  
  # Whitespace in function calls
  Text: =Upper("hello")        # Valid
  Text: =Upper ( "hello" )     # Valid but not recommended
```

#### **Comment Integration**:
```yaml
Properties:
  # Comments can appear between tokens
  Text: =/* Start */ Upper("hello") /* End */
  
  # Single-line comments
  OnSelect: =Set(MyVar, 10)    // Set variable value
  
  # Multi-line expressions with comments
  Items: =Sort(              /* Sort the filtered data */
    Filter(DataSource,       // Apply filter first
      Status = "Active"      /* Only active records */
    ),
    Name                     // Sort by name field
  )
```

#### **Unicode Support**:
```yaml
Properties:
  # Unicode in text literals
  Text: ="Bienvenidos"        # Spanish
  Text: ="欢迎"               # Chinese
  Text: ="مرحبا"              # Arabic
  
  # Unicode in identifiers (with quotes)
  Text: ='用户名'              # Chinese field name
  Text: ='Benutzername'       # German field name
```

### Grammar Best Practices

#### **Readability Guidelines**:
```yaml
Properties:
  # Use whitespace for clarity
  Text: =Value1 + Value2 * Value3    # Clear precedence
  Text: =(Value1 + Value2) * Value3  # Explicit grouping when needed
  
  # Break complex expressions across lines
  Items: =Filter(
    Sort(DataSource, Name),
    Status = "Active" And 
    Created >= DateAdd(Today(), -30, Days)
  )
  
  # Use meaningful variable names
  OnSelect: =
    Set(IsProcessing, true);
    Set(FilteredResults, Filter(Data, Criteria));
    Set(IsProcessing, false)
```

#### **Error Prevention**:
```yaml
Properties:
  # Always close parentheses and brackets
  Text: =Upper(Left(TextInput1.Text, 10))    # Properly nested
  
  # Use quotes consistently
  Text: ="Static text" & DynamicField        # Mixed literal and reference
  
  # Handle operator precedence explicitly
  Text: =(Price * Quantity) + Tax           # Clear calculation order
```

#### **Performance Considerations**:
```yaml
Properties:
  # Avoid deeply nested expressions in frequently updated properties
  # Instead of:
  Text: =If(Condition1, 
    Upper(Left(Concatenate(Field1, " ", Field2), 20)),
    Lower(Right(Concatenate(Field3, " ", Field4), 15))
  )
  
  # Use variables:
  OnVisible: =Set(DisplayText, 
    If(Condition1,
      Upper(Left(Field1 & " " & Field2, 20)),
      Lower(Right(Field3 & " " & Field4, 15))
    )
  )
```

## Power Fx Global Support and Internationalization

Power Fx is designed as a global product, supporting multiple languages and regions for both development and runtime experiences. This comprehensive guide covers language settings, authoring environment localization, and creating globally-aware applications.

### Overview

Power Fx provides global support through:
- **Localized Authoring**: Development tools adapt to author's language
- **Regional Number Formats**: Decimal separators adjust based on locale
- **Runtime Localization**: Apps display content in user's language
- **Date/Time Formatting**: Locale-aware date and time handling
- **Global Functions**: Built-in functions for international apps

#### **Core Principles**:
- Apps stored in language-agnostic format
- Multiple authors can edit same app in different languages
- Runtime adapts to user's language and region
- Follows Excel conventions for maximum familiarity

### Language Settings Configuration

#### **Platform-Specific Language Sources**:

**Native Studio/Player**:
```yaml
# Language provided by host operating system
Windows Settings:
  - Navigate to "All Settings"
  - Select "Time & language"
  - Configure language and regional format
  - Override decimal separator if needed
```

**Web Experiences**:
```yaml
# Language provided by browser
Browser Configuration:
  - Defaults to host operating system setting
  - Manual override available in browser settings
  - Affects Power Apps web studio and player
```

#### **Language Detection in Power Fx**:
```yaml
Properties:
  # Get current user's language
  Text: =Language()               # Returns "en-US", "es-ES", etc.
  
  # Use in conditional logic
  Text: =If(
    Left(Language(), 2) = "es",
    "Hola Mundo",
    "Hello World"
  )
  
  # Language-aware content selection
  Items: =Filter(
    ContentTable,
    LanguageCode = Left(Language(), 2) || 
    LanguageCode = "en"  // Fallback to English
  )
```

### Authoring Environment Localization

#### **Element Names in Formulas**:

**Always English Elements**:
```yaml
Properties:
  # Function names always English
  Text: =If(condition, "Yes", "No")
  Text: =Concatenate("Hello", " ", "World")
  Text: =Navigate(NextScreen, ScreenTransition.Fade)
  
  # Control property names always English
  Fill: =Button1.Fill
  Text: =TextInput1.Text
  Visible: =Gallery1.Selected.Name
  
  # Enumeration names always English
  Fill: =Color.Red
  BorderStyle: =BorderStyle.Solid
  Align: =Align.Center
  
  # Signal records always English
  Text: =Compass.Heading
  Text: =Location.Latitude
  Text: =App.ActiveScreen.Name
  
  # Operators always English
  Visible: =Value1 in Collection1
  Text: =Parent.Width
```

**Localized Elements**:
```yaml
# Control names default to English but can be renamed
Controls:
  Button1: "English default name"
  Botón1: "Spanish renamed control"
  Bouton1: "French renamed control"
  
# Collection names can be localized
Collections:
  MyCollection: "English collection name"
  MiColección: "Spanish collection name"
  MaCollection: "French collection name"
  
# Context variable names can be localized
Variables:
  MyVariable: "English variable"
  MiVariable: "Spanish variable"
  MaVariable: "French variable"
```

#### **Control Insertion and Naming**:
```yaml
# Spanish authoring experience shows localized control names:
# - "Casilla" (Checkbox)
# - "Botón" (Button)  
# - "Entrada de texto" (Text Input)

# But controls are inserted with English names:
# - Checkbox1
# - Button1
# - TextInput1

# Controls can be renamed after insertion
Properties:
  # Rename control via Content ribbon
  # Select control name on far left
  # Edit in dropdown text box
  Name: "Casilla1"  // Spanish name for checkbox
```

### Formula Separators and Chaining Operators

#### **Locale-Dependent Separators**:

**Decimal Separator Impact**:
```yaml
# English/UK/Japan (dot as decimal separator)
Decimal: "."         # 12.59
List: ","            # Function(arg1, arg2, arg3)
Chaining: ";"        # Statement1; Statement2; Statement3

# France/Spain/Germany (comma as decimal separator)  
Decimal: ","         # 12,59
List: ";"            # Function(arg1; arg2; arg3)
Chaining: ";;"       # Statement1;; Statement2;; Statement3
```

#### **Formula Examples by Locale**:

**English/UK/Japan Locale** (dot decimal separator):
```yaml
Properties:
  # Decimal numbers with dots
  Text: =12.59
  Text: =Value1 + 100.25
  
  # Function arguments separated by commas
  Text: =Concatenate("Hello", " ", "World")
  Items: =Sort(MyCollection, Name, Ascending)
  
  # Statements chained with semicolons
  OnSelect: =
    Set(IsLoading, true);
    Collect(MyData, RemoteSource);
    Set(IsLoading, false)
  
  # Complex formula example
  OnSelect: =If(
    Slider1.Value > 12.59,
    Notify("Valid!", NotificationType.Success);
    Navigate("NextScreen", ScreenTransition.None),
    Notify("Invalid, try again", NotificationType.Error)
  )
```

**French/Spanish/German Locale** (comma decimal separator):
```yaml
Properties:
  # Decimal numbers with commas
  Text: =12,59
  Text: =Value1 + 100,25
  
  # Function arguments separated by semicolons
  Text: =Concatenate("Hello"; " "; "World")
  Items: =Sort(MyCollection; Name; Ascending)
  
  # Statements chained with double semicolons
  OnSelect: =
    Set(IsLoading; true);;
    Collect(MyData; RemoteSource);;
    Set(IsLoading; false)
  
  # Same formula with European separators
  OnSelect: =If(
    Slider1.Value > 12,59;
    Notify("Valid!"; NotificationType.Success);;
    Navigate("NextScreen"; ScreenTransition.None);
    Notify("Invalid, try again"; NotificationType.Error)
  )
```

#### **Property Access Operator**:
```yaml
Properties:
  # Property selection operator (.) never changes
  Text: =Slider1.Value        # Always dot, regardless of locale
  Fill: =Button1.Fill         # Always dot, regardless of locale
  Items: =Gallery1.Selected   # Always dot, regardless of locale
```

#### **Multi-Author Collaboration**:
```yaml
# Formula stored internally in language-agnostic format
# Each author sees appropriate separators for their language

Internal Formula:
  "If(Slider1.Value > 12.59, Notify(...), Notify(...))"

English Author Sees:
  "If(Slider1.Value > 12.59, Notify(...), Notify(...))"

Spanish Author Sees:  
  "If(Slider1,Value > 12,59; Notify(...); Notify(...))"

# Both authors edit the same formula with locale-appropriate display
```

### Creating Global Applications

#### **Language Function Usage**:

**Basic Language Detection**:
```yaml
Properties:
  # Get full language tag
  Text: =Language()           # Returns "en-GB", "de-DE", "fr-FR", etc.
  
  # Extract language code only
  Text: =Left(Language(), 2)  # Returns "en", "de", "fr", etc.
  
  # Language-specific content
  Text: =Switch(
    Left(Language(), 2),
    "es", "Hola",
    "fr", "Bonjour", 
    "de", "Hallo",
    "Hello"  // Default English
  )
```

**Localization Table Pattern**:
```yaml
# Create localization table
LocalizationTable:
  - TextID: "Hello", LanguageTag: "en", LocalizedText: "Hello"
  - TextID: "Hello", LanguageTag: "es", LocalizedText: "Hola"
  - TextID: "Hello", LanguageTag: "fr", LocalizedText: "Bonjour"
  - TextID: "Hello", LanguageTag: "de", LocalizedText: "Hallo"
  - TextID: "Goodbye", LanguageTag: "en", LocalizedText: "Goodbye"
  - TextID: "Goodbye", LanguageTag: "es", LocalizedText: "Adiós"

Properties:
  # Lookup localized text with fallback
  Text: =LookUp(
    LocalizationTable,
    TextID = "Hello" && (
      LanguageTag = Left(Language(), 2) ||
      IsBlank(LanguageTag)
    )
  ).LocalizedText
  
  # More robust lookup with English fallback
  Text: =Coalesce(
    LookUp(LocalizationTable, 
      TextID = "Hello" && LanguageTag = Left(Language(), 2)
    ).LocalizedText,
    LookUp(LocalizationTable,
      TextID = "Hello" && LanguageTag = "en"
    ).LocalizedText,
    "Hello"  // Hard-coded fallback
  )
```

#### **UI Layout Considerations**:
```yaml
Properties:
  # Account for varying text lengths across languages
  Width: =Max(
    TextWidth("English Text", Font.Default, FontSize.Medium),
    TextWidth("Texto en Español", Font.Default, FontSize.Medium),
    TextWidth("Texte en Français", Font.Default, FontSize.Medium)
  ) + 20  // Add padding
  
  # Dynamic width based on current language
  Width: =TextWidth(
    LocalizedText,
    Font.Default,
    FontSize.Medium
  ) + 40
  
  # Responsive label sizing
  AutoHeight: =true
  Wrap: =true
  Width: =Parent.Width * 0.8
```

### Number, Date, and Time Formatting

#### **Text Function for Formatting**:

**Global-Aware Enumerations** (Recommended):
```yaml
Properties:
  # Date formatting with built-in localization
  Text: =Text(Now(), DateTimeFormat.LongDate)
  Text: =Text(Now(), DateTimeFormat.ShortDate)
  Text: =Text(Now(), DateTimeFormat.LongTime)
  Text: =Text(Now(), DateTimeFormat.ShortTime)
  
  # Number formatting with built-in localization
  Text: =Text(1234.56, NumberFormat.Currency)
  Text: =Text(0.75, NumberFormat.Percentage)
  Text: =Text(1234567, NumberFormat.Number)
  
  # Results vary by user's language:
  # English: "December 25, 2024", "$1,234.56", "75%"
  # Spanish: "25 de diciembre de 2024", "1.234,56 €", "75%"
  # German: "25. Dezember 2024", "1.234,56 €", "75%"
```

**Custom Format Strings**:
```yaml
Properties:
  # Custom format with language specification
  Text: =Text(Now(), "[$-en-US]dddd, mmmm dd, yyyy")
  Text: =Text(Now(), "[$-es-ES]dddd, dd 'de' mmmm 'de' yyyy")
  Text: =Text(Now(), "[$-de-DE]dddd, dd. mmmm yyyy")
  
  # Number formatting with locale
  Text: =Text(1234.56, "[$-en-US]$#,##0.00")
  Text: =Text(1234.56, "[$-de-DE]#.##0,00 €")
  Text: =Text(1234.56, "[$-fr-FR]#,##0.00 €")
  
  # The "[$-en-US]" prefix tells Text function the format language
  # This is automatically inserted based on authoring language
```

**Explicit Language Override**:
```yaml
Properties:
  # Third argument specifies output language
  Text: =Text(Now(), DateTimeFormat.LongDate, "en-US")
  Text: =Text(Now(), DateTimeFormat.LongDate, "es-ES") 
  Text: =Text(Now(), DateTimeFormat.LongDate, "de-DE")
  
  # Force specific currency format regardless of user locale
  Text: =Text(Amount, "[$-en-US]$#,##0.00", "en-US")
  Text: =Text(Amount, "[$-de-DE]#.##0,00 €", "de-DE")
```

### Reading User Input

#### **Value Conversion Functions**:

**Number Parsing**:
```yaml
Properties:
  # Parse number based on user's locale
  OnSelect: =Set(NumericValue, Value(TextInput1.Text))
  
  # Parse with explicit locale
  OnSelect: =Set(NumericValue, Value(TextInput1.Text, "en-US"))
  OnSelect: =Set(NumericValue, Value(TextInput1.Text, "de-DE"))
  
  # Examples by locale:
  # English: Value("12,345.67") = 12345.67
  # German: Value("12.345,67") = 12345.67
  # French: Value("12 345,67") = 12345.67
```

**Date Parsing**:
```yaml
Properties:
  # Parse date based on user's locale
  OnSelect: =Set(DateValue, DateValue(DateInput.Text))
  
  # Parse with explicit locale
  OnSelect: =Set(DateValue, DateValue(DateInput.Text, "en-US"))
  OnSelect: =Set(DateValue, DateValue(DateInput.Text, "de-DE"))
  
  # Examples by locale:
  # US: DateValue("1/2/01") = February 1, 2001
  # European: DateValue("1/2/01") = January 2, 2001
  # ISO: DateValue("2001-02-01") = February 1, 2001 (universal)
```

**Time Parsing**:
```yaml
Properties:
  # Parse time based on user's locale
  OnSelect: =Set(TimeValue, TimeValue(TimeInput.Text))
  
  # Parse with explicit locale
  OnSelect: =Set(TimeValue, TimeValue(TimeInput.Text, "en-US"))
  OnSelect: =Set(TimeValue, TimeValue(TimeInput.Text, "fr-FR"))
  
  # Examples:
  # 12-hour: TimeValue("11:43:02 PM") = 23:43:02
  # 24-hour: TimeValue("23:43:02") = 23:43:02
  # French: TimeValue("11:43:02") = 11:43:02
```

**DateTime Parsing**:
```yaml
Properties:
  # Parse date and time together
  OnSelect: =Set(DateTimeValue, DateTimeValue(DateTimeInput.Text))
  
  # Parse with explicit locale
  OnSelect: =Set(DateTimeValue, DateTimeValue(DateTimeInput.Text, "de-DE"))
  
  # Examples:
  # US: DateTimeValue("1/2/01 11:43:02 PM") = Feb 1, 2001 23:43:02
  # German: DateTimeValue("1.2.01 23:43:02") = Feb 1, 2001 23:43:02
  # ISO: DateTimeValue("2001-02-01T23:43:02") = Feb 1, 2001 23:43:02
```

#### **Input Validation with Localization**:
```yaml
Properties:
  # Validate numeric input
  Visible: =Or(
    IsBlank(TextInput1.Text),
    Not(IsNumeric(TextInput1.Text))
  )
  Text: =If(
    Left(Language(), 2) = "es",
    "Ingrese un número válido",
    "Enter a valid number"
  )
  
  # Validate date input
  Visible: =And(
    Not(IsBlank(DateInput.Text)),
    IsError(DateValue(DateInput.Text))
  )
  Text: =Switch(
    Left(Language(), 2),
    "es", "Formato de fecha inválido",
    "fr", "Format de date invalide", 
    "de", "Ungültiges Datumsformat",
    "Invalid date format"
  )
```

### Calendar and Clock Information

#### **Calendar Functions**:
```yaml
Properties:
  # Get localized month names
  Items: =Calendar.MonthsLong()      # January, February, March...
  Items: =Calendar.MonthsShort()     # Jan, Feb, Mar...
  
  # Get localized day names  
  Items: =Calendar.WeekdaysLong()    # Sunday, Monday, Tuesday...
  Items: =Calendar.WeekdaysShort()   # Sun, Mon, Tue...
  
  # Use in dropdown controls
  Items: =Calendar.MonthsLong()
  DefaultSelectedItems: =First(Calendar.MonthsLong())
  
  # Results vary by language:
  # English: "January", "February", "March"
  # Spanish: "enero", "febrero", "marzo"  
  # French: "janvier", "février", "mars"
  # German: "Januar", "Februar", "März"
```

#### **Clock Functions**:
```yaml
Properties:
  # Get localized time format information
  Text: =Clock.AmPm()               # "AM", "PM" or equivalent
  Text: =Clock.IsClock24()          # true/false for 24-hour clock
  
  # Use with time picker
  Items: =If(
    Clock.IsClock24(),
    [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23],
    [1,2,3,4,5,6,7,8,9,10,11,12]
  )
  
  # Conditional AM/PM display
  Visible: =Not(Clock.IsClock24())
  Items: =Clock.AmPm()
```

#### **Building Localized Date Pickers**:
```yaml
# Month dropdown
MonthDropdown:
  Items: =AddColumns(
    Calendar.MonthsLong(),
    "MonthNumber", 
    Value(Split(Concatenate("1,2,3,4,5,6,7,8,9,10,11,12"), ",").Result)
  )
  
# Day dropdown  
DayDropdown:
  Items: =Sequence(
    Day(DateAdd(Date(Year(Today()), MonthDropdown.Selected.MonthNumber, 1), -1, Days)),
    1
  )
  
# Year dropdown
YearDropdown:
  Items: =Sequence(100, Year(Today()) - 50)

# Construct selected date
SelectedDate:
  =Date(
    YearDropdown.Selected.Value,
    MonthDropdown.Selected.MonthNumber, 
    DayDropdown.Selected.Value
  )
```

### Advanced Global Patterns

#### **Multi-Language Content Management**:
```yaml
# Content table structure
ContentTable:
  - ID: "welcome_title", Language: "en", Content: "Welcome to Our App"
  - ID: "welcome_title", Language: "es", Content: "Bienvenido a Nuestra App"
  - ID: "welcome_title", Language: "fr", Content: "Bienvenue dans Notre App"
  - ID: "welcome_body", Language: "en", Content: "Please sign in to continue"
  - ID: "welcome_body", Language: "es", Content: "Por favor inicie sesión para continuar"
  - ID: "welcome_body", Language: "fr", Content: "Veuillez vous connecter pour continuer"

Properties:
  # Content lookup function
  OnStart: =Set(
    GetLocalizedText,
    Function(
      contentId As Text,
      Optional fallbackLang As Text
    ),
    With(
      {
        primaryLookup: LookUp(
          ContentTable,
          ID = contentId && Language = Left(Language(), 2)
        ),
        fallbackLookup: LookUp(
          ContentTable, 
          ID = contentId && Language = Coalesce(fallbackLang, "en")
        )
      },
      Coalesce(primaryLookup.Content, fallbackLookup.Content, contentId)
    )
  )
  
  # Usage
  Text: =GetLocalizedText("welcome_title")
  Text: =GetLocalizedText("welcome_body")
```

#### **Currency and Number Formatting by Region**:
```yaml
Properties:
  # Region-specific currency formatting
  Text: =Switch(
    Left(Language(), 2),
    "en", Text(Amount, "[$-en-US]$#,##0.00"),
    "de", Text(Amount, "[$-de-DE]#.##0,00 €"),
    "fr", Text(Amount, "[$-fr-FR]#,##0.00 €"),
    "es", Text(Amount, "[$-es-ES]#.##0,00 €"),
    "jp", Text(Amount, "[$-ja-JP]¥#,##0"),
    Text(Amount, NumberFormat.Currency)  // Default
  )
  
  # Percentage formatting by region
  Text: =Switch(
    Left(Language(), 2),
    "en", Text(Percentage, "0.00%"),
    "de", Text(Percentage, "0,00%"),
    "fr", Text(Percentage, "0,00 %"),
    Text(Percentage, NumberFormat.Percentage)
  )
```

#### **Time Zone Handling**:
```yaml
Properties:
  # Display time in user's time zone
  Text: =Text(
    DateAdd(UTCDateTime, TimeZoneInformation().Bias, Minutes),
    DateTimeFormat.LongDateTime
  )
  
  # Store UTC, display local
  OnSelect: =Patch(
    DataSource,
    ThisItem,
    {
      LastModified: UTCNow(),
      LastModifiedDisplay: Text(Now(), DateTimeFormat.ShortDateTime)
    }
  )
  
  # Time zone aware scheduling
  MeetingTime: =DateAdd(
    LocalMeetingTime,
    -TimeZoneInformation().Bias,
    Minutes
  )
```

#### **Right-to-Left (RTL) Language Support**:
```yaml
Properties:
  # Detect RTL languages
  IsRTL: =Left(Language(), 2) in ["ar", "he", "fa", "ur"]
  
  # Conditional layout alignment
  Align: =If(IsRTL, Align.Right, Align.Left)
  X: =If(IsRTL, Parent.Width - Self.Width, 0)
  
  # Icon direction for RTL
  Icon: =If(
    IsRTL,
    Icon.ChevronLeft,   // Reverse direction for RTL
    Icon.ChevronRight
  )
  
  # Text direction
  Text: =LocalizedText
  TextDirection: =If(IsRTL, TextDirection.RightToLeft, TextDirection.LeftToRight)
```

### Global Testing and Validation

#### **Multi-Language Testing Patterns**:
```yaml
Properties:
  # Test different language scenarios
  OnStart: =Set(TestLanguages, ["en-US", "es-ES", "fr-FR", "de-DE", "ja-JP"])
  
  # Language simulation for testing
  SimulatedLanguage: ="es-ES"  // Test with Spanish
  
  # Override Language() function for testing
  TestText: =Switch(
    SimulatedLanguage,
    "es-ES", "Hola Mundo",
    "fr-FR", "Bonjour le Monde", 
    "de-DE", "Hallo Welt",
    "Hello World"
  )
  
  # Test number formatting
  TestNumber: =Switch(
    SimulatedLanguage,
    "de-DE", "1.234,56",
    "fr-FR", "1 234,56",
    "1,234.56"
  )
```

#### **Validation Checklist**:
```yaml
# Global app validation checklist:
✓ All user-facing text has translations
✓ UI layouts accommodate longer translations  
✓ Number inputs work with locale-specific formats
✓ Date inputs accept locale-specific formats
✓ Currency displays correctly for target regions
✓ Time displays use appropriate 12/24 hour format
✓ Calendar controls show localized month/day names
✓ Error messages are translated
✓ Help text and tooltips are translated
✓ RTL languages layout correctly
✓ Font supports all required character sets
✓ Images/icons are culturally appropriate
```

### Global App Best Practices

#### **Development Guidelines**:
```yaml
# Use consistent localization keys
LocalizationKeys:
  - "app.title" (not "title")
  - "button.save" (not "save")
  - "error.required_field" (not "required")
  
# Provide context in localization table
LocalizationTable:
  - ID: "button.save", Context: "Save button on form", MaxLength: 10
  - ID: "error.required_field", Context: "Validation error", MaxLength: 50
  
# Plan for text expansion
EnglishText: "Save"           # 4 characters
GermanText: "Speichern"       # 10 characters (2.5x expansion)
FrenchText: "Sauvegarder"     # 12 characters (3x expansion)
```

#### **Performance Optimization**:
```yaml
Properties:
  # Cache localized content
  OnStart: =ClearCollect(
    LocalizedContent,
    Filter(
      ContentTable,
      Language = Left(Language(), 2) ||
      Language = "en"  // Include fallback
    )
  )
  
  # Efficient lookup
  Text: =LookUp(LocalizedContent, ID = "welcome_title").Content
  
  # Minimize Language() function calls
  OnStart: =Set(UserLanguage, Left(Language(), 2))
  Text: =If(UserLanguage = "es", "Hola", "Hello")
```

#### **Deployment Considerations**:
```yaml
# Environment-specific language handling
Properties:
  # Development environment override
  TestLanguage: ="es-ES"
  EffectiveLanguage: =If(
    App.Environment = "Development",
    Coalesce(TestLanguage, Language()),
    Language()
  )
  
  # Feature flag for new translations
  ShowNewTranslations: =LookUp(FeatureFlags, Name = "NewTranslations").Enabled
  Text: =If(
    ShowNewTranslations,
    GetLocalizedText("new_feature_text"),
    GetLocalizedText("original_text")
  )
```

## Power Fx Imperative Logic and Behavior Formulas

Power Fx supports both declarative formulas (for calculating values) and imperative logic (for performing actions and changing app state). This comprehensive guide covers imperative programming patterns, behavior formulas, and state management in Power Fx applications.

### Overview

Power Fx imperative logic enables:
- **State Changes**: Modify variables, collections, and data sources
- **Navigation Control**: Change screens and control user flow
- **Action Sequences**: Chain multiple operations together
- **Event Handling**: Respond to user interactions and system events
- **Side Effects**: Perform operations that affect app state

#### **Core Concepts**:
- **Behavior Formulas**: Execute when specific events occur
- **Action Functions**: Modify state rather than calculate values
- **Sequential Execution**: Actions performed in order with error handling
- **State Management**: Control variables, collections, and data sources

### Declarative vs Imperative Programming

#### **Declarative Formulas** (Automatic Recalculation):
```yaml
Properties:
  # Calculate value based on other values
  Color: =If(Value(TextBox1.Text) >= 0, Color.Black, Color.Red)
  
  # Automatically recalculates when TextBox1.Text changes
  Text: =Upper(TextInput1.Text)
  
  # Depends on multiple inputs
  Visible: =And(
    Not(IsBlank(NameInput.Text)),
    Not(IsBlank(EmailInput.Text))
  )
  
  # Complex calculations that update automatically
  Items: =Sort(
    Filter(DataSource, Status = StatusFilter.Selected.Value),
    ModifiedDate,
    Descending
  )
```

#### **Imperative Logic** (Event-Driven Actions):
```yaml
Properties:
  # User-initiated actions that change state
  OnSelect: =UpdateContext({ShowModal: true})
  
  # Navigate between screens
  OnSelect: =Navigate(NextScreen, ScreenTransition.SlideLeft)
  
  # Modify data sources
  OnSelect: =Patch(DataSource, ThisItem, {Status: "Completed"})
  
  # Multiple actions in sequence
  OnSelect: =
    Set(IsLoading, true);
    Collect(LocalData, RemoteDataSource);
    Set(IsLoading, false);
    Navigate(ResultsScreen)
```

### Behavior Formula Properties

#### **Common Behavior Properties**:

**Button and Control Events**:
```yaml
Properties:
  # Primary user interaction
  OnSelect: =Navigate(DetailScreen, ScreenTransition.Cover, {ID: ThisItem.ID})
  
  # Mouse/touch events
  OnMouseEnter: =UpdateContext({HoverState: true})
  OnMouseLeave: =UpdateContext({HoverState: false})
  
  # Focus events
  OnFocus: =Set(FocusedControl, Self.Name)
  OnBlur: =Set(FocusedControl, "")
```

**Screen Lifecycle Events**:
```yaml
Properties:
  # Screen becomes visible
  OnVisible: =
    UpdateContext({ScreenData: Blank()});
    ClearCollect(LocalCache, DataSource);
    Set(LoadComplete, true)
  
  # Screen becomes hidden
  OnHidden: =
    UpdateContext({UnsavedChanges: false});
    Clear(TempData)
  
  # App startup
  OnStart: =
    Set(AppVersion, "1.0.0");
    ClearCollect(GlobalSettings, SettingsSource);
    Set(InitComplete, true)
```

**Form Events**:
```yaml
Properties:
  # Form submission
  OnSuccess: =
    Notify("Record saved successfully", NotificationType.Success);
    Navigate(ListScreen, ScreenTransition.SlideRight)
  
  # Form errors
  OnFailure: =
    Notify("Error saving record: " & FirstError.Message, NotificationType.Error);
    UpdateContext({ShowErrorDetails: true})
  
  # Form reset
  OnReset: =
    UpdateContext({FormData: Blank()});
    ResetForm(Form1)
```

**Timer Events**:
```yaml
Properties:
  # Timer completion
  OnTimerEnd: =
    Set(TimerActive, false);
    Notify("Time's up!", NotificationType.Warning);
    UpdateContext({ShowResults: true})
  
  # Timer start
  OnTimerStart: =
    Set(StartTime, Now());
    UpdateContext({TimerRunning: true})
```

### Action Function Categories

#### **Navigation Functions**:

**Screen Navigation**:
```yaml
Properties:
  # Basic navigation
  OnSelect: =Navigate(NextScreen)
  
  # Navigation with transition
  OnSelect: =Navigate(DetailScreen, ScreenTransition.Fade)
  
  # Navigation with context data
  OnSelect: =Navigate(
    EditScreen, 
    ScreenTransition.Cover,
    {
      RecordID: Gallery1.Selected.ID,
      Mode: "Edit",
      ReturnScreen: App.ActiveScreen.Name
    }
  )
  
  # Conditional navigation
  OnSelect: =If(
    IsBlank(ValidationErrors),
    Navigate(SuccessScreen, ScreenTransition.SlideLeft),
    UpdateContext({ShowErrors: true})
  )
  
  # Return to previous screen
  OnSelect: =Back()
  
  # Navigate with screen replacement
  OnSelect: =Navigate(LoginScreen, ScreenTransition.None, {}, true)
```

**Deep Linking and URL Navigation**:
```yaml
Properties:
  # Launch external URL
  OnSelect: =Launch("https://www.microsoft.com")
  
  # Launch with parameters
  OnSelect: =Launch("mailto:user@company.com?subject=Hello&body=Message")
  
  # Launch app-specific URLs
  OnSelect: =Launch("ms-settings:network-wifi")
```

#### **Variable and State Management**:

**Global Variables**:
```yaml
Properties:
  # Set global variable
  OnSelect: =Set(CurrentUser, User())
  OnSelect: =Set(AppMode, "Production")
  OnSelect: =Set(LastAction, Now())
  
  # Conditional variable setting
  OnSelect: =Set(
    ThemeColor,
    If(Toggle1.Value, "#0078D4", "#323130")
  )
  
  # Multiple variable updates
  OnSelect: =
    Set(IsLoading, true);
    Set(ErrorMessage, "");
    Set(LastRefresh, Now())
```

**Context Variables** (Screen-Scoped):
```yaml
Properties:
  # Update single context variable
  OnSelect: =UpdateContext({SelectedTab: "Details"})
  
  # Update multiple context variables
  OnSelect: =UpdateContext({
    ShowModal: true,
    ModalType: "Confirmation",
    ModalMessage: "Are you sure you want to delete this item?"
  })
  
  # Clear context variables
  OnSelect: =UpdateContext({
    SelectedItem: Blank(),
    EditMode: false,
    ValidationErrors: Blank()
  })
  
  # Complex context updates
  OnSelect: =UpdateContext({
    FormData: Patch(
      FormData,
      {
        LastModified: Now(),
        ModifiedBy: User().Email
      }
    )
  })
```

#### **Collection Management**:

**Collection Creation and Population**:
```yaml
Properties:
  # Create new collection
  OnSelect: =ClearCollect(MyCollection, DataSource)
  
  # Add single item to collection
  OnSelect: =Collect(ShoppingCart, {
    Product: Gallery1.Selected,
    Quantity: Value(QuantityInput.Text),
    Price: Gallery1.Selected.Price
  })
  
  # Add multiple items
  OnSelect: =Collect(
    SelectedItems,
    Filter(Gallery1.AllItems, IsSelected)
  )
  
  # Clear collection
  OnSelect: =Clear(TempData)
  
  # Replace collection contents
  OnSelect: =ClearCollect(
    FilteredData,
    Filter(DataSource, Category = SelectedCategory)
  )
```

**Collection Modification**:
```yaml
Properties:
  # Update items in collection
  OnSelect: =UpdateIf(
    ShoppingCart,
    Product.ID = ThisItem.ID,
    {Quantity: Value(NewQuantityInput.Text)}
  )
  
  # Remove items from collection
  OnSelect: =RemoveIf(ShoppingCart, Product.ID = ThisItem.ID)
  
  # Remove single item
  OnSelect: =Remove(ShoppingCart, ThisItem)
  
  # Conditional collection updates
  OnSelect: =If(
    CountRows(Filter(ShoppingCart, Product.ID = NewItem.ID)) = 0,
    Collect(ShoppingCart, NewItem),
    UpdateIf(
      ShoppingCart,
      Product.ID = NewItem.ID,
      {Quantity: Quantity + 1}
    )
  )
```

#### **Data Source Operations**:

**Data Modification**:
```yaml
Properties:
  # Create new record
  OnSelect: =Patch(
    DataSource,
    Defaults(DataSource),
    {
      Name: NameInput.Text,
      Email: EmailInput.Text,
      CreatedDate: Now(),
      CreatedBy: User().Email
    }
  )
  
  # Update existing record
  OnSelect: =Patch(
    DataSource,
    Gallery1.Selected,
    {
      Status: "Completed",
      CompletedDate: Now(),
      CompletedBy: User().Email
    }
  )
  
  # Bulk update with condition
  OnSelect: =UpdateIf(
    DataSource,
    Status = "Pending" And CreatedDate < DateAdd(Today(), -30, Days),
    {Status: "Expired"}
  )
  
  # Delete record
  OnSelect: =Remove(DataSource, Gallery1.Selected)
  
  # Conditional delete with confirmation
  OnSelect: =If(
    ConfirmDelete,
    Remove(DataSource, ThisItem);
    UpdateContext({ConfirmDelete: false}),
    UpdateContext({ConfirmDelete: true})
  )
```

**Data Refresh and Synchronization**:
```yaml
Properties:
  # Refresh data source
  OnSelect: =Refresh(DataSource)
  
  # Refresh with notification
  OnSelect: =
    Set(IsRefreshing, true);
    Refresh(DataSource);
    Set(IsRefreshing, false);
    Notify("Data refreshed", NotificationType.Success)
  
  # Conditional refresh
  OnSelect: =If(
    DateDiff(LastRefresh, Now(), Minutes) > 5,
    Refresh(DataSource);
    Set(LastRefresh, Now())
  )
```

### Sequential Action Execution

#### **Action Chaining with Semicolons**:
```yaml
Properties:
  # Basic action sequence
  OnSelect: =
    UpdateContext({IsProcessing: true});
    Set(ProcessStartTime, Now());
    Navigate(ProcessingScreen)
  
  # Complex workflow sequence
  OnSelect: =
    // Validation
    UpdateContext({ValidationErrors: Blank()});
    If(IsBlank(NameInput.Text), 
      UpdateContext({ValidationErrors: "Name is required"}));
    
    // Process if valid
    If(IsBlank(ValidationErrors),
      Set(IsLoading, true);
      Patch(DataSource, FormData);
      ClearCollect(RefreshedData, DataSource);
      Set(IsLoading, false);
      Navigate(SuccessScreen, ScreenTransition.SlideLeft),
      
      // Handle validation errors
      UpdateContext({ShowValidation: true})
    )
  
  # Error handling in sequences
  OnSelect: =IfError(
    Set(IsProcessing, true);
    Patch(DataSource, FormData);
    Navigate(SuccessScreen),
    
    // Error handling
    Set(IsProcessing, false);
    UpdateContext({
      ErrorMessage: FirstError.Message,
      ShowError: true
    })
  )
```

#### **Execution Order and Dependencies**:
```yaml
Properties:
  # Actions execute in order - each waits for previous to complete
  OnSelect: =
    Set(Step1Complete, false);
    ClearCollect(Data1, Source1);      // Step 1: Load first dataset
    Set(Step1Complete, true);
    
    Set(Step2Complete, false);
    ClearCollect(Data2, Source2);      // Step 2: Load second dataset  
    Set(Step2Complete, true);
    
    // Step 3: Process combined data (depends on Step 1 & 2)
    ClearCollect(ProcessedData, 
      AddColumns(Data1, "Related", 
        LookUp(Data2, ID = Data1.RelatedID)
      )
    );
    
    Set(ProcessingComplete, true);
    Navigate(ResultsScreen)
```

### Error Handling in Imperative Logic

#### **IfError Function for Action Sequences**:
```yaml
Properties:
  # Basic error handling
  OnSelect: =IfError(
    Patch(DataSource, FormData),
    Notify("Error saving data: " & FirstError.Message, NotificationType.Error)
  )
  
  # Complex error handling with recovery
  OnSelect: =IfError(
    // Try primary operation
    ClearCollect(PrimaryData, PrimarySource);
    Set(DataLoadSuccess, true),
    
    // Fallback on error
    Set(DataLoadSuccess, false);
    ClearCollect(PrimaryData, CachedData);
    Notify("Using cached data due to connection error", NotificationType.Warning)
  )
  
  # Multiple error scenarios
  OnSelect: =IfError(
    Set(IsProcessing, true);
    Patch(DataSource, FormData);
    Navigate(SuccessScreen),
    
    // Handle different error types
    Switch(
      FirstError.Kind,
      ErrorKind.Network, Notify("Network error. Please check connection."),
      ErrorKind.Permission, Notify("You don't have permission for this action."),
      ErrorKind.Validation, Notify("Please check your input: " & FirstError.Message),
      Notify("An unexpected error occurred: " & FirstError.Message)
    );
    Set(IsProcessing, false)
  )
```

#### **IsError Function for Conditional Logic**:
```yaml
Properties:
  # Check for errors before proceeding
  OnSelect: =
    Set(ValidationResult, Value(TextInput1.Text));
    If(
      IsError(ValidationResult),
      UpdateContext({
        InputError: "Please enter a valid number",
        ShowError: true
      }),
      UpdateContext({
        InputError: "",
        ShowError: false,
        ProcessedValue: ValidationResult
      })
    )
  
  # Error checking in data operations
  OnSelect: =
    Set(LookupResult, LookUp(DataSource, ID = SelectedID));
    If(
      IsError(LookupResult),
      Notify("Record not found", NotificationType.Error),
      Navigate(DetailScreen, ScreenTransition.Cover, {Record: LookupResult})
    )
```

### Advanced Imperative Patterns

#### **State Machine Implementation**:
```yaml
Properties:
  # State management with actions
  OnSelect: =Switch(
    CurrentState,
    "Idle", 
      Set(CurrentState, "Loading");
      ClearCollect(Data, DataSource);
      Set(CurrentState, "Loaded"),
    
    "Loaded",
      Set(CurrentState, "Processing");
      UpdateIf(Data, Selected, {Status: "Processed"});
      Set(CurrentState, "Complete"),
    
    "Complete",
      Set(CurrentState, "Idle");
      Clear(Data),
    
    // Default case
    Notify("Invalid state: " & CurrentState, NotificationType.Error)
  )
  
  # State transitions with validation
  OnSelect: =
    Set(NewState, 
      Switch(
        CurrentState & "-" & ActionType,
        "Idle-Start", "Processing",
        "Processing-Complete", "Finished", 
        "Processing-Cancel", "Cancelled",
        "Finished-Reset", "Idle",
        CurrentState  // No change for invalid transitions
      )
    );
    
    If(
      NewState <> CurrentState,
      Set(CurrentState, NewState);
      UpdateContext({StateChanged: true}),
      Notify("Invalid action for current state", NotificationType.Warning)
    )
```

#### **Batch Operations**:
```yaml
Properties:
  # Process multiple items
  OnSelect: =
    Set(ProcessedCount, 0);
    Set(ErrorCount, 0);
    
    ForAll(
      Filter(Gallery1.AllItems, IsSelected),
      IfError(
        Patch(DataSource, ThisRecord, {Status: "Processed"});
        Set(ProcessedCount, ProcessedCount + 1),
        Set(ErrorCount, ErrorCount + 1)
      )
    );
    
    Notify(
      ProcessedCount & " items processed, " & ErrorCount & " errors",
      If(ErrorCount = 0, NotificationType.Success, NotificationType.Warning)
    )
  
  # Batch data synchronization
  OnSelect: =
    Set(SyncStartTime, Now());
    Set(SyncStatus, "Starting");
    
    // Upload local changes
    Set(SyncStatus, "Uploading");
    ForAll(
      Filter(LocalData, NeedsSync),
      Patch(RemoteDataSource, ThisRecord)
    );
    
    // Download remote changes
    Set(SyncStatus, "Downloading");
    ClearCollect(LocalData, RemoteDataSource);
    
    // Complete
    Set(SyncStatus, "Complete");
    Set(LastSyncTime, Now());
    Notify("Sync completed in " & 
      DateDiff(SyncStartTime, Now(), Seconds) & " seconds")
```

#### **Asynchronous-Style Operations**:
```yaml
Properties:
  # Simulate async operation with status tracking
  OnSelect: =
    Set(OperationId, GUID());
    Set(OperationStatus, "Started");
    UpdateContext({ShowProgress: true});
    
    // Start operation
    Set(OperationStatus, "Processing");
    ClearCollect(TempData, LargeDataSource);
    
    // Update progress
    Set(OperationStatus, "Transforming");
    ClearCollect(ProcessedData, 
      AddColumns(TempData, "Calculated", SomeCalculation)
    );
    
    // Finalize
    Set(OperationStatus, "Complete");
    UpdateContext({ShowProgress: false});
    Navigate(ResultsScreen, ScreenTransition.Fade)
  
  # Progress tracking pattern
  OnSelect: =
    Set(TotalSteps, 5);
    Set(CurrentStep, 0);
    
    Set(CurrentStep, 1); Set(StepStatus, "Loading data...");
    ClearCollect(Data, DataSource);
    
    Set(CurrentStep, 2); Set(StepStatus, "Validating...");
    UpdateIf(Data, IsBlank(RequiredField), {IsValid: false});
    
    Set(CurrentStep, 3); Set(StepStatus, "Processing...");
    ClearCollect(Results, ProcessData(Data));
    
    Set(CurrentStep, 4); Set(StepStatus, "Saving...");
    ForAll(Results, Patch(OutputSource, ThisRecord));
    
    Set(CurrentStep, 5); Set(StepStatus, "Complete");
    Navigate(SuccessScreen)
```

### Form and Control Imperative Logic

#### **Form Submission Workflows**:
```yaml
Properties:
  # Complete form submission process
  OnSelect: =
    // Clear previous state
    UpdateContext({
      ValidationErrors: Blank(),
      SubmissionInProgress: true
    });
    
    // Validate required fields
    If(IsBlank(NameInput.Text),
      UpdateContext({ValidationErrors: "Name is required"}));
    If(IsBlank(EmailInput.Text),
      UpdateContext({ValidationErrors: ValidationErrors & "Email is required"}));
    
    // Process if valid
    If(IsBlank(ValidationErrors),
      // Submit data
      Set(SubmissionResult, 
        Patch(DataSource, Defaults(DataSource), {
          Name: NameInput.Text,
          Email: EmailInput.Text,
          CreatedDate: Now()
        })
      );
      
      // Handle result
      If(IsError(SubmissionResult),
        UpdateContext({
          SubmissionInProgress: false,
          SubmissionError: FirstError.Message
        }),
        
        // Success
        UpdateContext({SubmissionInProgress: false});
        ResetForm(Self);
        Notify("Record saved successfully", NotificationType.Success);
        Navigate(ListScreen, ScreenTransition.SlideRight)
      ),
      
      // Validation failed
      UpdateContext({SubmissionInProgress: false})
    )
```

#### **Dynamic Control State Management**:
```yaml
Properties:
  # Toggle control visibility based on selections
  OnChange: =
    UpdateContext({
      ShowAdvanced: Self.Value,
      AdvancedFields: If(Self.Value, 
        ["Field1", "Field2", "Field3"], 
        []
      )
    });
    
    // Reset advanced fields when hidden
    If(Not(Self.Value),
      UpdateContext({
        AdvancedField1: "",
        AdvancedField2: "",
        AdvancedField3: ""
      })
    )
  
  # Cascade dropdown updates
  OnChange: =
    // Update dependent dropdown
    ClearCollect(
      SubCategories,
      Filter(AllSubCategories, ParentCategory = Self.Selected.Value)
    );
    
    // Reset dependent selections
    UpdateContext({
      SelectedSubCategory: Blank(),
      SelectedItem: Blank()
    });
    
    // Update form state
    UpdateContext({FormData: Patch(FormData, {
      Category: Self.Selected.Value,
      SubCategory: Blank(),
      Item: Blank()
    })})
```

### Performance Optimization in Imperative Logic

#### **Efficient Action Sequences**:
```yaml
Properties:
  # Batch operations instead of loops
  OnSelect: =
    // Instead of ForAll with individual Patch operations
    // Use bulk operations when possible
    
    Set(UpdatedRecords, 
      AddColumns(
        Filter(Gallery1.AllItems, IsSelected),
        "UpdatedStatus", "Processed",
        "UpdatedDate", Now()
      )
    );
    
    // Single bulk update
    ForAll(UpdatedRecords, 
      Patch(DataSource, {ID: ID}, {
        Status: UpdatedStatus,
        ProcessedDate: UpdatedDate
      })
    )
  
  # Minimize context variable updates
  OnSelect: =
    // Build complete context update object
    Set(NewContext, {
      Step: "Processing",
      Progress: 50,
      Message: "Processing data...",
      StartTime: Now(),
      IsVisible: true
    });
    
    // Single context update
    UpdateContext(NewContext)
```

#### **Memory Management**:
```yaml
Properties:
  # Clear temporary data after processing
  OnSelect: =
    // Process large dataset
    ClearCollect(TempResults, ProcessLargeData(DataSource));
    
    // Extract needed information
    Set(SummaryData, {
      Count: CountRows(TempResults),
      Total: Sum(TempResults, Amount),
      Average: Average(TempResults, Amount)
    });
    
    // Clear large temporary collection
    Clear(TempResults);
    
    // Navigate with summary
    Navigate(SummaryScreen, ScreenTransition.Fade, SummaryData)
  
  # Lazy loading patterns
  OnVisible: =
    If(IsBlank(ScreenData),
      Set(IsLoading, true);
      ClearCollect(ScreenData, 
        Filter(DataSource, Category = CurrentCategory)
      );
      Set(IsLoading, false)
    )
```

### Integration with Declarative Formulas

#### **Hybrid Declarative-Imperative Patterns**:
```yaml
Properties:
  # Use imperative logic to set up declarative formulas
  OnStart: =
    Set(FilterCriteria, {
      Status: "Active",
      MinDate: DateAdd(Today(), -30, Days),
      MaxDate: Today()
    })
  
  # Declarative formula uses imperative-set variables
  Items: =Filter(DataSource,
    Status = FilterCriteria.Status And
    CreatedDate >= FilterCriteria.MinDate And
    CreatedDate <= FilterCriteria.MaxDate
  )
  
  # Imperative actions update declarative dependencies
  OnSelect: =
    Set(FilterCriteria, Patch(FilterCriteria, {
      Status: StatusDropdown.Selected.Value,
      MinDate: DatePicker1.SelectedDate,
      MaxDate: DatePicker2.SelectedDate
    }))
  
  # Declarative formulas respond automatically to imperative changes
  Visible: =CountRows(
    Filter(DataSource,
      Status = FilterCriteria.Status And
      CreatedDate >= FilterCriteria.MinDate
    )
  ) > 0
```

### Imperative Logic Best Practices

#### **State Management Guidelines**:
```yaml
Properties:
  # Use descriptive variable names
  OnSelect: =
    Set(UserProfileLoadingInProgress, true);
    Set(UserProfileLastRefreshTime, Now());
    ClearCollect(UserProfileData, UserProfileService)
  
  # Group related state changes
  OnSelect: =UpdateContext({
    // Modal state
    ShowConfirmationModal: true,
    ConfirmationMessage: "Delete this record?",
    ConfirmationAction: "Delete",
    
    // Form state  
    FormReadOnly: true,
    FormValidationEnabled: false
  })
  
  # Initialize state consistently
  OnVisible: =
    If(IsBlank(ScreenInitialized),
      UpdateContext({
        SelectedTab: "Overview",
        ShowFilters: false,
        FilterApplied: false,
        SortOrder: "Name",
        SortDirection: "Ascending"
      });
      Set(ScreenInitialized, true)
    )
```

#### **Error Handling Best Practices**:
```yaml
Properties:
  # Comprehensive error handling
  OnSelect: =IfError(
    // Primary operation
    Set(OperationInProgress, true);
    Set(OperationResult, ProcessData(InputData));
    Set(OperationInProgress, false);
    Navigate(SuccessScreen, ScreenTransition.Fade, {
      Result: OperationResult
    }),
    
    // Error handling with context
    Set(OperationInProgress, false);
    UpdateContext({
      ErrorOccurred: true,
      ErrorMessage: FirstError.Message,
      ErrorKind: FirstError.Kind,
      ErrorTimestamp: Now(),
      ErrorContext: "ProcessData operation"
    });
    
    // Log error for debugging
    Collect(ErrorLog, {
      Timestamp: Now(),
      User: User().Email,
      Screen: App.ActiveScreen.Name,
      Error: FirstError.Message,
      Context: "User clicked process button"
    })
  )
```

#### **Performance Guidelines**:
```yaml
Properties:
  # Minimize action sequences
  OnSelect: =
    // Efficient: Single action with complex logic
    Set(ProcessingResult, 
      If(ValidateInput(UserInput),
        ProcessValidInput(UserInput),
        HandleInvalidInput(UserInput)
      )
    )
  
  # Rather than: Multiple conditional actions
  # OnSelect: =
  #   If(ValidateInput(UserInput),
  #     Set(Result, ProcessValidInput(UserInput)),
  #     Set(Result, HandleInvalidInput(UserInput))
  #   )
  
  # Use delegation-friendly patterns
  OnSelect: =
    // Server-side filtering before collection
    ClearCollect(FilteredData,
      Filter(LargeDataSource, Status = "Active")
    );
    
    // Local operations on filtered subset
    ClearCollect(ProcessedData,
      AddColumns(FilteredData, "DisplayText", 
        Name & " (" & Format & ")"
      )
    )
```

## Power Fx Operators and Identifiers Reference

Power Fx provides a comprehensive set of operators for building formulas that calculate values, compare data, manipulate text, and control application logic. This exhaustive reference covers all operator types, precedence rules, and identifier naming conventions.

### Operator Overview

Power Fx operators enable:
- **Arithmetic Operations**: Mathematical calculations with numbers
- **Comparison Operations**: Evaluate relationships between values
- **Logical Operations**: Boolean logic and conditional expressions
- **Text Operations**: String manipulation and concatenation
- **Reference Operations**: Access properties and fields
- **Membership Operations**: Test collection and string membership
- **Special Operations**: Disambiguation, scope control, and type conversion

#### **Operator Categories**:
- **Property Selectors**: Access control and data properties
- **Arithmetic Operators**: Mathematical operations
- **Comparison Operators**: Value relationship testing
- **Logical Operators**: Boolean operations
- **Text Operators**: String manipulation
- **Membership Operators**: Collection and substring testing
- **Scope Operators**: Record and control context
- **Disambiguation Operators**: Resolve naming conflicts

### Complete Operator Reference Table

#### **Property Access and Navigation**:
```yaml
Properties:
  # Property selector (dot notation)
  Text: =Slider1.Value          # Access control property
  Fill: =Color.Red              # Access enumeration value
  Items: =Gallery1.Selected     # Access selected item
  
  # Complex property chains
  Text: =Gallery1.Selected.Customer.Name
  Fill: =App.ActiveScreen.Fill
  Visible: =Parent.Visible
  
  # Collection field access
  Text: =MyCollection.Name
  Items: =DataSource.Category
```

**Operator**: `.` (Property Selector)
**Type**: Property access
**Syntax**: `Control.Property`, `Data.Field`, `Enumeration.Value`
**Description**: Extracts properties from controls, tables, or enumerations
**Note**: `!` can be used for backward compatibility

#### **Arithmetic Operators**:
```yaml
Properties:
  # Addition
  Text: =Value1 + Value2
  Text: =10 + 5                 # Result: 15
  Text: =Slider1.Value + 100
  
  # Subtraction and negation
  Text: =Value1 - Value2
  Text: =10 - 3                 # Result: 7
  Text: =-Slider1.Value         # Negative value
  
  # Multiplication
  Text: =Value1 * Value2
  Text: =Price * Quantity
  Text: =Slider1.Value * 0.8
  
  # Division
  Text: =Value1 / Value2
  Text: =Total / Count
  Text: =100 / 3                # Result: 33.333...
  
  # Exponentiation
  Text: =Base ^ Exponent
  Text: =2 ^ 3                  # Result: 8
  Text: =Slider1.Value ^ 2      # Square the value
  
  # Percentage
  Text: =Slider1.Value * 25%    # 25% of slider value
  Text: =20%                    # Equivalent to 0.2
  Text: =100% + 50%             # Result: 1.5
```

**Operators**: `+`, `-`, `*`, `/`, `^`, `%`
**Type**: Arithmetic operations
**Precedence**: `%` (highest), `^`, `*` `/`, `+` `-` (lowest)
**Description**: Perform mathematical calculations on numeric values

#### **Comparison Operators**:
```yaml
Properties:
  # Equality comparison
  Visible: =Price = 100
  Visible: =TextInput1.Text = "Admin"
  Visible: =Slider1.Value = Slider2.Value
  
  # Inequality comparison
  Visible: =Price <> 100        # Not equal to
  Visible: =Status <> "Complete"
  Visible: =User().Email <> CurrentUser
  
  # Greater than
  Visible: =Price > 100
  Visible: =DateValue > Today()
  Visible: =CountRows(Collection1) > 10
  
  # Greater than or equal
  Visible: =Price >= 100
  Visible: =Score >= PassingGrade
  Visible: =Slider1.Value >= Slider1.Min
  
  # Less than
  Visible: =Price < 100
  Visible: =ExpiryDate < Today()
  Visible: =RemainingCount < 5
  
  # Less than or equal
  Visible: =Price <= 100
  Visible: =Age <= MaxAge
  Visible: =Slider1.Value <= Slider1.Max
```

**Operators**: `=`, `<>`, `>`, `>=`, `<`, `<=`
**Type**: Comparison operations
**Returns**: Boolean (true/false)
**Description**: Compare values and return logical results

#### **Logical Operators**:
```yaml
Properties:
  # Logical AND (symbol form)
  Visible: =Price < 100 && Quantity > 0
  Visible: =IsActive && IsValid && IsApproved
  
  # Logical AND (keyword form - requires whitespace)
  Visible: =Price < 100 And Quantity > 0
  Visible: =Not(IsBlank(Name)) And Not(IsBlank(Email))
  
  # Logical OR (symbol form)
  Visible: =Status = "Draft" || Status = "Review"
  Visible: =IsAdmin || IsManager || IsOwner
  
  # Logical OR (keyword form - requires whitespace)
  Visible: =Status = "Draft" Or Status = "Review"
  Visible: =IsEmpty Or IsError Or IsBlank(Value)
  
  # Logical NOT (symbol form)
  Visible: =!IsHidden
  Visible: =!(Price > 1000)
  
  # Logical NOT (keyword form - requires whitespace)
  Visible: =Not IsHidden
  Visible: =Not (Price > 1000 And Quantity < 5)
```

**Operators**: `&&`/`And`, `||`/`Or`, `!`/`Not`
**Type**: Logical operations
**Returns**: Boolean (true/false)
**Description**: Combine and negate logical expressions
**Note**: Keyword forms require whitespace separation

#### **Text Concatenation Operator**:
```yaml
Properties:
  # Basic string concatenation
  Text: ="Hello" & " " & "World"        # Result: "Hello World"
  Text: =FirstName & " " & LastName
  Text: ="Value: " & Text(Slider1.Value)
  
  # Complex concatenation
  Text: ="User " & User().FullName & " logged in at " & Text(Now())
  Text: =Title & " - " & Subtitle & " (" & Status & ")"
  
  # Mixed data types (automatic conversion)
  Text: ="Count: " & CountRows(Collection1)
  Text: ="Price: $" & Text(Price, "##.00")
  Text: ="Date: " & Text(Today(), "mm/dd/yyyy")
  
  # Multi-line concatenation
  Text: =
    "Line 1" &
    Char(10) &           # New line character
    "Line 2" &
    Char(10) &
    "Line 3"
```

**Operator**: `&`
**Type**: String concatenation
**Description**: Combines multiple strings into a single continuous string
**Auto-conversion**: Non-text values automatically converted to text

#### **Membership Operators**:

**Collection/Table Membership**:
```yaml
Properties:
  # Case-insensitive membership (in)
  Visible: =Gallery1.Selected in SavedItems
  Visible: =CurrentUser in ApprovedUsers
  Items: =Filter(Products, Category in SelectedCategories)
  
  # Case-sensitive membership (exactin)
  Visible: =Gallery1.Selected exactin SavedItems
  Visible: =SelectedItem exactin PreciseCollection
  Items: =Filter(Data, Status exactin ValidStatuses)
  
  # Record membership in collections
  OnSelect: =If(
    ThisItem in FavoriteItems,
    Remove(FavoriteItems, ThisItem),
    Collect(FavoriteItems, ThisItem)
  )
```

**String Substring Testing**:
```yaml
Properties:
  # Case-insensitive substring search (in)
  Visible: ="windows" in "To display windows in the Windows operating system"
  Items: =Filter(Products, SearchText in ProductName)
  Visible: ="error" in ErrorMessage
  
  # Case-sensitive substring search (exactin)
  Visible: ="Windows" exactin "To display Windows in the Windows operating system"
  Items: =Filter(Products, "iPhone" exactin ProductName)
  Visible: ="ERROR" exactin LogMessage
  
  # Search validation
  Items: =Filter(
    Customers,
    SearchInput.Text in CustomerName ||
    SearchInput.Text in Email ||
    SearchInput.Text in Phone
  )
```

**Operators**: `in`, `exactin`
**Type**: Membership testing
**Description**: Test collection membership or substring presence
**Case sensitivity**: `in` is case-insensitive, `exactin` is case-sensitive

#### **Scope and Context Operators**:

**ThisItem Operator**:
```yaml
# Gallery control template
Gallery1:
  Items: =Employees

# Within gallery template
Properties:
  # Access current item fields
  Text: =ThisItem.'First Name' & " " & ThisItem.'Last Name'
  Fill: =If(ThisItem.Status = "Active", Color.Green, Color.Gray)
  Visible: =ThisItem.Department = "Sales"
  
  # Use in expressions
  OnSelect: =Navigate(
    DetailScreen,
    ScreenTransition.Cover,
    {SelectedEmployee: ThisItem}
  )
  
  # Complex ThisItem usage
  Text: =If(
    ThisItem.Salary > 50000,
    "Senior: " & ThisItem.Title,
    "Junior: " & ThisItem.Title
  )
```

**ThisRecord Operator**:
```yaml
Properties:
  # ForAll function with ThisRecord
  OnSelect: =ForAll(
    Employees,
    Patch(
      EmployeeData,
      ThisRecord,
      {LastUpdated: Now()}
    )
  )
  
  # Filter function with explicit ThisRecord
  Items: =Filter(
    Employees,
    StartsWith(ThisRecord.'First Name', "M")
  )
  
  # Implied ThisRecord (optional)
  Items: =Filter(
    Employees,
    StartsWith('First Name', "M")  # ThisRecord implied
  )
  
  # UpdateIf with ThisRecord
  OnSelect: =UpdateIf(
    Collection1,
    ThisRecord.Status = "Pending",
    {Status: "Approved", ApprovedBy: User().Email}
  )
```

**As Operator**:
```yaml
# Gallery with As operator
Gallery1:
  Items: =Employees As Employee

# Within gallery template
Properties:
  # Use custom name instead of ThisItem
  Text: =Employee.'First Name' & " " & Employee.'Last Name'
  Fill: =If(Employee.Status = "Active", Color.Green, Color.Gray)
  
  # Function scope with As operator
  Items: =ForAll(
    InactiveEmployees As Employee,
    Patch(
      Employees,
      Employee,
      {Status: 'Status (Employees)'.Active}
    )
  )
  
  # Nested scopes with As
  Text: =Concat(
    ForAll(Sequence(8) As Rank,
      Concat(
        ForAll(Sequence(8) As File,
          If(Mod(Rank.Value + File.Value, 2) = 1, " X ", " . ")
        ),
        Value
      ) & Char(10)
    ),
    Value
  )
```

**Self and Parent Operators**:
```yaml
Properties:
  # Self operator - reference current control
  Fill: =If(Self.Visible, Color.Blue, Color.Gray)
  Text: =Upper(Self.Text)
  OnSelect: =Set(LastClicked, Self.Name)
  
  # Parent operator - reference container control
  Width: =Parent.Width * 0.8
  Fill: =Parent.Fill
  Visible: =Parent.Visible And UserHasAccess
  
  # Nested container references
  X: =Parent.X + 10
  Y: =Parent.Y + Parent.Height + 5
  
  # Gallery template using Parent
  Fill: =If(
    ThisItem.IsSelected,
    Parent.TemplateFill,
    Color.Transparent
  )
```

**Operators**: `ThisItem`, `ThisRecord`, `As`, `Self`, `Parent`
**Type**: Scope and context reference
**Description**: Reference current record, control, or container context

#### **Disambiguation Operators**:

**Global Disambiguation**:
```yaml
Properties:
  # Access global variable when local field exists
  Text: =[@GlobalVariable]
  Text: =[@MyCollection]
  Text: =[@AppSetting]
  
  # In record scope with conflicting names
  Items: =AddColumns(
    Employees,
    "ManagerName",
    LookUp([@Employees], ID = Manager)  # Global collection reference
  )
  
  # Context variable disambiguation
  OnSelect: =UpdateContext({
    LocalValue: [@GlobalValue] + 10  # Use global, not local
  })
```

**Field Disambiguation**:
```yaml
Properties:
  # Table field disambiguation
  Items: =AddColumns(
    Orders,
    "CustomerInfo",
    Customers[@CustomerName]  # Specify table for field
  )
  
  # Nested scope disambiguation
  Items: =ForAll(
    OuterTable,
    AddColumns(
      InnerTable,
      "OuterValue",
      OuterTable[@FieldName]  # Access outer table field
    )
  )
```

**Operator**: `@`
**Type**: Disambiguation
**Syntax**: `[@GlobalName]`, `Table[@FieldName]`
**Description**: Resolve naming conflicts between scopes

#### **Language-Dependent Separators**:

**List Separators** (Function Arguments):
```yaml
# English/UK/Japan (comma as list separator)
Properties:
  Text: =Concatenate("Hello", " ", "World")
  Items: =Sort(MyCollection, Name, Ascending)
  Fill: =If(Condition, Color.Red, Color.Blue)

# France/Spain/Germany (semicolon as list separator)
Properties:
  Text: =Concatenate("Hello"; " "; "World")
  Items: =Sort(MyCollection; Name; Ascending)
  Fill: =If(Condition; Color.Red; Color.Blue)
```

**Decimal Separators**:
```yaml
# English/UK/Japan (dot as decimal separator)
Properties:
  Text: =12.59
  Text: =Value1 + 100.25

# France/Spain/Germany (comma as decimal separator)
Properties:
  Text: =12,59
  Text: =Value1 + 100,25
```

**Formula Chaining Separators**:
```yaml
# English/UK/Japan (semicolon for chaining)
Properties:
  OnSelect: =
    Set(IsLoading, true);
    Collect(MyData, DataSource);
    Set(IsLoading, false)

# France/Spain/Germany (double semicolon for chaining)
Properties:
  OnSelect: =
    Set(IsLoading, true);;
    Collect(MyData, DataSource);;
    Set(IsLoading, false)
```

**Separators**: `,` (list), `.`/`,` (decimal), `;`/`;;` (chaining)
**Type**: Language-dependent separators
**Description**: Separators adjust based on author's language settings

### Operator Precedence and Associativity

#### **Precedence Levels** (Highest to Lowest):

**1. Primary Expressions**:
```yaml
Properties:
  # Literals have highest precedence
  Text: =123
  Text: ="Hello"
  Text: =true
  
  # Identifiers
  Text: =MyVariable
  Text: =Control1
  
  # Parenthesized expressions
  Text: =(Value1 + Value2) * Value3
  
  # Function calls
  Text: =Upper("hello")
  Text: =Now()
```

**2. Postfix Operators**:
```yaml
Properties:
  # Percentage operator
  Text: =50%              # Equivalent to 0.5
  Text: =Slider1.Value * 25%
```

**3. Member Access**:
```yaml
Properties:
  # Property access (left-to-right)
  Text: =Gallery1.Selected.Customer.Name
  Fill: =App.ActiveScreen.Fill
  
  # Bang notation
  Text: =Table1!'Field Name'
```

**4. Prefix Operators**:
```yaml
Properties:
  # Unary minus and logical NOT
  Text: =-Value1
  Visible: =!IsHidden
  Visible: =Not IsBlank(TextInput1.Text)
```

**5. Exponentiation**:
```yaml
Properties:
  # Right-associative
  Text: =2 ^ 3 ^ 2        # Equivalent to 2 ^ (3 ^ 2) = 2 ^ 9 = 512
  Text: =Base ^ Exponent1 ^ Exponent2
```

**6. Multiplicative**:
```yaml
Properties:
  # Left-associative
  Text: =10 * 5 / 2       # Equivalent to (10 * 5) / 2 = 25
  Text: =Price * Quantity / ExchangeRate
```

**7. Additive**:
```yaml
Properties:
  # Left-associative
  Text: =10 + 5 - 2       # Equivalent to (10 + 5) - 2 = 13
  Text: =Base + Adjustment - Discount
```

**8. String Concatenation**:
```yaml
Properties:
  # Left-associative
  Text: ="A" & "B" & "C"  # Equivalent to ("A" & "B") & "C" = "ABC"
  Text: =FirstName & " " & MiddleName & " " & LastName
```

**9. Relational**:
```yaml
Properties:
  # Non-associative
  Visible: =Value1 < Value2 < Value3  # Error: not allowed
  Visible: =Value1 < Value2 And Value2 < Value3  # Correct
```

**10. Equality**:
```yaml
Properties:
  # Left-associative
  Visible: =A = B = C     # Equivalent to (A = B) = C (usually not desired)
  Visible: =A = B And B = C  # Usually intended meaning
```

**11. Membership**:
```yaml
Properties:
  # Non-associative
  Visible: ="text" in Field1 in Collection1  # Error: not allowed
  Visible: ="text" in Field1 And Field1 in Collection1  # Correct
```

**12. Logical AND**:
```yaml
Properties:
  # Left-associative
  Visible: =A && B && C   # Equivalent to (A && B) && C
  Visible: =Condition1 And Condition2 And Condition3
```

**13. Logical OR**:
```yaml
Properties:
  # Left-associative
  Visible: =A || B || C   # Equivalent to (A || B) || C
  Visible: =Condition1 Or Condition2 Or Condition3
```

#### **Precedence Examples**:
```yaml
Properties:
  # Arithmetic precedence
  Text: =2 + 3 * 4        # Result: 14 (not 20)
  Text: =(2 + 3) * 4      # Result: 20 (explicit grouping)
  
  # Comparison and logical precedence
  Visible: =X > 0 And Y < 10      # Equivalent to (X > 0) And (Y < 10)
  Visible: =Not X = 0             # Equivalent to Not (X = 0)
  
  # String concatenation precedence
  Text: ="Value: " & X + Y        # Equivalent to "Value: " & (X + Y)
  Text: =("Prefix" & X) + Y       # Explicit grouping for different meaning
  
  # Mixed operator precedence
  Visible: =A + B * C = D And E   # Equivalent to ((A + (B * C)) = D) And E
```

### Identifier Naming Conventions

#### **Valid Identifier Formats**:

**Simple Identifiers**:
```yaml
Properties:
  # Standard naming (no quotes needed)
  Text: =CustomerName
  Text: =ProductID  
  Text: =IsActive
  Text: =OrderDate
  
  # With numbers
  Text: =Customer123
  Text: =Field1Value
  Text: =Button2Text
  
  # With underscores
  Text: =First_Name
  Text: =Customer_ID
  Text: =Is_Valid_Email
```

**Single-Quoted Identifiers**:
```yaml
Properties:
  # Names with spaces
  Text: ='Customer Name'
  Text: ='Product Description'
  Text: ='Order Date'
  
  # Names with special characters
  Text: ='Email@Address'
  Text: ='Field-Name'
  Text: ='Column#1'
  
  # Names with quotes (escaped)
  Text: ='Name with "quotes"'
  Text: ='Field with ''apostrophe'''  # Two single quotes = one quote
  
  # Reserved words as identifiers
  Text: ='Text'
  Text: ='Value'
  Text: ='Name'
```

#### **Identifier Character Rules**:

**Start Characters**:
```yaml
# Valid identifier start characters:
- Letters (Unicode category: Letter)
- Underscore (_)

# Examples:
MyVariable      # Letter start
_PrivateField   # Underscore start
用户名           # Unicode letter (Chinese)
Benutzername    # Unicode letter (German)
```

**Continue Characters**:
```yaml
# Valid continuation characters:
- Start characters (letters, underscore)
- Decimal digits (0-9)
- Connecting characters (Unicode Pc category)
- Combining characters (Unicode Mc, Mn categories)
- Formatting characters (Unicode Cf category)

# Examples:
CustomerID123   # Letters and digits
Field_Name_1    # Letters, underscore, and digit
_internalVar2   # Underscore, letters, and digit
```

#### **Display Names vs Logical Names**:

**SharePoint/Dataverse Field Names**:
```yaml
# Display name (user-friendly)
Text: ='Custom Field'      # Shown in UI, may contain spaces
Text: ='Employee Name'     # Easy to understand
Text: ='Project Status'    # Descriptive

# Logical name (system-generated)
Text: =cr5e3_customfield   # Unique, no spaces, doesn't change
Text: =emp_name            # Database field name
Text: =proj_status         # System identifier

# Both reference the same field
Text: ='Custom Field'      # Using display name
Text: =cr5e3_customfield   # Using logical name (equivalent)
```

**Display Name Features**:
```yaml
# Display names are user-friendly
Characteristics:
  - User-friendly and intended for end users
  - May not be unique within entity
  - May change over time
  - Can contain spaces and Unicode characters
  - May be localized into different languages
  - Suggested by Power Fx IntelliSense
  
# Examples by language
English: 'Customer Name'
Spanish: 'Nombre del Cliente'
French: 'Nom du Client'
German: 'Kundenname'
```

**Logical Name Features**:
```yaml
# Logical names are system-level
Characteristics:
  - Guaranteed to be unique
  - Never change after creation
  - No spaces or special characters
  - Not localized (same across languages)
  - Used by professional developers
  - Not suggested by IntelliSense but can be typed
  
# System naming patterns
Examples:
  - cr5e3_customfield      # Custom field with prefix
  - new_projectstatus      # Custom field
  - emailaddress1          # System field
  - createdon              # Standard audit field
```

**Name Mapping and Translation**:
```yaml
# Behind-the-scenes mapping
Process:
  1. Formula uses display name: 'Custom Field'
  2. System maps to logical name: cr5e3_customfield
  3. Network traffic uses logical name
  4. Results mapped back to display name

# Cross-environment considerations
Properties:
  # Display names preferred for portability
  Text: ='Custom Field'    # Maps to local display name in target environment
  
  # Logical names may differ between environments
  Text: =cr5e3_customfield # Prefix 'cr5e3' may be different in target
  
  # System fields consistent across environments
  Text: =createdon         # Standard logical names work everywhere
  Text: =modifiedby        # Consistent system fields
```

**Name Disambiguation**:
```yaml
# When display names conflict, logical names are added
FieldReferences:
  'Custom Field'              # First field with this display name
  'Custom Field (cr5e3_alt)'  # Second field, disambiguated
  'Custom Field (cr5e3_new)'  # Third field, disambiguated

# Usage in formulas
Properties:
  Text: ='Custom Field'                    # First field
  Text: ='Custom Field (cr5e3_alt)'       # Second field
  Text: ='Custom Field (cr5e3_new)'       # Third field

# Disambiguation occurs for
Scenarios:
  - Multiple fields with same display name
  - Entity name conflicts
  - Option set value conflicts
  - Relationship name conflicts
  
# Disambiguation string format
Pattern: "'Display Name (logical_name)'"
Examples:
  - 'Status (statuscode)'
  - 'Status (new_customstatus)'
  - 'Customer (parentcustomerid)'
  - 'Customer (new_assignedcustomer)'
```

**Best Practices for Names**:
```yaml
# Recommended approach
Properties:
  # Use display names for readability and portability
  Text: ='Customer Name'        # Preferred
  Text: ='Order Date'           # Clear and portable
  Text: ='Project Status'       # Easy to understand
  
  # Use logical names only when necessary
  Text: =cr5e3_customfield     # When display name conflicts
  Text: =new_internalcode      # For system integrations
  
# Avoid mixing naming styles
BadPractice:
  Text: ='Customer Name' & " - " & orderdate & " (" & cr5e3_status & ")"
  
GoodPractice:
  Text: ='Customer Name' & " - " & 'Order Date' & " (" & 'Status' & ")"
```

### Advanced Operator Patterns

#### **Type Coercion with Operators**:
```yaml
Properties:
  # Automatic string conversion
  Text: ="Count: " & CountRows(Collection1)     # Number to string
  Text: ="Today: " & Today()                    # Date to string
  Text: ="Status: " & Toggle1.Value             # Boolean to string
  
  # Comparison type matching
  Visible: =TextInput1.Text = "123"             # String comparison
  Visible: =Value(TextInput1.Text) = 123        # Numeric comparison
  Visible: =DateValue(DateInput.Text) = Today() # Date comparison
  
  # Arithmetic type promotion
  Text: =IntegerValue + DecimalValue             # Integer promoted to decimal
  Text: =TextAsNumber + 10                      # Text converted to number
```

#### **Short-Circuit Evaluation**:
```yaml
Properties:
  # Logical AND short-circuits
  Visible: =Not(IsBlank(Record)) And Record.IsValid
  # If Record is blank, Record.IsValid is not evaluated
  
  # Logical OR short-circuits  
  Visible: =IsAdmin Or CheckPermissions(CurrentUser)
  # If IsAdmin is true, CheckPermissions() is not called
  
  # Safe property access
  Text: =If(
    Not(IsBlank(SelectedItem)),
    SelectedItem.Name,
    "No selection"
  )
  
  # Complex short-circuit patterns
  Visible: =IsBlank(ErrorMessage) Or ShowAllMessages Or CurrentUser.IsAdmin
  # Evaluation stops at first true condition
  
  # Avoiding expensive operations
  Items: =If(
    CountRows(LargeCollection) < 1000,  # Quick count check first
    Sort(LargeCollection, ComplexCalculation),  # Only if small enough
    TopN(LargeCollection, 100)          # Fallback for large collections
  )
```

#### **Operator Overloading by Type**:
```yaml
Properties:
  # Addition operator behavior varies by type
  Text: =123 + 456                    # Numeric addition: 579
  Text: =Time(10,30,0) + Time(2,15,0) # Time addition: 12:45:00
  Text: =Today() + 7                  # Date addition: Today + 7 days
  Text: =DateAdd(Today(), 1, Months)  # Date arithmetic with units
  
  # Subtraction behavior by type
  Text: =100 - 25                     # Numeric: 75
  Text: =Today() - DateAdd(Today(), -7, Days)  # Date difference: 7 days
  Text: =Time(15,30,0) - Time(10,15,0)        # Time difference: 5:15:00
  
  # Comparison operator behavior
  Visible: ="apple" < "banana"        # String lexicographic comparison
  Visible: =Date(2024,1,15) < Today() # Date chronological comparison
  Visible: =10.5 < 20                 # Numeric magnitude comparison
  Visible: =Time(9,0,0) < Time(17,0,0) # Time comparison
  
  # Concatenation with different types
  Text: ="Value: " & 123              # Number auto-converted
  Text: ="Date: " & Today()           # Date auto-converted  
  Text: ="Status: " & true            # Boolean auto-converted
```

#### **Complex Expression Patterns**:
```yaml
Properties:
  # Nested operations with mixed types
  Text: =Text(
    Sum(
      Filter(Orders, Status = "Complete"),
      Total
    ),
    "Currency"
  )
  
  # Conditional chains with multiple operators
  Fill: =Switch(
    Status,
    "Active", Color.Green,
    "Warning", Color.Orange,
    "Error", Color.Red,
    Color.Gray
  )
  
  # Complex logical expressions
  Visible: =(
    UserRole = "Admin" Or 
    UserRole = "Manager"
  ) And (
    Not(IsReadOnly) And 
    HasPermission("Edit")
  )
  
  # Multi-level property access with safety
  Text: =If(
    Not(IsBlank(Gallery1.Selected)) And
    Not(IsBlank(Gallery1.Selected.Customer)) And
    Not(IsBlank(Gallery1.Selected.Customer.PrimaryContact)),
    Gallery1.Selected.Customer.PrimaryContact.Email,
    "No contact email"
  )
  
  # Mathematical expressions with precedence
  Text: =((BasePrice * (1 + TaxRate)) * Quantity) - DiscountAmount
  
  # String manipulation chains
  Text: =Proper(
    Trim(
      Substitute(
        Substitute(TextInput1.Text, "  ", " "),
        Chr(9), " "  # Replace tabs with spaces
      )
    )
  )
```

#### **Operator Chaining Patterns**:
```yaml
Properties:
  # Property access chains
  Text: =App.ActiveScreen.Parent.Fill
  Text: =Gallery1.Selected.Customer.Address.Street
  Fill: =Parent.Parent.TemplateFill
  
  # Function call chains
  Text: =Upper(
    Left(
      Trim(CustomerName),
      10
    )
  )
  
  # Arithmetic operation chains
  Text: =BaseAmount * TaxRate + ShippingCost - DiscountAmount
  
  # Logical operation chains
  Visible: =IsActive And IsVisible And HasPermission And Not(IsLocked)
  
  # Comparison chains (with explicit grouping)
  Visible: =(MinValue <= CurrentValue) And (CurrentValue <= MaxValue)
  
  # Mixed operator chains
  Text: ="Total: " & Text(
    (Price * Quantity * (1 + TaxRate)) - Discount,
    "Currency"
  )
```

#### **Error-Safe Operator Patterns**:
```yaml
Properties:
  # Safe division
  Text: =If(
    Denominator <> 0,
    Numerator / Denominator,
    0
  )
  
  # Safe property access with multiple checks
  Text: =If(
    And(
      Not(IsBlank(SelectedRecord)),
      Not(IsError(SelectedRecord)),
      Not(IsBlank(SelectedRecord.RelatedData))
    ),
    SelectedRecord.RelatedData.DisplayValue,
    "N/A"
  )
  
  # Safe string operations
  Text: =If(
    Not(IsBlank(FirstName)) And Not(IsBlank(LastName)),
    FirstName & " " & LastName,
    Coalesce(FirstName, LastName, "Unknown")
  )
  
  # Safe numeric operations
  Text: =If(
    IsNumeric(TextInput1.Text),
    Text(Value(TextInput1.Text) * 1.1, "Currency"),
    "Invalid number"
  )
  
  # Safe date operations
  Text: =If(
    Not(IsBlank(DatePicker1.SelectedDate)) And
    DatePicker1.SelectedDate >= Today(),
    Text(DatePicker1.SelectedDate, "Short Date"),
    "Please select a valid future date"
  )
```

#### **Performance-Optimized Operator Usage**:
```yaml
Properties:
  # Efficient filtering with operator precedence
  Items: =Filter(
    LargeDataSource,
    Status = "Active" And                    # Simple equality first
    Category in SelectedCategories And       # Membership test
    Amount >= MinAmount And Amount <= MaxAmount  # Range filters
  )
  
  # Minimize complex calculations in frequently updated properties
  OnVisible: =Set(
    CalculatedValues,
    AddColumns(
      BaseData,
      "FormattedPrice", Text(Price, "Currency"),
      "IsExpired", ExpiryDate < Today(),
      "DaysRemaining", DateDiff(Today(), ExpiryDate, Days)
    )
  );
  
  Items: =CalculatedValues  # Use pre-calculated values
  
  # Efficient string building
  Text: =Concatenate(
    "Order #", OrderNumber,
    " - ", CustomerName,
    " (", Text(OrderDate, "Short Date"), ")"
  )
  # More efficient than multiple & operations
  
  # Cache expensive lookups
  OnVisible: =Set(
    UserPermissions,
    LookUp(PermissionsTable, UserID = User().Email)
  );
  
  Visible: =UserPermissions.CanEdit  # Use cached lookup
```

### Operator Best Practices

#### **Readability Guidelines**:
```yaml
Properties:
  # Use parentheses for clarity
  Text: =(Price * Quantity) + Tax          # Clear order
  Text: =(StartDate <= Today()) And (EndDate >= Today())  # Explicit grouping
  
  # Break complex expressions across lines
  OnVisible: =
    Set(IsValidUser, 
      Not(IsBlank(User().Email)) And 
      User().Email in AuthorizedUsers
    );
    Set(HasAccess,
      IsValidUser And (
        UserRole = "Admin" Or 
        UserRole = "Editor"
      )
    )
  
  # Use meaningful variable names
  OnSelect: =
    Set(CalculatedTotal, (BasePrice * Quantity) + TaxAmount);
    Set(DiscountedTotal, CalculatedTotal * (1 - DiscountRate))
  
  # Group related conditions logically
  Visible: =
    // User permissions
    (UserRole = "Admin" Or UserRole = "Manager") And
    // Record state
    (Status <> "Deleted" And Status <> "Archived") And
    // Time constraints
    (StartDate <= Today() And EndDate >= Today())
  
  # Use consistent spacing around operators
  Text: =FirstName & " " & LastName        # Consistent spacing
  Value: =(Base + Adjustment) * Multiplier # Clear arithmetic grouping
```

#### **Performance Considerations**:
```yaml
Properties:
  # Efficient operator usage
  Items: =Filter(
    LargeDataSource,
    Status = "Active" And        # Simple equality first
    Category in SelectedCategories And    # Membership test
    Amount > MinAmount           # Range filter last
  )
  
  # Avoid expensive operations in loops
  Items: =AddColumns(
    Products,
    "FormattedPrice",
    Text(Price, "Currency")      # Formatting operation per row
  )
  
  # Cache complex calculations
  OnVisible: =Set(
    TotalRevenue,
    Sum(Filter(Orders, Status = "Complete"), Amount)
  );
  Text: =Text(TotalRevenue, "Currency")  # Use cached value
  
  # Minimize delegation breaks
  Items: =
    // Delegable operations first
    Filter(
      LargeDataSource,
      Status = "Active" And
      CreatedDate >= DateAdd(Today(), -30, Days)
    )
    // Non-delegable operations on filtered subset
    |> AddColumns(
      "DisplayName",
      FirstName & " " & LastName
    )
  
  # Use short-circuit evaluation strategically
  Visible: =
    IsQuickCheck Or              # Fast check first
    ExpensiveValidation()        # Only if quick check fails
```

#### **Error Prevention**:
```yaml
Properties:
  # Safe property access
  Text: =If(
    Not(IsBlank(SelectedRecord)) And Not(IsError(SelectedRecord)),
    SelectedRecord.Name,
    "No data available"
  )
  
  # Type-safe operations
  Text: =If(
    IsNumeric(TextInput1.Text),
    Text(Value(TextInput1.Text) * 2, "General Number"),
    "Please enter a valid number"
  )
  
  # Null-safe string operations
  Text: =Coalesce(FirstName, "") & " " & Coalesce(LastName, "")
  
  # Safe division operations
  Value: =If(Denominator = 0, 0, Numerator / Denominator)
  
  # Defensive date operations
  Text: =If(
    Not(IsBlank(SelectedDate)) And SelectedDate >= Date(1900,1,1),
    Text(SelectedDate, "Short Date"),
    "Invalid date"
  )
  
  # Collection safety checks
  Text: =If(
    CountRows(MyCollection) > 0,
    First(MyCollection).Name,
    "No items"
  )
```

#### **Maintainability Practices**:
```yaml
Properties:
  # Use descriptive variable names for complex expressions
  OnStart: =
    Set(MaxAllowedAge, 65);
    Set(MinRequiredExperience, 2);
    Set(ValidDepartments, ["Sales", "Marketing", "Engineering"])
  
  Visible: =
    Age <= MaxAllowedAge And
    Experience >= MinRequiredExperience And
    Department in ValidDepartments
  
  # Break complex formulas into logical components
  OnVisible: =
    // Calculate user permissions
    Set(UserCanEdit, UserRole in ["Admin", "Editor"]);
    Set(UserCanDelete, UserRole = "Admin");
    Set(UserCanView, true);
    
    // Calculate record state
    Set(RecordIsEditable, Status in ["Draft", "Review"]);
    Set(RecordIsLocked, LockDate > DateAdd(Today(), -7, Days));
  
  Visible: =UserCanEdit And RecordIsEditable And Not(RecordIsLocked)
  
  # Document complex operator precedence
  Value: =
    // Order of operations: %, ^, *, /, +, -
    BaseAmount + (BaseAmount * TaxRate) + ShippingCost - 
    (BaseAmount * DiscountRate)
    
  # Alternative with explicit grouping for clarity
  Value: =
    BaseAmount + 
    (BaseAmount * TaxRate) + 
    ShippingCost - 
    (BaseAmount * DiscountRate)
```

#### **Debugging and Testing Patterns**:
```yaml
Properties:
  # Add intermediate variables for debugging
  OnSelect: =
    Set(DebugStep1, ValidateInput(UserInput));
    Set(DebugStep2, ProcessInput(DebugStep1));
    Set(DebugStep3, SaveResult(DebugStep2));
    
    If(IsError(DebugStep3),
      Collect(DebugLog, {
        Timestamp: Now(),
        Step: "SaveResult",
        Input: DebugStep2,
        Error: FirstError.Message
      })
    )
  
  # Use conditional compilation patterns
  Text: =If(
    App.Environment = "Development",
    "Debug: " & VariableName & " = " & Text(VariableValue),
    NormalDisplayText
  )
  
  # Validate operator precedence with test cases
  TestResults: =
    {
      SimpleArithmetic: 2 + 3 * 4,        # Expected: 14
      WithParentheses: (2 + 3) * 4,       # Expected: 20
      MixedTypes: "Count: " & 5 + 3,      # Expected: "Count: 8"
      LogicalPrecedence: true Or false And false  # Expected: true
    }
```

#### **Localization and Globalization**:
```yaml
Properties:
  # Account for locale-dependent operators
  OnStart: =Set(
    UserLocale,
    If(
      "," in Text(1.5),  # Check if comma is decimal separator
      "European",
      "English"
    )
  );
  
  # Use appropriate separators for user's locale
  Formula: =If(
    UserLocale = "European",
    "Concatenate(""Hello""; "" ""; ""World"")",  # Semicolon separators
    "Concatenate(""Hello"", "" "", ""World"")"   # Comma separators
  )
  
  # Design operator usage for multiple languages
  Text: =Switch(
    Left(Language(), 2),
    "es", FirstName & " " & LastName,     # Standard concatenation
    "ja", LastName & " " & FirstName,     # Different order for Japanese
    FirstName & " " & LastName            # Default
  )
```

#### **Security and Validation**:
```yaml
Properties:
  # Input sanitization with operators
  CleanedInput: =
    Trim(
      Substitute(
        Substitute(
          Substitute(TextInput1.Text, "<", ""),
          ">", ""
        ),
        "&", ""
      )
    )
  
  # Permission checking with logical operators
  CanPerformAction: =
    (UserRole = "Admin") Or
    (UserRole = "Manager" And RecordOwner = User().Email) Or
    (UserRole = "User" And RecordOwner = User().Email And Status = "Draft")
  
  # Data validation chains
  IsValidData: =
    Not(IsBlank(RequiredField1)) And
    Not(IsBlank(RequiredField2)) And
    IsNumeric(NumericField) And
    Value(NumericField) > 0 And
    Not(IsBlank(EmailField)) And
    IsMatch(EmailField, Email)
```

#### **Code Organization Patterns**:
```yaml
Properties:
  # Group operators by purpose
  // Validation operators
  IsValidInput: =Not(IsBlank(TextInput1.Text)) And Len(TextInput1.Text) >= 3
  IsValidEmail: =IsMatch(EmailInput.Text, Email)
  IsValidDate: =Not(IsBlank(DatePicker1.SelectedDate))
  
  // Calculation operators  
  SubTotal: =Price * Quantity
  TaxAmount: =SubTotal * TaxRate
  TotalAmount: =SubTotal + TaxAmount
  
  // Display operators
  FormattedTotal: =Text(TotalAmount, "Currency")
  FormattedDate: =Text(OrderDate, "Short Date")
  
  # Use consistent operator patterns across the app
  StandardValidation: =
    Not(IsBlank({FieldName})) And
    Len({FieldName}) >= {MinLength} And
    Len({FieldName}) <= {MaxLength}
```

## Power Fx Regular Expressions Reference

Power Fx provides comprehensive regular expression support through the **IsMatch**, **Match**, and **MatchAll** functions. This exhaustive guide covers pattern matching, text validation, and data extraction using regular expressions in canvas apps.

### Overview

Regular expressions in Power Fx enable:
- **Pattern Matching**: Validate text input formats
- **Text Extraction**: Extract specific data from strings
- **Data Validation**: Ensure user input meets requirements
- **Search and Filter**: Find content matching specific patterns
- **Text Processing**: Advanced string manipulation and parsing

#### **Core Functions**:
- **IsMatch()**: Test if text matches a pattern (returns true/false)
- **Match()**: Extract first match and submatches from text
- **MatchAll()**: Extract all matches and submatches from text
- **Predefined Patterns**: Built-in patterns for common scenarios

### Regular Expression Functions

#### **IsMatch Function**:
```yaml
Properties:
  # Basic pattern validation
  Visible: =IsMatch(TextInput1.Text, "\d{3}-\d{3}-\d{4}")  # Phone format
  Visible: =IsMatch(EmailInput.Text, Match.Email)          # Email validation
  Visible: =IsMatch(ZipCodeInput.Text, "\d{5}(-\d{4})?")  # ZIP code format
  
  # Case-insensitive matching
  Visible: =IsMatch(StatusInput.Text, "active|inactive", MatchOptions.IgnoreCase)
  
  # Complete string matching
  Visible: =IsMatch(ProductCode.Text, "^[A-Z]{3}\d{4}$", MatchOptions.Complete)
  
  # Multi-line text validation
  Visible: =IsMatch(
    TextArea1.Text, 
    "^.+$", 
    MatchOptions.Multiline
  )
  
  # Complex validation with options
  Visible: =IsMatch(
    InputField.Text,
    "(?i)^[a-z]+\s+\d{2,4}$",  # Inline case-insensitive option
    MatchOptions.Complete
  )
```

#### **Match Function**:
```yaml
Properties:
  # Extract first match
  Text: =Match(FullName.Text, "(\w+)\s+(\w+)").FullMatch     # Full name match
  Text: =Match(PhoneNumber.Text, "(\d{3})-(\d{3})-(\d{4})").FullMatch
  
  # Extract named submatches
  Text: =Match(
    EmailAddress.Text,
    "(?<username>\w+)@(?<domain>\w+\.\w+)"
  ).username  # Extract username part
  
  Text: =Match(
    DateString.Text,
    "(?<month>\d{1,2})/(?<day>\d{1,2})/(?<year>\d{4})"
  ).year     # Extract year part
  
  # Extract numbered submatches
  Text: =Match(
    ProductInfo.Text,
    "(\w+)-(\d+)-(\w+)",
    MatchOptions.NumberedSubMatches
  ).SubMatches.1  # Second submatch (index 1)
  
  # Safe extraction with error handling
  Text: =If(
    IsMatch(DataInput.Text, "ID:(\d+)"),
    Match(DataInput.Text, "ID:(\d+)").SubMatches.1,
    "No ID found"
  )
  
  # Complex pattern extraction
  Text: =Match(
    LogEntry.Text,
    "(?<timestamp>\d{4}-\d{2}-\d{2})\s+(?<level>\w+):\s+(?<message>.*)"
  ).message
```

#### **MatchAll Function**:
```yaml
Properties:
  # Extract all matches
  Items: =MatchAll(DocumentText.Text, "\b\w+@\w+\.\w+\b")  # All email addresses
  Items: =MatchAll(PhoneList.Text, "\d{3}-\d{3}-\d{4}")   # All phone numbers
  Items: =MatchAll(ProductCodes.Text, "[A-Z]{3}\d{4}")    # All product codes
  
  # Extract with named submatches
  Items: =MatchAll(
    ContactList.Text,
    "(?<name>\w+\s+\w+)\s+(?<phone>\d{3}-\d{3}-\d{4})"
  )
  
  # Process multiple matches
  Items: =ForAll(
    MatchAll(TaggedText.Text, "#(?<tag>\w+)"),
    {
      Tag: tag,
      Length: Len(tag)
    }
  )
  
  # Extract data from structured text
  Items: =MatchAll(
    CSVData.Text,
    "(?<name>[^,]+),(?<age>\d+),(?<city>[^,\r\n]+)"
  )
  
  # Complex multi-line extraction
  Items: =MatchAll(
    LogFile.Text,
    "(?m)^(?<date>\d{4}-\d{2}-\d{2})\s+(?<time>\d{2}:\d{2}:\d{2})\s+(?<level>\w+):\s+(?<message>.*)$"
  )
```

### Pattern Elements

#### **Literal Characters**:
```yaml
Properties:
  # Direct character matching
  Visible: =IsMatch(Input.Text, "hello")        # Matches "hello" exactly
  Visible: =IsMatch(Input.Text, "user@domain")  # Matches literal @ symbol
  
  # Escaped special characters
  Visible: =IsMatch(Input.Text, "price\$\d+")   # Matches "price$" + digits
  Visible: =IsMatch(Input.Text, "file\.txt")    # Matches "file.txt" exactly
  Visible: =IsMatch(Input.Text, "\[info\]")     # Matches "[info]" exactly
  
  # Unicode characters
  Visible: =IsMatch(Input.Text, "café")         # Matches accented characters
  Visible: =IsMatch(Input.Text, "\u0041")       # Matches "A" (Unicode)
  Visible: =IsMatch(Input.Text, "\x41")         # Matches "A" (hex)
  
  # Special character codes
  Visible: =IsMatch(Input.Text, "line1\r\nline2")  # Carriage return + newline
  Visible: =IsMatch(Input.Text, "tab\there")       # Tab character
  Visible: =IsMatch(Input.Text, "form\fdata")      # Form feed
```

#### **Character Classes**:
```yaml
Properties:
  # Basic character classes
  Visible: =IsMatch(Input.Text, "[abc]")        # Matches a, b, or c
  Visible: =IsMatch(Input.Text, "[a-z]")        # Any lowercase letter
  Visible: =IsMatch(Input.Text, "[A-Za-z0-9]")  # Alphanumeric character
  Visible: =IsMatch(Input.Text, "[^0-9]")       # NOT a digit
  
  # Predefined character classes
  Visible: =IsMatch(Input.Text, "\d")           # Digit (0-9)
  Visible: =IsMatch(Input.Text, "\w")           # Word character (letter/digit/_)
  Visible: =IsMatch(Input.Text, "\s")           # Whitespace character
  Visible: =IsMatch(Input.Text, ".")            # Any character (except newline)
  
  # Negated character classes
  Visible: =IsMatch(Input.Text, "\D")           # Non-digit
  Visible: =IsMatch(Input.Text, "\W")           # Non-word character
  Visible: =IsMatch(Input.Text, "\S")           # Non-whitespace
  
  # Unicode categories
  Visible: =IsMatch(Input.Text, "\p{Lu}")       # Uppercase letter
  Visible: =IsMatch(Input.Text, "\p{Ll}")       # Lowercase letter
  Visible: =IsMatch(Input.Text, "\p{Nd}")       # Decimal digit
  Visible: =IsMatch(Input.Text, "\p{Pc}")       # Connector punctuation
  Visible: =IsMatch(Input.Text, "\P{L}")        # NOT a letter
  
  # Complex character classes
  Visible: =IsMatch(Input.Text, "[\w\-\.]")     # Word char, hyphen, or period
  Visible: =IsMatch(Input.Text, "[a-zA-Z\s]")   # Letters and whitespace
```

#### **Quantifiers**:
```yaml
Properties:
  # Basic quantifiers
  Visible: =IsMatch(Input.Text, "a?")           # Zero or one 'a'
  Visible: =IsMatch(Input.Text, "a*")           # Zero or more 'a'
  Visible: =IsMatch(Input.Text, "a+")           # One or more 'a'
  
  # Exact count quantifiers
  Visible: =IsMatch(Input.Text, "a{3}")         # Exactly 3 'a's
  Visible: =IsMatch(Input.Text, "a{2,5}")       # Between 2 and 5 'a's
  Visible: =IsMatch(Input.Text, "a{3,}")        # At least 3 'a's
  
  # Lazy quantifiers (non-greedy)
  Visible: =IsMatch(Input.Text, "a+?")          # One or more 'a' (minimal)
  Visible: =IsMatch(Input.Text, "a*?")          # Zero or more 'a' (minimal)
  Visible: =IsMatch(Input.Text, "a{2,5}?")      # Between 2-5 'a' (minimal)
  
  # Common patterns with quantifiers
  Visible: =IsMatch(Input.Text, "\d{3}-\d{3}-\d{4}")    # Phone: 123-456-7890
  Visible: =IsMatch(Input.Text, "[A-Z]{2,3}\d{4,6}")    # Code: AB1234 or ABC123456
  Visible: =IsMatch(Input.Text, "\w+@\w+\.\w{2,4}")     # Email pattern
```

#### **Anchors and Assertions**:
```yaml
Properties:
  # Position anchors
  Visible: =IsMatch(Input.Text, "^start")       # Begins with "start"
  Visible: =IsMatch(Input.Text, "end$")         # Ends with "end"
  Visible: =IsMatch(Input.Text, "^exact$")      # Exactly "exact"
  
  # Word boundaries
  Visible: =IsMatch(Input.Text, "\bword\b")     # Whole word "word"
  Visible: =IsMatch(Input.Text, "\Bpart\B")     # "part" NOT at word boundary
  
  # Lookahead assertions
  Visible: =IsMatch(Input.Text, "pass(?=word)")  # "pass" followed by "word"
  Visible: =IsMatch(Input.Text, "file(?!\.exe)") # "file" NOT followed by ".exe"
  
  # Lookbehind assertions
  Visible: =IsMatch(Input.Text, "(?<=pre)fix")   # "fix" preceded by "pre"
  Visible: =IsMatch(Input.Text, "(?<!in)valid")  # "valid" NOT preceded by "in"
  
  # Complex boundary patterns
  Visible: =IsMatch(
    Input.Text,
    "(?<=\s|^)\d+(?=\s|$)"  # Number at word boundaries
  )
```

#### **Groups and Alternation**:
```yaml
Properties:
  # Basic grouping
  Visible: =IsMatch(Input.Text, "(abc)+")       # One or more "abc"
  Visible: =IsMatch(Input.Text, "(cat|dog)")    # Either "cat" or "dog"
  
  # Named capture groups
  Text: =Match(
    Input.Text,
    "(?<first>\w+)\s+(?<last>\w+)"
  ).first  # Extract first name
  
  # Numbered capture groups (with option)
  Text: =Match(
    Input.Text,
    "(\w+)\s+(\w+)",
    MatchOptions.NumberedSubMatches
  ).SubMatches.1  # Second group
  
  # Non-capturing groups
  Visible: =IsMatch(Input.Text, "(?:cat|dog)s")  # "cats" or "dogs"
  
  # Complex alternation
  Visible: =IsMatch(
    Input.Text,
    "(mobile|cell|phone)\s*:?\s*\d{3}-\d{3}-\d{4}"
  )
  
  # Nested groups
  Text: =Match(
    Input.Text,
    "(?<area>(?<state>[A-Z]{2})-(?<city>\w+))"
  ).city
```

#### **Backreferences**:
```yaml
Properties:
  # Named backreferences
  Visible: =IsMatch(
    Input.Text,
    "(?<word>\w+)\s+\k<word>"  # Repeated word
  )
  
  # Numbered backreferences (with option)
  Visible: =IsMatch(
    Input.Text,
    "(\w+)\s+\1",  # Same word repeated
    MatchOptions.NumberedSubMatches
  )
  
  # Complex backreference patterns
  Visible: =IsMatch(
    Input.Text,
    "(?<open><\w+>).*?(?<close></\w+>)"  # HTML-like tags
  )
```

### Match Options

#### **MatchOptions.Contains** (Default):
```yaml
Properties:
  # Default behavior - pattern can be anywhere in text
  Visible: =IsMatch("Hello World", "World")           # True
  Visible: =IsMatch("abc123def", "\d+")               # True (finds 123)
  Visible: =IsMatch("The quick brown fox", "quick")   # True
  
  # Explicit Contains option
  Visible: =IsMatch(
    "Find this pattern",
    "pattern",
    MatchOptions.Contains
  )
```

#### **MatchOptions.Complete**:
```yaml
Properties:
  # Pattern must match entire string
  Visible: =IsMatch("123", "\d+", MatchOptions.Complete)     # True
  Visible: =IsMatch("abc123", "\d+", MatchOptions.Complete)  # False
  
  # Equivalent to anchoring with ^ and $
  Visible: =IsMatch("123", "^\d+$")                   # Same as Complete
  
  # Form validation examples
  Visible: =IsMatch(
    ZipCodeInput.Text,
    "\d{5}(-\d{4})?",
    MatchOptions.Complete
  )
  
  Visible: =IsMatch(
    PhoneInput.Text,
    "\d{3}-\d{3}-\d{4}",
    MatchOptions.Complete
  )
```

#### **MatchOptions.BeginsWith**:
```yaml
Properties:
  # Pattern must be at start of string
  Visible: =IsMatch("Hello World", "Hello", MatchOptions.BeginsWith)  # True
  Visible: =IsMatch("Hello World", "World", MatchOptions.BeginsWith)  # False
  
  # Equivalent to ^ anchor
  Visible: =IsMatch("Hello World", "^Hello")          # Same as BeginsWith
  
  # URL validation
  Visible: =IsMatch(
    URLInput.Text,
    "https?://",
    MatchOptions.BeginsWith
  )
```

#### **MatchOptions.EndsWith**:
```yaml
Properties:
  # Pattern must be at end of string
  Visible: =IsMatch("document.pdf", "\.pdf", MatchOptions.EndsWith)   # True
  Visible: =IsMatch("document.pdf", "doc", MatchOptions.EndsWith)     # False
  
  # Equivalent to $ anchor
  Visible: =IsMatch("document.pdf", "\.pdf$")         # Same as EndsWith
  
  # File extension validation
  Visible: =IsMatch(
    FilenameInput.Text,
    "\.(jpg|png|gif)",
    MatchOptions.EndsWith & MatchOptions.IgnoreCase
  )
```

#### **MatchOptions.IgnoreCase**:
```yaml
Properties:
  # Case-insensitive matching
  Visible: =IsMatch("HELLO", "hello", MatchOptions.IgnoreCase)        # True
  Visible: =IsMatch("MiXeD CaSe", "mixed case", MatchOptions.IgnoreCase)  # True
  
  # Protocol matching
  Visible: =IsMatch(
    "FILE://server/path",
    "^file://",
    MatchOptions.BeginsWith & MatchOptions.IgnoreCase
  )
  
  # Status validation
  Visible: =IsMatch(
    StatusInput.Text,
    "^(active|inactive|pending)$",
    MatchOptions.Complete & MatchOptions.IgnoreCase
  )
  
  # Inline case-insensitive option
  Visible: =IsMatch("HELLO", "(?i)hello")             # Equivalent
```

#### **MatchOptions.Multiline**:
```yaml
Properties:
  # ^ and $ match line boundaries, not just string boundaries
  Items: =MatchAll(
    MultilineText.Text,
    "^Line \d+$",
    MatchOptions.Multiline
  )
  
  # Extract headers from each line
  Items: =MatchAll(
    DocumentText.Text,
    "^# (?<title>.+)$",
    MatchOptions.Multiline
  )
  
  # Inline multiline option
  Items: =MatchAll(
    TextContent.Text,
    "(?m)^ERROR: (?<message>.+)$"
  )
```

#### **MatchOptions.DotAll**:
```yaml
Properties:
  # Dot (.) matches newline characters
  Text: =Match(
    "Line 1" & Char(10) & "Line 2",
    "Line 1.*Line 2",
    MatchOptions.DotAll
  ).FullMatch  # Matches across lines
  
  # Without DotAll, . doesn't match newlines
  Text: =Match(
    "Line 1" & Char(10) & "Line 2",
    "Line 1.*Line 2"
  ).FullMatch  # Returns blank
  
  # Inline DotAll option
  Text: =Match(
    MultilineContent.Text,
    "(?s)start.*end"
  ).FullMatch
```

#### **MatchOptions.FreeSpacing**:
```yaml
Properties:
  # Whitespace and comments ignored in pattern
  Visible: =IsMatch(
    "2024-12-25",
    "(?x)               # Free spacing mode
     \d{4}              # Year (4 digits)
     -                  # Separator
     \d{2}              # Month (2 digits)  
     -                  # Separator
     \d{2}              # Day (2 digits)",
    MatchOptions.FreeSpacing
  )
  
  # Complex email pattern with documentation
  Visible: =IsMatch(
    EmailInput.Text,
    "(?x)
     (?<username>       # Username part
       [a-zA-Z0-9._%-]+ # Allow letters, numbers, and common symbols
     )
     @                  # At symbol
     (?<domain>         # Domain part
       [a-zA-Z0-9.-]+   # Domain name
       \.               # Dot
       [a-zA-Z]{2,4}    # Top-level domain
     )",
    MatchOptions.Complete & MatchOptions.FreeSpacing
  )
```

#### **MatchOptions.NumberedSubMatches**:
```yaml
Properties:
  # Enable traditional numbered capture groups
  Text: =Match(
    "John Doe",
    "(\w+) (\w+)",
    MatchOptions.NumberedSubMatches
  ).SubMatches.1  # "Doe" (second group)
  
  # Numbered backreferences
  Visible: =IsMatch(
    "hello hello",
    "(\w+) \1",
    MatchOptions.NumberedSubMatches
  )
  
  # Cannot mix named and numbered captures
  # This would cause an error:
  # "(?<first>\w+) (\w+)" with NumberedSubMatches
```

#### **Combined Options**:
```yaml
Properties:
  # Multiple options with & operator
  Visible: =IsMatch(
    URLInput.Text,
    "^https?://[a-z0-9.-]+\.[a-z]{2,4}$",
    MatchOptions.Complete & MatchOptions.IgnoreCase
  )
  
  # Email validation with multiple options
  Visible: =IsMatch(
    EmailInput.Text,
    "^\w+@\w+\.\w{2,4}$",
    MatchOptions.Complete & MatchOptions.IgnoreCase
  )
  
  # Multi-line processing with case insensitivity
  Items: =MatchAll(
    LogText.Text,
    "^(?<level>error|warning|info): (?<message>.+)$",
    MatchOptions.Multiline & MatchOptions.IgnoreCase
  )
```

### Predefined Patterns

#### **Match Enum Patterns**:
```yaml
Properties:
  # Basic character patterns
  Visible: =IsMatch(Input.Text, Match.Any)              # Any single character
  Visible: =IsMatch(Input.Text, Match.Digit)            # Single digit (0-9)
  Visible: =IsMatch(Input.Text, Match.Letter)           # Single letter
  Visible: =IsMatch(Input.Text, Match.Space)            # Single whitespace
  Visible: =IsMatch(Input.Text, Match.NonSpace)         # Single non-whitespace
  
  # Multiple character patterns
  Visible: =IsMatch(Input.Text, Match.MultipleDigits)   # One or more digits
  Visible: =IsMatch(Input.Text, Match.MultipleLetters)  # One or more letters
  Visible: =IsMatch(Input.Text, Match.MultipleSpaces)   # One or more spaces
  Visible: =IsMatch(Input.Text, Match.MultipleNonSpaces) # One or more non-spaces
  
  # Optional patterns
  Visible: =IsMatch(Input.Text, Match.OptionalDigits)   # Zero or more digits
  Visible: =IsMatch(Input.Text, Match.OptionalLetters)  # Zero or more letters
  Visible: =IsMatch(Input.Text, Match.OptionalSpaces)   # Zero or more spaces
  
  # Punctuation patterns
  Visible: =IsMatch(Input.Text, Match.Period)           # Period (.)
  Visible: =IsMatch(Input.Text, Match.Comma)            # Comma (,)
  Visible: =IsMatch(Input.Text, Match.Hyphen)           # Hyphen (-)
  Visible: =IsMatch(Input.Text, Match.LeftParen)        # Left parenthesis (
  Visible: =IsMatch(Input.Text, Match.RightParen)       # Right parenthesis )
  Visible: =IsMatch(Input.Text, Match.Tab)              # Tab character
```

#### **Match.Email Pattern**:
```yaml
Properties:
  # Email validation
  Visible: =IsMatch(EmailInput.Text, Match.Email, MatchOptions.Complete)
  
  # Extract all email addresses from text
  Items: =MatchAll(DocumentText.Text, Match.Email)
  
  # Email validation with custom requirements
  Visible: =And(
    IsMatch(EmailInput.Text, Match.Email, MatchOptions.Complete),
    Not("test" in Lower(EmailInput.Text))  # Exclude test emails
  )
  
  # Extract email components
  Text: =Match(
    EmailInput.Text,
    "(?<local>[^@]+)@(?<domain>[^@]+)",
    MatchOptions.Complete
  ).local  # Username part
  
  # Business email validation
  Visible: =And(
    IsMatch(EmailInput.Text, Match.Email, MatchOptions.Complete),
    Not(IsMatch(
      EmailInput.Text,
      "@(gmail|yahoo|hotmail|outlook)\.com$",
      MatchOptions.EndsWith & MatchOptions.IgnoreCase
    ))
  )
```

#### **Combining Predefined Patterns**:
```yaml
Properties:
  # Phone number pattern
  Visible: =IsMatch(
    PhoneInput.Text,
    Match.Digit & Match.Digit & Match.Digit & Match.Hyphen &
    Match.Digit & Match.Digit & Match.Digit & Match.Hyphen &
    Match.MultipleDigits
  )
  
  # Product code pattern (ABC123)
  Visible: =IsMatch(
    ProductInput.Text,
    Match.Letter & Match.Letter & Match.Letter & Match.MultipleDigits,
    MatchOptions.Complete
  )
  
  # Date pattern (MM/DD/YYYY)
  Visible: =IsMatch(
    DateInput.Text,
    Match.Digit & Match.Digit & "/" & 
    Match.Digit & Match.Digit & "/" &
    Match.Digit & Match.Digit & Match.Digit & Match.Digit,
    MatchOptions.Complete
  )
  
  # Custom identifier pattern
  Visible: =IsMatch(
    IDInput.Text,
    Match.Letter & Match.OptionalLetters & Match.Hyphen & Match.MultipleDigits
  )
```

### Common Pattern Examples

#### **Data Validation Patterns**:
```yaml
Properties:
  # US Phone number formats
  Visible: =IsMatch(
    PhoneInput.Text,
    "^(\+1\s?)?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}$",
    MatchOptions.Complete
  )
  
  # Social Security Number
  Visible: =IsMatch(
    SSNInput.Text,
    "^\d{3}-\d{2}-\d{4}$",
    MatchOptions.Complete
  )
  
  # Credit card number (basic)
  Visible: =IsMatch(
    CardInput.Text,
    "^\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}$",
    MatchOptions.Complete
  )
  
  # US ZIP code
  Visible: =IsMatch(
    ZipInput.Text,
    "^\d{5}(-\d{4})?$",
    MatchOptions.Complete
  )
  
  # IP Address (basic)
  Visible: =IsMatch(
    IPInput.Text,
    "^(\d{1,3}\.){3}\d{1,3}$",
    MatchOptions.Complete
  )
  
  # URL validation
  Visible: =IsMatch(
    URLInput.Text,
    "^https?://[\w.-]+\.[\w]{2,4}(/.*)?$",
    MatchOptions.Complete & MatchOptions.IgnoreCase
  )
```

#### **Password Strength Patterns**:
```yaml
Properties:
  # At least 8 characters with mixed case and numbers
  Visible: =And(
    Len(PasswordInput.Text) >= 8,
    IsMatch(PasswordInput.Text, "[A-Z]"),      # Has uppercase
    IsMatch(PasswordInput.Text, "[a-z]"),      # Has lowercase
    IsMatch(PasswordInput.Text, "\d"),         # Has digit
    IsMatch(PasswordInput.Text, "[^A-Za-z0-9]") # Has special char
  )
  
  # Strong password (single regex)
  Visible: =IsMatch(
    PasswordInput.Text,
    "^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{8,}$",
    MatchOptions.Complete
  )
  
  # No common weak patterns
  Visible: =And(
    IsMatch(PasswordInput.Text, "^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$"),
    Not(IsMatch(PasswordInput.Text, "password|123456|qwerty", MatchOptions.IgnoreCase))
  )
```

#### **Date and Time Patterns**:
```yaml
Properties:
  # ISO date format (YYYY-MM-DD)
  Visible: =IsMatch(
    DateInput.Text,
    "^\d{4}-\d{2}-\d{2}$",
    MatchOptions.Complete
  )
  
  # US date format (MM/DD/YYYY)
  Visible: =IsMatch(
    DateInput.Text,
    "^(0[1-9]|1[0-2])/(0[1-9]|[12][0-9]|3[01])/\d{4}$",
    MatchOptions.Complete
  )
  
  # Time format (HH:MM or HH:MM:SS)
  Visible: =IsMatch(
    TimeInput.Text,
    "^([01]?[0-9]|2[0-3]):[0-5][0-9](:[0-5][0-9])?$",
    MatchOptions.Complete
  )
  
  # DateTime extraction
  Text: =Match(
    LogEntry.Text,
    "(?<date>\d{4}-\d{2}-\d{2})\s+(?<time>\d{2}:\d{2}:\d{2})"
  ).date
```

#### **File and Path Patterns**:
```yaml
Properties:
  # File extension validation
  Visible: =IsMatch(
    FilenameInput.Text,
    "\.(pdf|doc|docx|xls|xlsx|ppt|pptx)$",
    MatchOptions.EndsWith & MatchOptions.IgnoreCase
  )
  
  # Image file validation
  Visible: =IsMatch(
    ImageInput.Text,
    "\.(jpg|jpeg|png|gif|bmp|svg)$",
    MatchOptions.EndsWith & MatchOptions.IgnoreCase
  )
  
  # Windows file path
  Visible: =IsMatch(
    PathInput.Text,
    "^[A-Z]:\\\\([^\\\\/:*?""<>|]+\\\\)*[^\\\\/:*?""<>|]*$",
    MatchOptions.Complete
  )
  
  # Extract filename from path
  Text: =Match(
    FilePath.Text,
    "([^/\\\\]+)$"
  ).FullMatch
```

#### **Code and Format Patterns**:
```yaml
Properties:
  # Hex color code
  Visible: =IsMatch(
    ColorInput.Text,
    "^#[0-9A-Fa-f]{6}$",
    MatchOptions.Complete
  )
  
  # GUID/UUID format
  Visible: =IsMatch(
    GUIDInput.Text,
    "^[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}$",
    MatchOptions.Complete
  )
  
  # License plate (US format)
  Visible: =IsMatch(
    PlateInput.Text,
    "^[A-Z0-9]{1,8}$",
    MatchOptions.Complete
  )
  
  # Version number (semantic versioning)
  Visible: =IsMatch(
    VersionInput.Text,
    "^\d+\.\d+\.\d+(-[\w.]+)?$",
    MatchOptions.Complete
  )
```

### Text Processing and Extraction

#### **Data Parsing Examples**:
```yaml
Properties:
  # Parse CSV-like data
  Items: =ForAll(
    MatchAll(
      CSVText.Text,
      "(?<name>[^,]+),(?<age>\d+),(?<email>[^,\r\n]+)"
    ),
    {
      Name: Trim(name),
      Age: Value(age),
      Email: Trim(email)
    }
  )
  
  # Extract structured log data
  Items: =ForAll(
    MatchAll(
      LogData.Text,
      "(?<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\s+\[(?<level>\w+)\]\s+(?<message>.*)"
    ),
    {
      Timestamp: DateTimeValue(timestamp),
      Level: Upper(level),
      Message: message
    }
  )
  
  # Parse configuration file
  Items: =ForAll(
    MatchAll(
      ConfigText.Text,
      "^(?<key>\w+)\s*=\s*(?<value>.+)$",
      MatchOptions.Multiline
    ),
    {
      Key: key,
      Value: value
    }
  )
```

#### **Text Cleaning and Transformation**:
```yaml
Properties:
  # Extract clean phone numbers
  Items: =ForAll(
    MatchAll(ContactText.Text, "\d{3}[-.]?\d{3}[-.]?\d{4}"),
    {
      Original: FullMatch,
      Clean: Substitute(Substitute(FullMatch, "-", ""), ".", "")
    }
  )
  
  # Extract and format URLs
  Items: =ForAll(
    MatchAll(
      DocumentText.Text,
      "https?://[^\s<>\"]+",
      MatchOptions.IgnoreCase
    ),
    {
      URL: FullMatch,
      Domain: Match(FullMatch, "://([^/]+)").SubMatches.1
    }
  )
  
  # Clean and validate input
  Text: =If(
    IsMatch(
      Trim(UserInput.Text),
      "^[\w\s.-]{2,50}$",
      MatchOptions.Complete
    ),
    Proper(Trim(UserInput.Text)),
    "Invalid input"
  )
```

### Advanced Pattern Techniques

#### **Complex Validation Scenarios**:
```yaml
Properties:
  # Business rules validation
  Visible: =And(
    // Required format: ABC-12345
    IsMatch(
      ProductCode.Text,
      "^[A-Z]{3}-\d{5}$",
      MatchOptions.Complete
    ),
    // First letter indicates category
    Left(ProductCode.Text, 1) in ["A", "B", "C"],
    // Code not in blacklist
    Not(ProductCode.Text in BlacklistedCodes)
  )
  
  # Conditional validation
  Visible: =Switch(
    DocumentType.Selected.Value,
    "Invoice", IsMatch(
      DocumentNumber.Text,
      "^INV-\d{4}-\d{6}$",
      MatchOptions.Complete
    ),
    "Receipt", IsMatch(
      DocumentNumber.Text,
      "^REC-\d{8}$",
      MatchOptions.Complete
    ),
    "Order", IsMatch(
      DocumentNumber.Text,
      "^ORD-[A-Z]{2}\d{6}$",
      MatchOptions.Complete
    ),
    false
  )
```

#### **Performance Optimization**:
```yaml
Properties:
  # Cache expensive regex operations
  OnVisible: =Set(
    EmailPattern,
    "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
  );
  
  Visible: =IsMatch(EmailInput.Text, EmailPattern, MatchOptions.Complete)
  
  # Use simple checks before complex regex
  Visible: =And(
    Len(InputText.Text) >= 5,               # Quick length check
    "@" in InputText.Text,                  # Quick content check
    IsMatch(InputText.Text, Match.Email)    # Full validation
  )
  
  # Limit regex complexity for real-time validation
  Visible: =If(
    Len(InputText.Text) < 100,
    IsMatch(InputText.Text, ComplexPattern),
    IsMatch(InputText.Text, SimplePattern)
  )
```

### Regular Expression Best Practices

#### **Pattern Design Guidelines**:
```yaml
Properties:
  # Use anchors for exact matching
  Visible: =IsMatch(Input.Text, "^exact$", MatchOptions.Complete)  # Preferred
  Visible: =IsMatch(Input.Text, "exact", MatchOptions.Complete)    # Also works
  
  # Escape special characters properly
  Visible: =IsMatch(Input.Text, "price: \$\d+\.\d{2}")  # Escape $ and .
  
  # Use character classes efficiently
  Visible: =IsMatch(Input.Text, "[0-9]")     # Less efficient
  Visible: =IsMatch(Input.Text, "\d")        # More efficient
  
  # Group alternatives efficiently
  Visible: =IsMatch(Input.Text, "(cat|dog|bird)")      # Good
  Visible: =IsMatch(Input.Text, "cat|dog|bird")        # Also good
  
  # Use non-capturing groups when not extracting
  Visible: =IsMatch(Input.Text, "(?:https?|ftp)://")   # Non-capturing
  Visible: =IsMatch(Input.Text, "(https?|ftp)://")     # Capturing (unnecessary)
```

#### **Error Prevention and Debugging**:
```yaml
Properties:
  # Test patterns incrementally
  Step1: =IsMatch(Input.Text, "\d")                    # Has digit?
  Step2: =IsMatch(Input.Text, "\d{3}")                 # Has 3 digits?
  Step3: =IsMatch(Input.Text, "\d{3}-\d{3}-\d{4}")     # Full phone pattern?
  
  # Use Match.Email for email validation
  Visible: =IsMatch(EmailInput.Text, Match.Email)      # Recommended
  // Instead of custom email regex which can be error-prone
  
  # Validate user input before processing
  ProcessedData: =If(
    IsMatch(RawInput.Text, "^[\w\s.,!?-]+$"),
    MatchAll(RawInput.Text, YourPattern),
    []  // Return empty if input invalid
  )
  
  # Handle missing matches gracefully
  ExtractedValue: =If(
    IsMatch(Input.Text, Pattern),
    Match(Input.Text, Pattern).SubMatches.1,
    DefaultValue
  )
```

#### **Maintenance and Documentation**:
```yaml
Properties:
  # Document complex patterns
  PhonePattern: ="^\d{3}-\d{3}-\d{4}$"  # US phone format: 123-456-7890
  Visible: =IsMatch(PhoneInput.Text, PhonePattern, MatchOptions.Complete)
  
  # Use descriptive variable names
  EmailValidationPattern: =Match.Email
  StrongPasswordPattern: ="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{8,}$"
  
  # Break complex patterns into parts
  AreaCodePattern: ="\d{3}"
  ExchangePattern: ="\d{3}"
  NumberPattern: ="\d{4}"
  PhonePattern: =AreaCodePattern & "-" & ExchangePattern & "-" & NumberPattern
  
  # Use FreeSpacing for complex patterns
  ComplexDatePattern: ="(?x)
    ^                    # Start of string
    \d{4}                # Year (4 digits)
    -                    # Separator
    (0[1-9]|1[0-2])      # Month (01-12)
    -                    # Separator
    (0[1-9]|[12]\d|3[01]) # Day (01-31)
    $                    # End of string"
```

## Power Fx Tables Reference

Power Fx provides comprehensive table manipulation capabilities for working with structured data in canvas apps. This exhaustive guide covers table creation, record manipulation, field operations, and advanced data processing patterns.

### Overview

Tables in Power Fx are fundamental data structures that enable:
- **Data Storage**: Organize information in structured records and fields
- **Data Manipulation**: Filter, sort, transform, and aggregate data
- **Record Operations**: Create, modify, and access individual records
- **Field Management**: Work with columns and data types consistently
- **Collection Management**: Handle dynamic data sets and user interactions

#### **Core Concepts**:
- **Table**: Collection of records with consistent field structure
- **Record**: Individual data item containing named fields
- **Field**: Specific piece of information within a record
- **Column**: Same field across multiple records in a table

### Table Structure and Elements

#### **Records - Individual Data Items**:
```yaml
Properties:
  # Inline record definition
  RecordExample: ={ Name: "John Doe", Age: 30, Active: true }
  
  # Record with calculated fields
  CalculatedRecord: ={
    FirstName: "Jane",
    LastName: "Smith", 
    FullName: "Jane" & " " & "Smith",
    CreatedDate: Now()
  }
  
  # Record with nested structure
  NestedRecord: ={
    Employee: {
      Name: "Alice Johnson",
      Department: "Engineering",
      Contact: {
        Email: "alice@company.com",
        Phone: "555-0123"
      }
    }
  }
  
  # Record from form controls
  FormRecord: ={
    ProductName: TextInput_Name.Text,
    ProductPrice: Value(TextInput_Price.Text),
    InStock: Checkbox_Available.Value,
    Category: Dropdown_Category.Selected.Value
  }
  
  # Record with special characters in field names
  SpecialFieldRecord: ={
    'Product Name': "Widget A",
    'Unit Price': 15.99,
    'Quantity on Hand': 25,
    'Last Updated': Today()
  }
```

#### **Fields - Data Elements Within Records**:
```yaml
Properties:
  # Access fields using dot notation
  Text: =First(Products).Name                    # Get Name field from first record
  Text: =ThisItem.ProductName                    # Access field in gallery item
  Text: =LookUp(Customers, ID = 123).'Full Name' # Field with space in name
  
  # Field validation and formatting
  Text: =If(
    IsBlank(ThisItem.Email),
    "No email provided",
    ThisItem.Email
  )
  
  # Field calculations
  Text: =ThisItem.Quantity * ThisItem.UnitPrice  # Calculate total
  Text: =DateDiff(ThisItem.StartDate, Today())   # Days since start
  
  # Field type checking
  Visible: =IsNumeric(ThisItem.Score)            # Check if field is numeric
  Visible: =Not(IsBlank(ThisItem.Description))   # Check if field has value
  
  # Field concatenation
  Text: =Concatenate(
    ThisItem.FirstName, 
    " ", 
    ThisItem.LastName
  )
  
  # Field with default values
  Text: =If(
    IsBlank(ThisItem.Status),
    "Pending",
    ThisItem.Status
  )
```

#### **Columns - Consistent Field Structure**:
```yaml
Properties:
  # Single column extraction
  Items: =ShowColumns(Products, "ProductName")   # Show only ProductName column
  Items: =Products.ProductName                   # Shorthand for single column
  
  # Multiple column selection
  Items: =ShowColumns(
    Employees,
    "FirstName",
    "LastName", 
    "Department"
  )
  
  # Column operations
  Items: =RenameColumns(
    Products,
    "ProductName", "Name",
    "UnitPrice", "Price"
  )
  
  # Add calculated columns
  Items: =AddColumns(
    Orders,
    "Total", Quantity * UnitPrice,
    "Status", If(
      Shipped,
      "Completed",
      "Pending"
    )
  )
  
  # Drop unwanted columns
  Items: =DropColumns(
    DetailedProducts,
    "InternalNotes",
    "SupplierCode"
  )
  
  # Column data type consistency
  Items: =AddColumns(
    ImportedData,
    "NumericValue", Value(TextValue),
    "DateValue", DateValue(DateText),
    "BooleanValue", If(StatusText = "Active", true, false)
  )
```

#### **Tables - Complete Data Structures**:
```yaml
Properties:
  # Inline table definition using Table function
  StaticTable: =Table(
    { Name: "Alice", Age: 30, Department: "Sales" },
    { Name: "Bob", Age: 25, Department: "Engineering" },
    { Name: "Carol", Age: 35, Department: "Marketing" }
  )
  
  # Single-column table with brackets
  SimpleList: =["Apple", "Banana", "Orange", "Grape"]
  
  # Complex table with nested data
  ComplexTable: =Table(
    {
      Product: "Laptop",
      Specifications: {
        CPU: "Intel i7",
        RAM: "16GB",
        Storage: "512GB SSD"
      },
      PriceHistory: [
        { Date: Date(2024,1,1), Price: 1200 },
        { Date: Date(2024,6,1), Price: 1100 }
      ]
    },
    {
      Product: "Monitor",
      Specifications: {
        Size: "27 inch",
        Resolution: "4K",
        Panel: "IPS"
      },
      PriceHistory: [
        { Date: Date(2024,1,1), Price: 400 },
        { Date: Date(2024,6,1), Price: 350 }
      ]
    }
  )
  
  # Table from collection or data source
  Items: =Filter(
    Products,
    Category = "Electronics" And
    InStock = true
  )
  
  # Table with calculations
  Items: =AddColumns(
    Sales,
    "Commission", Amount * 0.05,
    "Quarter", "Q" & Text(
      RoundUp(Month(SaleDate) / 3, 0)
    )
  )
```

### Table Creation and Definition

#### **Inline Table Creation**:
```yaml
Properties:
  # Basic table with consistent structure
  ProductCatalog: =Table(
    { ID: 1, Name: "Widget A", Price: 29.99, InStock: true },
    { ID: 2, Name: "Widget B", Price: 34.99, InStock: false },
    { ID: 3, Name: "Widget C", Price: 19.99, InStock: true }
  )
  
  # Configuration table
  AppSettings: =Table(
    { Setting: "Theme", Value: "Dark" },
    { Setting: "Language", Value: "English" },
    { Setting: "AutoSave", Value: "true" },
    { Setting: "NotificationsEnabled", Value: "false" }
  )
  
  # Lookup table for validation
  StatusOptions: =Table(
    { Value: "Draft", Label: "Draft", Color: "Gray" },
    { Value: "Review", Label: "Under Review", Color: "Yellow" },
    { Value: "Approved", Label: "Approved", Color: "Green" },
    { Value: "Rejected", Label: "Rejected", Color: "Red" }
  )
  
  # Menu structure table
  NavigationMenu: =Table(
    { Screen: "Home", Icon: "Home", Order: 1, Visible: true },
    { Screen: "Products", Icon: "ShoppingCart", Order: 2, Visible: true },
    { Screen: "Reports", Icon: "BarChart", Order: 3, Visible: varUserRole = "Admin" },
    { Screen: "Settings", Icon: "Settings", Order: 4, Visible: varUserRole = "Admin" }
  )
```

#### **Single-Column Tables**:
```yaml
Properties:
  # Simple value lists
  Colors: =["Red", "Green", "Blue", "Yellow", "Purple"]
  Numbers: =[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  Priorities: =["Low", "Medium", "High", "Critical"]
  
  # Equivalent Table function syntax
  ColorsTable: =Table(
    { Value: "Red" },
    { Value: "Green" },
    { Value: "Blue" },
    { Value: "Yellow" },
    { Value: "Purple" }
  )
  
  # Dynamic single-column table
  YearsList: =ForAll(
    Sequence(10),
    Year(Today()) - Value + 1
  )
  
  # Filtered single-column extraction
  ProductNames: =Distinct(Products, ProductName)
  ActiveUserEmails: =Filter(Users, Active).Email
  
  # Single-column calculations
  TotalsByCategory: =GroupBy(
    Sales,
    "Category",
    "TotalSales",
    Sum(Amount)
  ).TotalSales
```

#### **Nested Table Structures**:
```yaml
Properties:
  # Table with nested records
  CompanyStructure: =Table(
    {
      Department: "Engineering",
      Manager: { Name: "Alice Smith", Email: "alice@company.com" },
      Employees: Table(
        { Name: "Bob Johnson", Role: "Senior Developer" },
        { Name: "Carol Davis", Role: "Junior Developer" }
      ),
      Budget: 150000
    },
    {
      Department: "Sales", 
      Manager: { Name: "David Wilson", Email: "david@company.com" },
      Employees: Table(
        { Name: "Eva Brown", Role: "Sales Manager" },
        { Name: "Frank Miller", Role: "Sales Rep" }
      ),
      Budget: 100000
    }
  )
  
  # Product with variants table
  ProductWithVariants: =Table(
    {
      ProductName: "T-Shirt",
      BasePrice: 19.99,
      Variants: Table(
        { Size: "S", Color: "Red", Stock: 10 },
        { Size: "M", Color: "Blue", Stock: 15 },
        { Size: "L", Color: "Green", Stock: 8 }
      )
    }
  )
  
  # Order with line items
  OrderStructure: =Table(
    {
      OrderID: "ORD-001",
      CustomerName: "John Doe",
      OrderDate: Date(2024,6,15),
      LineItems: Table(
        { Product: "Widget A", Quantity: 2, Price: 29.99 },
        { Product: "Widget B", Quantity: 1, Price: 34.99 }
      ),
      ShippingAddress: {
        Street: "123 Main St",
        City: "Anytown",
        State: "CA",
        ZipCode: "12345"
      }
    }
  )
```

### Table Manipulation Functions

#### **Filtering and Selection**:
```yaml
Properties:
  # Basic filtering
  Items: =Filter(Products, InStock = true)
  Items: =Filter(Orders, OrderDate >= DateAdd(Today(), -30))
  Items: =Filter(Employees, Department = "Sales" And Active = true)
  
  # Complex filtering with multiple conditions
  Items: =Filter(
    Sales,
    SaleDate >= DateValue("2024-01-01") And
    SaleDate <= DateValue("2024-12-31") And
    Amount > 1000 And
    Region in ["North", "South"]
  )
  
  # Text-based filtering
  Items: =Filter(
    Customers,
    StartsWith(CompanyName, TextInput_Search.Text) Or
    Contains(Lower(ContactEmail), Lower(TextInput_Search.Text))
  )
  
  # Filtering with related data
  Items: =Filter(
    Orders,
    CountRows(
      Filter(OrderItems, OrderID = Orders[@OrderID] And BackOrdered = true)
    ) = 0
  )
  
  # Advanced filtering with calculations
  Items: =Filter(
    Products,
    (QuantityOnHand / AverageMonthlyUsage) < 2  // Less than 2 months inventory
  )
  
  # Multiple table filtering
  Items: =Filter(
    Employees,
    ID in ShowColumns(
      Filter(Assignments, ProjectStatus = "Active"),
      "EmployeeID"
    ).EmployeeID
  )
```

#### **Sorting and Ordering**:
```yaml
Properties:
  # Simple sorting
  Items: =Sort(Products, ProductName)                # Ascending by name
  Items: =Sort(Orders, OrderDate, Descending)        # Descending by date
  Items: =Sort(Employees, LastName, FirstName)       # Multiple columns
  
  # Complex sorting with calculations
  Items: =Sort(
    AddColumns(Products, "TotalValue", QuantityOnHand * UnitPrice),
    TotalValue,
    Descending
  )
  
  # Conditional sorting
  Items: =Sort(
    Tasks,
    Switch(
      Priority,
      "Critical", 1,
      "High", 2,
      "Medium", 3,
      "Low", 4,
      5
    ),
    DueDate
  )
  
  # Sorting with nested fields
  Items: =Sort(Orders, Customer.CompanyName, Customer.ContactName)
  
  # Custom sort order
  Items: =Sort(
    StatusReports,
    LookUp(
      ["New", "In Progress", "Review", "Complete"],
      Value = StatusReports[@Status]
    ).Value,
    Ascending
  )
  
  # Sort by calculated relevance
  Items: =Sort(
    Filter(
      Products,
      Contains(Lower(ProductName), Lower(SearchText.Text))
    ),
    Len(ProductName) - Len(SearchText.Text)  // Shorter names first
  )
```

#### **Grouping and Aggregation**:
```yaml
Properties:
  # Basic grouping
  Items: =GroupBy(Sales, "Region", "RegionalSales")
  Items: =GroupBy(Orders, "CustomerID", "CustomerOrders")
  
  # Grouping with calculations
  Items: =GroupBy(
    Sales,
    "SalesRep",
    "TotalSales",
    Sum(Amount),
    "OrderCount",
    CountRows(Sales),
    "AverageOrder",
    Average(Amount)
  )
  
  # Multiple field grouping
  Items: =GroupBy(
    Transactions,
    "Year",
    "Quarter",
    "QuarterlyData"
  )
  
  # Complex grouping with preprocessing
  Items: =GroupBy(
    AddColumns(
      Orders,
      "Month", Text(OrderDate, "yyyy-mm"),
      "Year", Year(OrderDate)
    ),
    "Year",
    "Month",
    "MonthlyOrders",
    "MonthlyRevenue",
    Sum(TotalAmount)
  )
  
  # Nested grouping results
  Items: =ForAll(
    GroupBy(Sales, "Region", "RegionalData"),
    {
      Region: Region,
      MonthlyBreakdown: GroupBy(
        RegionalData,
        "Month",
        "MonthlySales"
      ),
      TotalRegionalSales: Sum(RegionalData, Amount)
    }
  )
  
  # Grouping with filtering
  Items: =GroupBy(
    Filter(
      Sales,
      SaleDate >= DateAdd(Today(), -365)  // Last year only
    ),
    "ProductCategory",
    "CategorySales",
    "Revenue",
    Sum(Amount),
    "Units",
    Sum(Quantity)
  )
```

#### **Column Operations**:
```yaml
Properties:
  # Add calculated columns
  Items: =AddColumns(
    Orders,
    "DaysOld", DateDiff(OrderDate, Today()),
    "Status", If(
      Shipped,
      "Shipped",
      If(DaysOld > 7, "Overdue", "Pending")
    ),
    "Priority", Switch(
      CustomerTier,
      "Gold", "High",
      "Silver", "Medium", 
      "Low"
    )
  )
  
  # Rename columns for display
  Items: =RenameColumns(
    Products,
    "ProductName", "Name",
    "UnitPrice", "Price",
    "QuantityOnHand", "In Stock"
  )
  
  # Select specific columns
  Items: =ShowColumns(
    Employees,
    "FullName",
    "Department",
    "Email",
    "HireDate"
  )
  
  # Remove sensitive columns
  Items: =DropColumns(
    CustomerData,
    "SSN",
    "CreditCardNumber",
    "InternalNotes"
  )
  
  # Complex column transformations
  Items: =AddColumns(
    RawData,
    "FormattedDate", Text(DateValue(DateString), "mm/dd/yyyy"),
    "CleanedName", Proper(Trim(Lower(NameField))),
    "ParsedAmount", Value(
      Substitute(
        Substitute(AmountString, "$", ""),
        ",", ""
      )
    ),
    "Category", LookUp(
      CategoryMappings,
      Code = RawData[@CategoryCode]
    ).CategoryName
  )
  
  # Conditional column additions
  Items: =AddColumns(
    BaseData,
    "AdditionalInfo", If(
      ShowDetailed,
      LookUp(DetailedData, ID = BaseData[@ID]),
      Blank()
    )
  )
```

### Advanced Table Operations

#### **Table Joining and Relationships**:
```yaml
Properties:
  # Basic lookup join
  Items: =AddColumns(
    Orders,
    "CustomerInfo", LookUp(
      Customers,
      CustomerID = Orders[@CustomerID]
    )
  )
  
  # Multiple table enrichment
  Items: =AddColumns(
    OrderItems,
    "ProductInfo", LookUp(
      Products,
      ProductID = OrderItems[@ProductID]
    ),
    "OrderInfo", LookUp(
      Orders, 
      OrderID = OrderItems[@OrderID]
    ),
    "ExtendedPrice", Quantity * LookUp(
      Products,
      ProductID = OrderItems[@ProductID]
    ).UnitPrice
  )
  
  # Complex relationship handling
  Items: =AddColumns(
    Employees,
    "Manager", LookUp(
      Employees,
      EmployeeID = Employees[@ManagerID]
    ),
    "DirectReports", Filter(
      Employees,
      ManagerID = Employees[@EmployeeID]
    ),
    "ProjectCount", CountRows(
      Filter(
        ProjectAssignments,
        EmployeeID = Employees[@EmployeeID] And
        Status = "Active"
      )
    )
  )
  
  # Cross-reference tables
  Items: =AddColumns(
    Products,
    "Categories", Filter(
      ProductCategories,
      ProductID = Products[@ProductID]
    ),
    "Suppliers", Filter(
      ProductSuppliers, 
      ProductID = Products[@ProductID]
    ),
    "AverageRating", Average(
      Filter(
        ProductReviews,
        ProductID = Products[@ProductID]
      ),
      Rating
    )
  )
```

#### **Table Transformation and Reshaping**:
```yaml
Properties:
  # Pivot-like transformations
  Items: =ForAll(
    Distinct(Sales, Region),
    {
      Region: Result,
      Q1Sales: Sum(
        Filter(Sales, Region = Result And Quarter = "Q1"),
        Amount
      ),
      Q2Sales: Sum(
        Filter(Sales, Region = Result And Quarter = "Q2"), 
        Amount
      ),
      Q3Sales: Sum(
        Filter(Sales, Region = Result And Quarter = "Q3"),
        Amount
      ),
      Q4Sales: Sum(
        Filter(Sales, Region = Result And Quarter = "Q4"),
        Amount
      )
    }
  )
  
  # Unpivot operations
  Items: =ForAll(
    QuarterlyData,
    [
      { Quarter: "Q1", Region: Region, Sales: Q1Sales },
      { Quarter: "Q2", Region: Region, Sales: Q2Sales },
      { Quarter: "Q3", Region: Region, Sales: Q3Sales },
      { Quarter: "Q4", Region: Region, Sales: Q4Sales }
    ]
  )
  
  # Flatten nested structures
  Items: =Ungroup(
    ForAll(
      Orders,
      ForAll(
        OrderItems,
        {
          OrderID: Orders[@OrderID],
          CustomerName: Orders[@CustomerName],
          OrderDate: Orders[@OrderDate],
          ProductName: ProductName,
          Quantity: Quantity,
          UnitPrice: UnitPrice,
          LineTotal: Quantity * UnitPrice
        }
      )
    ),
    "Value"
  )
  
  # Hierarchical data flattening
  Items: =Ungroup(
    ForAll(
      Departments,
      ForAll(
        Employees,
        AddColumns(
          Employees,
          "DepartmentName", Departments[@DepartmentName],
          "DepartmentHead", Departments[@HeadOfDepartment]
        )
      )
    ),
    "Value"
  )
```

#### **Data Validation and Cleanup**:
```yaml
Properties:
  # Data quality validation
  Items: =AddColumns(
    ImportedData,
    "ValidationErrors", Concatenate(
      If(IsBlank(CustomerName), "Missing customer name; ", ""),
      If(Not(IsNumeric(Amount)), "Invalid amount; ", ""),
      If(Not(IsMatch(Email, Match.Email)), "Invalid email; ", ""),
      If(IsBlank(OrderDate), "Missing order date; ", "")
    ),
    "IsValid", And(
      Not(IsBlank(CustomerName)),
      IsNumeric(Amount),
      IsMatch(Email, Match.Email),
      Not(IsBlank(OrderDate))
    )
  )
  
  # Data standardization
  Items: =AddColumns(
    RawCustomers,
    "StandardizedPhone", Substitute(
      Substitute(
        Substitute(Phone, "(", ""),
        ")", ""
      ),
      "-", ""
    ),
    "CleanedEmail", Lower(Trim(Email)),
    "FormattedName", Proper(Trim(CustomerName)),
    "NormalizedState", Upper(Left(State, 2))
  )
  
  # Duplicate detection
  Items: =AddColumns(
    Customers,
    "DuplicateCount", CountRows(
      Filter(
        Customers,
        Email = Customers[@Email] Or
        (Phone = Customers[@Phone] And Not(IsBlank(Phone)))
      )
    ),
    "IsDuplicate", CountRows(
      Filter(
        Customers,
        Email = Customers[@Email] And
        ID <> Customers[@ID]
      )
    ) > 0
  )
  
  # Data enrichment and completion
  Items: =AddColumns(
    Addresses,
    "CompleteAddress", Concatenate(
      If(IsBlank(StreetAddress), "", StreetAddress & ", "),
      If(IsBlank(City), "", City & ", "),
      If(IsBlank(State), "", State & " "),
      If(IsBlank(ZipCode), "", ZipCode)
    ),
    "Region", LookUp(
      StateRegionMapping,
      StateCode = Addresses[@State]
    ).Region,
    "TimeZone", LookUp(
      ZipCodeData,
      ZipCode = Addresses[@ZipCode]
    ).TimeZone
  )
```

### Record Scope and Context

#### **Record Scope in Table Functions**:
```yaml
Properties:
  # Filter with record scope
  Items: =Filter(
    Products,
    UnitPrice > 100 And                    # Direct field access
    QuantityOnHand < ReorderLevel And      # Multiple fields
    Category in ["Electronics", "Gadgets"] # Field comparison
  )
  
  # AddColumns with calculations
  Items: =AddColumns(
    Orders,
    "Subtotal", Quantity * UnitPrice,      # Calculate from fields
    "Tax", (Quantity * UnitPrice) * TaxRate,
    "Total", (Quantity * UnitPrice) * (1 + TaxRate),
    "Discount", If(
      Quantity >= 10,
      (Quantity * UnitPrice) * 0.1,       # Volume discount
      0
    )
  )
  
  # Complex record scope operations
  Items: =AddColumns(
    Employees,
    "Tenure", DateDiff(HireDate, Today()),
    "SalaryGrade", Switch(
      Salary,
      Salary < 50000, "Entry",
      Salary < 75000, "Mid",
      Salary < 100000, "Senior",
      "Executive"
    ),
    "NextReviewDate", DateAdd(LastReviewDate, 365),
    "IsEligibleForBonus", And(
      DateDiff(HireDate, Today()) >= 365,
      PerformanceRating >= 3,
      Department <> "Intern"
    )
  )
  
  # Nested record scope
  Items: =AddColumns(
    Customers,
    "RecentOrders", Filter(
      Orders,
      CustomerID = Customers[@CustomerID] And    # Disambiguation needed
      OrderDate >= DateAdd(Today(), -90)
    ),
    "RecentOrderCount", CountRows(
      Filter(
        Orders,
        CustomerID = Customers[@CustomerID] And
        OrderDate >= DateAdd(Today(), -90)
      )
    ),
    "LastOrderAmount", LookUp(
      Orders,
      CustomerID = Customers[@CustomerID] And
      OrderDate = Max(
        Filter(
          Orders,
          CustomerID = Customers[@CustomerID]
        ),
        OrderDate
      )
    ).TotalAmount
  )
```

#### **Disambiguation with @ Operator**:
```yaml
Properties:
  # Global variable access in record scope
  Items: =Filter(
    Products,
    Category = [@varSelectedCategory] And  # Global variable
    UnitPrice <= [@varMaxPrice]            # Global variable
  )
  
  # Nested table disambiguation
  Items: =AddColumns(
    Orders,
    "ItemDetails", ForAll(
      Filter(
        OrderItems,
        OrderID = Orders[@OrderID]         # Outer table reference
      ),
      {
        ProductName: LookUp(
          Products,
          ProductID = OrderItems[@ProductID] # Inner table reference
        ).ProductName,
        Quantity: Quantity,                 # Inner table field
        OrderDate: Orders[@OrderDate]       # Outer table field
      }
    )
  )
  
  # Complex disambiguation scenarios
  Items: =ForAll(
    Departments,
    {
      DepartmentName: DepartmentName,      # Current record field
      EmployeeCount: CountRows(
        Filter(
          Employees,
          DepartmentID = Departments[@DepartmentID] # Outer scope
        )
      ),
      Manager: LookUp(
        Employees,
        EmployeeID = Departments[@ManagerID] And
        Active = [@varActiveFilter]         # Global variable
      ),
      Budget: Budget * [@varBudgetMultiplier] # Global calculation
    }
  )
  
  # Collection access in nested scopes
  Items: =ForAll(
    colRegions,
    {
      Region: RegionName,
      TopSalesRep: First(
        Sort(
          Filter(
            colSalesReps,
            Region = colRegions[@RegionName] And  # Outer collection
            Active = [@varShowActiveOnly]         # Global variable
          ),
          TotalSales,
          Descending
        )
      ).FullName
    }
  )
```

#### **Context Variables in Record Scope**:
```yaml
Properties:
  # Using With for complex calculations
  Items: =With(
    {
      CurrentDate: Today(),
      FiscalYearStart: Date(Year(Today()), 4, 1)  # April 1st
    },
    Filter(
      Transactions,
      TransactionDate >= FiscalYearStart And
      TransactionDate <= CurrentDate And
      Amount > 0
    )
  )
  
  # Complex context with multiple variables
  Items: =With(
    {
      SearchTerm: Lower(Trim(TextInput_Search.Text)),
      MinPrice: Value(TextInput_MinPrice.Text),
      MaxPrice: Value(TextInput_MaxPrice.Text),
      SelectedCategories: Split(Dropdown_Categories.Selected.Value, ",")
    },
    Filter(
      Products,
      (IsBlank(SearchTerm) Or 
       Contains(Lower(ProductName), SearchTerm) Or
       Contains(Lower(Description), SearchTerm)) And
      (IsBlank(MinPrice) Or UnitPrice >= MinPrice) And
      (IsBlank(MaxPrice) Or UnitPrice <= MaxPrice) And
      (IsEmpty(SelectedCategories) Or Category in SelectedCategories)
    )
  )
  
  # Nested With statements
  Items: =With(
    { BaseFilters: Filter(Orders, OrderDate >= DateAdd(Today(), -30)) },
    With(
      { 
        TotalRevenue: Sum(BaseFilters, TotalAmount),
        AverageOrder: Average(BaseFilters, TotalAmount)
      },
      AddColumns(
        BaseFilters,
        "RevenuePercent", TotalAmount / TotalRevenue * 100,
        "AboveAverage", TotalAmount > AverageOrder
      )
    )
  )
```

### Data Source Integration

#### **Working with External Data Sources**:
```yaml
Properties:
  # SharePoint list integration
  Items: =Filter(
    'SharePoint List',
    Status.Value = "Active" And
    Modified >= DateAdd(Today(), -7)
  )
  
  # SQL Server data with relationships
  Items: =AddColumns(
    Filter(SQLOrders, OrderStatus = "Pending"),
    "CustomerDetails", LookUp(
      SQLCustomers,
      CustomerID = SQLOrders[@CustomerID]
    ),
    "OrderItems", Filter(
      SQLOrderItems,
      OrderID = SQLOrders[@OrderID]
    )
  )
  
  # Excel workbook data
  Items: =AddColumns(
    'Excel Workbook'.Products,
    "CategoryName", LookUp(
      'Excel Workbook'.Categories,
      CategoryID = 'Excel Workbook'.Products[@CategoryID]
    ).CategoryName
  )
  
  # Dataverse table operations
  Items: =Filter(
    Accounts,
    statuscode = 'Status Code (Accounts)'.Active And
    createdon >= DateAdd(Today(), -30) And
    ownerid = User()
  )
  
  # Web API / REST service data
  Items: =AddColumns(
    Filter(
      ParseJSON(APIConnector.GetProducts()).value,
      available = true
    ),
    "FormattedPrice", Text(price, "Currency"),
    "StockStatus", If(
      inventory_count > 10,
      "In Stock",
      If(inventory_count > 0, "Low Stock", "Out of Stock")
    )
  )
```

#### **Collection Management**:
```yaml
Properties:
  # Initialize collection from data source
  OnStart: =ClearCollect(
    colProducts,
    AddColumns(
      Products,
      "Selected", false,
      "OriginalIndex", Value
    )
  );
  
  # Dynamic collection updates
  OnSelect: =Patch(
    colShoppingCart,
    Defaults(colShoppingCart),
    {
      ProductID: ThisItem.ID,
      ProductName: ThisItem.Name,
      Quantity: Value(TextInput_Quantity.Text),
      UnitPrice: ThisItem.Price,
      DateAdded: Now()
    }
  );
  
  # Collection aggregation and summary
  Items: =AddColumns(
    GroupBy(
      colOrderItems,
      "ProductID",
      "Items"
    ),
    "ProductName", LookUp(
      colProducts,
      ID = ProductID
    ).Name,
    "TotalQuantity", Sum(Items, Quantity),
    "TotalAmount", Sum(Items, Quantity * UnitPrice),
    "AveragePrice", Average(Items, UnitPrice)
  )
  
  # Collection synchronization
  OnVisible: =If(
    CountRows(colLocalData) = 0 Or
    DateDiff(varLastSync, Now(), Minutes) > 15,
    ClearCollect(
      colLocalData,
      Filter(
        RemoteDataSource,
        Modified >= varLastSync
      )
    );
    Set(varLastSync, Now())
  );
  
  # Batch collection operations
  OnSelect: =ForAll(
    Filter(colSelectedItems, Selected = true),
    Patch(
      Orders,
      Defaults(Orders),
      {
        ProductID: ProductID,
        Quantity: Quantity,
        OrderDate: Today(),
        Status: "Pending"
      }
    )
  );
```

### Performance Optimization

#### **Efficient Table Operations**:
```yaml
Properties:
  # Minimize nested operations
  Items: =With(
    { FilteredOrders: Filter(Orders, OrderDate >= DateAdd(Today(), -30)) },
    AddColumns(
      FilteredOrders,
      "CustomerName", LookUp(Customers, ID = FilteredOrders[@CustomerID]).Name,
      "ItemCount", CountRows(
        Filter(OrderItems, OrderID = FilteredOrders[@OrderID])
      )
    )
  )
  
  # Cache lookup tables
  OnStart: =Set(
    varCustomerLookup,
    AddColumns(
      Customers,
      "LookupKey", ID
    )
  );
  
  Items: =AddColumns(
    Orders,
    "CustomerName", LookUp(
      varCustomerLookup,
      LookupKey = Orders[@CustomerID]
    ).Name
  )
  
  # Reduce data transfer
  Items: =ShowColumns(
    Filter(
      Products,
      Category = "Electronics" And InStock = true
    ),
    "ProductName",
    "Price", 
    "Description"
  )
  
  # Optimize filtering order
  Items: =Filter(
    Products,
    InStock = true And              // Boolean first (fastest)
    Category = "Electronics" And    // Exact match second
    Price >= 100 And                // Numeric comparison
    Contains(ProductName, SearchText) // Text search last (slowest)
  )
  
  # Use delegation-friendly operations
  Items: =Filter(
    LargeDataSource,
    Status = "Active" And
    CreatedDate >= DateAdd(Today(), -90)  // Delegable operations
  )
```

#### **Memory Management**:
```yaml
Properties:
  # Limit result sets
  Items: =FirstN(
    Sort(
      Filter(Products, Category = "Electronics"),
      ProductName
    ),
    100  // Limit to first 100 results
  )
  
  # Progressive loading
  Items: =If(
    CountRows(colDisplayData) < varPageSize * varCurrentPage,
    Concat(
      colDisplayData,
      FirstN(
        Filter(
          AllData,
          Not(ID in ShowColumns(colDisplayData, "ID").ID)
        ),
        varPageSize
      )
    ),
    colDisplayData
  )
  
  # Clear unused collections
  OnHidden: =Clear(colTemporaryData);
  
  # Selective data loading
  Items: =If(
    varLoadDetails,
    AddColumns(
      BaseData,
      "DetailedInfo", LookUp(DetailTable, ID = BaseData[@ID])
    ),
    BaseData  // Load basic data only
  )
```

### Table Best Practices

#### **Data Structure Design**:
```yaml
Properties:
  # Consistent field naming
  WellStructuredTable: =Table(
    { ID: 1, Name: "Product A", Price: 29.99, IsActive: true },
    { ID: 2, Name: "Product B", Price: 34.99, IsActive: false }
  )
  
  # Avoid deeply nested structures when possible
  FlattenedDesign: =Table(
    { 
      OrderID: 1, 
      CustomerID: 101, 
      CustomerName: "John Doe",
      ProductID: 201,
      ProductName: "Widget",
      Quantity: 2
    }
  )
  
  # Use consistent data types
  TypeConsistentTable: =Table(
    { 
      ID: 1,                    // Number
      Name: "Item 1",           // Text
      Price: 29.99,             // Number (decimal)
      InStock: true,            // Boolean
      CreatedDate: Today()      // Date
    }
  )
  
  # Plan for extensibility
  ExtensibleStructure: =Table(
    {
      ID: 1,
      Name: "Base Item",
      Properties: {             // Flexible properties object
        Color: "Red",
        Size: "Large",
        Material: "Cotton"
      },
      Tags: ["featured", "new", "popular"], // Array for multiple values
      Metadata: {
        CreatedBy: User().Email,
        Version: "1.0"
      }
    }
  )
```

#### **Error Handling and Validation**:
```yaml
Properties:
  # Safe field access
  Text: =If(
    IsBlank(ThisItem) Or IsBlank(ThisItem.Name),
    "No name available",
    ThisItem.Name
  )
  
  # Defensive table operations
  Items: =If(
    CountRows(SourceData) > 0,
    Filter(
      SourceData,
      Not(IsBlank(RequiredField)) And
      IsNumeric(NumericField)
    ),
    Table()  // Return empty table if no source data
  )
  
  # Graceful lookup failures
  Text: =With(
    { FoundItem: LookUp(LookupTable, ID = ThisItem.LookupID) },
    If(
      IsBlank(FoundItem),
      "Item not found",
      FoundItem.DisplayName
    )
  )
  
  # Validation before operations
  Items: =If(
    And(
      Not(IsBlank(FilterValue)),
      IsNumeric(FilterValue),
      FilterValue > 0
    ),
    Filter(Products, Price >= FilterValue),
    Products  // Return all if filter invalid
  )
  
  # Error boundary patterns
  Items: =IfError(
    AddColumns(
      ComplexDataOperation,
      "CalculatedField", RiskyCalculation
    ),
    AddColumns(
      ComplexDataOperation,
      "CalculatedField", "Error in calculation"
    )
  )
```

## Power Fx Variables Reference

Power Fx provides comprehensive variable management capabilities for storing and manipulating data across different scopes in canvas apps. This exhaustive guide covers global variables, context variables, collections, and advanced variable patterns.

### Overview

Variables in Power Fx enable:
- **State Management**: Store and maintain application state across user interactions
- **Data Persistence**: Temporarily hold calculated values and user inputs
- **Screen Communication**: Pass data between screens and contexts
- **Performance Optimization**: Cache expensive calculations and API results
- **User Experience**: Create dynamic, responsive applications with stateful behavior

#### **Variable Types**:
- **Global Variables**: App-wide scope, accessible from anywhere
- **Context Variables**: Screen-specific scope, great for parameters
- **Collections**: Table variables for storing and manipulating data sets
- **Built-in Variables**: System-provided variables and functions

#### **Core Principles**:
- **Favor Formulas**: Use automatic recalculation when possible
- **Minimize State**: Keep variables to essential use cases only
- **Clear Naming**: Use descriptive variable names for maintainability
- **Proper Scope**: Choose the right variable type for your needs

### Global Variables

#### **Creating and Setting Global Variables**:
```yaml
Properties:
  # Basic global variable creation with Set function
  OnSelect: =Set(varUserName, "John Doe")
  OnSelect: =Set(varCounter, 0)
  OnSelect: =Set(varIsLoggedIn, true)
  OnSelect: =Set(varCurrentDate, Today())
  
  # Setting multiple variables in sequence
  OnSelect: =Set(varFirstName, "Jane");
             Set(varLastName, "Smith");
             Set(varFullName, varFirstName & " " & varLastName)
  
  # Global variables with calculated values
  OnSelect: =Set(varTotalSales, Sum(Sales, Amount))
  OnSelect: =Set(varAverageRating, Average(Reviews, Rating))
  OnSelect: =Set(varDaysRemaining, DateDiff(Today(), ProjectDeadline))
  
  # Global variables with complex data types
  OnSelect: =Set(varSelectedCustomer, LookUp(Customers, ID = Gallery1.Selected.ID))
  OnSelect: =Set(varUserPreferences, {
    Theme: "Dark",
    Language: "English", 
    NotificationsEnabled: true,
    AutoSave: false
  })
  
  # Global variables with conditional logic
  OnSelect: =Set(varUserRole, 
    If(
      User().Email in AdminEmails,
      "Admin",
      If(
        User().Email in ManagerEmails,
        "Manager",
        "User"
      )
    )
  )
  
  # Global variables from form data
  OnSelect: =Set(varFormData, {
    ProductName: TextInput_Name.Text,
    Price: Value(TextInput_Price.Text),
    Category: Dropdown_Category.Selected.Value,
    Description: TextInput_Description.Text,
    InStock: Checkbox_InStock.Value
  })
```

#### **Reading and Using Global Variables**:
```yaml
Properties:
  # Direct variable usage
  Text: =varUserName
  Visible: =varIsLoggedIn
  Default: =varCounter
  
  # Variables in calculations
  Text: ="Welcome back, " & varUserName & "!"
  Text: =Text(varTotalSales, "Currency")
  Visible: =varUserRole = "Admin"
  
  # Variables in conditional logic
  Text: =If(
    varIsLoggedIn,
    "Dashboard",
    "Please log in"
  )
  
  # Variables in data operations
  Items: =Filter(Products, Category = varSelectedCategory)
  Items: =Sort(Orders, OrderDate, If(varSortAscending, Ascending, Descending))
  
  # Variables with complex objects
  Text: =varSelectedCustomer.CompanyName
  Text: =varUserPreferences.Theme
  Visible: =varUserPreferences.NotificationsEnabled
  
  # Variables in formulas with error handling
  Text: =If(
    IsBlank(varSelectedItem),
    "No item selected",
    varSelectedItem.Name
  )
  
  # Global variables in nested operations
  Items: =Filter(
    Sales,
    SalesRep = varCurrentUser.FullName And
    SaleDate >= varReportStartDate And
    SaleDate <= varReportEndDate
  )
```

#### **Global Variable Patterns**:
```yaml
Properties:
  # Application state management
  OnStart: =Set(varAppVersion, "2.1.0");
           Set(varEnvironment, "Production");
           Set(varLastSync, Now());
           Set(varUserPreferences, {
             Theme: "Light",
             Language: "English",
             ItemsPerPage: 25,
             AutoRefresh: true
           })
  
  # User session management
  OnVisible: =Set(varSessionStart, Now());
            Set(varPageViews, varPageViews + 1);
            Set(varLastActivity, Now())
  
  # Navigation state
  OnSelect: =Set(varCurrentScreen, "ProductDetails");
           Set(varNavigationHistory, 
             Concat(varNavigationHistory, [varCurrentScreen])
           );
           Navigate(ProductDetailScreen)
  
  # Data caching patterns
  OnVisible: =If(
    DateDiff(varLastProductUpdate, Now(), Minutes) > 30,
    Set(varProductCache, Products);
    Set(varLastProductUpdate, Now())
  )
  
  # Configuration management
  OnStart: =Set(varAppConfig, {
    APIEndpoint: "https://api.company.com/v1/",
    MaxRetries: 3,
    TimeoutSeconds: 30,
    EnableLogging: true,
    CacheExpirationMinutes: 60
  })
  
  # Feature flags
  OnStart: =Set(varFeatureFlags, {
    EnableAdvancedReporting: true,
    ShowBetaFeatures: false,
    EnableOfflineMode: true,
    UseNewUI: false
  })
```

### Context Variables

#### **Creating and Setting Context Variables**:
```yaml
Properties:
  # Basic context variable creation with UpdateContext
  OnSelect: =UpdateContext({ locUserName: "Alice Johnson" })
  OnSelect: =UpdateContext({ locCounter: 0 })
  OnSelect: =UpdateContext({ locShowDetails: true })
  
  # Setting multiple context variables
  OnSelect: =UpdateContext({
    locSelectedItem: Gallery1.Selected,
    locEditMode: true,
    locLastUpdate: Now()
  })
  
  # Context variables with calculations
  OnSelect: =UpdateContext({
    locSubtotal: Sum(Gallery_Items.AllItems, Quantity * Price),
    locTax: Sum(Gallery_Items.AllItems, Quantity * Price) * 0.08,
    locTotal: (Sum(Gallery_Items.AllItems, Quantity * Price)) * 1.08
  })
  
  # Conditional context variables
  OnSelect: =UpdateContext({
    locValidationMessage: If(
      IsBlank(TextInput_Email.Text),
      "Email is required",
      If(
        Not(IsMatch(TextInput_Email.Text, Match.Email)),
        "Please enter a valid email address",
        ""
      )
    ),
    locIsValid: And(
      Not(IsBlank(TextInput_Email.Text)),
      IsMatch(TextInput_Email.Text, Match.Email)
    )
  })
  
  # Context variables from complex operations
  OnSelect: =UpdateContext({
    locSearchResults: Filter(
      Products,
      Contains(Lower(ProductName), Lower(TextInput_Search.Text)) Or
      Contains(Lower(Description), Lower(TextInput_Search.Text))
    ),
    locResultCount: CountRows(
      Filter(
        Products,
        Contains(Lower(ProductName), Lower(TextInput_Search.Text))
      )
    ),
    locSearchTime: Now()
  })
```

#### **Context Variables with Navigate**:
```yaml
Properties:
  # Pass context variables during navigation
  OnSelect: =Navigate(
    ProductDetailScreen,
    ScreenTransition.SlideLeft,
    {
      locSelectedProduct: ThisItem,
      locPreviousScreen: "ProductList",
      locEditMode: false,
      locShowActions: varUserRole = "Admin"
    }
  )
  
  # Navigation with complex context
  OnSelect: =Navigate(
    ReportScreen,
    ScreenTransition.Fade,
    {
      locReportType: Dropdown_ReportType.Selected.Value,
      locDateRange: {
        StartDate: DatePicker_Start.SelectedDate,
        EndDate: DatePicker_End.SelectedDate
      },
      locFilters: {
        Category: Dropdown_Category.Selected.Value,
        Region: Dropdown_Region.Selected.Value,
        SalesRep: Dropdown_SalesRep.Selected.Value
      },
      locParameters: {
        ShowDetails: Toggle_Details.Value,
        GroupByMonth: Toggle_Monthly.Value,
        IncludeCharts: Toggle_Charts.Value
      }
    }
  )
  
  # Navigation with user context
  OnSelect: =Navigate(
    UserProfileScreen,
    ScreenTransition.SlideUp,
    {
      locUserData: LookUp(Users, Email = User().Email),
      locEditPermissions: User().Email = varCurrentUser.Email Or varUserRole = "Admin",
      locReturnScreen: varCurrentScreen,
      locTimestamp: Now()
    }
  )
  
  # Conditional navigation with context
  OnSelect: =If(
    varIsLoggedIn,
    Navigate(
      DashboardScreen,
      ScreenTransition.None,
      {
        locWelcomeMessage: "Welcome back, " & varUserName,
        locLastLogin: varLastLoginTime,
        locNotificationCount: CountRows(
          Filter(Notifications, Read = false And UserId = varCurrentUser.ID)
        )
      }
    ),
    Navigate(
      LoginScreen,
      ScreenTransition.SlideRight,
      {
        locRedirectAfterLogin: "Dashboard",
        locMessage: "Please log in to continue"
      }
    )
  )
```

#### **Context Variable Usage Patterns**:
```yaml
Properties:
  # Form state management
  OnVisible: =UpdateContext({
    locFormMode: "New",
    locOriginalData: Blank(),
    locHasChanges: false,
    locValidationErrors: Table()
  })
  
  # Real-time validation
  OnChange: =UpdateContext({
    locEmailError: If(
      IsBlank(Text),
      "Email is required",
      If(
        Not(IsMatch(Text, Match.Email)),
        "Invalid email format",
        ""
      )
    ),
    locFormValid: And(
      Not(IsBlank(TextInput_Email.Text)),
      IsMatch(TextInput_Email.Text, Match.Email),
      Not(IsBlank(TextInput_Name.Text)),
      Len(TextInput_Name.Text) >= 2
    )
  })
  
  # Screen state management
  OnVisible: =UpdateContext({
    locLoading: true,
    locError: "",
    locDataLastUpdated: Blank(),
    locCurrentPage: 1,
    locItemsPerPage: 20
  });
  
  # With for complex local calculations
  OnSelect: =With(
    {
      locOrderTotal: Sum(Gallery_OrderItems.AllItems, Quantity * UnitPrice),
      locTaxRate: LookUp(TaxRates, State = varCurrentUser.State).Rate
    },
    UpdateContext({
      locSubtotal: locOrderTotal,
      locTax: locOrderTotal * locTaxRate,
      locGrandTotal: locOrderTotal * (1 + locTaxRate),
      locCanSubmit: locOrderTotal > 0 And Not(IsBlank(locTaxRate))
    })
  )
  
  # Search and filter state
  OnChange: =UpdateContext({
    locSearchTerm: Lower(Trim(Text)),
    locSearchActive: Not(IsBlank(Trim(Text))),
    locFilteredResults: If(
      IsBlank(Trim(Text)),
      Products,
      Filter(
        Products,
        Contains(Lower(ProductName), Lower(Trim(Text))) Or
        Contains(Lower(Category), Lower(Trim(Text)))
      )
    )
  })
```

### Collections

#### **Creating and Managing Collections**:
```yaml
Properties:
  # Initialize collections with ClearCollect
  OnStart: =ClearCollect(
    colShoppingCart,
    Table()  // Empty table with no schema yet
  );
  
  OnStart: =ClearCollect(
    colUserPreferences,
    {
      Theme: "Light",
      Language: "English",
      NotificationsEnabled: true,
      ItemsPerPage: 25
    }
  );
  
  # Collect single values
  OnSelect: =Collect(
    colRecentSearches,
    TextInput_Search.Text
  );
  
  # Collect records
  OnSelect: =Collect(
    colFavoriteProducts,
    {
      ProductID: ThisItem.ID,
      ProductName: ThisItem.Name,
      DateAdded: Now(),
      UserID: User().Email
    }
  );
  
  # Collect from data sources
  OnVisible: =ClearCollect(
    colActiveProducts,
    Filter(Products, Active = true And InStock > 0)
  );
  
  # Complex collection initialization
  OnStart: =ClearCollect(
    colOrderItems,
    ForAll(
      Sequence(5),
      {
        LineNumber: Value,
        ProductID: Blank(),
        ProductName: "",
        Quantity: 0,
        UnitPrice: 0,
        Total: 0,
        Selected: false
      }
    )
  );
```

#### **Collection Operations and Manipulation**:
```yaml
Properties:
  # Add items to collections
  OnSelect: =Collect(
    colAuditLog,
    {
      Action: "Product Added",
      ProductID: varNewProduct.ID,
      UserEmail: User().Email,
      Timestamp: Now(),
      Details: "Product: " & varNewProduct.Name
    }
  );
  
  # Update collection items
  OnSelect: =UpdateIf(
    colShoppingCart,
    ProductID = ThisItem.ProductID,
    {
      Quantity: Quantity + 1,
      LastModified: Now()
    }
  );
  
  # Remove items from collections
  OnSelect: =RemoveIf(
    colTempData,
    DateDiff(CreatedDate, Now(), Hours) > 24
  );
  
  # Patch collection records
  OnSelect: =Patch(
    colUserSettings,
    LookUp(colUserSettings, SettingName = "Theme"),
    {
      SettingValue: "Dark",
      LastModified: Now(),
      ModifiedBy: User().Email
    }
  );
  
  # Complex collection updates
  OnSelect: =ForAll(
    Filter(colOrderItems, Selected = true),
    Patch(
      colOrderItems,
      ThisRecord,
      {
        Status: "Processing",
        ProcessedDate: Now(),
        ProcessedBy: varCurrentUser.ID
      }
    )
  );
  
  # Collection aggregations
  Text: =Concatenate(
    "Total Items: ", CountRows(colShoppingCart), ", ",
    "Total Value: ", Text(Sum(colShoppingCart, Quantity * UnitPrice), "Currency")
  )
```

#### **Collection Data Patterns**:
```yaml
Properties:
  # Shopping cart pattern
  OnSelect: =If(
    CountRows(
      Filter(colShoppingCart, ProductID = ThisItem.ID)
    ) > 0,
    // Update existing item
    UpdateIf(
      colShoppingCart,
      ProductID = ThisItem.ID,
      { Quantity: Quantity + 1 }
    ),
    // Add new item
    Collect(
      colShoppingCart,
      {
        ProductID: ThisItem.ID,
        ProductName: ThisItem.Name,
        UnitPrice: ThisItem.Price,
        Quantity: 1,
        DateAdded: Now()
      }
    )
  );
  
  # Form draft storage
  OnChange: =Patch(
    colFormDrafts,
    Defaults(colFormDrafts),
    {
      FormName: "ProductEntry",
      UserEmail: User().Email,
      DraftData: {
        ProductName: TextInput_Name.Text,
        Description: TextInput_Description.Text,
        Price: Value(TextInput_Price.Text),
        Category: Dropdown_Category.Selected.Value
      },
      LastSaved: Now()
    }
  );
  
  # Recent activity tracking
  OnSelect: =Collect(
    colUserActivity,
    {
      ActivityType: "Screen View",
      ScreenName: varCurrentScreen,
      Timestamp: Now(),
      UserEmail: User().Email,
      SessionID: varSessionID
    }
  );
  // Keep only last 100 activities
  If(
    CountRows(colUserActivity) > 100,
    RemoveIf(
      colUserActivity,
      Timestamp < Last(
        FirstN(
          Sort(colUserActivity, Timestamp, Ascending),
          CountRows(colUserActivity) - 100
        )
      ).Timestamp
    )
  );
  
  # Offline data synchronization
  OnVisible: =If(
    Connection.Connected,
    // Sync pending changes
    ForAll(
      Filter(colOfflineChanges, Synced = false),
      Patch(
        RemoteDataSource,
        ThisRecord.RemoteRecord,
        ThisRecord.Changes
      );
      Patch(
        colOfflineChanges,
        ThisRecord,
        { Synced: true, SyncDate: Now() }
      )
    );
    // Remove synced records
    RemoveIf(colOfflineChanges, Synced = true)
  );
```

#### **Collection Performance Optimization**:
```yaml
Properties:
  # Efficient collection queries
  Items: =With(
    { 
      searchTerm: Lower(Trim(TextInput_Search.Text)),
      selectedCategory: Dropdown_Category.Selected.Value
    },
    Filter(
      colProducts,
      (IsBlank(searchTerm) Or Contains(Lower(ProductName), searchTerm)) And
      (IsBlank(selectedCategory) Or Category = selectedCategory)
    )
  )
  
  # Minimize collection operations
  OnSelect: =With(
    {
      existingItem: LookUp(colShoppingCart, ProductID = ThisItem.ID)
    },
    If(
      IsBlank(existingItem),
      Collect(colShoppingCart, newItemRecord),
      UpdateIf(colShoppingCart, ProductID = ThisItem.ID, updatedFields)
    )
  );
  
  # Batch collection operations
  OnSelect: =ClearCollect(
    colSelectedItems,
    ForAll(
      Filter(Gallery1.AllItems, CheckBox_Select.Value),
      {
        ItemID: ThisRecord.ID,
        ItemName: ThisRecord.Name,
        SelectedDate: Now(),
        SelectedBy: User().Email
      }
    )
  );
  
  # Collection cleanup strategies
  OnStart: =// Clean up old temporary data
           RemoveIf(
             colTempData, 
             DateDiff(CreatedDate, Now(), Days) > 7
           );
           // Limit collection size
           If(
             CountRows(colAuditLog) > 1000,
             RemoveIf(
               colAuditLog,
               CreatedDate < Last(
                 FirstN(
                   Sort(colAuditLog, CreatedDate, Descending),
                   800
                 )
               ).CreatedDate
             )
           );
```

### Advanced Variable Patterns

#### **Variable Naming Conventions**:
```yaml
Properties:
  # Global variables (var prefix)
  OnStart: =Set(varCurrentUser, User());
           Set(varAppVersion, "2.1.0");
           Set(varEnvironment, "Production");
           Set(varFeatureFlags, { EnableBeta: false });
  
  # Context variables (loc prefix)
  OnVisible: =UpdateContext({
    locSelectedItem: Blank(),
    locEditMode: false,
    locValidationErrors: Table(),
    locFormData: Defaults(Products)
  });
  
  # Collections (col prefix)
  OnStart: =ClearCollect(colUserPreferences, userPreferencesData);
           ClearCollect(colRecentSearches, Table());
           ClearCollect(colShoppingCart, Table());
  
  # Descriptive naming
  OnSelect: =Set(varCustomerSearchResults, searchResults);
           Set(varOrderProcessingStatus, "InProgress");
           Set(varPaymentValidationErrors, validationResults);
           UpdateContext({
             locProductDetailLoading: true,
             locInventoryCheckComplete: false,
             locPriceCalculationResult: calculatedPrice
           });
```

#### **Variable State Management Patterns**:
```yaml
Properties:
  # Application initialization
  OnStart: =// Initialize global application state
           Set(varAppState, {
             Version: "2.1.0",
             Environment: "Production",
             StartTime: Now(),
             SessionID: GUID()
           });
           
           // Initialize user session
           Set(varUserSession, {
             User: User(),
             LoginTime: Now(),
             LastActivity: Now(),
             Permissions: getUserPermissions(User().Email)
           });
           
           // Initialize feature flags
           Set(varFeatures, {
             EnableAdvancedReporting: true,
             ShowBetaFeatures: false,
             EnableOfflineSync: true,
             UseNewNavigationUI: false
           });
           
           // Initialize application preferences
           Set(varPreferences, {
             Theme: "Light",
             Language: "English",
             ItemsPerPage: 25,
             AutoSave: true,
             NotificationsEnabled: true
           });
  
  # Screen state initialization
  OnVisible: =// Reset screen state
            UpdateContext({
              locScreenState: {
                Loading: false,
                Error: "",
                LastRefresh: Blank(),
                HasChanges: false
              },
              locUIState: {
                SelectedTab: "Overview",
                ShowDetails: false,
                SortDirection: "Ascending",
                FilterActive: false
              },
              locDataState: {
                CurrentPage: 1,
                ItemsPerPage: 20,
                TotalItems: 0,
                FilterCriteria: {}
              }
            });
  
  # Form state management
  OnVisible: =UpdateContext({
    locFormState: {
      Mode: "New",  // New, Edit, View
      IsDirty: false,
      IsValid: false,
      ValidationErrors: Table(),
      OriginalData: Blank()
    },
    locFormData: If(
      varEditMode,
      varSelectedRecord,
      Defaults(DataSource)
    )
  });
  
  # Complex workflow state
  OnVisible: =UpdateContext({
    locWorkflowState: {
      CurrentStep: 1,
      TotalSteps: 5,
      StepData: Table(
        { Step: 1, Title: "Basic Info", Complete: false, Valid: false },
        { Step: 2, Title: "Details", Complete: false, Valid: false },
        { Step: 3, Title: "Review", Complete: false, Valid: false },
        { Step: 4, Title: "Approval", Complete: false, Valid: false },
        { Step: 5, Title: "Confirmation", Complete: false, Valid: false }
      ),
      CanProceed: false,
      CanGoBack: false
    }
  });
```

#### **Error Handling and Validation Patterns**:
```yaml
Properties:
  # Comprehensive error handling
  OnSelect: =With(
    {
      validationResult: validateFormData(locFormData)
    },
    If(
      validationResult.IsValid,
      // Success path
      UpdateContext({
        locSubmitting: true,
        locError: ""
      });
      IfError(
        Patch(DataSource, Defaults(DataSource), locFormData),
        // Error handling
        UpdateContext({
          locSubmitting: false,
          locError: "Failed to save: " & FirstError.Message,
          locValidationErrors: Table({
            Field: "General",
            Message: FirstError.Message
          })
        }),
        // Success handling
        UpdateContext({
          locSubmitting: false,
          locSuccess: true,
          locFormState: {
            Mode: "View",
            IsDirty: false,
            IsValid: true
          }
        });
        Set(varLastSaveTime, Now())
      ),
      // Validation failed
      UpdateContext({
        locValidationErrors: validationResult.Errors,
        locError: "Please correct the errors below"
      })
    )
  );
  
  # Defensive variable access
  Text: =If(
    IsBlank(varSelectedCustomer),
    "No customer selected",
    If(
      IsBlank(varSelectedCustomer.CompanyName),
      "Company name not available",
      varSelectedCustomer.CompanyName
    )
  );
  
  # Safe collection operations
  Items: =If(
    IsEmpty(colProducts),
    Table({ Message: "No products available" }),
    colProducts
  );
  
  # Variable validation patterns
  OnChange: =With(
    {
      inputValue: Value(Text),
      isValid: And(
        Not(IsBlank(Text)),
        IsNumeric(Text),
        Value(Text) > 0,
        Value(Text) <= 1000
      )
    },
    UpdateContext({
      locInputValid: isValid,
      locInputError: If(
        isValid,
        "",
        Switch(
          true,
          IsBlank(Text), "Value is required",
          Not(IsNumeric(Text)), "Value must be a number",
          Value(Text) <= 0, "Value must be greater than 0",
          Value(Text) > 1000, "Value cannot exceed 1000",
          "Invalid value"
        )
      ),
      locInputValue: If(isValid, inputValue, 0)
    })
  );
```

#### **Performance and Memory Management**:
```yaml
Properties:
  # Variable cleanup on screen exit
  OnHidden: =// Clear large collections
            Clear(colTempData);
            Clear(colSearchResults);
            
            // Reset context variables
            UpdateContext({
              locLargeDataSet: Blank(),
              locImageCache: Table(),
              locProcessingResults: Table()
            });
            
            // Update global cleanup tracker
            Set(varLastCleanup, Now());
  
  # Lazy loading patterns
  Items: =If(
    IsBlank(colCachedData) Or 
    DateDiff(varLastDataRefresh, Now(), Minutes) > 30,
    
    // Load data and cache
    ClearCollect(
      colCachedData,
      FirstN(DataSource, 100)  // Limit initial load
    );
    Set(varLastDataRefresh, Now());
    colCachedData,
    
    // Return cached data
    colCachedData
  );
  
  # Progressive data loading
  OnSelect: =If(
    locCurrentPage * locItemsPerPage >= CountRows(colDisplayData),
    
    // Load next page
    Collect(
      colDisplayData,
      FirstN(
        Filter(
          DataSource,
          Not(ID in ShowColumns(colDisplayData, "ID").ID)
        ),
        locItemsPerPage
      )
    );
    UpdateContext({ locCurrentPage: locCurrentPage + 1 })
  );
  
  # Memory-efficient operations
  Items: =With(
    {
      filteredData: Filter(
        LargeDataSource,
        Category = varSelectedCategory And
        Active = true
      )
    },
    If(
      CountRows(filteredData) > 1000,
      FirstN(
        Sort(filteredData, LastModified, Descending),
        1000
      ),
      filteredData
    )
  );
```

### Variable Best Practices

#### **Design Principles**:
```yaml
Properties:
  # Prefer formulas over variables when possible
  Text: =TextInput1.Text & " " & TextInput2.Text  // Good - automatic recalculation
  
  # Use variables for state that can't be calculated
  OnSelect: =Set(varLastAction, "ButtonClicked");  // Good - user action tracking
           Set(varClickCount, varClickCount + 1)   // Good - accumulating state
  
  # Clear variable purposes and scoping
  OnStart: =// Global application state
           Set(varCurrentUser, getCurrentUser());
           Set(varAppSettings, loadAppSettings());
           
           // Feature toggles
           Set(varFeatures, loadFeatureFlags());
  
  OnVisible: =// Screen-specific state
            UpdateContext({
              locEditMode: false,
              locSelectedItem: Blank()
            });
  
  # Consistent naming and documentation
  OnStart: =// User session variables
           Set(varCurrentUser, User());      // Current authenticated user
           Set(varUserRole, getUserRole());  // User's role/permissions
           Set(varSessionID, GUID());        // Unique session identifier
           
           // Application state variables  
           Set(varAppMode, "Normal");        // Normal, Maintenance, Debug
           Set(varEnvironment, "Prod");      // Prod, Test, Dev
           Set(varVersion, "2.1.0");         // Application version
```

#### **Debugging and Monitoring**:
```yaml
Properties:
  # Variable debugging patterns
  OnSelect: =// Debug mode variable tracking
           If(
             varDebugMode,
             Collect(
               colDebugLog,
               {
                 Action: "VariableSet",
                 VariableName: "varSelectedCustomer", 
                 Value: varSelectedCustomer.ID,
                 Timestamp: Now(),
                 Screen: varCurrentScreen
               }
             )
           );
  
  # Variable state monitoring
  Text: =If(
    varDebugMode,
    Concatenate(
      "Variables: ",
      "User=", varCurrentUser.FullName, " | ",
      "Role=", varUserRole, " | ",
      "Session=", Left(varSessionID, 8), " | ",
      "Screen=", varCurrentScreen
    ),
    ""
  );
  
  # Variable validation in debug mode
  OnVisible: =If(
    varDebugMode,
    // Validate critical variables exist
    Set(varDebugMessages, 
      Concatenate(
        If(IsBlank(varCurrentUser), "Missing varCurrentUser; ", ""),
        If(IsBlank(varUserRole), "Missing varUserRole; ", ""),
        If(IsBlank(varAppSettings), "Missing varAppSettings; ", "")
      )
    )
  );
  
  # Variable performance monitoring
  OnSelect: =With(
    { startTime: Now() },
    
    // Perform variable operations
    Set(varProcessingResult, complexCalculation());
    
    // Track performance
    Set(varPerformanceLog,
      Patch(varPerformanceLog, 
        Defaults(varPerformanceLog),
        {
          Operation: "ComplexCalculation",
          Duration: DateDiff(startTime, Now(), Milliseconds),
          Timestamp: Now()
        }
      )
    )
  );
```

## Advanced Power Fx Patterns

### Error Handling:

```yaml
Properties:
  # Basic error handling
  Text: =IfError(Value(TextInput1.Text), 0)
  
  # Complex error handling
  OnSelect: =IfError(
    Patch(DataSource, ThisItem, {Status: "Updated"}),
    UpdateContext({LastError: FirstError.Message}),
    UpdateContext({LastError: ""})
  )
```

### Conditional Logic:

```yaml
Properties:
  # Simple conditional
  Text: =If(Score >= 90, "A", If(Score >= 80, "B", "C"))
  
  # Switch statement equivalent
  Text: =Switch(
    Grade,
    "A", "Excellent",
    "B", "Good", 
    "C", "Average",
    "Unknown"
  )
```

### Formula Chaining:

```yaml
Properties:
  # Multiple actions with semicolon
  OnSelect: =
    Set(IsLoading, true);
    ClearCollect(Data, RemoteSource);
    Set(IsLoading, false);
    Navigate(ResultScreen)
```

### Dynamic Formulas:

```yaml
Properties:
  # Dynamic property calculation
  Fill: =ColorValue("#" & HexInput.Text)
  Height: =Parent.Height * (Slider1.Value / 100)
  X: =Switch(
    AlignmentChoice.Selected.Value,
    "Left", 0,
    "Center", (Parent.Width - Self.Width) / 2,
    "Right", Parent.Width - Self.Width
  )
```

### Debugging Techniques:
- Use `Trace()` function for debugging in test scenarios
- Break complex formulas into steps using variables
- Test formulas with sample data before applying to real data sources
- Use `Notify()` to display intermediate values during development

## Schema Compliance

Always validate your YAML against the official schema:
- Use the schema URL for validation: `http://powerapps.com/schemas/pa-yaml/v3.0/pa.schema`
- Ensure all required properties are present
- Follow the exact structure defined in the schema
- Test formulas for Power Fx compliance

## Power Fx YAML Formula Grammar Reference

Power Fx integrates with YAML to provide a standardized way of representing formulas and component definitions in canvas app source files. This exhaustive guide covers YAML formula syntax, grammar rules, component structure, and best practices for editing canvas apps as code.

### Overview

The YAML formula grammar enables:
- **Source Code Management**: Edit canvas apps as structured text files
- **Version Control**: Track changes and collaborate using Git and other SCM tools
- **Automation**: Build CI/CD pipelines and automated testing for canvas apps
- **Code Review**: Review formula changes in familiar text-based environments
- **Template Creation**: Create reusable app structures and component libraries

#### **Core Principles**:
- **Excel Consistency**: Uses Excel's leading `=` convention for formulas
- **YAML Compatibility**: Leverages standard YAML syntax with Power Fx extensions
- **Escape Mechanism**: The `=` prefix prevents YAML from interpreting formula syntax
- **Multi-line Support**: Supports both single-line and multi-line formula expressions
- **Component Structure**: Defines component hierarchy and property bindings

#### **Key Features**:
- **Leading Equals**: All formulas must begin with `=` for consistency and parsing
- **Single-line Formulas**: Compact syntax for simple expressions
- **Multi-line Formulas**: Block scalar notation for complex formulas
- **Component Instances**: YAML object notation with `As` operator for typing
- **Property Definitions**: Input and output properties for component encapsulation

### Leading Equals Sign Requirement

#### **Fundamental Rule - All Formulas Start with =**:
```yaml
Properties:
  # Basic formula expressions
  Visible: =true
  X: =34
  Width: =Parent.Width * 0.8
  Text: ="Hello, World"
  
  # Complex calculations
  Value: =Sum(Filter(Sales, Region = "North"), Amount)
  Color: =If(ThisItem.Status = "Active", Color.Green, Color.Red)
  
  # Multi-line formula with leading equals
  Text: |
    ="Hello, " &
    "World"
  
  # Conditional expressions
  Items: =If(
    IsBlank(SearchText.Text),
    AllProducts,
    Filter(AllProducts, Contains(ProductName, SearchText.Text))
  )
  
  # Variable assignments
  OnSelect: =Set(varCurrentUser, User());
           Set(varIsLoggedIn, true);
           Navigate(DashboardScreen)
  
  # Complex data operations
  OnVisible: =ClearCollect(
    colFilteredData,
    Filter(
      Sales,
      SaleDate >= DateAdd(Today(), -30) And
      Region = varSelectedRegion
    )
  )
```

#### **Why Leading Equals is Required**:
```yaml
Properties:
  # Consistency with Excel
  # Excel cells use = to denote formulas vs static values
  Value: =SUM(A1:A10)  # Formula in Excel
  Value: =Sum(DataTable, Amount)  # Formula in Power Fx
  
  # YAML Escape Mechanism  
  # Without =, YAML would interpret as scalar values
  Time: 1:00     # YAML would parse as 60 seconds
  Time: =1:00    # Power Fx interprets as time literal
  
  # Future Compatibility
  # Allows mixing formulas and static values
  StaticText: "Hello World"        # Static value (no =)
  DynamicText: ="Hello " & User()  # Formula (with =)
  
  # Prevents YAML Type Conversion
  # YAML has implicit typing that conflicts with Power Fx
  Number: 007        # YAML converts to 7
  Number: ="007"     # Power Fx preserves as text "007"
  
  Boolean: true      # YAML boolean
  Boolean: =true     # Power Fx boolean expression
```

#### **Leading Equals in Different Contexts**:
```yaml
Properties:
  # Control properties
  Text: ="Current user: " & User().FullName
  Visible: =varUserRole = "Admin"
  X: =Parent.Width / 2 - Self.Width / 2
  Y: =Header.Y + Header.Height + 10
  
  # Event properties
  OnSelect: =Set(varSelectedItem, ThisItem);
           Navigate(DetailScreen)
  OnVisible: =Refresh(DataSource);
            UpdateContext({ locLoading: false })
  OnChange: =UpdateContext({
    locIsValid: Not(IsBlank(Text))
  })
  
  # Data binding properties  
  Items: =Filter(Products, Category = Gallery1.Selected.Category)
  Default: =LookUp(Users, Email = User().Email).DisplayName
  
  # Component property bindings
  UserName: =varCurrentUser.FullName
  ProfileImage: =varCurrentUser.Photo
  IsAdmin: =varUserRole in ["Admin", "SuperAdmin"]
  
  # Formula-driven styling
  Fill: =If(
    ThisItem.IsSelected,
    RGBA(0, 120, 212, 1),    # Selected blue
    RGBA(255, 255, 255, 1)   # Default white
  )
  
  Color: =Switch(
    ThisItem.Priority,
    "High", Color.Red,
    "Medium", Color.Orange,
    "Low", Color.Green,
    Color.Gray
  )
```

### Single-Line Formulas

#### **Basic Single-Line Syntax**:
```yaml
Properties:
  # Standard format: Name: SPACE = Expression
  Text1: ="Hello, World"
  Text2: ="Hello " & ", " & "World"
  Number1: =34
  Boolean1: =true
  Time1: =1:34
  
  # Simple calculations
  Width: =Parent.Width * 0.8
  Height: =Self.Width * 0.6
  X: =(Parent.Width - Self.Width) / 2
  Y: =Previous.Y + Previous.Height + 10
  
  # Function calls
  Text: =Upper(TextInput1.Text)
  Value: =Sum(DataTable, Amount)
  Visible: =CountRows(FilteredData) > 0
  Color: =ColorValue("#FF5733")
  
  # Variable references
  Default: =varCurrentUser.Email
  Items: =colFilteredProducts
  Selected: =varSelectedItem
  
  # Conditional expressions (single line)
  Text: =If(IsBlank(UserInput.Text), "Enter text", UserInput.Text)
  Fill: =If(ThisItem.IsActive, Color.Green, Color.LightGray)
  Visible: =varUserRole = "Admin" Or varUserRole = "Manager"
```

#### **Single-Line Formula Rules and Restrictions**:
```yaml
Properties:
  # SPACE between colon and equals is REQUIRED
  Text: ="Hello"          # ✓ Correct - space before =
  # Text:="Hello"         # ✗ Error - no space before =
  
  # Pound sign # is NOT ALLOWED (YAML comment interpretation)
  # Text: ="Hello #PowerApps"    # ✗ Error - # treated as comment
  # Use multi-line for # characters
  
  # Colon : is NOT ALLOWED (YAML map interpretation)  
  # Time: ="Meeting at 3:30 PM"  # ✗ Error - : creates new mapping
  # Use multi-line for : characters
  
  # Comments must use Power Fx syntax, not YAML
  Value: =42 // This is a Power Fx comment
  # Value: =42 # This would be a YAML comment and cause errors
  
  # No YAML escaping with quotes or backslashes
  # Use multi-line formulas if escaping is needed
  Text: ="""Quoted text"""     # ✓ Power Fx triple quotes work
  # Text: ="\"Quoted\""         # ✗ YAML escaping not supported
  
  # Long expressions should use multi-line for readability
  SimpleCalc: =A + B * C        # ✓ Good for single line
  # ComplexCalc: =If(IsBlank(A), "", If(IsNumeric(A), Text(A), A)) # Consider multi-line
```

#### **Single-Line Formula Examples by Type**:
```yaml
Properties:
  # Text and string operations
  FullName: =FirstName.Text & " " & LastName.Text
  DisplayText: =Upper(Left(ProductName, 1)) & Lower(Mid(ProductName, 2))
  CleanInput: =Trim(Substitute(UserInput.Text, "  ", " "))
  
  # Numeric calculations
  Total: =Quantity.Value * UnitPrice.Value
  Percentage: =Round((ActualValue / TargetValue) * 100, 1)
  TaxAmount: =Round(Subtotal * TaxRate, 2)
  
  # Date and time operations
  DaysOld: =DateDiff(CreatedDate, Today())
  NextWeek: =DateAdd(Today(), 7, Days)
  FormattedDate: =Text(SelectedDate, "mm/dd/yyyy")
  
  # Boolean logic
  IsValid: =Not(IsBlank(Email.Text)) And IsMatch(Email.Text, Match.Email)
  CanEdit: =varUserRole = "Admin" Or Owner = User().Email
  ShowDetails: =Toggle1.Value And CountRows(DetailData) > 0
  
  # Lookup and data operations
  CustomerName: =LookUp(Customers, ID = ThisItem.CustomerID).CompanyName
  ProductCount: =CountRows(Filter(Products, CategoryID = ThisItem.ID))
  LastOrder: =Max(Filter(Orders, CustomerID = ThisItem.ID), OrderDate)
  
  # Control references
  NextX: =PreviousControl.X + PreviousControl.Width + 20
  CenterY: =(Parent.Height - Self.Height) / 2
  RelativeWidth: =Parent.Width - LeftMargin - RightMargin
```

#### **Performance Considerations for Single-Line Formulas**:
```yaml
Properties:
  # Cache expensive lookups in variables
  # Instead of repeated lookups:
  # Text1: =LookUp(ExpensiveTable, ID = ThisItem.ID).Field1
  # Text2: =LookUp(ExpensiveTable, ID = ThisItem.ID).Field2
  
  # Use variables to cache:
  OnVisible: =Set(varCachedRecord, LookUp(ExpensiveTable, ID = ThisItem.ID))
  Text1: =varCachedRecord.Field1
  Text2: =varCachedRecord.Field2
  
  # Use With for complex single expressions
  Value: =With({calc: Quantity * Price}, calc * (1 + TaxRate))
  
  # Minimize function calls in frequently evaluated properties
  Visible: =varIsVisible  # Better than: CountRows(SomeTable) > 0
  
  # Use delegation-friendly operations when possible
  Items: =Filter(LargeDataSource, Status = "Active")  # Delegable
  # Avoid: Filter(LargeDataSource, Len(Description) > 100)  # Not delegable
```

### Multi-Line Formulas

#### **Basic Multi-Line Syntax with Block Scalars**:
```yaml
Properties:
  # Standard multi-line format using | (pipe)
  Text1: |
    ="Hello, World"
  
  # Complex multi-line expression
  Text2: |
    ="Hello" &
    "," &
    "World"
  
  # Multi-line with preserved trailing newlines (|+)
  Description: |+
    ="Product description" &
    Char(10) &  // Line break
    "Additional details"
  
  # Multi-line with stripped trailing newlines (|-)
  CleanText: |-
    =Trim(
      Substitute(
        Substitute(UserInput.Text, Char(10), " "),
        "  ", " "
      )
    )
  
  # Complex calculation with proper indentation
  TotalCost: |
    =Sum(
      Filter(
        OrderItems,
        OrderID = ThisItem.ID
      ),
      Quantity * UnitPrice
    ) + 
    If(
      ShippingRequired,
      ShippingCost,
      0
    )
```

#### **Advanced Multi-Line Formula Patterns**:
```yaml
Properties:
  # Complex conditional logic
  StatusText: |
    =Switch(
      ThisItem.Status,
      "Draft", "Document is being prepared",
      "Review", "Document is under review by " & ReviewerName,
      "Approved", "Document approved on " & Text(ApprovedDate, "mm/dd/yyyy"),
      "Rejected", "Document rejected: " & RejectionReason,
      "Unknown status"
    )
  
  # Multi-step data processing
  ProcessedData: |
    =AddColumns(
      Filter(
        RawData,
        Not(IsBlank(ImportantField)) And
        IsNumeric(NumericField) And
        DateValue(DateField) >= DateAdd(Today(), -90)
      ),
      "CleanedText", Trim(Lower(TextField)),
      "CalculatedValue", NumericField * Multiplier,
      "CategoryName", LookUp(
        Categories,
        CategoryID = RawData[@CategoryID]
      ).Name,
      "IsRecent", DateValue(DateField) >= DateAdd(Today(), -30)
    )
  
  # Complex validation logic
  ValidationMessage: |
    =If(
      IsBlank(EmailInput.Text),
      "Email address is required",
      If(
        Not(IsMatch(EmailInput.Text, Match.Email)),
        "Please enter a valid email address",
        If(
          Len(EmailInput.Text) > 100,
          "Email address is too long (maximum 100 characters)",
          If(
            Contains(EmailInput.Text, "@tempmail.") Or
            Contains(EmailInput.Text, "@10minutemail."),
            "Temporary email addresses are not allowed",
            "" // No validation errors
          )
        )
      )
    )
  
  # Multi-line event handling
  OnSelect: |
    =// Validate input
    UpdateContext({
      locValidationErrors: Table()
    });
    
    // Check required fields
    If(
      IsBlank(NameInput.Text),
      UpdateContext({
        locValidationErrors: Collect(
          locValidationErrors,
          { Field: "Name", Message: "Name is required" }
        )
      })
    );
    
    // Process if valid
    If(
      CountRows(locValidationErrors) = 0,
      // Save data
      Patch(
        DataSource,
        Defaults(DataSource),
        {
          Name: NameInput.Text,
          Email: EmailInput.Text,
          CreatedDate: Now(),
          CreatedBy: User().Email
        }
      );
      
      // Show success message
      Notify("Record saved successfully", NotificationType.Success);
      
      // Navigate to list
      Navigate(ListScreen, ScreenTransition.SlideLeft),
      
      // Show validation errors
      UpdateContext({
        locShowValidation: true
      })
    )
```

#### **Multi-Line Formula with Comments and Documentation**:
```yaml
Properties:
  # Using Power Fx comments within multi-line formulas
  ComplexCalculation: |
    =// Calculate base amount
    With(
      {
        // Get the base price from lookup table
        basePrice: LookUp(
          PricingTable,
          ProductCode = ThisItem.Code And
          EffectiveDate <= Today()
        ).BasePrice,
        
        // Calculate quantity discount
        qtyDiscount: If(
          Quantity >= 100, 0.15,      // 15% for 100+
          If(Quantity >= 50, 0.10,    // 10% for 50+
            If(Quantity >= 10, 0.05,  // 5% for 10+
              0                       // No discount for <10
            )
          )
        )
      },
      
      // Final calculation with all factors
      basePrice * Quantity * (1 - qtyDiscount) * (1 + TaxRate)
    )
  
  # Multi-line collection operations
  OnVisible: |
    =// Clear any existing temporary data
    Clear(colTempResults);
    
    // Load and process data
    ClearCollect(
      colTempResults,
      ForAll(
        // Get all active products
        Filter(Products, Active = true),
        
        // Add calculated fields
        {
          ProductID: ID,
          ProductName: Name,
          CurrentPrice: Price,
          
          // Calculate market position
          MarketPosition: Switch(
            true,
            Price < Average(Products, Price) * 0.8, "Budget",
            Price > Average(Products, Price) * 1.2, "Premium", 
            "Standard"
          ),
          
          // Get sales data
          RecentSales: CountRows(
            Filter(
              Sales,
              ProductID = ID And
              SaleDate >= DateAdd(Today(), -30)
            )
          ),
          
          // Calculate inventory status
          InventoryStatus: If(
            CurrentStock <= ReorderLevel, "Low",
            If(CurrentStock <= ReorderLevel * 2, "Medium", "Good")
          )
        }
      )
    );
    
    // Update last refresh time
    Set(varLastRefresh, Now())
```

#### **Multi-Line Formula Best Practices**:
```yaml
Properties:
  # Proper indentation (at least one space from parent level)
  WellFormatted: |
    =If(
      IsBlank(InputValue),
      "No input provided",
      Upper(InputValue)
    )
  
  # Use meaningful variable names in With statements
  CalculationWithContext: |
    =With(
      {
        userPreferences: LookUp(UserSettings, UserID = User().Email),
        currentDateTime: Now(),
        businessHours: {
          Start: Time(8, 0, 0),
          End: Time(17, 0, 0)
        }
      },
      
      If(
        Time(currentDateTime) >= businessHours.Start And
        Time(currentDateTime) <= businessHours.End,
        userPreferences.BusinessGreeting,
        userPreferences.AfterHoursGreeting
      )
    )
  
  # Break complex expressions into logical chunks
  StepByStepProcessing: |
    =// Step 1: Validate and clean input
    With(
      { cleanedInput: Trim(Lower(UserInput.Text)) },
      
      // Step 2: Apply business rules
      If(
        Len(cleanedInput) >= 3,
        
        // Step 3: Search and return results
        Filter(
          ProductCatalog,
          Contains(Lower(ProductName), cleanedInput) Or
          Contains(Lower(Description), cleanedInput) Or
          Contains(Lower(Category), cleanedInput)
        ),
        
        // Return empty if input too short
        Table()
      )
    )
  
  # Document complex logic with comments
  BusinessRuleImplementation: |
    =// Implementation of business rule BR-2024-001
    // "Discount calculation based on customer tier and order volume"
    
    With(
      {
        // Get customer tier from lookup
        customerTier: LookUp(
          CustomerTiers,
          CustomerID = ThisItem.CustomerID
        ).TierLevel,
        
        // Calculate order volume
        orderVolume: Sum(OrderItems, Quantity * UnitPrice)
      },
      
      // Apply tiered discount structure
      Switch(
        customerTier,
        "Platinum", orderVolume * 0.20,  // 20% discount
        "Gold", orderVolume * 0.15,      // 15% discount  
        "Silver", orderVolume * 0.10,    // 10% discount
        "Bronze", If(
          orderVolume > 1000,
          orderVolume * 0.05,            // 5% for large orders
          0                              // No discount
        ),
        0  // Default: no discount
      )
    )
```

### Component Instance Definition

#### **Basic Component Instance Syntax**:
```yaml
# Standard component instance format:
# Name As ComponentType.Template:
#   Property: =Expression
#   ...

Properties:
  # Basic control instances
  Label1 As Label:
    Text: ="Hello, World"
    X: =20
    Y: =40
    Fill: =Color.White
  
  TextInput1 As TextInput:
    Default: =""
    X: =Label1.X
    Y: =Label1.Y + Label1.Height + 10
    Width: =300
    Height: =32
  
  Button1 As Button:
    Text: ="Submit"
    X: =TextInput1.X
    Y: =TextInput1.Y + TextInput1.Height + 10
    OnSelect: =SubmitForm(Form1)
  
  # Container controls with nested components
  Gallery1 As Gallery.horizontalGallery:
    Items: =Products
    Fill: =Color.White
    
    # Nested components within gallery
    Label_ProductName As Label:
      Text: =ThisItem.ProductName
      X: =10
      Y: =10
      
    Label_Price As Label:
      Text: =Text(ThisItem.Price, "Currency")
      X: =10
      Y: =Label_ProductName.Y + Label_ProductName.Height + 5
      
    Button_Select As Button:
      Text: ="Select"
      X: =Parent.Width - Self.Width - 10
      Y: =10
      OnSelect: =Set(varSelectedProduct, ThisItem)
```

#### **Component Types and Templates**:
```yaml
Properties:
  # Gallery with different templates
  HorizontalGallery As Gallery.horizontalGallery:
    Items: =Products
    TemplateFill: =Color.White
    TemplateSize: =200
  
  VerticalGallery As Gallery.verticalGallery:
    Items: =Orders
    TemplateFill: =If(ThisItem.Status = "Urgent", Color.LightRed, Color.White)
    TemplateSize: =80
  
  FlexGallery As Gallery.galleryFlexHeight:
    Items: =Comments
    TemplateFill: =Color.LightGray
    
  # Form controls
  EditForm As EditForm:
    DataSource: =Customers
    Item: =Gallery1.Selected
    DefaultMode: =FormMode.Edit
    
  DisplayForm As DisplayForm:
    DataSource: =Products
    Item: =varSelectedProduct
  
  # Input controls with specific configurations
  DatePicker1 As DatePicker:
    DefaultDate: =Today()
    StartYear: =Year(Today()) - 10
    EndYear: =Year(Today()) + 5
    
  Dropdown1 As Dropdown:
    Items: =Categories
    DefaultSelectedItems: =Filter(Categories, IsDefault = true)
    
  ComboBox1 As ComboBox:
    Items: =Countries
    SearchItems: =Filter(Countries, Contains(CountryName, ComboBox1.SearchText))
  
  # Media and visualization controls
  Image1 As Image:
    Image: =varSelectedProduct.ProductImage
    ImagePosition: =ImagePosition.Fit
    
  Chart1 As PieChart:
    Items: =SalesByRegion
    Label: ="Region"
    Value: ="SalesAmount"
    
  PowerBI1 As PowerBITile:
    TileUrl: =varPowerBITileURL
    WorkspaceId: =varWorkspaceID
```

#### **Component Naming with Special Characters**:
```yaml
Properties:
  # Names with spaces - use single quotes around entire left side
  'Product Name Label' As Label:
    Text: =ThisItem.ProductName
    X: =10
    Y: =10
  
  # Alternative: use double quotes for entire left side
  "Customer Email Input" As TextInput:
    Default: =ThisItem.Email
    Format: =TextFormat.Email
  
  # Complex names requiring escaping
  '''User''s Profile Image''' As Image:
    Image: =ThisItem.ProfilePhoto
  
  # Names with special characters
  'Price ($)' As Label:
    Text: =Text(ThisItem.Price, "Currency")
  
  # Programmatically generated component names
  'Dynamic Component 1' As Label:
    Text: ="Generated content"
    Visible: =varShowDynamicComponents
```

#### **Complex Component Hierarchies**:
```yaml
Properties:
  # Container with multiple nested levels
  Container1 As GroupContainer:
    Fill: =Color.LightGray
    
    # Header section
    HeaderContainer As GroupContainer:
      Fill: =Color.White
      Height: =60
      
      Title As Label:
        Text: ="Product Details"
        Font: =Font.'Segoe UI Semibold'
        Size: =18
        X: =20
        Y: =15
        
      CloseButton As Button:
        Text: ="×"
        X: =Parent.Width - Self.Width - 10
        Y: =10
        OnSelect: =Set(varShowDetails, false)
    
    # Content section  
    ContentContainer As GroupContainer:
      Y: =HeaderContainer.Height
      Height: =Parent.Height - HeaderContainer.Height
      
      ProductImage As Image:
        Image: =varSelectedProduct.Image
        X: =20
        Y: =20
        Width: =200
        Height: =200
        
      DetailsContainer As GroupContainer:
        X: =ProductImage.X + ProductImage.Width + 20
        Y: =ProductImage.Y
        Width: =Parent.Width - X - 20
        
        ProductName As Label:
          Text: =varSelectedProduct.Name
          Font: =Font.'Segoe UI Semibold'
          Size: =16
          
        ProductDescription As Label:
          Text: =varSelectedProduct.Description
          Y: =ProductName.Y + ProductName.Height + 10
          Size: =12
          
        PriceLabel As Label:
          Text: =Text(varSelectedProduct.Price, "Currency")
          Y: =ProductDescription.Y + ProductDescription.Height + 10
          Color: =Color.Green
          Font: =Font.'Segoe UI Semibold'
```

#### **Component Property Inheritance and Overrides**:
```yaml
Properties:
  # Base template properties
  BaseButton As Button:
    Fill: =RGBA(0, 120, 212, 1)  # Base blue
    Color: =Color.White
    Font: =Font.'Segoe UI'
    Size: =14
    Height: =40
    
  # Specialized buttons inheriting and overriding
  SaveButton As Button:
    Text: ="Save"
    Fill: =Color.Green           # Override base color
    OnSelect: =SubmitForm(Form1)
    
  CancelButton As Button:
    Text: ="Cancel"
    Fill: =Color.Red             # Override base color
    OnSelect: =ResetForm(Form1); Navigate(ListScreen)
    
  # Dynamic property overrides
  StatusButton As Button:
    Text: =ThisItem.StatusText
    Fill: =Switch(               # Dynamic color based on status
      ThisItem.Status,
      "Active", Color.Green,
      "Pending", Color.Orange,
      "Inactive", Color.Gray,
      Color.LightBlue           # Default color
    )
    OnSelect: =ToggleStatus(ThisItem.ID)
```

### Component Definition and Properties

#### **Simple Component Property Definition**:
```yaml
Properties:
  # Custom component definition
  DateRangePicker As CanvasComponent:
    # Input properties (customizable by app maker)
    DefaultStart: |-
      =// Input property - customizable default for component instance
      Now()
      
    DefaultEnd: |-
      =// Input property - customizable default for component instance  
      DateAdd(Now(), 1, Days)
      
    ShowTime: |-
      =// Input property - boolean flag for time display
      false
      
    # Output properties (calculated, not modifiable by app maker)
    SelectedStart: =DatePicker1.SelectedDate     // Output property
    SelectedEnd: =DatePicker2.SelectedDate       // Output property
    IsValidRange: =SelectedStart <= SelectedEnd  // Output property
    
    # Internal component logic
    DatePicker1 As DatePicker:
      DefaultDate: =DateRangePicker.DefaultStart
      Visible: =true
      
    DatePicker2 As DatePicker:
      DefaultDate: =DateRangePicker.DefaultEnd
      Visible: =true
      
    ValidationLabel As Label:
      Text: =If(
        DateRangePicker.IsValidRange,
        "",
        "End date must be after start date"
      )
      Color: =Color.Red
      Visible: =Not(DateRangePicker.IsValidRange)
```

#### **Advanced Component Property Patterns**:
```yaml
Properties:
  # Complex component with multiple property types
  ProductSearchComponent As CanvasComponent:
    # Input properties for configuration
    DataSource: |-
      =// Input: The table/collection to search
      Products
      
    SearchFields: |-
      =// Input: Fields to search in (comma-separated text)
      "ProductName,Description,Category"
      
    MaxResults: |-
      =// Input: Maximum number of results to return
      50
      
    EnableFilters: |-
      =// Input: Whether to show category filters
      true
      
    # Output properties for data flow
    SearchResults: =Gallery_Results.AllItems
    SelectedItem: =Gallery_Results.Selected
    HasResults: =CountRows(Gallery_Results.AllItems) > 0
    SearchTerm: =TextInput_Search.Text
    
    # Internal component implementation
    TextInput_Search As TextInput:
      HintText: ="Search products..."
      OnChange: =UpdateContext({
        locSearchTerm: Lower(Trim(Text))
      })
      
    Dropdown_Category As Dropdown:
      Items: =Distinct(ProductSearchComponent.DataSource, Category)
      Visible: =ProductSearchComponent.EnableFilters
      
    Gallery_Results As Gallery.verticalGallery:
      Items: =With(
        {
          searchFields: Split(ProductSearchComponent.SearchFields, ","),
          searchTerm: locSearchTerm,
          selectedCategory: Dropdown_Category.Selected.Result
        },
        
        FirstN(
          Filter(
            ProductSearchComponent.DataSource,
            (IsBlank(searchTerm) Or 
             Contains(Lower(ProductName), searchTerm) Or
             Contains(Lower(Description), searchTerm)) And
            (IsBlank(selectedCategory) Or Category = selectedCategory)
          ),
          ProductSearchComponent.MaxResults
        )
      )
```

#### **Component Metadata and Documentation**:
```yaml
Properties:
  # Well-documented component with metadata
  ReusableDataCard As CanvasComponent:
    # === INPUT PROPERTIES ===
    # Configuration properties
    Title: |-
      =// Input: Display title for the card
      // Type: Text
      // Default: "Data Card"
      "Data Card"
      
    DataValue: |-
      =// Input: Primary data value to display
      // Type: Any (Text, Number, Date, etc.)
      // Default: Blank
      Blank()
      
    DataFormat: |-
      =// Input: Format string for data value
      // Type: Text
      // Examples: "Currency", "mm/dd/yyyy", "0.00%"
      // Default: "" (no formatting)
      ""
      
    ShowTrend: |-
      =// Input: Whether to show trend indicator
      // Type: Boolean
      // Default: false
      false
      
    TrendValue: |-
      =// Input: Trend percentage (-100 to 100)
      // Type: Number
      // Default: 0
      0
      
    # Styling properties
    BackgroundColor: |-
      =// Input: Card background color
      // Type: Color
      // Default: White
      Color.White
      
    # === OUTPUT PROPERTIES ===
    # Data flow outputs
    FormattedValue: =If(
      IsBlank(ReusableDataCard.DataFormat),
      Text(ReusableDataCard.DataValue),
      Text(ReusableDataCard.DataValue, ReusableDataCard.DataFormat)
    )
    
    TrendDirection: =If(
      ReusableDataCard.TrendValue > 0, "Up",
      If(ReusableDataCard.TrendValue < 0, "Down", "Flat")
    )
    
    TrendColor: =Switch(
      TrendDirection,
      "Up", Color.Green,
      "Down", Color.Red,
      Color.Gray
    )
    
    # === INTERNAL IMPLEMENTATION ===
    # Component UI structure
    CardContainer As GroupContainer:
      Fill: =ReusableDataCard.BackgroundColor
      BorderColor: =Color.LightGray
      BorderThickness: =1
      
      TitleLabel As Label:
        Text: =ReusableDataCard.Title
        Font: =Font.'Segoe UI Semibold'
        Size: =12
        Color: =Color.DarkGray
        
      ValueLabel As Label:
        Text: =ReusableDataCard.FormattedValue
        Font: =Font.'Segoe UI'
        Size: =24
        Y: =TitleLabel.Y + TitleLabel.Height + 5
        
      TrendIcon As Icon:
        Icon: =Switch(
          ReusableDataCard.TrendDirection,
          "Up", Icon.ChevronUp,
          "Down", Icon.ChevronDown,
          Icon.Remove
        )
        Color: =ReusableDataCard.TrendColor
        Visible: =ReusableDataCard.ShowTrend
        X: =ValueLabel.X + ValueLabel.Width + 10
        Y: =ValueLabel.Y
        
      TrendLabel As Label:
        Text: =Abs(ReusableDataCard.TrendValue) & "%"
        Color: =ReusableDataCard.TrendColor
        Visible: =ReusableDataCard.ShowTrend
        X: =TrendIcon.X + TrendIcon.Width + 5
        Y: =TrendIcon.Y
```

### YAML Compatibility and Constraints

#### **YAML Comments vs Power Fx Comments**:
```yaml
Properties:
  # YAML comments are NOT preserved in source format
  # This comment will be lost during processing
  
  # Use Power Fx comments within formulas instead
  CalculationWithComments: |
    =// This Power Fx comment is preserved
    /* This block comment is also preserved */
    With(
      { baseValue: 100 },
      baseValue * 1.25  // 25% markup
    )
  
  # Power Fx comments in single-line formulas
  Price: =BasePrice * 1.08  // Add 8% tax
  
  # Block comments for complex explanations
  BusinessRule: |
    =/* 
       Business Rule BR-2024-003:
       Calculate customer discount based on:
       1. Customer tier (Bronze/Silver/Gold/Platinum)
       2. Order volume (quantity-based discounts)
       3. Historical loyalty (years as customer)
    */
    With(
      {
        tierDiscount: Switch(
          CustomerTier,
          "Platinum", 0.20,
          "Gold", 0.15,
          "Silver", 0.10,
          "Bronze", 0.05,
          0
        ),
        volumeDiscount: If(OrderQuantity >= 100, 0.10, 0),
        loyaltyDiscount: If(YearsAsCustomer >= 5, 0.05, 0)
      },
      // Apply best single discount (don't stack)
      Max(tierDiscount, volumeDiscount, loyaltyDiscount)
    )
```

#### **Common YAML Parsing Pitfalls and Solutions**:
```yaml
Properties:
  # Problem: # character treated as YAML comment
  # ErrorExample: ="Hello #PowerApps"  # ✗ Causes parsing error
  
  # Solution: Use multi-line format for # characters
  TextWithHash: |
    ="Hello #PowerApps"
  
  # Problem: : character creates new YAML mapping
  # ErrorExample: ="Meeting at 3:30 PM"  # ✗ Causes parsing error
  
  # Solution: Use multi-line format for : characters  
  TimeText: |
    ="Meeting at 3:30 PM"
  
  # Problem: Record syntax conflicts with YAML
  # ErrorExample: ={ a: 1, b: 2 }  # ✗ YAML interprets as nested mapping
  
  # Solution: Use multi-line format for record literals
  RecordValue: |
    ={ a: 1, b: 2, c: 3 }
  
  # Problem: Duplicate property names (YAML allows, we error)
  Text: ="First value"
  # Text: ="Second value"  # ✗ Error - duplicate property name
  
  # Solution: Use unique property names
  Text1: ="First value"
  Text2: ="Second value"
  
  # Problem: YAML type conversion conflicts
  # Number: 007        # YAML converts 007 to 7
  # Boolean: yes       # YAML converts "yes" to true
  
  # Solution: Use formulas to preserve exact values
  Number: ="007"       # Preserves as text "007"
  Boolean: ="yes"      # Preserves as text "yes"
  ExplicitBoolean: =true  # Explicit Power Fx boolean
```

#### **YAML Block Scalar Variations**:
```yaml
Properties:
  # Standard block scalar (|) - preserves line breaks
  StandardBlock: |
    ="Line 1" &
    Char(10) &
    "Line 2"
  
  # Block scalar with preserved trailing newlines (|+)
  PreserveTrailing: |+
    ="Content with" &
    Char(10) &
    "trailing newlines"
  
  # Block scalar with stripped trailing newlines (|-)  
  StripTrailing: |-
    ="Content without" &
    Char(10) &
    "trailing newlines"
  
  # Folded block scalar (>) - converts line breaks to spaces
  # Note: Power Fx expressions should use | variants for clarity
  FoldedBlock: >
    This is a long text that will be
    folded into a single line with
    spaces between the original lines
  
  # Complex multi-line with proper indentation
  ComplexFormula: |
    =// All lines must be indented from the parent level
    With(
      {
        // Nested indentation for readability
        calculatedValue: If(
          IsBlank(InputValue),
          0,
          Value(InputValue)
        ),
        
        // Another calculated variable
        multiplier: If(
          IsPremiumCustomer,
          1.5,
          1.0
        )
      },
      
      // Final result calculation
      calculatedValue * multiplier
    )
```

#### **Error Prevention Strategies**:
```yaml
Properties:
  # Validate YAML structure before Power Fx parsing
  # Use consistent indentation (spaces, not tabs)
  # Proper alignment for nested structures
  
  WellStructuredComponent As Gallery:
    Items: =FilteredData
    TemplateFill: =Color.White
    
    # Nested components properly indented
    ItemLabel As Label:
      Text: =ThisItem.Name
      X: =10
      Y: =10
      
    ItemButton As Button:
      Text: ="Select"
      X: =Parent.Width - Self.Width - 10
      Y: =ItemLabel.Y
      OnSelect: =Set(varSelected, ThisItem)
  
  # Escape problematic characters using multi-line
  ProblematicText: |
    ="Text with: colons and #hashtags"
  
  # Use Power Fx string functions for special characters
  SpecialChars: =Char(35) & "hashtag " & Char(58) & "colon"
  
  # Validate property names are unique within scope
  UniqueProperty1: ="Value 1"
  UniqueProperty2: ="Value 2"
  
  # Use meaningful names to avoid conflicts
  CustomerName: =ThisItem.CustomerName
  ProductName: =ThisItem.ProductName
  # Not: Name: =ThisItem.CustomerName and Name: =ThisItem.ProductName
```

### YAML Formula Grammar Best Practices

#### **Code Organization and Structure**:
```yaml
Properties:
  # Group related properties together
  # === POSITIONING ===
  X: =20
  Y: =HeaderSection.Y + HeaderSection.Height + 10
  Width: =Parent.Width - 40
  Height: =200
  
  # === STYLING ===
  Fill: =Color.White
  BorderColor: =Color.LightGray
  BorderThickness: =1
  
  # === DATA BINDING ===
  Items: =Filter(Products, Category = SelectedCategory)
  Default: =First(FilteredProducts)
  
  # === EVENT HANDLING ===
  OnSelect: =Set(varSelectedItem, ThisItem);
           Navigate(DetailScreen)
  OnVisible: =Refresh(DataSource)
  
  # Use consistent naming conventions
  # Controls: PascalCase (Button1, CustomerGallery)
  # Variables: camelCase with prefix (varCurrentUser, locSelectedItem)
  # Properties: PascalCase (Text, OnSelect, Visible)
```

#### **Performance and Maintainability**:
```yaml
Properties:
  # Cache expensive calculations in variables
  OnStart: |
    =Set(varExpensiveLookup, 
      AddColumns(
        Customers,
        "FullAddress", Address & ", " & City & ", " & State
      )
    )
  
  # Reference cached data instead of recalculating
  Items: =varExpensiveLookup
  
  # Use With for complex single calculations
  DisplayText: |
    =With(
      {
        user: LookUp(Users, Email = User().Email),
        preferences: LookUp(UserPreferences, UserID = user.ID)
      },
      "Welcome " & user.DisplayName & 
      " (" & preferences.PreferredLanguage & ")"
    )
  
  # Break complex formulas into manageable pieces
  ValidationResult: |
    =// Step 1: Basic validation
    With(
      { basicChecks: ValidateBasicFields(FormData) },
      
      // Step 2: Business rule validation
      If(
        basicChecks.IsValid,
        ValidateBusinessRules(FormData),
        basicChecks
      )
    )
  
  # Document complex business logic
  PricingCalculation: |
    =// Implements pricing model v2.1
    // Reference: Business Requirements Document BRD-2024-001
    
    With(
      {
        // Base pricing from catalog
        basePrice: LookUp(PriceCatalog, ProductID = ThisItem.ID).Price,
        
        // Customer-specific adjustments
        customerMultiplier: LookUp(
          CustomerTiers,
          TierLevel = ThisItem.Customer.Tier
        ).PriceMultiplier,
        
        // Volume-based discounts
        volumeDiscount: If(
          ThisItem.Quantity >= 100, 0.15,     // 15% for 100+
          If(ThisItem.Quantity >= 50, 0.10,   // 10% for 50+
            If(ThisItem.Quantity >= 10, 0.05, // 5% for 10+
              0                                // No discount
            )
          )
        )
      },
      
      // Final price calculation
      basePrice * customerMultiplier * (1 - volumeDiscount)
    )
```

#### **Testing and Debugging Support**:
```yaml
Properties:
  # Include debug information in development mode
  DebugInfo: |
    =If(
      varDebugMode,
      "Debug: " & 
      "User=" & varCurrentUser.Email & 
      ", Role=" & varUserRole & 
      ", Screen=" & varCurrentScreen,
      ""
    )
  
  # Validation helpers for troubleshooting
  ValidationDetails: |
    =If(
      varShowValidation,
      Concatenate(
        "Field Errors: ",
        CountRows(Filter(ValidationErrors, Severity = "Error")), 
        ", Warnings: ",
        CountRows(Filter(ValidationErrors, Severity = "Warning"))
      ),
      ""
    )
  
  # Error boundaries for production resilience
  SafeCalculation: |
    =IfError(
      // Primary calculation
      ComplexBusinessLogic(InputData),
      
      // Fallback if calculation fails
      {
        Result: "Calculation Error",
        ErrorMessage: FirstError.Message,
        Timestamp: Now()
      }
    )
  
  # Progressive enhancement patterns
  EnhancedFeature: |
    =If(
      varFeatureFlags.EnableAdvancedCalculations,
      // Enhanced calculation when feature is enabled
      AdvancedCalculationEngine(DataSet),
      
      // Fallback to basic calculation
      BasicCalculationEngine(DataSet)
    )
```

## Additional Resources
- Official Schema: https://raw.githubusercontent.com/microsoft/PowerApps-Tooling/refs/heads/master/schemas/pa-yaml/v3.0/pa.schema.yaml
- Power Fx Documentation: https://learn.microsoft.com/en-us/power-platform/power-fx/formula-reference-canvas-apps
- Power Apps YAML Documentation: https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/power-apps-yaml
- Working with Formulas: https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/working-with-formulas
