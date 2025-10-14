# Sketch Health Dashboard - Implementation Continuation Prompt

## 📋 Current Project State

### ✅ Completed Features (Phase 1 - 75% Complete)

**UI/UX Enhancements:**
- ✅ Onboarding modal with 3-step flow (goal selection, daily goals)
- ✅ Custom workout builder with full CRUD functionality
- ✅ Torn paper effect on all cards using CSS pseudo-elements
- ✅ Confetti celebration on workout completion
- ✅ Animated progress bars with smooth transitions
- ✅ Updated hand-drawn logo (heart + dumbbell + pencil)

**Nutrition Tracking Infrastructure:**
- ✅ Meal categorization UI (Breakfast, Lunch, Dinner, Snacks tabs)
- ✅ Macronutrient display (Protein/Carbs/Fat progress bars)
- ✅ Enhanced Gemini API integration for macro breakdown
- ✅ Data model supporting structured meals object
- ✅ `selectMealType()` and `addMealWithAI()` functions

**Technical Improvements:**
- ✅ All JavaScript indentation errors fixed
- ✅ Meal tab button CSS styling
- ✅ Firestore data structure for meals and macros

### 🔧 Remaining Phase 1 Work (25%)

**Critical Functions Needed:**
1. **renderMeals()** - Display logged meals by category with delete buttons
2. **renderMacros()** - Update macro progress bars and calculate totals
3. **Integration** - Add function calls to `renderTodaysData()` and `init()`
4. **Hand-drawn Icons** - Add SVG doodles for water glass, food items, workout icons
5. **Customizable Theme** - Settings panel for ink color and highlighter color

**File Locations:**
- Meal logging UI: `index.html` lines 662-694
- Macro tracking display: `index.html` lines 717-761
- `addMealWithAI()`: `index.html` lines 1077-1154
- `selectMealType()`: `index.html` lines 1062-1075
- `renderTodaysData()`: `index.html` lines 1166+

---

## 🚀 Phase 2: Progressive Overload & Exercise Tracking

### Overview
Transform workout tracking from simple checkboxes to detailed logging for muscle building. Implement progressive overload tracking, exercise history, and performance analytics.

### Features to Implement

#### 2.1 Detailed Exercise Logging
**Replace Checkbox System:**
- Current: Simple checkbox per exercise
- New: Input fields for each set (weight, reps, RPE/RIR)
- UI: Collapsible set-by-set entry within each exercise card

**Implementation Steps:**
1. Update `renderTodaysData()` workout display
2. Create `logExerciseSet()` function
3. Modify data model to store set-by-set data:
```javascript
workout: {
  exercises: [
    {
      name: "Bench Press",
      sets: [
        {setNumber: 1, weight: 185, reps: 8, rpe: 7},
        {setNumber: 2, weight: 185, reps: 7, rpe: 8},
        {setNumber: 3, weight: 175, reps: 8, rpe: 8}
      ]
    }
  ]
}
```

#### 2.2 Progressive Overload Tracking
**Show Previous Performance:**
- Display last week's numbers when logging today's workout
- Highlight improvements (green) or decreases (yellow)
- Calculate total volume (weight × reps × sets)

**Implementation:**
1. Create `getExerciseHistory()` - Query last 4 weeks of data for exercise
2. Create `calculateProgress()` - Compare current vs previous session
3. Add UI indicators showing "Last time: 185lbs × 8 reps"

#### 2.3 Exercise-Specific History
**Performance Charts:**
- Create modal/page showing exercise progression
- Line chart for max weight over time
- Volume chart (total weight moved)
- Rep PR tracking

**Libraries Needed:**
- Chart.js or lightweight alternative
- Integration: Add "View History" button to each exercise

#### 2.4 Personal Records (PR) Detection
**Auto-detection:**
- Monitor weight × reps for 1RM PRs
- Track volume PRs
- Trigger confetti + special badge on PR

