# Sudoku Game Analytics Documentation

## Overview
This Sudoku game includes comprehensive analytics tracking using the `AnalyticsManager` class. All game interactions, performance metrics, and player behavior are tracked and reported.

## Analytics Architecture

### AnalyticsManager.js
The core analytics system that:
- Tracks game sessions and levels
- Records user actions and tasks
- Calculates XP based on performance
- Submits reports to React Native WebView or parent frames
- Persists pending reports in localStorage when offline

## What's Being Tracked

### 1. **Session Initialization**
- **Game ID**: `NumberSudoku`
- **Session Name**: `session_{timestamp}_{randomId}`
- **Session ID**: Unique identifier for each game session
- **Timestamp**: ISO 8601 format

### 2. **Level Tracking**
Each puzzle is tracked as a "level" with identifier: `sudoku_{difficulty}_{timestamp}`

#### Level Metrics:
- `levelId`: Unique level identifier
- `successful`: Whether puzzle was completed correctly
- `timeTaken`: Time in milliseconds
- `xpEarned`: XP earned for this puzzle
- `tasks`: Array of submit attempts

### 3. **User Actions Tracked**

#### Game Controls:
- **Undo Button**: Tracks each undo operation
  - Counter: `undo_button_clicks`
  - Current count: `undo_count`

- **Erase Button**: Tracks cell erases
  - Counter: `erase_button_clicks`
  - Incremented each time erase is used

- **Notes Mode**: Tracks note-taking mode toggles
  - Counter: `notes_mode_toggles`
  - Current state: `notes_mode_active` (true/false)

- **Theme Toggle**: Tracks light/dark mode switches
  - Counter: `theme_toggles`
  - Current theme: `current_theme` (light/dark)

- **Difficulty Changes**: Tracks when player changes difficulty
  - Counter: `difficulty_changes`
  - Previous: `difficulty_changed_from`
  - New: `difficulty_changed_to`

#### Gameplay Metrics:
- **Move Tracking**:
  - `total_moves`: Total number of cells filled
  - `correct_moves`: Number of correct number placements
  - `incorrect_moves`: Number of incorrect placements
  - `moves_made`: Moves during current puzzle

- **Submit Attempts**:
  - `submit_attempts`: Number of times player checked solution
  - Each attempt recorded as a task with:
    - Question: "Submit Attempt #{number}"
    - Correct choice: "solved"
    - Choice made: "solved" or "has_errors"
    - Time taken and XP awarded

### 4. **Performance Metrics**

#### Accuracy:
- `accuracy_percent`: Overall accuracy of filled cells
- `correct_cells`: Number of correctly filled cells
- `wrong_cells`: Number of incorrectly filled cells
- `filled_cells`: Total filled cells vs 81

#### Timing:
- `time_seconds`: Total time spent on puzzle (seconds)
- `completion_time_ms`: Total completion time (milliseconds)
- `puzzle_start_time`: ISO timestamp when puzzle started

#### Difficulty:
- `difficulty`: Current difficulty level (easy/medium/hard)

### 5. **XP System**

XP is calculated based on:
- **Base XP**: 100 points
- **Time Bonus**: Up to 50 points (decreases with time)
  - Formula: `max(0, 50 - floor(timeMs / 10000))`
- **Attempt Bonus**: Up to 50 points (decreases with attempts)
  - Formula: `max(0, 50 - (submitAttempts - 1) * 10)`
- **Difficulty Multiplier**:
  - Easy: 1x
  - Medium: 1.5x
  - Hard: 2x

Final XP: `(baseXP + timeBonus + attemptBonus) * difficultyMultiplier`

#### XP Breakdown Metrics:
- `xp_breakdown_base`: Base XP earned
- `xp_breakdown_time_bonus`: Time-based bonus
- `xp_breakdown_attempt_bonus`: Attempt-based bonus
- `xp_breakdown_difficulty_multiplier`: Difficulty multiplier applied
- `xpEarnedTotal`: Total XP for session

### 6. **Completion Status**
- `puzzle_completed`: Set to "true" when puzzle is solved
- `session_abandoned`: Set to "true" if player leaves without completing

## Data Structure

