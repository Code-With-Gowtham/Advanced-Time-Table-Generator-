# Advanced Timetable Generator

A sophisticated desktop application built with Electron, React, and Node.js for generating optimized timetables across multiple classes using constraint-based scheduling algorithms.

## Features

### Core Functionality
- **Multi-Class Timetable Generation**: Generate timetables for multiple classes simultaneously
- **Constraint-Based Scheduling**: Advanced algorithm respecting all hard and soft constraints
- **Teacher Clash Prevention**: Global availability matrix prevents scheduling conflicts
- **Priority-Based Placement**: HIGH priority subjects get optimal time slots
- **Teacher Preferences**: Respects teacher day and time slot preferences
- **Editable Timetables**: Manual editing with real-time conflict validation
- **Workload Management**: Comprehensive teacher workload tracking and visualization
- **Export Functionality**: Export to PDF and Excel formats

### Technical Architecture

#### 1. Data Models

**Global Settings**
- Number of day orders (e.g., 5)
- Hours per day (e.g., 6)
- Break hours configuration

**Classes**
- Class name (e.g., First Year, Second Year, Third Year , Final Year)
- Associated subjects

**Subjects**
- Subject name
- Class association
- Weekly required hours
- Priority level (HIGH/MEDIUM/LOW)

**Teachers**
- Teacher name
- Maximum hours per day (hard constraint)
- Preferred day orders (soft constraint)
- Preferred time slots (morning/afternoon) (soft constraint)

**Allocations**
- Subject-to-teacher mapping
- One teacher per subject
- One teacher can handle multiple subjects

**Timetable**
- Class × Day Order × Hour grid
- Subject and teacher assignments
- Conflict tracking

#### 2. Global Clash-Prevention Strategy

The application uses a **Teacher Availability Matrix**:

```
teacherAvailability[teacherId][dayOrder][hour] = true/false
```

**Process:**
1. Initialize all slots as available (true)
2. Mark break hours as unavailable (false)
3. When assigning a slot:
   - Check if teacher is available
   - Mark slot as unavailable after assignment
4. Prevents double-booking across all classes

**Teacher Daily Hour Counter:**
```
teacherDailyHours[teacherId][dayOrder] = hourCount
```

Ensures teachers don't exceed their daily maximum hours.

#### 3. Preference Scoring Mechanism

Each potential slot is scored based on multiple factors:

```javascript
score = 0

// Teacher preferred day orders (+50 points)
if (teacher prefers this day) score += 50

// Teacher preferred time slots (+30-40 points)
if (morning preference && morning slot) score += 30
if (afternoon preference && afternoon slot) score += 30
if (specific hour match) score += 40

// Priority-based scoring (+20 points)
if (HIGH priority && morning slot) score += 20

// Distribution bonus (+10 points)
if (subject not yet scheduled on this day) score += 10

// Penalties (-10 points)
if (consecutive slot for same subject) score -= 10
```

The algorithm selects the slot with the highest score.

#### 4. Priority-Based Subject Placement

**Scheduling Order:**
1. Sort allocations by priority: HIGH → MEDIUM → LOW
2. Within same priority, sort by weekly hours (descending)
3. Schedule HIGH priority subjects first to get best slots
4. LOW priority subjects fill remaining slots

**Benefits:**
- Core subjects get morning slots
- Lab subjects get optimal time blocks
- Less critical subjects adapt to remaining availability

#### 5. Multi-Class Timetable Generation Algorithm

**Algorithm Flow:**

```
1. INITIALIZATION
   - Load all classes, subjects, teachers, allocations
   - Create teacher availability matrix (global)
   - Initialize teacher daily hour counters
   - Create empty timetable structure for all classes

2. PRIORITIZATION
   - Sort allocations by priority (HIGH → MEDIUM → LOW)
   - Within priority, sort by weekly hours (descending)

3. SCHEDULING LOOP
   For each allocation (in priority order):
     For each required hour of that subject:
       a. FIND BEST SLOT
          - Iterate through all day orders and hours
          - Skip break hours
          - Check hard constraints:
            * Class slot available?
            * Teacher available? (no clash)
            * Teacher under daily limit?
          - Calculate preference score for valid slots
          - Select slot with highest score
       
       b. ASSIGN SLOT
          - Update timetable
          - Mark teacher as busy
          - Increment teacher daily hours
          - Decrement subject remaining hours

4. VALIDATION
   - Ensure all subjects scheduled
   - Verify no teacher clashes
   - Check all daily limits respected

5. OUTPUT
   - Convert to database format
   - Calculate statistics
   - Return timetable with metrics
```

**Key Features:**
- **Slot-by-slot generation**: Evaluates each slot globally
- **Backtracking support**: Returns error if constraints cannot be satisfied
- **Optimization**: Maximizes preference satisfaction while respecting constraints