**Implementation:**
1. Create `checkForPR()` function after each set logged
2. Store PRs in `userProfile.personalRecords`
3. Add PR history view

#### 2.5 Integrated Rest Timer
**Workout Flow Enhancement:**
- After logging a set, show "Start Rest Timer" button
- Pre-populated with exercise-appropriate rest (60s accessories, 3-5min compounds)
- Countdown with audio alert

**Integration Point:**
- Modify existing timer modal
- Add `startExerciseRestTimer(exercise, setNumber)` function

---

## 🎮 Phase 3: Gamification & Visualization

### Overview
Add game-like elements and visual progress tracking to increase user engagement and motivation.

### Features to Implement

#### 3.1 Achievement System
**Badge Categories:**
- **Consistency**: Streak-based (3, 7, 14, 30, 60, 100 day streaks)
- **Milestones**: "First PR", "100th Workout", "1000 Total Reps"
- **Performance**: "Bench 225lbs", "Squat 315lbs", "500lb Total"
- **Nutrition**: "7 Day Macro Hit", "30 Days Water Goal"

**Data Structure:**
```javascript
achievements: {
  earned: [
    {id: 'streak_7', name: 'Week Warrior', earnedDate: timestamp, tier: 'bronze'},
    {id: 'bench_225', name: 'Two Plate Club', earnedDate: timestamp, tier: 'gold'}
  ],
  progress: {
    'streak_30': {current: 23, target: 30, description: 'Train 30 days in a row'}
  }
}
```

**Implementation:**
1. Create `checkAchievements()` - Run after each workout/nutrition log
2. Design SVG badges (notepad themed)
3. Achievement showcase page/modal
4. Progress notifications (toast-style)

#### 3.2 Enhanced Streak Tracking
**Multiple Streak Types:**
- Workout streak (current & longest)
- Macro goal streak
- Water intake streak
- Combined "Perfect Day" streak (workout + macros + water)

**Visual Indicators:**
- Flame emoji 🔥 with number
- Progress circle for weekly goal
- Calendar heat map view

#### 3.3 Monthly Calendar View
**Historical Visualization:**
- Calendar grid showing entire month
- Color-coding by workout type:
  - Push: Red
  - Pull: Blue  
  - Legs: Green
  - Cardio: Orange
  - Rest: Gray
- Click day to view details
- Show streak indicators

**Library Option:** 
- FullCalendar.js (or build custom with CSS Grid)

#### 3.4 Progress Charts & Analytics
**Weekly/Monthly Trends:**
- Total weight lifted per week (bar chart)
- Average daily calories (line chart)
- Workout frequency (scatter plot)
- Body metrics over time (if tracking weight)

**Dashboard Cards:**
- "This Week" summary cards
- Comparison to previous week
- Monthly averages

**Implementation:**
1. Create `calculateWeeklyStats()` function
2. Integrate Chart.js or similar
3. Add "Analytics" tab to Historical Log section

---

## ⚙️ Phase 4: Enhanced Data & Settings

### Overview
Add user customization options and expand data tracking capabilities.

### Features to Implement

#### 4.1 Settings Panel
**User Preferences UI:**
- Location: Expand profile dropdown menu
- Add "Settings" option that opens modal/page

**Configurable Options:**
- Daily macro goals (protein/carbs/fat targets)
- Water intake goal (glasses per day)
- Liquid calorie limit
- Weekly workout target
- Theme customization (ink color, highlighter color)
- Notification preferences

**Data Storage:**
```javascript
userProfile: {
  settings: {
    macroGoals: {protein: 150, carbs: 200, fat: 65},
    waterGoal: 8,
    liquidCaloriesGoal: 500,
    weeklyWorkoutGoal: 4,
    theme: {inkColor: '#1E3A8A', highlighterColor: '#60A5FA'}
  }
}
```