### Report Format
```json
{
  "gameId": "NumberSudoku",
  "name": "session_1234567890_abc123",
  "sessionId": "1234567890-xyz789",
  "timestamp": "2026-03-16T10:30:00.000Z",
  "xpEarnedTotal": 250,
  "xpEarned": 250,
  "xpTotal": 250,
  "bestXp": 250,
  "rawData": [
    {"key": "difficulty", "value": "medium"},
    {"key": "submit_attempts", "value": "2"},
    {"key": "accuracy_percent", "value": "95.5"},
    {"key": "total_moves", "value": "45"},
    {"key": "correct_moves", "value": "43"},
    {"key": "incorrect_moves", "value": "2"},
    // ... more metrics
  ],
  "diagnostics": {
    "levels": [
      {
        "levelId": "sudoku_medium_1234567890",
        "successful": true,
        "timeTaken": 245000,
        "timeDirection": false,
        "xpEarned": 250,
        "tasks": [
          {
            "taskId": "submit_attempt_1",
            "question": "Submit Attempt #1",
            "options": "[]",
            "correctChoice": "solved",
            "choiceMade": "has_errors",
            "successful": false,
            "timeTaken": 180000,
            "xpEarned": 10
          },
          {
            "taskId": "submit_attempt_2",
            "question": "Submit Attempt #2",
            "options": "[]",
            "correctChoice": "solved",
            "choiceMade": "solved",
            "successful": true,
            "timeTaken": 245000,
            "xpEarned": 50
          }
        ]
      }
    ]
  }
}
```

## Integration Points

### React Native WebView
Analytics are automatically sent to React Native apps via:
```javascript
window.ReactNativeWebView.postMessage(JSON.stringify(payload));
```

### Parent Window (iframe)
For embedded games, analytics are sent to parent via:
```javascript
window.parent.postMessage(payload, targetOrigin);
```

### Custom Bridge
Games can define custom handlers:
```javascript
window.myJsAnalytics = {
  trackGameSession: function(payload) {
    // Custom handling
  }
};
```

### Offline Persistence
If delivery fails, reports are saved to localStorage under key:
- `ignite_pending_sessions_jsplugin`

Pending reports are flushed when:
- Browser comes online
- Page loads
- 2 seconds after submit attempt

## Console Logging

All analytics events are logged to console for debugging:
```
[Analytics] Initialized for: NumberSudoku
[Analytics] New game started: sudoku_easy_1234567890
[Analytics] Submit attempt #1 { correct: 40, wrong: 5, accuracy: 88.9%, filled: 45/81 }
[Analytics] Puzzle completed! { timeTaken: 163.45s, baseXP: 100, timeBonus: 34, attemptBonus: 40, totalXP: 174 }
[Analytics] REPORT SUBMITTED
```

## Lifecycle Events

### Game Start
1. Analytics initialized with game ID and session
2. New level started with unique levelId
3. Puzzle start time recorded
4. Difficulty and timestamp logged

### During Gameplay
1. Each move validated (correct/incorrect)
2. UI interactions tracked (undo, erase, notes)
3. Difficulty/theme changes recorded
4. Metrics updated continuously

### Submit Attempt
1. Board validation performed
2. Accuracy calculated
3. Submit attempt recorded as task
4. All current metrics logged

### Completion
1. XP calculated with bonuses
2. Level marked successful
3. Final metrics added
4. Report submitted to all bridges

### Abandonment
1. On page unload, if puzzle incomplete
2. Level marked unsuccessful
3. Session marked as abandoned
4. Report still submitted for tracking

## Usage Example

```javascript
// Analytics is automatically initialized in script.js
const analytics = new AnalyticsManager();
analytics.initialize('NumberSudoku', 'session_name', 'session_id');

// Start tracking a puzzle
analytics.startLevel('sudoku_easy_123456');

// Add custom metrics
analytics.addRawMetric('custom_metric', 'value');

// Record task/action
analytics.recordTask(
  'sudoku_easy_123456',
  'task_id',
  'Question text',
  'correct_answer',
  'user_answer',
  1000, // time in ms
  50    // XP earned
);

// Complete level
analytics.endLevel('sudoku_easy_123456', true, 180000, 200);

// Submit report
analytics.submitReport();
```

## Privacy & Data

- No personal information is collected
- No location data is tracked
- Analytics are gameplay-focused only
- Data is used for:
  - Game improvement
  - Understanding player behavior
  - Balancing difficulty
  - XP system calibration

## Testing

To view analytics output:
1. Open browser console (F12)
2. Play a complete game
3. Look for `[Analytics]` prefixed messages
4. Full report is logged when puzzle is completed
