# GitHub Markdown Diagrams and Tables Guide

## Creating Diagrams

GitHub supports four types of diagram creation in markdown:

### 1. Mermaid Diagrams

Mermaid syntax renders text into visual diagrams, supporting a wide variety of chart and diagram types.

**Syntax**: Use fenced code block with `mermaid` identifier

#### Statistical/Data Visualization Charts

**XY Chart (Line/Bar Charts)**:
````markdown
```mermaid
xychart-beta
    title "Sales Revenue"
    x-axis [Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec]
    y-axis "Revenue (in $)" 4000 --> 11000
    bar [5000, 6000, 7500, 8200, 9500, 10500, 11000, 10200, 9200, 8500, 7000, 6000]
    line [5000, 6000, 7500, 8200, 9500, 10500, 11000, 10200, 9200, 8500, 7000, 6000]
```
````

**⚠️ Important XY Chart Restrictions:**
- All x-axis labels must be wrapped in quotes, especially when using CJK characters: `x-axis ["标签1", "标签2"]`
- Decimal numbers in data arrays may cause parsing errors. Use integers when possible: `bar [1, 2, 3]` instead of `bar [1.2, 2.4, 3.6]`
- If decimals are necessary, ensure proper syntax validation

**Example with CJK characters:**
````markdown
```mermaid
xychart-beta
    title "主要股灾恢复至峰值前水平所需时间（年）"
    x-axis ["2020新冠", "1962肯尼迪", "1987黑一", "1990储贷", "1970股灾", "1966信贷", "1946战后", "1937衰退", "2008金融", "1973石油", "2000互联网", "1929大萧条"]
    y-axis "恢复年数" 0 --> 25
    bar [1, 1, 2, 2, 2, 2, 3, 4, 6, 6, 15, 25]
```
````

**Pie Chart**:
````markdown
```mermaid
pie title Pets adopted by volunteers
    "Dogs" : 386
    "Cats" : 85
    "Rats" : 15
```
````

**Quadrant Chart**:
````markdown
```mermaid
quadrantChart
    title Reach and influence
    x-axis Low Reach --> High Reach
    y-axis Low Influence --> High Influence
    quadrant-1 We should expand
    quadrant-2 Need to promote
    quadrant-3 Re-evaluate
    quadrant-4 May be improved
    Campaign A: [0.3, 0.6]
    Campaign B: [0.45, 0.23]
    Campaign C: [0.57, 0.69]
```
````

**Sankey Diagram**:
````markdown
```mermaid
sankey-beta
    Agricultural 'waste',Bio-conversion,124.729
    Bio-conversion,Liquid,0.597
    Bio-conversion,Losses,26.862
    Bio-conversion,Solid,280.322
    Bio-conversion,Gas,81.144
```
````

**Radar Chart**:
````markdown
```mermaid
radar
    title Skills Assessment
    JavaScript : 80
    React : 75
    Node.js : 70
    Python : 65
    SQL : 85
    Docker : 60
```
````

**Treemap**:
````markdown
```mermaid
treemap
    title Project Structure
    Frontend
        React: 45
        Vue: 30
        Angular: 25
    Backend
        Node.js: 40
        Python: 35
        Java: 25
```
````

#### Flow and Process Diagrams

**Flow Chart**:
````markdown
```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```
````

**Sequence Diagram**:
````markdown
```mermaid
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->>John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail!
    John-->>Alice: Great!
    John->>Bob: How about you?
    Bob-->>John: Jolly good!
```
````

**State Diagram**:
````markdown
```mermaid
stateDiagram-v2
    [*] --> Still
    Still --> [*]
    Still --> Moving
    Moving --> Still
    Moving --> Crash
    Crash --> [*]
```
````

#### Project Management Diagrams

**Gantt Chart**:
````markdown
```mermaid
gantt
    title A Gantt Diagram
    dateFormat  YYYY-MM-DD
    section Section
    A task           :a1, 2014-01-01, 30d
    Another task     :after a1  , 20d
    section Another
    Task in sec      :2014-01-12  , 12d
    another task      : 24d
```
````

**Timeline**:
````markdown
```mermaid
timeline
    title History of Social Media Platform
    2002 : LinkedIn
    2004 : Facebook
         : Google
    2005 : Youtube
    2006 : Twitter
```
````