#### 4.2 Customizable Goals
**Dynamic Target Updates:**
- Goal input forms with validation
- Real-time progress bar adjustments
- Save to Firestore `userProfile`
- Load goals on app init

**Implementation:**
1. Create `updateUserSettings()` function
2. Modify `renderMacros()` to use custom goals
3. Add settings modal UI

#### 4.3 Workout Templates Library
**Pre-built Routines:**
- Beginner/Intermediate/Advanced programs
- PPL (Push/Pull/Legs) splits
- Full body routines
- Specialization programs

**Custom Template Creation:**
- Save current workout as template
- Template management (edit, delete, duplicate)
- Share templates via export/import

**Data Structure:**
```javascript
templates: {
  predefined: [...],
  custom: [
    {id: 'custom_1', name: 'My Chest Day', exercises: [...]}
  ]
}
```

#### 4.4 Data Export/Import
**User Data Portability:**
- Export all data as JSON
- Import from previous export
- Backup/restore functionality
- Download as CSV for spreadsheet analysis

---

## 🔒 Phase 5: Security & Technical Improvements

### Overview
Address security vulnerabilities and improve code maintainability through modularization.

### Features to Implement

#### 5.1 Secure Gemini API Calls
**Current Issue:**
- API key exposed in client-side code: `index.html` line 1087
- Vulnerable to theft and abuse

**Solution - Firebase Cloud Function:**
```javascript
// functions/estimateNutrition.js
exports.estimateNutrition = functions.https.onCall(async (data, context) => {
  // Verify authentication
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
  }
  
  const {description} = data;
  const apiKey = functions.config().gemini.key; // Stored securely
  
  // Make API call server-side
  const response = await fetch(geminiUrl, {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({...})
  });
  
  return await response.json();
});
```

**Client-Side Update:**
```javascript
// Replace direct API call with Cloud Function call
const estimateNutrition = firebase.functions().httpsCallable('estimateNutrition');
const result = await estimateNutrition({description: desc});
```

**Steps:**
1. Set up Firebase Functions
2. Create nutrition estimation endpoint
3. Store API key as environment variable
4. Update client code to call Cloud Function
5. Remove exposed API key

#### 5.2 Firebase App Check
**Request Verification:**
- Prevent unauthorized access to Firestore
- Validate requests come from your app only
- Block bots and scrapers

**Implementation:**
```javascript
// Initialize App Check
const appCheck = firebase.appCheck();
appCheck.activate('RECAPTCHA_V3_SITE_KEY', true);
```

**Steps:**
1. Register app with Firebase App Check
2. Add reCAPTCHA site key
3. Configure Firestore rules to require App Check
4. Test verification flow

#### 5.3 Code Modularization
**Current Structure:**
- Single 1600+ line HTML file
- All JavaScript inline
- Difficult to maintain and test

**Target Structure:**
```
/src
  /js
    auth.js          - Authentication logic
    ui.js            - DOM rendering and updates
    firestore.js     - Database operations
    workouts.js      - Workout-specific functions
    nutrition.js     - Meal logging and macro tracking
    analytics.js     - Charts and statistics
    achievements.js  - Gamification logic
  /css
    main.css         - Extracted styles
    components.css   - Reusable component styles
  /assets
    /icons           - SVG icons and doodles
```

**Migration Strategy:**
1. Create module structure
2. Extract functions by domain (auth, UI, data)
3. Use ES6 modules with imports
4. Set up build process (optional: Vite/Webpack)
5. Update HTML to reference modules
6. Test thoroughly after refactor

#### 5.4 Error Handling & User Feedback
**Improve Robustness:**
- Try-catch blocks on all async operations
- User-friendly error messages
- Loading states for all async actions
- Offline mode handling
- Retry logic for failed requests

**Toast Notification System:**
```javascript
// Add notification library or create custom
function showToast(message, type = 'info') {
  // type: success, error, warning, info
  // Display non-intrusive notification
}
```