#### 6. Editable Timetable Validation Logic

**Manual Edit Process:**

1. **User clicks cell** to edit
2. **Display available options**:
   - List all subject-teacher allocations for that class
   - Option to clear slot

3. **Validate change**:
   ```javascript
   validateSlotChange(classId, dayOrder, hour, newTeacherId):
     // Check teacher clash
     if (teacher busy in another class at this time):
       return { valid: false, reason: "Teacher clash" }
     
     // Check daily limit
     if (teacher exceeds max hours for this day):
       return { valid: false, reason: "Daily limit exceeded" }
     
     return { valid: true }
   ```

4. **Apply or reject**:
   - If valid: Update database and UI
   - If invalid: Show error message with reason

**Conflict Highlighting:**
- Red border for invalid changes
- Tooltip shows reason (clash/limit)
- Confirmation dialog for override (future feature)

#### 7. Electron Setup and Build Process

**Project Structure:**
```
timetable-generator/
├── main.js                 # Electron main process
├── preload.js             # Secure IPC bridge
├── package.json           # Dependencies and scripts
├── webpack.config.js      # React bundler config
├── database/
│   └── DatabaseService.js # SQLite database layer
├── core/
│   └── TimetableGenerator.js # Core algorithm
├── services/
│   └── ExportService.js   # PDF/Excel export
├── src/
│   ├── index.js          # React entry point
│   ├── components/       # React components
│   └── styles/           # CSS stylesheets
└── renderer/
    ├── index.html        # HTML template
    └── bundle.js         # Compiled React app
```

**Security Features:**
- Context isolation enabled
- Node integration disabled in renderer
- Secure IPC communication via preload script
- Content Security Policy

**Build Commands:**
```bash
# Install dependencies
npm install

# Development mode (with DevTools)
npm run dev

# Production mode
npm start

# Build Windows executable
npm run build:win
```

**IPC Communication Pattern:**
```javascript
// Renderer (React)
const result = await window.api.generateTimetable()

// Preload (Bridge)
contextBridge.exposeInMainWorld('api', {
  generateTimetable: () => ipcRenderer.invoke('generate-timetable')
})

// Main (Electron)
ipcMain.handle('generate-timetable', async () => {
  // Process and return result
})
```

## Installation

### Prerequisites
- Node.js 18+ 
- npm or yarn

### Setup

1. Clone the repository:
```bash
git clone https://github.com/ASWINNIDESH/Advanced Time Table Generator.git
cd Advanced Time Table Generator
```

2. Install dependencies:
```bash
npm install
```

3. Build the React application:
```bash
npx webpack
```

4. Run the application:
```bash
npm start
```

For development with DevTools:
```bash
npm run dev
```

## Usage Guide

### Step 1: Configure Global Settings
- Set number of day orders (e.g., 5)
- Set hours per day (e.g., 6)
- Optionally specify break hours (e.g., 3,4)

### Step 2: Add Classes
- Add classes like "First Year", "Second Year", "Third Year"

### Step 3: Add Subjects
- For each class, add subjects
- Specify weekly required hours
- Set priority level (HIGH for core/lab subjects)

### Step 4: Add Teachers
- Add teacher names
- Set maximum hours per day (hard limit)
- Optionally specify preferred days and time slots

### Step 5: Create Allocations
- Map each subject to a teacher
- One teacher can handle multiple subjects

### Step 6: Generate Timetable
- Click "Generate Timetable"
- Algorithm will create optimized schedule
- View statistics (utilization, preference match)

### Step 7: Edit and Export
- Enable edit mode to manually adjust slots
- Changes are validated in real-time
- Export to PDF or Excel

## Algorithm Highlights

### Constraint Satisfaction
- **Hard Constraints** (must be satisfied):
  - No teacher clashes
  - Teacher daily hour limits
  - Exact subject coverage

- **Soft Constraints** (optimized):
  - Teacher preferences
  - Subject priorities
  - Distribution rules

### Performance
- Efficient slot selection using scoring
- Global availability tracking prevents conflicts
- Single-pass generation with backtracking support

### Quality Metrics
- Utilization rate: Percentage of filled slots
- Preference match rate: Percentage of slots matching teacher preferences
- Zero clashes guaranteed

## Technology Stack

- **Frontend**: React 18
- **Backend**: Node.js
- **Desktop Framework**: Electron 28
- **Database**: SQLite (better-sqlite3)
- **Export**: jsPDF, xlsx
- **Build**: Webpack, Babel
- **Packaging**: electron-builder

## License

MIT

## Author

Developed as an advanced constraint-based scheduling system for educational institutions.
