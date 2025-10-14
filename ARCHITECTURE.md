# Sketch Health - System Architecture

## System Overview

```mermaid
graph TB
    subgraph "Client Layer"
        UI[User Interface<br/>HTML + Tailwind CSS]
        App[Application State<br/>JavaScript]
    end
    
    subgraph "Firebase Services"
        Auth[Firebase Auth<br/>Google OAuth]
        FS[Firestore Database<br/>Real-time Sync]
        CF[Cloud Functions<br/>Secure API]
    end
    
    subgraph "External APIs"
        Gemini[Google Gemini AI<br/>Nutrition Analysis]
    end
    
    UI --> App
    App --> Auth
    App --> FS
    App --> CF
    CF --> Gemini
    FS -.Real-time Updates.-> App
```

## Data Flow Architecture

### Current State (Pre-Enhancement)

```mermaid
graph LR
    User[User] --> Browser[Browser]
    Browser --> App[App Object]
    App --> Firebase[Firebase Auth/Firestore]
    App --> GeminiAPI[Gemini API Direct]
    
    style GeminiAPI fill:#ff6b6b
```

### Target State (Post-Enhancement)

```mermaid
graph LR
    User[User] --> Browser[Browser]
    Browser --> Modules[Modular JS]
    Modules --> Firebase[Firebase Services]
    Modules --> CloudFn[Cloud Functions]
    CloudFn --> Gemini[Gemini API Secure]
    
    style CloudFn fill:#51cf66
    style Gemini fill:#51cf66
```

## Component Architecture

### Phase 1: Core Components

```mermaid
graph TB
    subgraph "Authentication"
        A1[Sign In/Up]
        A2[User Profile]
        A3[Onboarding]
    end
    
    subgraph "Workout Management"
        W1[Workout Selector]
        W2[Custom Builder]
        W3[Exercise Logger]
        W4[Timer]
    end
    
    subgraph "Nutrition Tracking"
        N1[Meal Logger]
        N2[Macro Tracker]
        N3[AI Analysis]
        N4[Water/Liquid Cal]
    end
    
    subgraph "Visualization"
        V1[Progress Bars]
        V2[Streak Display]
        V3[Historical Log]
        V4[Charts]
    end
    
    A3 --> W1
    A3 --> N1
    W2 --> W1
    N3 --> N2
    W3 --> V1
    N2 --> V1
```

## Database Schema Evolution

### Before Enhancements

```mermaid
erDiagram
    USER ||--o{ DAILY_LOG : has
    USER ||--|| USER_PROFILE : has
    
    DAILY_LOG {
        date date PK
        liquidCalories number
        waterIntake number
        excessCalories number
        todaysWorkouts array
    }
    
    USER_PROFILE {
        userId string PK
        workoutStreak number
        lastWorkoutDate string
    }
```

### After Phase 1 Enhancements

```mermaid
erDiagram
    USER ||--o{ DAILY_LOG : has
    USER ||--|| USER_PROFILE : has
    USER ||--o{ CUSTOM_WORKOUT : creates
    USER ||--o{ EXERCISE_HISTORY : tracks
    USER ||--|| ACHIEVEMENTS : earns
    
    DAILY_LOG {
        date date PK
        liquidCalories number
        waterIntake number
        protein number
        carbs number
        fat number
        meals object
        todaysWorkouts array
    }
    
    USER_PROFILE {
        userId string PK
        workoutStreak number
        lastWorkoutDate string
        onboardingCompleted boolean
        primaryGoal string
        waterGoal number
        macroGoals object
        themePreferences object
    }
    
    CUSTOM_WORKOUT {
        workoutId string PK
        name string
        duration number
        exercises array
        createdAt timestamp
    }
    
    EXERCISE_HISTORY {
        exerciseName string PK
        history array
        personalRecords object
    }
    
    ACHIEVEMENTS {
        userId string PK
        unlockedAchievements array
        streaks object
    }
```

## Feature Dependency Map