**Implementation Points:**
- Wrap all API calls
- Add loading spinners
- Handle Firestore connection issues
- Validate user input before submission

#### 5.5 Performance Optimization
**Loading Improvements:**
- Lazy load chart libraries
- Debounce input handlers
- Implement virtual scrolling for long lists
- Cache computed values
- Optimize Firestore queries (use indexes)

**Best Practices:**
- Bundle size analysis
- Code splitting
- Image optimization (SVG only currently - good!)
- Minimize re-renders

---

## 📝 Implementation Prompt Template

### For Continuing Phase 1:
```
Continue implementing Phase 1 features for the Sketch Health Dashboard. Current progress is 75% complete.

Remaining work:
1. Add renderMeals() function to display logged meals by category
2. Add renderMacros() function to calculate and update macro progress bars
3. Update renderTodaysData() to call new rendering functions
4. Initialize meal tab selection in init() method
5. Add hand-drawn SVG doodles for water, food, and workout icons
6. Implement theme customization settings

Key file locations:
- Main app code: index.html (lines 800-1600)
- Meal logging: lines 1062-1154
- Render functions: lines 1166+

Technical context:
- Single-file HTML app with inline JavaScript
- Firebase Firestore for data storage
- Gemini AI for nutrition estimation
- Tailwind CSS + custom styles
- Kalam font for hand-written aesthetic
```

### For Starting Phase 2:
```
Implement Phase 2: Progressive Overload & Exercise Tracking for Sketch Health Dashboard.

Current state:
- Phase 1 complete with basic workout tracking via checkboxes
- Need to enhance to detailed set-by-set logging
- Support progressive overload tracking and PR detection

Key features to add:
1. Replace exercise checkboxes with set-by-set input (weight, reps, RPE)
2. Show previous performance when logging workouts
3. Calculate and display volume progression
4. Add exercise-specific history charts
5. Implement PR detection with celebrations
6. Integrate rest timer into workout flow

Technical requirements:
- Modify workout data model in Firestore
- Add Chart.js for exercise history visualization
- Create new functions: logExerciseSet(), getExerciseHistory(), checkForPR()
- Update renderTodaysData() workout display

Reference existing code:
- Workout rendering: index.html lines 1166-1200
- Workout completion: lines 1027-1053
- Timer modal: lines 698-710
```

### For Starting Phase 3:
```
Implement Phase 3: Gamification & Visualization for Sketch Health Dashboard.

Goal: Add achievement system, enhanced streak tracking, monthly calendar view, and progress analytics.

Key features:
1. Achievement/badge system with categories (consistency, milestones, performance)
2. Multiple streak types (workout, macro, water, perfect day)
3. Monthly calendar visualization with workout type color-coding
4. Progress charts (weekly weight, calories, workout frequency)

Technical approach:
- Create achievements.js module with badge logic
- Integrate Chart.js for analytics
- Build calendar view (FullCalendar or custom CSS Grid)
- Add achievement notification system

Data model additions:
- userProfile.achievements: {earned: [], progress: {}}
- userProfile.streaks: {workout: {current, longest}, macros: {}, water: {}}

Reference:
- Current streak display: index.html lines 571-581
- Profile data model: lines 883-900
```

### For Starting Phase 4:
```
Implement Phase 4: Enhanced Data & Settings for Sketch Health Dashboard.

Focus: User customization, settings panel, workout templates, data export.

Key features:
1. Settings modal with all user preferences
2. Customizable macro/water/workout goals
3. Workout template library (predefined + custom)
4. Data export/import functionality

Implementation:
- Create settings modal UI in profile dropdown
- Add updateUserSettings() function
- Create template management system
- Implement JSON export/import

Technical notes:
- Store all settings in userProfile.settings object
- Make all goals dynamic based on user preferences
- Template system similar to workout builder
- Export should include all user data (workouts, meals, progress)
```