**User Journey**:
````markdown
```mermaid
journey
    title My working day
    section Go to work
      Make tea: 5: Me
      Go upstairs: 3: Me
      Do work: 1: Me, Cat
    section Go home
      Go downstairs: 5: Me
      Sit down: 5: Me
```
````

#### Technical Diagrams

**Class Diagram**:
````markdown
```mermaid
classDiagram
    Animal <|-- Duck
    Animal <|-- Fish
    Animal <|-- Zebra
    Animal : +int age
    Animal : +String gender
    Animal: +isMammal()
    Animal: +mate()
```
````

**Entity Relationship Diagram**:
````markdown
```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ LINE-ITEM : contains
    CUSTOMER }|..|{ DELIVERY-ADDRESS : uses
```
````

**Mindmap**:
````markdown
```mermaid
mindmap
  root((mindmap))
    Origins
      Long history
      ::icon(fa fa-book)
      Popularisation
        British popular psychology author Tony Buzan
    Research
      On effectiveness<br/>and features
      On Automatic creation
        Uses
            Creative techniques
            Strategic planning
            Argument mapping
```
````

### 2. GeoJSON Maps

Create interactive maps by specifying geographic coordinates to define geographic data.

**Syntax**: Use fenced code block with `geojson` identifier

**Example**:
````markdown
```geojson
{
  "type": "Polygon",
  "coordinates": [
    [
      [-90, 30],
      [-90, 35],
      [-85, 35],
      [-85, 30],
      [-90, 30]
    ]
  ]
}
```
````

### 3. TopoJSON Maps

Similar to GeoJSON, but with more compact representation. Supports points, lines, and polygon geometries.

**Syntax**: Use fenced code block with `topojson` identifier

### 4. ASCII STL 3D Models

Create interactive 3D models directly in markdown.

**Syntax**: Use fenced code block with `stl` identifier

**Available in**:
- GitHub Issues
- GitHub Discussions
- Pull requests
- Wikis
- Markdown files

**Common Rendering Issues and Solutions:**

1. **CJK Character Issues**: Always wrap text containing Chinese, Japanese, or Korean characters in quotes
   - ✅ Correct: `x-axis ["中文", "日本語", "한국어"]`
   - ❌ Incorrect: `x-axis [中文, 日本語, 한국어]`

2. **Decimal Number Parsing**: GitHub's Mermaid parser may have issues with decimal numbers
   - ✅ Safer: Use integers when possible
   - ⚠️ If decimals needed: Test thoroughly and consider rounding

3. **Syntax Validation**: Always preview diagrams before publishing
   - Use GitHub's preview feature
   - Check for parsing errors in the rendered output

**Note**: You may observe errors if you run a third-party Mermaid plugin when using Mermaid syntax on GitHub.

## Creating Tables

### Basic Syntax

Use pipes `|` and hyphens `-` to create tables:

```markdown
| First Header | Second Header |
| ------------ | ------------- |
| Content Cell | Content Cell  |
```

### Formatting Requirements

- Include a blank line before the table for proper rendering
- Pipes at table ends are optional
- Must have at least three hyphens in header row
- Cells can vary in width

### Text Alignment

Use colons to control text alignment:

```markdown
| Left Align | Center Align | Right Align |
| :--------- | :----------: | ----------: |
| Left       | Center       | Right       |
| Text       | Text         | Text        |
```

- Left alignment: `:---`
- Center alignment: `:---:`
- Right alignment: `---:`

### Cell Content Formatting

Table cells support inline formatting:

```markdown
| Feature | Syntax        | Example                 |
| ------- | ------------- | ----------------------- |
| Code    | \`code\`      | `printf("Hello");`      |
| Links   | [text](url)   | [GitHub](github.com)    |
| Bold    | **text**      | **Important content**   |
| Italic  | *text*        | *Emphasized content*    |
```

### Special Characters

To include a pipe `|` in a cell, escape it with a backslash:

```markdown
| Character | Escape Method |
| --------- | ------------- |
| Pipe      | \|            |
```

### Editing Tips

When editing tables frequently, it's recommended to use a fixed-width font for proper alignment.

---

*Reference: GitHub Flavored Markdown Spec*