```mermaid
graph TB
    subgraph "Foundation"
        F1[Firebase Setup]
        F2[Authentication]
        F3[Basic UI]
    end
    
    subgraph "Phase 1 Dependencies"
        P1A[Onboarding] --> F2
        P1B[Custom Workouts] --> F3
        P1C[Macro Tracking] --> F3
        P1D[Visual Enhancements] --> F3
    end
    
    subgraph "Phase 2 Dependencies"
        P2A[Exercise Logging] --> P1B
        P2B[Exercise History] --> P2A
        P2C[PR Tracking] --> P2B
    end
    
    subgraph "Phase 3 Dependencies"
        P3A[Achievements] --> P2C
        P3A --> P1C
        P3B[Calendar View] --> P2B
        P3C[Charts] --> P2B
    end
    
    subgraph "Phase 4 Dependencies"
        P4A[Settings Panel] --> P1A
        P4B[Custom Goals] --> P4A
    end
    
    subgraph "Phase 5 Dependencies"
        P5A[Cloud Functions] --> P1C
        P5B[Refactoring] --> P5A
        P5C[Security] --> P5B
    end
    
    F1 --> F2
    F2 --> F3
```

## User Journey Map

### New User Flow

```mermaid
journey
    title New User Onboarding Experience
    section Landing
        See landing page: 5: User
        Read benefits: 4: User
        Click Sign Up: 5: User
    section Onboarding
        Sign in with Google: 4: User
        Choose fitness goal: 5: User
        Set daily goals: 4: User
        Complete onboarding: 5: User
    section First Use
        View dashboard: 5: User
        Add first workout: 5: User
        Log first meal: 4: User
        Track water: 4: User
        Complete workout: 5: User
        See confetti: 5: User
```

### Returning User Flow

```mermaid
journey
    title Returning User Daily Flow
    section Morning
        Open app: 5: User
        Log breakfast: 5: User
        Add water: 4: User
    section Afternoon
        Log lunch: 5: User
        Start workout: 5: User
        Log exercise sets: 4: User
        Use rest timer: 5: User
        Complete workout: 5: User
        Celebrate achievement: 5: User
    section Evening
        Log dinner: 5: User
        Review daily progress: 5: User
        Check streak: 5: User
        View history: 4: User
```

## API Integration Pattern

### Nutrition Analysis Flow

```mermaid
sequenceDiagram
    participant U as User
    participant UI as UI Layer
    participant CF as Cloud Function
    participant G as Gemini API
    participant DB as Firestore
    
    U->>UI: Enter food description
    UI->>UI: Show loading state
    UI->>CF: Call analyzeFoodNutrition()
    CF->>CF: Validate auth
    CF->>G: Request nutrition data
    G-->>CF: Return macros
    CF-->>UI: Return structured data
    UI->>UI: Display results
    U->>UI: Confirm and log
    UI->>DB: Save meal entry
    DB-->>UI: Confirm save
    UI->>UI: Update UI
```

### Exercise History Flow

```mermaid
sequenceDiagram
    participant U as User
    participant UI as UI Layer
    participant DB as Firestore
    participant C as Chart Library
    
    U->>UI: Complete exercise set
    UI->>DB: Save set data
    DB-->>UI: Confirm save
    UI->>UI: Check for PR
    alt New Personal Record
        UI->>UI: Show celebration
        UI->>DB: Update PR
    end
    U->>UI: View history
    UI->>DB: Fetch exercise history
    DB-->>UI: Return historical data
    UI->>C: Generate chart
    C-->>UI: Render visualization
    UI->>U: Display history
```

## State Management Strategy

```mermaid
graph TB
    subgraph "Application State"
        LocalState[Local State<br/>Immediate UI]
        TodayState[Today's Data<br/>Current Session]
        UserProfile[User Profile<br/>Persistent Settings]
        Cache[Local Cache<br/>Offline Support]
    end
    
    subgraph "Firestore"
        DailyDocs[Daily Logs]
        ProfileDoc[Profile]
        CustomDocs[Custom Workouts]
        HistoryDocs[Exercise History]
    end
    
    LocalState --> TodayState
    TodayState --> DailyDocs
    UserProfile --> ProfileDoc
    CustomDocs --> Cache
    HistoryDocs --> Cache
    
    DailyDocs -.Real-time.-> TodayState
    ProfileDoc -.Real-time.-> UserProfile
```

## Security Architecture

### Current Issues (To Be Fixed in Phase 5)

```mermaid
graph LR
    Client[Client Code] -->|Exposed API Key| Gemini[Gemini API]
    Client -->|Direct Access| Firebase[Firebase]
    
    style Client fill:#ff6b6b
    style Gemini fill:#ff6b6b
```

### Target Security Model