### For Starting Phase 5:
```
Implement Phase 5: Security & Technical Improvements for Sketch Health Dashboard.

Critical security fixes and code quality improvements.

Priority tasks:
1. Move Gemini API calls to Firebase Cloud Function (HIGH PRIORITY - security risk)
2. Implement Firebase App Check
3. Refactor codebase into modular structure
4. Add comprehensive error handling
5. Optimize performance

Security focus:
- Remove exposed API key from client code (line 1087)
- Set up Firebase Cloud Functions
- Configure App Check with reCAPTCHA
- Update Firestore security rules

Code quality:
- Extract inline JavaScript to separate modules
- Create auth.js, ui.js, firestore.js, workouts.js, nutrition.js
- Implement proper error boundaries
- Add loading states and user feedback
- Performance profiling and optimization
```

---

## 🧪 Testing Guidelines

### Before Considering Phase Complete:
1. **Functionality**: All features work as described
2. **Data Persistence**: Changes save and load correctly from Firestore
3. **Error Handling**: Graceful handling of edge cases
4. **UI/UX**: Smooth animations, no layout shifts
5. **Mobile Responsive**: Test on mobile viewport
6. **Cross-browser**: Test in Chrome, Firefox, Safari
7. **Performance**: No lag with typical data volumes

### Regression Testing:
- Existing features still work after changes
- No console errors
- No broken links or buttons
- Authentication flow intact
- Data migration (if schema changed)

---

## 📚 Key Technical References

**Current Tech Stack:**
- Firebase SDK v11.6.1
- Tailwind CSS (via CDN)
- Kalam font (Google Fonts)
- Canvas Confetti library
- Gemini 2.5 Flash API

**Important Code Patterns:**
```javascript
// Firestore save pattern
saveData(type) {
  if (type === 'daily') {
    const docRef = doc(this.db, 'artifacts', appId, 'users', this.userId, 'dailyLogs', dateString);
    await setDoc(docRef, this.todayState, {merge: true});
  }
}

// Real-time listener pattern
onSnapshot(docRef, (doc) => {
  this.todayState = doc.data();
  this.renderTodaysData();
});

// Modal control pattern
function openModal() {
  modal.classList.remove('hidden');
  setTimeout(() => modalContent.style.transform = 'scale(1)', 10);
}
```

**Firestore Collections Structure:**
```
/artifacts/{appId}
  /users/{userId}
    /dailyLogs/{YYYY-MM-DD}
    /customWorkouts/{workoutId}
  /userProfiles/{userId}
```

---

## 💡 Development Tips

1. **Incremental Development**: Implement one feature at a time, test thoroughly
2. **Data Migration**: Consider backward compatibility when changing data models
3. **Console Logging**: Add debug logs liberally, remove before production
4. **Git Commits**: Commit after each working feature
5. **Documentation**: Update this guide as you implement features
6. **User Feedback**: Test with real workout scenarios

## 🎯 Success Metrics

**Phase 1 Success:**
- Users can log meals by category with AI
- Macro tracking works and displays correctly
- All UI elements render properly
- Theme customization available

**Phase 2 Success:**
- Detailed workout logging captures all set data
- Progressive overload is trackable
- Exercise history provides actionable insights
- PRs are automatically detected

**Phase 3 Success:**
- Achievement system motivates continued use
- Calendar view provides clear progress visualization
- Analytics dashboard shows meaningful trends
- Streak tracking encourages consistency

**Phase 4 Success:**
- Settings are easily customizable
- Templates reduce friction in workout creation
- Data export allows user control of information
- Goals adapt to individual user needs

**Phase 5 Success:**
- API keys are secure (not exposed)
- App passes security audit
- Code is maintainable and testable
- Performance meets web vitals standards

---

**Last Updated**: Phase 1 - 75% Complete
**Next Priority**: Complete Phase 1 (renderMeals, renderMacros functions)