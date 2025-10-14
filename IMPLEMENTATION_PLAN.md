# Sketch Health - Comprehensive Implementation Plan

## Table of Contents
1. [Overview](#overview)
2. [Current Architecture Analysis](#current-architecture-analysis)
3. [Phase 1: Core High-Impact Features](#phase-1-core-high-impact-features)
4. [Phase 2: Progressive Overload & Exercise Tracking](#phase-2-progressive-overload--exercise-tracking)
5. [Phase 3: Gamification & Visualization](#phase-3-gamification--visualization)
6. [Phase 4: Enhanced Data & Settings](#phase-4-enhanced-data--settings)
7. [Phase 5: Security & Technical Improvements](#phase-5-security--technical-improvements)
8. [Data Model Evolution](#data-model-evolution)
9. [Technical Considerations](#technical-considerations)

---

## Overview

This document outlines a phased approach to enhance Sketch Health with new features while maintaining the charming notepad aesthetic and improving user experience.

**Implementation Strategy**: Iterative phases with user feedback loops
**Timeline**: Each phase can be implemented independently
**Architecture**: Progressive enhancement from single-file to modular structure

---

## Current Architecture Analysis

### Strengths
- Clean notepad aesthetic with hand-drawn elements
- Real-time Firebase sync
- Simple, intuitive UI
- AI-powered calorie estimation
- Built-in timer for exercises

### Current Data Structure
```javascript
// Firestore: artifacts/{appId}/users/{userId}/dailyLogs/{date}
{
  liquidCalories: number,
  waterIntake: number,
  excessCalories: number,
  excessLog: Array<{calories, desc}>,
  todaysWorkouts: Array<{
    id: string,
    instanceId: string,
    name: string,
    isComplete: boolean,
    completedExercises?: string[],
    cardioType?: string,
    duration?: number
  }>
}

// Firestore: artifacts/{appId}/users/{userId}/data/userProfile
{
  workoutStreak: number,
  lastWorkoutDate: string
}
```

### Areas for Improvement
1. Limited workout customization
2. No progressive overload tracking
3. Missing macro tracking
4. No onboarding experience
5. API keys exposed in client code
6. Limited gamification elements

---

## Phase 1: Core High-Impact Features

### 1.1 Onboarding Modal

**Purpose**: Guide first-time users and personalize their experience

**Implementation**:
```html
<!-- Add to body, before loading-container -->
<div id="onboarding-modal" class="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center hidden z-50">
  <div class="bg-yellow-100 rounded-lg p-8 max-w-2xl mx-4 border-4 border-blue-800 shadow-2xl">
    <!-- Step 1: Welcome -->
    <div id="onboarding-step-1" class="onboarding-step">
      <h2 class="text-3xl font-bold notepad-header mb-4">Welcome to Sketch Health! 📝💪</h2>
      <p class="text-lg mb-6">Let's personalize your fitness journey in just a few steps.</p>
      <button class="btn btn-primary w-full" onclick="app.onboarding.nextStep()">Get Started</button>
    </div>
    
    <!-- Step 2: Goal Selection -->
    <div id="onboarding-step-2" class="onboarding-step hidden">
      <h2 class="text-2xl font-bold notepad-header mb-4">What's your primary goal?</h2>
      <div class="space-y-3">
        <button class="btn btn-secondary w-full text-left" onclick="app.onboarding.selectGoal('build_muscle')">
          💪 Build Muscle
        </button>
        <button class="btn btn-secondary w-full text-left" onclick="app.onboarding.selectGoal('lose_weight')">
          🏃 Lose Weight
        </button>
        <button class="btn btn-secondary w-full text-left" onclick="app.onboarding.selectGoal('maintain_fitness')">
          ⚖️ Maintain Fitness
        </button>
      </div>
    </div>
    
    <!-- Step 3: Daily Goals -->
    <div id="onboarding-step-3" class="onboarding-step hidden">
      <h2 class="text-2xl font-bold notepad-header mb-4">Set Your Daily Goals</h2>
      <div class="space-y-4">
        <div>
          <label class="block font-semibold mb-2">Water Goal (glasses/day)</label>
          <input type="number" id="onboarding-water-goal" value="8" min="1" max="20" class="w-full px-3 py-2">
        </div>
        <div>
          <label class="block font-semibold mb-2">Max Liquid Calories (kcal/day)</label>
          <input type="number" id="onboarding-liquid-goal" value="500" min="0" max="2000" step="50" class="w-full px-3 py-2">
        </div>
        <div>
          <label class="block font-semibold mb-2">Workout Days per Week</label>
          <input type="number" id="onboarding-workout-days" value="3" min="1" max="7" class="w-full px-3 py-2">
        </div>
      </div>
      <button class="btn btn-primary w-full mt-6" onclick="app.onboarding.complete()">Start My Journey!</button>
    </div>
  </div>
</div>
```

**Data Structure Addition**:
```javascript
// Add to userProfile
{
  onboardingCompleted: boolean,
  primaryGoal: 'build_muscle' | 'lose_weight' | 'maintain_fitness',
  waterGoal: number,
  liquidCaloriesGoal: number,
  weeklyWorkoutGoal: number,
  createdAt: timestamp
}
```

**JavaScript Logic**:
```javascript
onboarding: {
  currentStep: 1,
  selectedGoal: null,
  
  init() {
    // Check if user has completed onboarding
    if (!app.userProfile.onboardingCompleted) {
      document.getElementById('onboarding-modal').classList.remove('hidden');
    }
  },
  
  nextStep() {
    document.getElementById(`onboarding-step-${this.currentStep}`).classList.add('hidden');
    this.currentStep++;
    document.getElementById(`onboarding-step-${this.currentStep}`).classList.remove('hidden');
  },
  
  selectGoal(goal) {
    this.selectedGoal = goal;
    this.nextStep();
  },
  
  async complete() {
    app.userProfile.onboardingCompleted = true;
    app.userProfile.primaryGoal = this.selectedGoal;
    app.userProfile.waterGoal = parseInt(document.getElementById('onboarding-water-goal').value);
    app.userProfile.liquidCaloriesGoal = parseInt(document.getElementById('onboarding-liquid-goal').value);
    app.userProfile.weeklyWorkoutGoal = parseInt(document.getElementById('onboarding-workout-days').value);
    app.userProfile.createdAt = new Date().toISOString();
    
    await app.saveData('profile');
    document.getElementById('onboarding-modal').classList.add('hidden');
  }
}
```

---

### 1.2 Custom Workout Builder

**Purpose**: Allow users to create personalized workout routines

**UI Component** (New page/modal):
```html
<div id="workout-builder-modal" class="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center hidden z-50 overflow-y-auto">
  <div class="bg-yellow-100 rounded-lg p-8 max-w-4xl mx-4 my-8 border-4 border-blue-800 max-h-[90vh] overflow-y-auto">
    <div class="flex justify-between items-center mb-6">
      <h2 class="text-3xl font-bold notepad-header">Custom Workout Builder</h2>
      <button onclick="app.workoutBuilder.close()" class="btn btn-secondary">✕ Close</button>
    </div>
    
    <!-- Workout Name -->
    <div class="mb-6">
      <label class="block font-bold mb-2">Workout Name</label>
      <input type="text" id="custom-workout-name" placeholder="e.g., My Chest Day" class="w-full px-3 py-2">
    </div>
    
    <!-- Duration Estimate -->
    <div class="mb-6">
      <label class="block font-bold mb-2">Estimated Duration (minutes)</label>
      <input type="number" id="custom-workout-duration" value="45" min="5" max="180" class="w-full px-3 py-2">
    </div>
    
    <!-- Exercise List -->
    <div class="mb-6">
      <div class="flex justify-between items-center mb-4">
        <h3 class="text-xl font-bold">Exercises</h3>
        <button onclick="app.workoutBuilder.addExercise()" class="btn btn-primary">+ Add Exercise</button>
      </div>
      
      <div id="custom-exercises-list" class="space-y-4">
        <!-- Exercise items will be added here dynamically -->
      </div>
    </div>
    
    <!-- Action Buttons -->
    <div class="flex gap-4">
      <button onclick="app.workoutBuilder.save()" class="btn btn-primary flex-1">Save Workout</button>
      <button onclick="app.workoutBuilder.close()" class="btn btn-secondary flex-1">Cancel</button>
    </div>
  </div>
</div>

<!-- Exercise Item Template -->
<template id="exercise-item-template">
  <div class="exercise-item bg-white p-4 rounded-lg border-2 border-blue-800">
    <div class="flex gap-4 items-start">
      <div class="flex-1 space-y-3">
        <input type="text" placeholder="Exercise name (e.g., Bench Press)" class="exercise-name w-full px-3 py-2">
        <div class="grid grid-cols-3 gap-3">
          <div>
            <label class="block text-sm font-semibold mb-1">Sets</label>
            <input type="number" class="exercise-sets w-full px-3 py-2" value="3" min="1" max="10">
          </div>
          <div>
            <label class="block text-sm font-semibold mb-1">Reps</label>
            <input type="text" class="exercise-reps w-full px-3 py-2" value="10-12" placeholder="8-12 or 60s">
          </div>
          <div>
            <label class="block text-sm font-semibold mb-1">Rest (sec)</label>
            <input type="number" class="exercise-rest w-full px-3 py-2" value="90" min="30" max="300" step="15">
          </div>
        </div>
      </div>
      <button class="btn btn-danger btn-sm mt-8" onclick="app.workoutBuilder.removeExercise(this)">✕</button>
    </div>
  </div>
</template>
```

**Data Structure**:
```javascript
// Firestore: artifacts/{appId}/users/{userId}/customWorkouts/{workoutId}
{
  id: string,
  name: string,
  duration: number, // minutes
  createdAt: timestamp,
  lastModified: timestamp,
  exercises: Array<{
    name: string,
    sets: number,
    reps: string, // Can be "10-12" or "60s hold"
    restTime: number // seconds
  }>
}
```

**JavaScript Implementation**:
```javascript
workoutBuilder: {
  currentWorkout: null,
  isEditing: false,
  
  open(workoutId = null) {
    this.isEditing = !!workoutId;
    if (workoutId) {
      // Load existing workout for editing
      this.loadWorkout(workoutId);
    } else {
      // New workout
      this.currentWorkout = null;
      document.getElementById('custom-workout-name').value = '';
      document.getElementById('custom-workout-duration').value = '45';
      document.getElementById('custom-exercises-list').innerHTML = '';
    }
    document.getElementById('workout-builder-modal').classList.remove('hidden');
  },
  
  close() {
    document.getElementById('workout-builder-modal').classList.add('hidden');
  },
  
  addExercise() {
    const template = document.getElementById('exercise-item-template');
    const clone = template.content.cloneNode(true);
    document.getElementById('custom-exercises-list').appendChild(clone);
  },
  
  removeExercise(button) {
    button.closest('.exercise-item').remove();
  },
  
  async save() {
    const name = document.getElementById('custom-workout-name').value.trim();
    if (!name) {
      alert('Please enter a workout name');
      return;
    }
    
    const exercises = [];
    document.querySelectorAll('.exercise-item').forEach(item => {
      const exerciseName = item.querySelector('.exercise-name').value.trim();
      if (exerciseName) {
        exercises.push({
          name: exerciseName,
          sets: parseInt(item.querySelector('.exercise-sets').value),
          reps: item.querySelector('.exercise-reps').value.trim(),
          restTime: parseInt(item.querySelector('.exercise-rest').value)
        });
      }
    });
    
    if (exercises.length === 0) {
      alert('Please add at least one exercise');
      return;
    }
    
    const workoutId = this.currentWorkout?.id || `custom_${Date.now()}`;
    const workoutData = {
      id: workoutId,
      name: name,
      duration: parseInt(document.getElementById('custom-workout-duration').value),
      exercises: exercises,
      lastModified: new Date().toISOString()
    };
    
    if (!this.isEditing) {
      workoutData.createdAt = new Date().toISOString();
    }
    
    // Save to Firestore
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
    const docRef = doc(app.db, 'artifacts', appId, 'users', app.userId, 'customWorkouts', workoutId);
    await setDoc(docRef, workoutData);
    
    // Add to local workouts object
    app.workouts[workoutId] = workoutData;
    
    // Update workout selector
    this.updateWorkoutSelector();
    
    this.close();
    alert(`Workout "${name}" saved successfully!`);
  },
  
  updateWorkoutSelector() {
    const selector = document.getElementById('workoutSelector');
    // Remove old custom options
    Array.from(selector.options).forEach(option => {
      if (option.value.startsWith('custom_')) {
        option.remove();
      }
    });
    
    // Add custom workouts
    Object.keys(app.workouts).forEach(workoutId => {
      if (workoutId.startsWith('custom_')) {
        const workout = app.workouts[workoutId];
        const option = document.createElement('option');
        option.value = workoutId;
        option.textContent = `${workout.name} (${workout.duration} Mins) [Custom]`;
        selector.insertBefore(option, selector.querySelector('[value="cardio"]'));
      }
    });
  },
  
  async loadUserWorkouts() {
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
    const workoutsRef = collection(app.db, 'artifacts', appId, 'users', app.userId, 'customWorkouts');
    const querySnapshot = await getDocs(workoutsRef);
    
    querySnapshot.forEach(doc => {
      app.workouts[doc.id] = doc.data();
    });
    
    this.updateWorkoutSelector();
  }
}
```

---

### 1.3 Enhanced Nutrition with Macros

**Purpose**: Provide complete nutritional tracking with protein, carbs, and fat

**UI Changes**:
```html
<!-- Replace existing Nutrition Hub content -->
<div class="card p-6">
  <h2 class="font-bold text-xl mb-4 notepad-header">Nutrition Hub</h2>
  
  <!-- Macros Overview -->
  <div class="mb-6 p-4 bg-white rounded-lg border-2 border-blue-800">
    <h3 class="font-bold text-lg mb-3 notepad-header">Today's Macros</h3>
    <div class="grid grid-cols-3 gap-4 mb-4">
      <div class="text-center">
        <p class="text-sm text-blue-600 font-semibold">Protein</p>
        <p class="text-2xl font-bold notepad-header"><span id="proteinDisplay">0</span>g</p>
        <p class="text-xs text-blue-400">Goal: <span id="proteinGoal">150</span>g</p>
      </div>
      <div class="text-center">
        <p class="text-sm text-blue-600 font-semibold">Carbs</p>
        <p class="text-2xl font-bold notepad-header"><span id="carbsDisplay">0</span>g</p>
        <p class="text-xs text-blue-400">Goal: <span id="carbsGoal">200</span>g</p>
      </div>
      <div class="text-center">
        <p class="text-sm text-blue-600 font-semibold">Fat</p>
        <p class="text-2xl font-bold notepad-header"><span id="fatDisplay">0</span>g</p>
        <p class="text-xs text-blue-400">Goal: <span id="fatGoal">60</span>g</p>
      </div>
    </div>
    
    <!-- Macro Progress Bars -->
    <div class="space-y-2">
      <div>
        <div class="progress-bar">
          <div id="proteinBar" class="progress-bar-fill" style="width: 0%; background-color: #EF4444;"></div>
        </div>
      </div>
      <div>
        <div class="progress-bar">
          <div id="carbsBar" class="progress-bar-fill" style="width: 0%; background-color: #F59E0B;"></div>
        </div>
      </div>
      <div>
        <div class="progress-bar">
          <div id="fatBar" class="progress-bar-fill" style="width: 0%; background-color: #10B981;"></div>
        </div>
      </div>
    </div>
  </div>
  
  <!-- Meal Logging Sections -->
  <div class="space-y-4 mb-6">
    <div class="meal-section" data-meal="breakfast">
      <div class="flex justify-between items-center mb-2">
        <h3 class="font-bold notepad-header">🌅 Breakfast</h3>
        <button onclick="app.nutrition.addMeal('breakfast')" class="btn btn-secondary btn-sm py-1 px-3 text-xs">+ Add Food</button>
      </div>
      <div id="breakfast-list" class="space-y-2 text-sm"></div>
    </div>
    
    <div class="meal-section" data-meal="lunch">
      <div class="flex justify-between items-center mb-2">
        <h3 class="font-bold notepad-header">☀️ Lunch</h3>
        <button onclick="app.nutrition.addMeal('lunch')" class="btn btn-secondary btn-sm py-1 px-3 text-xs">+ Add Food</button>
      </div>
      <div id="lunch-list" class="space-y-2 text-sm"></div>
    </div>
    
    <div class="meal-section" data-meal="dinner">
      <div class="flex justify-between items-center mb-2">
        <h3 class="font-bold notepad-header">🌙 Dinner</h3>
        <button onclick="app.nutrition.addMeal('dinner')" class="btn btn-secondary btn-sm py-1 px-3 text-xs">+ Add Food</button>
      </div>
      <div id="dinner-list" class="space-y-2 text-sm"></div>
    </div>
    
    <div class="meal-section" data-meal="snacks">
      <div class="flex justify-between items-center mb-2">
        <h3 class="font-bold notepad-header">🍎 Snacks & Extras</h3>
        <button onclick="app.nutrition.addMeal('snacks')" class="btn btn-secondary btn-sm py-1 px-3 text-xs">+ Add Food</button>
      </div>
      <div id="snacks-list" class="space-y-2 text-sm"></div>
    </div>
  </div>
  
  <!-- Liquid Calories (keep existing) -->
  <div class="mb-6">
    <h3 class="font-semibold notepad-header mb-2">Liquid Calories</h3>
    <!-- Keep existing liquid calories UI -->
  </div>
  
  <!-- Water Intake (keep existing) -->
  <div>
    <h3 class="font-semibold notepad-header mb-2">Water Intake</h3>
    <!-- Keep existing water intake UI -->
  </div>
</div>

<!-- Food Entry Modal -->
<div id="food-entry-modal" class="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center hidden z-50">
  <div class="bg-yellow-100 rounded-lg p-6 max-w-md mx-4 border-4 border-blue-800">
    <h3 class="text-2xl font-bold notepad-header mb-4">Add Food</h3>
    <input type="text" id="food-description" placeholder="Describe your food..." class="w-full px-3 py-2 mb-4">
    <div class="flex gap-3">
      <button id="analyze-food-btn" onclick="app.nutrition.analyzeFood()" class="btn btn-primary flex-1">
        <span id="analyze-text">Analyze</span>
        <svg id="analyze-spinner" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white hidden" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      </button>
      <button onclick="app.nutrition.closeFoodModal()" class="btn btn-secondary flex-1">Cancel</button>
    </div>
    <div id="food-result" class="mt-4 hidden">
      <div class="bg-white p-4 rounded-lg border-2 border-blue-800">
        <div class="grid grid-cols-2 gap-3 text-sm mb-3">
          <div><strong>Calories:</strong> <span id="result-calories"></span></div>
          <div><strong>Protein:</strong> <span id="result-protein"></span>g</div>
          <div><strong>Carbs:</strong> <span id="result-carbs"></span>g</div>
          <div><strong>Fat:</strong> <span id="result-fat"></span>g</div>
        </div>
        <button onclick="app.nutrition.confirmFood()" class="btn btn-primary w-full">Add to Log</button>
      </div>
    </div>
  </div>
</div>
```

**Updated Gemini API Call**:
```javascript
nutrition: {
  currentMeal: null,
  currentFoodData: null,
  
  addMeal(mealType) {
    this.currentMeal = mealType;
    document.getElementById('food-entry-modal').classList.remove('hidden');
    document.getElementById('food-description').value = '';
    document.getElementById('food-result').classList.add('hidden');
  },
  
  closeFoodModal() {
    document.getElementById('food-entry-modal').classList.add('hidden');
  },
  
  async analyzeFood() {
    const description = document.getElementById('food-description').value.trim();
    if (!description) return;
    
    const analyzeBtn = document.getElementById('analyze-food-btn');
    const analyzeText = document.getElementById('analyze-text');
    const analyzeSpinner = document.getElementById('analyze-spinner');
    
    analyzeBtn.disabled = true;
    analyzeText.textContent = 'Analyzing...';
    analyzeSpinner.classList.remove('hidden');
    
    try {
      const apiKey = "AIzaSyD6imNqPY8RfloxyOxxbYz_pvYScIxmkvk";
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
      
      const payload = {
        contents: [{
          parts: [{
            text: `Analyze this food and provide nutritional information: ${description}. 
                   Give estimates for calories, protein (grams), carbohydrates (grams), and fat (grams).`
          }]
        }],
        generationConfig: {
          responseMimeType: "application/json",
          responseSchema: {
            type: "OBJECT",
            properties: {
              calories: { type: "NUMBER" },
              protein: { type: "NUMBER" },
              carbs: { type: "NUMBER" },
              fat: { type: "NUMBER" }
            },
            required: ["calories", "protein", "carbs", "fat"]
          }
        }
      };
      
      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });
      
      if (!response.ok) throw new Error(`API error: ${response.statusText}`);
      
      const result = await response.json();
      this.currentFoodData = JSON.parse(result.candidates[0].content.parts[0].text);
      
      // Display results
      document.getElementById('result-calories').textContent = this.currentFoodData.calories;
      document.getElementById('result-protein').textContent = this.currentFoodData.protein;
      document.getElementById('result-carbs').textContent = this.currentFoodData.carbs;
      document.getElementById('result-fat').textContent = this.currentFoodData.fat;
      document.getElementById('food-result').classList.remove('hidden');
      
    } catch (error) {
      console.error("Food analysis failed:", error);
      alert("Sorry, I couldn't analyze that food. Please try again.");
    } finally {
      analyzeBtn.disabled = false;
      analyzeText.textContent = 'Analyze';
      analyzeSpinner.classList.add('hidden');
    }
  },
  
  confirmFood() {
    const description = document.getElementById('food-description').value.trim();
    const mealData = {
      description: description,
      ...this.currentFoodData,
      timestamp: new Date().toISOString()
    };
    
    // Add to appropriate meal in todayState
    if (!app.todayState.meals) {
      app.todayState.meals = {
        breakfast: [],
        lunch: [],
        dinner: [],
        snacks: []
      };
    }
    
    app.todayState.meals[this.currentMeal].push(mealData);
    
    // Update macro totals
    app.todayState.protein = (app.todayState.protein || 0) + this.currentFoodData.protein;
    app.todayState.carbs = (app.todayState.carbs || 0) + this.currentFoodData.carbs;
    app.todayState.fat = (app.todayState.fat || 0) + this.currentFoodData.fat;
    app.todayState.totalCalories = (app.todayState.totalCalories || 0) + this.currentFoodData.calories;
    
    app.saveData('daily');
    this.closeFoodModal();
  },
  
  renderMeals() {
    const meals = app.todayState.meals || { breakfast: [], lunch: [], dinner: [], snacks: [] };
    
    ['breakfast', 'lunch', 'dinner', 'snacks'].forEach(mealType => {
      const listElement = document.getElementById(`${mealType}-list`);
      listElement.innerHTML = meals[mealType].map((food, index) => `
        <div class="flex justify-between items-center bg-yellow-50 p-2 rounded-md">
          <div class="flex-1">
            <p class="font-semibold">${food.description}</p>
            <p class="text-xs text-blue-600">${food.calories}cal | P:${food.protein}g C:${food.carbs}g F:${food.fat}g</p>
          </div>
          <button onclick="app.nutrition.removeFood('${mealType}', ${index})" class="btn btn-danger btn-sm py-1 px-2 text-xs">✕</button>
        </div>
      `).join('') || '<p class="text-blue-400 text-xs italic">No items added</p>';
    });
    
    // Update macro displays and bars
    this.updateMacroDisplays();
  },
  
  updateMacroDisplays() {
    const state = app.todayState;
    const goals = app.userProfile;
    
    document.getElementById('proteinDisplay').textContent = state.protein || 0;
    document.getElementById('carbsDisplay').textContent = state.carbs || 0;
    document.getElementById('fatDisplay').textContent = state.fat || 0;
    
    document.getElementById('proteinBar').style.width = `${Math.min(100, ((state.protein || 0) / (goals.proteinGoal || 150)) * 100)}%`;
    document.getElementById('carbsBar').style.width = `${Math.min(100, ((state.carbs || 0) / (goals.carbsGoal || 200)) * 100)}%`;
    document.getElementById('fatBar').style.width = `${Math.min(100, ((state.fat || 0) / (goals.fatGoal || 60)) * 100)}%`;
  },
  
  removeFood(mealType, index) {
    const food = app.todayState.meals[mealType][index];
    
    // Subtract from totals
    app.todayState.protein -= food.protein;
    app.todayState.carbs -= food.carbs;
    app.todayState.fat -= food.fat;
    app.todayState.totalCalories -= food.calories;
    
    // Remove from meal
    app.todayState.meals[mealType].splice(index, 1);
    
    app.saveData('daily');
  }
}
```

**Data Structure Updates**:
```javascript
// Add to dailyLogs
{
  // ... existing fields
  meals: {
    breakfast: Array<FoodItem>,
    lunch: Array<FoodItem>,
    dinner: Array<FoodItem>,
    snacks: Array<FoodItem>
  },
  protein: number,
  carbs: number,
  fat: number,
  totalCalories: number
}

// Add to userProfile
{
  // ... existing fields
  proteinGoal: number,
  carbsGoal: number,
  fatGoal: number
}

type FoodItem = {
  description: string,
  calories: number,
  protein: number,
  carbs: number,
  fat: number,
  timestamp: string
}
```

---

### 1.4 Visual Enhancements

#### Hand-Drawn Doodles
```css
/* Add to style section */
.doodle-water {
  display: inline-block;
  width: 32px;
  height: 32px;
  background-image: url('data:image/svg+xml,...'); /* Water glass doodle */
}

.doodle-food {
  display: inline-block;
  width: 32px;
  height: 32px;
  background-image: url('data:image/svg+xml,...'); /* Food plate doodle */
}

.doodle-checkmark {
  display: inline-block;
  width: 24px;
  height: 24px;
  background-image: url('data:image/svg+xml,...'); /* Hand-drawn checkmark */
}
```

#### Torn Paper Edges
```css
.card {
  /* Add torn paper effect */
  box-shadow: 
    2px 2px 4px rgba(0,0,0,0.1),
    inset 0 0 0 2px rgba(191, 219, 254, 0.3);
  position: relative;
}

.card::after {
  content: '';
  position: absolute;
  top: -3px;
  left: 0;
  right: 0;
  height: 6px;
  background-image: 
    repeating-linear-gradient(
      90deg,
      transparent,
      transparent 10px,
      rgba(191, 219, 254, 0.5) 10px,
      rgba(191, 219, 254, 0.5) 11px
    );
}
```

#### Confetti Animation
```javascript
// Add confetti library or custom implementation
function celebrateWorkoutCompletion() {
  // Use canvas-confetti library or custom implementation
  confetti({
    particleCount: 100,
    spread: 70,
    origin: { y: 0.6 },
    colors: ['#1E3A8A', '#60A5FA', '#93C5FD', '#FEF9C3']
  });
}

// Modify completeWorkout function
completeWorkout(instanceId) {
  const workout = this.todayState.todaysWorkouts.find(w => w.instanceId === instanceId);
  if (!workout || workout.isComplete) return;
  
  workout.isComplete = true;
  
  // Show confetti!
  celebrateWorkoutCompletion();
  
  // ... rest of existing logic
}
```

#### Animated Progress Bars
```css
.progress-bar-fill {
  transition: width 0.6s cubic-bezier(0.65, 0, 0.35, 1);
}

/* Add pulse animation when goal is reached */
@keyframes pulse-glow {
  0%, 100% { box-shadow: 0 0 5px rgba(96, 165, 250, 0.5); }
  50% { box-shadow: 0 0 20px rgba(96, 165, 250, 0.8); }
}

.progress-bar-fill.goal-reached {
  animation: pulse-glow 2s infinite;
}
```

#### Customizable Theme
```html
<!-- Add to profile dropdown -->
<div id="profileDropdown" class="...">
  <button id="settingsBtn" class="w-full text-left px-4 py-2 hover:bg-blue-50 text-blue-900 font-semibold">
    ⚙️ Settings
  </button>
  <button id="logoutBtn" class="...">
    Log out
  </button>
</div>

<!-- Settings Modal -->
<div id="settings-modal" class="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center hidden z-50">
  <div class="bg-yellow-100 rounded-lg p-8 max-w-md mx-4 border-4 border-blue-800">
    <h2 class="text-2xl font-bold notepad-header mb-6">Theme Settings</h2>
    
    <div class="space-y-6">
      <div>
        <label class="block font-bold mb-3">Ink Color</label>
        <div class="grid grid-cols-4 gap-3">
          <button class="ink-color-btn w-12 h-12 rounded-full border-4" style="background-color: #1E3A8A;" data-color="#1E3A8A" onclick="app.theme.setInkColor('#1E3A8A')"></button>
          <button class="ink-color-btn w-12 h-12 rounded-full border-4" style="background-color: #000000;" data-color="#000000" onclick="app.theme.setInkColor('#000000')"></button>
          <button class="ink-color-btn w-12 h-12 rounded-full border-4" style="background-color: #059669;" data-color="#059669" onclick="app.theme.setInkColor('#059669')"></button>
          <button class="ink-color-btn w-12 h-12 rounded-full border-4" style="background-color: #7C3AED;" data-color="#7C3AED" onclick="app.theme.setInkColor('#7C3AED')"></button>
        </div>
      </div>
      
      <div>
        <label class="block font-bold mb-3">Highlighter Color</label>
        <div class="grid grid-cols-4 gap-3">
          <button class="highlight-color-btn w-12 h-12 rounded-full border-4" style="background-color: #60A5FA;" data-color="#60A5FA" onclick="app.theme.setHighlightColor('#60A5FA')"></button>
          <button class="highlight-color-btn w-12 h-12 rounded-full border-4" style="background-color: #34D399;" data-color="#34D399" onclick="app.theme.setHighlightColor('#34D399')"></button>
          <button class="highlight-color-btn w-12 h-12 rounded-full border-4" style="background-color: #FBBF24;" data-color="#FBBF24" onclick="app.theme.setHighlightColor('#FBBF24')"></button>
          <button class="highlight-color-btn w-12 h-12 rounded-full border-4" style="background-color: #F472B6;" data-color="#F472B6" onclick="app.theme.setHighlightColor('#F472B6')"></button>
        </div>
      </div>
    </div>
    
    <button onclick="app.theme.closeSettings()" class="btn btn-primary w-full mt-6">Done</button>
  </div>
</div>
```

```javascript
theme: {
  inkColor: '#1E3A8A',
  highlightColor: '#60A5FA',
  
  init() {
    // Load saved theme
    if (app.userProfile.inkColor) {
      this.inkColor = app.userProfile.inkColor;
      this.highlightColor = app.userProfile.highlightColor;
      this.applyTheme();
    }
  },
  
  setInkColor(color) {
    this.inkColor = color;
    this.applyTheme();
    this.saveTheme();
  },
  
  setHighlightColor(color) {
    this.highlightColor = color;
    this.applyTheme();
    this.saveTheme();
  },
  
  applyTheme() {
    document.documentElement.style.setProperty('--ink-color', this.inkColor);
    document.documentElement.style.setProperty('--highlight-color', this.highlightColor);
  },
  
  async saveTheme() {
    app.userProfile.inkColor = this.inkColor;
    app.userProfile.highlightColor = this.highlightColor;
    await app.saveData('profile');
  },
  
  openSettings() {
    document.getElementById('settings-modal').classList.remove('hidden');
  },
  
  closeSettings() {
    document.getElementById('settings-modal').classList.add('hidden');
  }
}
```

```css
/* Update CSS to use CSS variables */
:root {
  --ink-color: #1E3A8A;
  --highlight-color: #60A5FA;
}

body {
  color: var(--ink-color);
}

.notepad-header {
  color: var(--ink-color);
}

.progress-bar-fill {
  background-color: var(--highlight-color);
}

/* Update other color references */
```

---

## Phase 2: Progressive Overload & Exercise Tracking

### 2.1 Detailed Exercise Logging

**Purpose**: Track weight and reps for each set to enable progressive overload

**UI Changes**:
```html
<!-- Replace checkbox exercise items with detailed logging -->
<div class="exercise-log-item bg-yellow-50 p-4 rounded-lg border-2 border-blue-800 mb-3">
  <div class="flex justify-between items-start mb-3">
    <h4 class="font-bold text-lg">${exerciseName}</h4>
    <button onclick="app.exercises.viewHistory('${exerciseName}')" class="btn btn-secondary btn-sm py-1 px-2 text-xs">
      📊 History
    </button>
  </div>
  
  <!-- Previous Performance -->
  <div class="mb-3 p-2 bg-blue-100 rounded text-sm">
    <p class="font-semibold">Last time: 185 lbs × 10, 8, 7 reps</p>
    <p class="text-xs text-blue-600">Beat it by adding weight or reps! 💪</p>
  </div>
  
  <!-- Set Logging -->
  <div class="space-y-2">
    ${[1,2,3].map(setNum => `
      <div class="flex items-center gap-2">
        <span class="font-semibold w-16">Set ${setNum}:</span>
        <input type="number" placeholder="Weight" class="set-weight px-2 py-1 w-20" />
        <span>lbs ×</span>
        <input type="number" placeholder="Reps" class="set-reps px-2 py-1 w-16" />
        <span>reps</span>
        <button onclick="app.exercises.logSet('${exerciseName}', ${setNum})" class="btn btn-primary btn-sm py-1 px-3 text-xs">✓</button>
      </div>
    `).join('')}
  </div>
  
  <!-- Quick Actions -->
  <div class="flex gap-2 mt-3">
    <button onclick="app.exercises.useLastWeight('${exerciseName}')" class="btn btn-secondary btn-sm flex-1 text-xs">
      Use Last Weight
    </button>
    <button onclick="app.exercises.startRestTimer(90)" class="btn btn-secondary btn-sm flex-1 text-xs">
      ⏱️ Rest Timer
    </button>
  </div>
</div>
```

**Data Structure**:
```javascript
// Firestore: artifacts/{appId}/users/{userId}/exerciseHistory/{exerciseName}
{
  exerciseName: string,
  history: Array<{
    date: string, // ISO date
    sets: Array<{
      weight: number,
      reps: number,
      rpe?: number // Rate of Perceived Exertion (optional)
    }>,
    notes?: string
  }>,
  personalRecords: {
    max1RM: { weight: number, date: string },
    maxVolume: { weight: number, reps: number, date: string },
    maxReps: { weight: number, reps: number, date: string }
  }
}
```

**Implementation**:
```javascript
exercises: {
  async logSet(exerciseName, setNumber) {
    const weightInput = document.querySelector(`[data-exercise="${exerciseName}"] [data-set="${setNumber}"] .set-weight`);
    const repsInput = document.querySelector(`[data-exercise="${exerciseName}"] [data-set="${setNumber}"] .set-reps`);
    
    const weight = parseFloat(weightInput.value);
    const reps = parseInt(repsInput.value);
    
    if (!weight || !reps) {
      alert('Please enter both weight and reps');
      return;
    }
    
    // Store in workout instance
    const workout = app.todayState.todaysWorkouts.find(w => !w.isComplete);
    if (!workout.exerciseData) workout.exerciseData = {};
    if (!workout.exerciseData[exerciseName]) {
      workout.exerciseData[exerciseName] = { sets: [] };
    }
    
    workout.exerciseData[exerciseName].sets[setNumber - 1] = { weight, reps };
    
    // Visual feedback
    weightInput.classList.add('bg-green-100');
    repsInput.classList.add('bg-green-100');
    
    await app.saveData('daily');
  },
  
  async saveExerciseHistory() {
    // Called when workout is completed
    const completedWorkout = app.todayState.todaysWorkouts.find(w => w.isComplete);
    if (!completedWorkout.exerciseData) return;
    
    const today = app.getTodayDateString();
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
    
    for (const [exerciseName, data] of Object.entries(completedWorkout.exerciseData)) {
      const historyRef = doc(app.db, 'artifacts', appId, 'users', app.userId, 'exerciseHistory', exerciseName);
      const historyDoc = await getDoc(historyRef);
      
      let history = historyDoc.exists() ? historyDoc.data() : {
        exerciseName: exerciseName,
        history: [],
        personalRecords: {}
      };
      
      // Add today's session
      history.history.push({
        date: today,
        sets: data.sets,
        notes: data.notes || ''
      });
      
      // Update PRs
      this.updatePersonalRecords(history, data.sets);
      
      // Keep only last 100 sessions
      if (history.history.length > 100) {
        history.history = history.history.slice(-100);
      }
      
      await setDoc(historyRef, history);
    }
  },
  
  updatePersonalRecords(history, sets) {
    sets.forEach(set => {
      const estimated1RM = set.weight * (1 + set.reps / 30); // Epley formula
      
      if (!history.personalRecords.max1RM || estimated1RM > history.personalRecords.max1RM.weight) {
        history.personalRecords.max1RM = {
          weight: estimated1RM,
          actualWeight: set.weight,
          reps: set.reps,
          date: app.getTodayDateString()
        };
      }
      
      const volume = set.weight * set.reps;
      if (!history.personalRecords.maxVolume || volume > history.personalRecords.maxVolume.volume) {
        history.personalRecords.maxVolume = {
          volume: volume,
          weight: set.weight,
          reps: set.reps,
          date: app.getTodayDateString()
        };
      }
    });
  },
  
  async viewHistory(exerciseName) {
    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
    const historyRef = doc(app.db, 'artifacts', appId, 'users', app.userId, 'exerciseHistory', exerciseName);
    const historyDoc = await getDoc(historyRef);
    
    if (!historyDoc.exists()) {
      alert('No history for this exercise yet. Complete it first!');
      return;
    }
    
    const history = historyDoc.data();
    this.showHistoryModal(history);
  },
  
  showHistoryModal(history) {
    // Create and show modal with chart
    // Use Chart.js or similar library
    const modalHTML = `
      <div class="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center z-50">
        <div class="bg-yellow-100 rounded-lg p-8 max-w-4xl mx-4 max-h-[90vh] overflow-y-auto border-4 border-blue-800">
          <h2 class="text-2xl font-bold notepad-header mb-4">${history.exerciseName} History</h2>
          
          <!-- Personal Records -->
          <div class="grid grid-cols-2 gap-4 mb-6">
            <div class="bg-white p-4 rounded-lg border-2 border-blue-800">
              <p class="text-sm text-blue-600 font-semibold">Est. 1RM</p>
              <p class="text-3xl font-bold">${history.personalRecords.max1RM?.weight.toFixed(0)} lbs</p>
              <p class="text-xs text-blue-400">${history.personalRecords.max1RM?.date}</p>
            </div>
            <div class="bg-white p-4 rounded-lg border-2 border-blue-800">
              <p class="text-sm text-blue-600 font-semibold">Best Volume Set</p>
              <p class="text-3xl font-bold">${history.personalRecords.maxVolume?.volume} lbs</p>
              <p class="text-xs text-blue-400">${history.personalRecords.maxVolume?.weight}×${history.personalRecords.maxVolume?.reps}</p>
            </div>
          </div>
          
          <!-- Chart -->
          <canvas id="exercise-chart" class="mb-6"></canvas>
          
          <!-- Recent Sessions -->
          <div class="space-y-2">
            <h3 class="font-bold text-lg">Recent Sessions</h3>
            ${history.history.slice(-10).reverse().map(session => `
              <div class="bg-white p-3 rounded border-2 border-blue-800">
                <p class="font-semibold mb-1">${new Date(session.date).toLocaleDateString()}</p>
                <p class="text-sm">${session.sets.map(s => `${s.weight}×${s.reps}`).join(', ')}</p>
              </div>
            `).join('')}
          </div>
          
          <button onclick="this.closest('.fixed').remove()" class="btn btn-primary w-full mt-6">Close</button>
        </div>
      </div>
    `;
    
    document.body.insertAdjacentHTML('beforeend', modalHTML);
    this.renderExerciseChart(history);
  },
  
  renderExerciseChart(history) {
    // Implement chart using Chart.js
    // Show weight progression over time
  }
}
```

---

## Phase 3: Gamification & Visualization

### 3.1 Achievements System

**Implementation**: Create a badges/achievements tracking system

**Data Structure**:
```javascript
// Firestore: artifacts/{appId}/users/{userId}/data/achievements
{
  unlockedAchievements: Array<{
    id: string,
    unlockedAt: timestamp,
    progress?: number
  }>,
  streaks: {
    workout: { current: number, longest: number },
    water: { current: number, longest: number },
    nutrition: { current: number, longest: number }
  }
}

// Achievement definitions (hardcoded in app)
const ACHIEVEMENTS = {
  streak_starter: {
    id: 'streak_starter',
    name: 'Streak Starter',
    description: 'Complete workouts 3 days in a row',
    icon: '🔥',
    condition: (profile) => profile.workoutStreak >= 3
  },
  hydration_hero: {
    id: 'hydration_hero',
    name: 'Hydration Hero',
    description: 'Meet your water goal for 7 consecutive days',
    icon: '💧',
    condition: (achievements) => achievements.streaks.water.current >= 7
  },
  // ... more achievements
}
```

---

## Phase 4: Enhanced Data & Settings

### 4.1 Settings Panel

Implement comprehensive user preferences including macro goals, theme preferences, and notification settings.

---

## Phase 5: Security & Technical Improvements

### 5.1 Firebase Cloud Function for Gemini API

**Purpose**: Secure API key server-side

**Setup**:
```bash
npm install -g firebase-tools
firebase login
firebase init functions
```

**Cloud Function**:
```javascript
// functions/index.js
const functions = require('firebase-functions');
const { GoogleGenerativeAI } = require('@google/generative-ai');

exports.analyzeFoodNutrition = functions.https.onCall(async (data, context) => {
  // Verify authentication
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'User must be authenticated');
  }
  
  const { foodDescription } = data;
  
  if (!foodDescription) {
    throw new functions.https.HttpsError('invalid-argument', 'Food description is required');
  }
  
  try {
    const genAI = new GoogleGenerativeAI(functions.config().gemini.key);
    const model = genAI.getGenerativeModel({ model: "gemini-2.5-flash-preview-05-20" });
    
    const result = await model.generateContent({
      contents: [{
        parts: [{
          text: `Analyze this food and provide nutritional information: ${foodDescription}`
        }]
      }],
      generationConfig: {
        responseMimeType: "application/json",
        responseSchema: {
          type: "OBJECT",
          properties: {
            calories: { type: "NUMBER" },
            protein: { type: "NUMBER" },
            carbs: { type: "NUMBER" },
            fat: { type: "NUMBER" }
          }
        }
      }
    });
    
    return JSON.parse(result.response.text());
    
  } catch (error) {
    console.error('Gemini API error:', error);
    throw new functions.https.HttpsError('internal', 'Failed to analyze food');
  }
});
```

**Client-side update**:
```javascript
async analyzeFood() {
  const description = document.getElementById('food-description').value.trim();
  if (!description) return;
  
  try {
    // Call Cloud Function instead of direct API
    const analyzeFoodNutrition = firebase.functions().httpsCallable('analyzeFoodNutrition');
    const result = await analyzeFoodNutrition({ foodDescription: description });
    
    this.currentFoodData = result.data;
    // ... rest of logic
    
  } catch (error) {
    console.error("Food analysis failed:", error);
    alert("Sorry, I couldn't analyze that food. Please try again.");
  }
}
```

### 5.2 Code Refactoring

**Structure**:
```
health-dashboard/
├── index.html
├── js/
│   ├── auth.js          # Authentication logic
│   ├── firestore.js     # Database operations
│   ├── ui.js            # UI rendering
│   ├── workouts.js      # Workout management
│   ├── nutrition.js     # Nutrition tracking
│   ├── achievements.js  # Gamification
│   └── main.js          # App initialization
├── css/
│   ├── theme.css        # Theme and colors
│   └── components.css   # Component styles
└── assets/
    └── doodles/         # SVG doodles
```

---

## Data Model Evolution

### Complete Firestore Structure

```
artifacts/{appId}/users/{userId}/
├── dailyLogs/{date}
│   ├── liquidCalories: number
│   ├── waterIntake: number
│   ├── protein: number
│   ├── carbs: number
│   ├── fat: number
│   ├── totalCalories: number
│   ├── meals: {
│   │   breakfast: FoodItem[],
│   │   lunch: FoodItem[],
│   │   dinner: FoodItem[],
│   │   snacks: FoodItem[]
│   ├── }
│   └── todaysWorkouts: WorkoutInstance[]
│
├── customWorkouts/{workoutId}
│   ├── id: string
│   ├── name: string
│   ├── duration: number
│   ├── exercises: Exercise[]
│   ├── createdAt: timestamp
│   └── lastModified: timestamp
│
├── exerciseHistory/{exerciseName}
│   ├── exerciseName: string
│   ├── history: Session[]
│   └── personalRecords: PRs
│
└── data/
    ├── userProfile
    │   ├── onboardingCompleted: boolean
    │   ├── primaryGoal: string
    │   ├── workoutStreak: number
    │   ├── lastWorkoutDate: string
    │   ├── waterGoal: number
    │   ├── liquidCaloriesGoal: number
    │   ├── proteinGoal: number
    │   ├── carbsGoal: number
    │   ├── fatGoal: number
    │   ├── inkColor: string
    │   └── highlightColor: string
    │
    └── achievements
        ├── unlockedAchievements: Achievement[]
        └── streaks: Streaks
```

---

## Technical Considerations

### Performance
- Lazy load exercise history
- Implement pagination for meal logs
- Use Firestore query limits
- Cache user workouts locally

### Error Handling
- Implement retry logic for API calls
- Show user-friendly error messages
- Log errors to Firebase Analytics
- Implement offline support with Firestore cache

### Testing
- Test onboarding flow
- Test workout builder
- Test API rate limits
- Test offline functionality

### Browser Compatibility
- Test on Chrome, Firefox, Safari
- Test responsive design on mobile
- Test touch interactions
- Ensure progressive enhancement

---

## Implementation Order Recommendation

### Week 1-2: Phase 1 Foundation
1. Day 1-2: Onboarding modal
2. Day 3-5: Custom workout builder
3. Day 6-8: Enhanced nutrition with macros
4. Day 9-10: Visual enhancements (doodles, torn edges)
5. Day 11-12: Animations and theme customization
6. Day 13-14: Testing and bug fixes

### Week 3-4: Phase 2 Progressive Overload
1. Detailed exercise logging UI
2. Exercise history storage
3. Personal records tracking
4. Charts and visualizations
5. Integration with rest timer

### Week 5: Phase 3 Gamification
1. Achievements system
2. Enhanced streak tracking
3. Monthly calendar view
4. Celebration animations

### Week 6: Phase 4 Settings
1. Customizable goals
2. Settings panel
3. User preferences

### Week 7-8: Phase 5 Security & Refactoring
1. Firebase Cloud Functions setup
2. Code refactoring
3. Performance optimization
4. Comprehensive testing

---

## Next Steps

1. Review this plan and prioritize features
2. Set up development environment
3. Create git repository for version control
4. Start with Phase 1, feature by feature
5. Test each feature thoroughly before moving to next
6. Gather user feedback after each phase
7. Iterate based on feedback

This plan provides a roadmap for transforming Sketch Health into a comprehensive, engaging fitness tracking application while maintaining its unique notepad aesthetic.