```mermaid
graph TB
    subgraph "Client Layer"
        Browser[Browser]
    end
    
    subgraph "Security Layer"
        AppCheck[Firebase App Check<br/>Bot Protection]
        Auth[Firebase Auth<br/>User Identity]
    end
    
    subgraph "Backend Layer"
        CF[Cloud Functions<br/>API Keys Hidden]
    end
    
    subgraph "Data Layer"
        Rules[Firestore Rules<br/>Access Control]
        DB[(Firestore)]
    end
    
    Browser --> AppCheck
    AppCheck --> Auth
    Auth --> CF
    Auth --> Rules
    Rules --> DB
    CF --> External[External APIs]
    
    style AppCheck fill:#51cf66
    style Auth fill:#51cf66
    style Rules fill:#51cf66
```

## Performance Optimization Strategy

```mermaid
graph TB
    subgraph "Loading Optimization"
        L1[Lazy Load<br/>Exercise History]
        L2[Pagination<br/>Historical Logs]
        L3[Image Optimization<br/>Doodles/Icons]
    end
    
    subgraph "Data Optimization"
        D1[Firestore Indexes]
        D2[Query Limits]
        D3[Local Caching]
        D4[Offline Support]
    end
    
    subgraph "Rendering Optimization"
        R1[Virtual Scrolling]
        R2[Debounced Updates]
        R3[CSS Animations]
    end
    
    L1 --> D3
    L2 --> D2
    D1 --> D2
    D3 --> D4
    R2 --> D3
```

## Module Structure (Phase 5)

```mermaid
graph TB
    subgraph "Core Modules"
        Main[main.js<br/>App Init]
        Auth[auth.js<br/>Authentication]
        UI[ui.js<br/>DOM Updates]
    end
    
    subgraph "Feature Modules"
        Workouts[workouts.js<br/>Workout Management]
        Nutrition[nutrition.js<br/>Nutrition Tracking]
        Exercises[exercises.js<br/>Exercise Logging]
        Achievements[achievements.js<br/>Gamification]
    end
    
    subgraph "Utility Modules"
        Firestore[firestore.js<br/>Database Ops]
        Theme[theme.js<br/>Theming]
        Utils[utils.js<br/>Helpers]
    end
    
    Main --> Auth
    Main --> UI
    Main --> Workouts
    Main --> Nutrition
    Workouts --> Firestore
    Nutrition --> Firestore
    Exercises --> Firestore
    UI --> Theme
    Auth --> Firestore
```

## Testing Strategy

```mermaid
graph TB
    subgraph "Unit Tests"
        U1[State Management]
        U2[Data Transformations]
        U3[Calculations]
    end
    
    subgraph "Integration Tests"
        I1[Firebase Operations]
        I2[API Calls]
        I3[Authentication Flow]
    end
    
    subgraph "E2E Tests"
        E1[User Onboarding]
        E2[Workout Logging]
        E3[Nutrition Tracking]
    end
    
    subgraph "Manual Tests"
        M1[UI/UX Testing]
        M2[Cross-Browser]
        M3[Mobile Responsiveness]
    end
```

## Deployment Pipeline

```mermaid
graph LR
    Dev[Development] --> Test[Testing]
    Test --> Stage[Staging]
    Stage --> Prod[Production]
    
    Dev -->|Local Server| DevTest[Manual Testing]
    Test -->|Firebase Emulator| AutoTest[Automated Tests]
    Stage -->|Preview Deploy| UAT[User Testing]
    Prod -->|Firebase Hosting| Live[Live App]
```

## Key Technical Decisions

### Technology Stack

| Layer | Technology | Reasoning |
|-------|-----------|-----------|
| Frontend | HTML/CSS/JS | Simple, no build step needed initially |
| Styling | Tailwind CSS | Rapid prototyping, utility-first |
| Database | Firestore | Real-time sync, easy setup |
| Auth | Firebase Auth | Google OAuth integration |
| Functions | Cloud Functions | Serverless, auto-scaling |
| AI | Google Gemini | Powerful, cost-effective |
| Charts | Chart.js | Lightweight, well-documented |

### Design Patterns

| Pattern | Use Case |
|---------|----------|
| Observer | Real-time Firestore listeners |
| Factory | Creating workout instances |
| Singleton | App state management |
| Strategy | Different workout types |
| Decorator | Theme customization |

### Data Storage Strategy

| Data Type | Storage Location | Sync Strategy |
|-----------|-----------------|---------------|
| Today's data | Firestore + Local State | Real-time |
| User profile | Firestore | Real-time |
| Custom workouts | Firestore + Local Cache | On-demand |
| Exercise history | Firestore | Lazy load |
| Achievements | Firestore | On login |
| Theme preferences | Firestore + LocalStorage | Immediate |

## Progressive Enhancement Approach

1. **Core Experience** (Works everywhere)
   - Basic workout logging
   - Simple nutrition tracking
   - Manual data entry

2. **Enhanced Experience** (Modern browsers)
   - Real-time sync
   - AI nutrition analysis
   - Smooth animations

3. **Premium Experience** (Latest browsers)
   - Advanced charts
   - Offline support
   - Push notifications

## Migration Strategy

### Phase 1 to Phase 2

```javascript
// Add migration function to handle schema changes
async function migrateToPhase2() {
  // Convert simple exercise completion to detailed logging
  const workouts = await getDocs(dailyLogsRef);
  workouts.forEach(async (doc) => {
    const data = doc.data();
    if (data.todaysWorkouts) {
      // Transform old format to new format
      const updatedWorkouts = data.todaysWorkouts.map(workout => ({
        ...workout,
        exerciseData: workout.completedExercises ? 
          convertToDetailedLogs(workout.completedExercises) : {}
      }));
      await updateDoc(doc.ref, { todaysWorkouts: updatedWorkouts });
    }
  });
}
```

## Scalability Considerations

### Database Design

- Use subcollections for exercise history (better query performance)
- Implement data archiving after 1 year
- Use Firestore aggregate queries for statistics
- Implement pagination for all lists

### Code Organization

- Keep modules under 500 lines
- Use lazy loading for heavy features
- Implement code splitting for large dependencies
- Use service workers for offline support

### Cost Optimization

- Cache frequently accessed data
- Batch Firestore writes where possible
- Use Gemini Flash model for nutrition (faster, cheaper)
- Implement rate limiting for API calls

## Future Considerations

### Potential Enhancements (Beyond Phase 5)

1. **Social Features**
   - Share workouts with friends
   - Compete on leaderboards
   - Follow other users

2. **Advanced Analytics**
   - Body measurements tracking
   - Progress photos
   - AI workout recommendations

3. **Integration**
   - Connect with fitness trackers
   - Import from other apps
   - Export to spreadsheets

4. **Monetization**
   - Premium features
   - Personal training marketplace
   - Nutrition coaching

## Success Metrics

### Phase 1 KPIs

- Onboarding completion rate > 80%
- Custom workout creation rate > 50%
- Daily active users
- Average session duration
- Feature adoption rate

### Technical Metrics

- Page load time < 2s
- Time to interactive < 3s
- First contentful paint < 1s
- API response time < 500ms
- Crash-free rate > 99%

---

## Quick Reference

### File Structure After Refactoring

```
health-dashboard/
├── index.html                 # Main HTML file
├── IMPLEMENTATION_PLAN.md     # Detailed implementation plan
├── ARCHITECTURE.md           # This file
├── README.md                 # Project documentation
│
├── js/
│   ├── main.js              # App initialization
│   ├── auth.js              # Authentication logic
│   ├── firestore.js         # Database operations
│   ├── ui.js                # UI rendering
│   ├── workouts.js          # Workout management
│   ├── nutrition.js         # Nutrition tracking
│   ├── exercises.js         # Exercise logging
│   ├── achievements.js      # Gamification
│   ├── theme.js             # Theme management
│   └── utils.js             # Utility functions
│
├── css/
│   ├── theme.css            # Theme and colors
│   ├── components.css       # Component styles
│   └── animations.css       # Animation definitions
│
├── assets/
│   ├── doodles/            # SVG doodles
│   └── icons/              # App icons
│
└── functions/
    ├── index.js             # Cloud Functions
    └── package.json         # Function dependencies
```

### Command Quick Reference

```bash
# Development
python -m http.server 8000  # Start local server

# Firebase (Phase 5)
firebase init                # Initialize Firebase
firebase deploy              # Deploy to production
firebase deploy --only functions  # Deploy only functions
firebase emulators:start     # Start local emulators

# Testing
npm test                     # Run unit tests (Phase 5)
```

### Resource Links

- [Firebase Documentation](https://firebase.google.com/docs)
- [Firestore Security Rules](https://firebase.google.com/docs/firestore/security/get-started)
- [Google Gemini API](https://ai.google.dev/docs)
- [Chart.js Documentation](https://www.chartjs.org/docs/latest/)
- [Tailwind CSS](https://tailwindcss.com/docs)