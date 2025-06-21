# How to Implement State Machine Programming in TwinCAT
### Autor: Hristo Ganchev (https://github.com/hganchev)
#### Date: 2025-21-06

![Alt text](StateMachine.webp)

*Master the art of structured automation programming with state machines - the foundation of robust industrial control systems*

## Table of Contents
- [Introduction](#introduction)
- [What is a State Machine?](#what-is-a-state-machine)
- [State Machine Design Patterns](#state-machine-design-patterns)
- [Implementation with CASE Statements](#implementation-with-case-statements)
- [Error Handling in State Machines](#error-handling-in-state-machines)
- [Practical Example 1: Traffic Light Controller](#practical-example-1-traffic-light-controller)
- [Practical Example 2: Conveyor Control System](#practical-example-2-conveyor-control-system)
- [Best Practices](#best-practices)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)
- [Conclusion](#conclusion)

## Introduction

State machine programming is one of the most powerful and widely-used programming patterns in industrial automation. Whether you're controlling a simple traffic light or a complex manufacturing line, state machines provide a structured, predictable way to manage system behavior.

In this comprehensive guide, we'll explore how to implement robust state machines in TwinCAT 3, covering everything from basic concepts to advanced error handling techniques.

## What is a State Machine?

A state machine is a programming model that describes the behavior of a system using:
- **States**: Distinct conditions or modes of operation
- **Transitions**: Rules for moving between states
- **Events**: Triggers that cause state changes
- **Actions**: Operations performed in each state

### Benefits of State Machine Programming
- **Predictable behavior**: Clear understanding of system states
- **Easy debugging**: Simple to trace execution flow
- **Maintainable code**: Structured and readable
- **Safety**: Controlled state transitions prevent unexpected behavior

## State Machine Design Patterns

### 1. Simple Linear State Machine
Best for sequential processes with clear start-to-finish flow.

```iecst
TYPE E_SimpleStates :
(
    STATE_INIT := 0,
    STATE_START,
    STATE_PROCESS,
    STATE_COMPLETE,
    STATE_ERROR := 99
);
END_TYPE
```

### 2. Hierarchical State Machine
For complex systems with sub-states and parallel operations.

```iecst
TYPE E_MainStates :
(
    MAIN_IDLE := 0,
    MAIN_RUNNING,
    MAIN_MAINTENANCE,
    MAIN_ERROR := 99
);
END_TYPE

TYPE E_RunningSubStates :
(
    RUN_LOADING := 100,
    RUN_PROCESSING,
    RUN_UNLOADING
);
END_TYPE
```

### 3. Event-Driven State Machine
Responds to external events and conditions.

```iecst
TYPE E_Events :
(
    EVENT_NONE := 0,
    EVENT_START_BUTTON,
    EVENT_STOP_BUTTON,
    EVENT_EMERGENCY_STOP,
    EVENT_CYCLE_COMPLETE
);
END_TYPE
```

## Implementation with CASE Statements

### Basic State Machine Structure

Here's the fundamental structure for implementing a state machine in TwinCAT:

```iecst
PROGRAM StateMachine
VAR
    eCurrentState    : E_States := STATE_INIT;
    eLastState       : E_States;
    bStateChanged    : BOOL;
    tStateTimer      : TON;
    bStartRequest    : BOOL;
    bStopRequest     : BOOL;
    bErrorActive     : BOOL;
END_VAR

// Detect state changes
bStateChanged := (eCurrentState <> eLastState);
eLastState := eCurrentState;

// Reset timer on state change
IF bStateChanged THEN
    tStateTimer(IN := FALSE);
END_IF

// State machine execution
CASE eCurrentState OF
    
    STATE_INIT:
        // Initialization actions
        IF bStartRequest AND NOT bErrorActive THEN
            eCurrentState := STATE_RUNNING;
        END_IF
    
    STATE_RUNNING:
        tStateTimer(IN := TRUE, PT := T#5S);
        
        // Running actions here
        
        IF bStopRequest THEN
            eCurrentState := STATE_STOPPING;
        ELSIF bErrorActive THEN
            eCurrentState := STATE_ERROR;
        ELSIF tStateTimer.Q THEN
            eCurrentState := STATE_COMPLETE;
        END_IF
    
    STATE_STOPPING:
        // Stop sequence actions
        IF (* stop conditions met *) THEN
            eCurrentState := STATE_IDLE;
        END_IF
    
    STATE_COMPLETE:
        // Completion actions
        eCurrentState := STATE_IDLE;
    
    STATE_IDLE:
        // Idle state actions
        IF bStartRequest THEN
            eCurrentState := STATE_INIT;
        END_IF
    
    STATE_ERROR:
        // Error handling
        IF (* error cleared *) THEN
            eCurrentState := STATE_IDLE;
        END_IF
        
ELSE
    // Default case - should never reach here
    eCurrentState := STATE_ERROR;
END_CASE

// Call timer function block
tStateTimer();
```

### Advanced State Machine with Function Block

For reusable state machines, create a function block:

```iecst
FUNCTION_BLOCK FB_StateMachine
VAR_INPUT
    bEnable         : BOOL;
    bReset          : BOOL;
    bStart          : BOOL;
    bStop           : BOOL;
END_VAR

VAR_OUTPUT
    eCurrentState   : E_States;
    bBusy          : BOOL;
    bDone          : BOOL;
    bError         : BOOL;
    nErrorID       : UDINT;
END_VAR

VAR
    eState         : E_States := STATE_INIT;
    tStateTimer    : TON;
    bStateChanged  : BOOL;
    eLastState     : E_States;
END_VAR

// State change detection
bStateChanged := (eState <> eLastState);
eLastState := eState;

// Reset outputs
bDone := FALSE;
bError := FALSE;
nErrorID := 0;

IF NOT bEnable THEN
    eState := STATE_DISABLED;
    RETURN;
END_IF

IF bReset THEN
    eState := STATE_INIT;
END_IF

// Main state machine logic
CASE eState OF
    // ... implement states here
END_CASE

// Update outputs
eCurrentState := eState;
bBusy := (eState > STATE_IDLE) AND (eState < STATE_ERROR);
```

## Error Handling in State Machines

### Centralized Error Handling

```iecst
FUNCTION_BLOCK FB_ErrorHandler
VAR_INPUT
    bErrorCondition1    : BOOL;
    bErrorCondition2    : BOOL;
    bErrorCondition3    : BOOL;
    bAcknowledge       : BOOL;
END_VAR

VAR_OUTPUT
    bErrorActive       : BOOL;
    nErrorCode        : UDINT;
    sErrorMessage     : STRING(255);
END_VAR

VAR
    bErrorLatch       : BOOL;
END_VAR

// Error detection
IF bErrorCondition1 THEN
    bErrorLatch := TRUE;
    nErrorCode := 1001;
    sErrorMessage := 'Sensor 1 fault detected';
ELSIF bErrorCondition2 THEN
    bErrorLatch := TRUE;
    nErrorCode := 1002;
    sErrorMessage := 'Motor overload protection';
ELSIF bErrorCondition3 THEN
    bErrorLatch := TRUE;
    nErrorCode := 1003;
    sErrorMessage := 'Communication timeout';
END_IF

// Error reset
IF bAcknowledge AND bErrorLatch THEN
    bErrorLatch := FALSE;
    nErrorCode := 0;
    sErrorMessage := '';
END_IF

bErrorActive := bErrorLatch;
```

### Error Recovery Patterns

```iecst
TYPE E_ErrorStates :
(
    ERROR_NONE := 0,
    ERROR_DETECTED,
    ERROR_RECOVERY,
    ERROR_MANUAL_INTERVENTION
);
END_TYPE

VAR
    eErrorState : E_ErrorStates;
    nRetryCount : INT;
    nMaxRetries : INT := 3;
END_VAR

CASE eErrorState OF
    ERROR_NONE:
        // Normal operation
        
    ERROR_DETECTED:
        // Stop current operation
        // Log error
        nRetryCount := nRetryCount + 1;
        
        IF nRetryCount <= nMaxRetries THEN
            eErrorState := ERROR_RECOVERY;
        ELSE
            eErrorState := ERROR_MANUAL_INTERVENTION;
        END_IF
    
    ERROR_RECOVERY:
        // Attempt automatic recovery
        IF (* recovery successful *) THEN
            eErrorState := ERROR_NONE;
            nRetryCount := 0;
        ELSE
            eErrorState := ERROR_DETECTED;
        END_IF
    
    ERROR_MANUAL_INTERVENTION:
        // Wait for operator intervention
        IF bOperatorReset THEN
            eErrorState := ERROR_NONE;
            nRetryCount := 0;
        END_IF
END_CASE
```

## Practical Example 1: Traffic Light Controller

Let's implement a complete traffic light control system:

```iecst
TYPE E_TrafficStates :
(
    TRAFFIC_OFF := 0,
    TRAFFIC_INIT,
    TRAFFIC_RED,
    TRAFFIC_RED_YELLOW,
    TRAFFIC_GREEN,
    TRAFFIC_YELLOW,
    TRAFFIC_FLASH_RED,
    TRAFFIC_ERROR := 99
);
END_TYPE

PROGRAM TrafficLightController
VAR
    eCurrentState       : E_TrafficStates := TRAFFIC_OFF;
    eLastState         : E_TrafficStates;
    bStateChanged      : BOOL;
    
    // Timers
    tRedTimer          : TON;
    tRedYellowTimer    : TON;
    tGreenTimer        : TON;
    tYellowTimer       : TON;
    tFlashTimer        : TON;
    
    // Time constants
    tRedTime           : TIME := T#30S;
    tRedYellowTime     : TIME := T#2S;
    tGreenTime         : TIME := T#25S;
    tYellowTime        : TIME := T#3S;
    tFlashInterval     : TIME := T#500MS;
    
    // Inputs
    bSystemEnable      : BOOL;
    bEmergencyStop     : BOOL;
    bMaintenanceMode   : BOOL;
    
    // Outputs
    bRedLight          : BOOL;
    bYellowLight       : BOOL;
    bGreenLight        : BOOL;
    
    // Status
    bFlashState        : BOOL;
END_VAR

// State change detection
bStateChanged := (eCurrentState <> eLastState);
eLastState := eCurrentState;

// Reset all lights
bRedLight := FALSE;
bYellowLight := FALSE;
bGreenLight := FALSE;

// Main state machine
CASE eCurrentState OF
    
    TRAFFIC_OFF:
        // All lights off
        IF bSystemEnable AND NOT bEmergencyStop THEN
            eCurrentState := TRAFFIC_INIT;
        END_IF
    
    TRAFFIC_INIT:
        // Initialization sequence
        bRedLight := TRUE;
        tRedTimer(IN := TRUE, PT := T#2S);
        
        IF tRedTimer.Q THEN
            eCurrentState := TRAFFIC_RED;
        END_IF
    
    TRAFFIC_RED:
        bRedLight := TRUE;
        tRedTimer(IN := TRUE, PT := tRedTime);
        
        IF tRedTimer.Q THEN
            eCurrentState := TRAFFIC_RED_YELLOW;
        END_IF
    
    TRAFFIC_RED_YELLOW:
        bRedLight := TRUE;
        bYellowLight := TRUE;
        tRedYellowTimer(IN := TRUE, PT := tRedYellowTime);
        
        IF tRedYellowTimer.Q THEN
            eCurrentState := TRAFFIC_GREEN;
        END_IF
    
    TRAFFIC_GREEN:
        bGreenLight := TRUE;
        tGreenTimer(IN := TRUE, PT := tGreenTime);
        
        IF tGreenTimer.Q THEN
            eCurrentState := TRAFFIC_YELLOW;
        END_IF
    
    TRAFFIC_YELLOW:
        bYellowLight := TRUE;
        tYellowTimer(IN := TRUE, PT := tYellowTime);
        
        IF tYellowTimer.Q THEN
            eCurrentState := TRAFFIC_RED;
        END_IF
    
    TRAFFIC_FLASH_RED:
        // Emergency flashing red
        tFlashTimer(IN := TRUE, PT := tFlashInterval);
        
        IF tFlashTimer.Q THEN
            bFlashState := NOT bFlashState;
            tFlashTimer(IN := FALSE);
        END_IF
        
        bRedLight := bFlashState;
        
        IF NOT bEmergencyStop AND bSystemEnable THEN
            eCurrentState := TRAFFIC_INIT;
        END_IF
    
    TRAFFIC_ERROR:
        // Error state - flash all lights
        tFlashTimer(IN := TRUE, PT := T#250MS);
        
        IF tFlashTimer.Q THEN
            bFlashState := NOT bFlashState;
            tFlashTimer(IN := FALSE);
        END_IF
        
        bRedLight := bFlashState;
        bYellowLight := bFlashState;
        bGreenLight := bFlashState;

ELSE
    eCurrentState := TRAFFIC_ERROR;
END_CASE

// Global conditions
IF bEmergencyStop OR bMaintenanceMode THEN
    eCurrentState := TRAFFIC_FLASH_RED;
    // Reset all timers
    tRedTimer(IN := FALSE);
    tRedYellowTimer(IN := FALSE);
    tGreenTimer(IN := FALSE);
    tYellowTimer(IN := FALSE);
ELSIF NOT bSystemEnable THEN
    eCurrentState := TRAFFIC_OFF;
END_IF

// Reset timers on state change
IF bStateChanged THEN
    tRedTimer(IN := FALSE);
    tRedYellowTimer(IN := FALSE);
    tGreenTimer(IN := FALSE);
    tYellowTimer(IN := FALSE);
    tFlashTimer(IN := FALSE);
END_IF

// Execute timer function blocks
tRedTimer();
tRedYellowTimer();
tGreenTimer();
tYellowTimer();
tFlashTimer();
```

## Practical Example 2: Conveyor Control System

A more complex example with multiple conveyors and sensors:

```iecst
TYPE E_ConveyorStates :
(
    CONV_DISABLED := 0,
    CONV_INIT,
    CONV_IDLE,
    CONV_LOADING,
    CONV_RUNNING,
    CONV_UNLOADING,
    CONV_STOPPING,
    CONV_CLEANING,
    CONV_ERROR := 99
);
END_TYPE

FUNCTION_BLOCK FB_ConveyorControl
VAR_INPUT
    bEnable            : BOOL;
    bReset             : BOOL;
    bStart             : BOOL;
    bStop              : BOOL;
    bCleaningMode      : BOOL;
    
    // Sensors
    bLoadingSensor     : BOOL;
    bUnloadingSensor   : BOOL;
    bProductPresent    : BOOL;
    bMotorOverload     : BOOL;
    bEstopPressed      : BOOL;
END_VAR

VAR_OUTPUT
    eCurrentState      : E_ConveyorStates;
    bMotorRun          : BOOL;
    bLoadingValve      : BOOL;
    bUnloadingValve    : BOOL;
    bStatusLED         : BOOL;
    bAlarmLight        : BOOL;
    
    // Status
    bReady             : BOOL;
    bBusy              : BOOL;
    bError             : BOOL;
    nErrorCode         : UDINT;
END_VAR

VAR
    eState             : E_ConveyorStates := CONV_DISABLED;
    eLastState         : E_ConveyorStates;
    bStateChanged      : BOOL;
    
    // Timers
    tLoadingTimer      : TON;
    tRunTimer          : TON;
    tUnloadingTimer    : TON;
    tStopTimer         : TON;
    tCleaningTimer     : TON;
    
    // Parameters
    tLoadingTime       : TIME := T#5S;
    tRunTime           : TIME := T#30S;
    tUnloadingTime     : TIME := T#3S;
    tStopTime          : TIME := T#2S;
    tCleaningTime      : TIME := T#60S;
    
    // Internal flags
    bCycleComplete     : BOOL;
    nProductCount      : UDINT;
END_VAR

// State change detection
bStateChanged := (eState <> eLastState);
eLastState := eState;

// Reset outputs
bMotorRun := FALSE;
bLoadingValve := FALSE;
bUnloadingValve := FALSE;
bStatusLED := FALSE;
bAlarmLight := FALSE;
bReady := FALSE;
bBusy := FALSE;
bError := FALSE;
nErrorCode := 0;

// Main state machine
CASE eState OF
    
    CONV_DISABLED:
        bAlarmLight := TRUE;
        
        IF bEnable AND NOT bEstopPressed THEN
            eState := CONV_INIT;
        END_IF
    
    CONV_INIT:
        bStatusLED := TRUE;
        
        // Initialize system
        nProductCount := 0;
        bCycleComplete := FALSE;
        
        // Check safety conditions
        IF bMotorOverload THEN
            eState := CONV_ERROR;
            nErrorCode := 2001;
        ELSE
            eState := CONV_IDLE;
        END_IF
    
    CONV_IDLE:
        bReady := TRUE;
        bStatusLED := TRUE;
        
        IF bStart AND bLoadingSensor THEN
            eState := CONV_LOADING;
        ELSIF bCleaningMode THEN
            eState := CONV_CLEANING;
        END_IF
    
    CONV_LOADING:
        bBusy := TRUE;
        bLoadingValve := TRUE;
        tLoadingTimer(IN := TRUE, PT := tLoadingTime);
        
        IF tLoadingTimer.Q AND bProductPresent THEN
            eState := CONV_RUNNING;
        ELSIF tLoadingTimer.Q AND NOT bProductPresent THEN
            eState := CONV_ERROR;
            nErrorCode := 2002; // Loading timeout
        END_IF
    
    CONV_RUNNING:
        bBusy := TRUE;
        bMotorRun := TRUE;
        bStatusLED := TRUE;
        tRunTimer(IN := TRUE, PT := tRunTime);
        
        IF bUnloadingSensor THEN
            eState := CONV_UNLOADING;
        ELSIF tRunTimer.Q THEN
            eState := CONV_ERROR;
            nErrorCode := 2003; // Product not reaching unloading position
        ELSIF bStop THEN
            eState := CONV_STOPPING;
        END_IF
    
    CONV_UNLOADING:
        bBusy := TRUE;
        bUnloadingValve := TRUE;
        tUnloadingTimer(IN := TRUE, PT := tUnloadingTime);
        
        IF tUnloadingTimer.Q THEN
            nProductCount := nProductCount + 1;
            bCycleComplete := TRUE;
            eState := CONV_IDLE;
        END_IF
    
    CONV_STOPPING:
        bStatusLED := TRUE;
        tStopTimer(IN := TRUE, PT := tStopTime);
        
        // Controlled stop sequence
        IF tStopTimer.Q THEN
            eState := CONV_IDLE;
        END_IF
    
    CONV_CLEANING:
        bMotorRun := TRUE;
        tCleaningTimer(IN := TRUE, PT := tCleaningTime);
        
        // Cleaning cycle with reversed direction
        IF tCleaningTimer.Q OR NOT bCleaningMode THEN
            eState := CONV_IDLE;
        END_IF
    
    CONV_ERROR:
        bError := TRUE;
        bAlarmLight := TRUE;
        
        // Error recovery
        IF bReset AND NOT bMotorOverload THEN
            eState := CONV_INIT;
        END_IF

ELSE
    eState := CONV_ERROR;
    nErrorCode := 2999; // Unknown state
END_CASE

// Global safety conditions
IF bEstopPressed OR NOT bEnable THEN
    eState := CONV_DISABLED;
ELSIF bMotorOverload AND (eState <> CONV_ERROR) THEN
    eState := CONV_ERROR;
    nErrorCode := 2001;
END_IF

// Reset timers on state change
IF bStateChanged THEN
    tLoadingTimer(IN := FALSE);
    tRunTimer(IN := FALSE);
    tUnloadingTimer(IN := FALSE);
    tStopTimer(IN := FALSE);
    tCleaningTimer(IN := FALSE);
END_IF

// Update outputs
eCurrentState := eState;

// Execute timers
tLoadingTimer();
tRunTimer();
tUnloadingTimer();
tStopTimer();
tCleaningTimer();
```

## Best Practices

### 1. State Naming Conventions
```iecst
// Use descriptive prefixes
TYPE E_PumpStates :
(
    PUMP_OFF := 0,          // Clear prefix
    PUMP_STARTING,          // Action-based names
    PUMP_RUNNING,
    PUMP_STOPPING,
    PUMP_MAINTENANCE,
    PUMP_ERROR := 99        // Error state at high value
);
END_TYPE
```

### 2. Timeout Management
```iecst
// Always use timeouts for safety
IF tOperationTimer.Q THEN
    eState := STATE_ERROR;
    nErrorCode := 1001; // Operation timeout
END_IF
```

### 3. State Documentation
```iecst
CASE eCurrentState OF
    STATE_INIT:
        (* 
         * Purpose: Initialize system components
         * Entry: System enabled and no errors
         * Exit: All components ready
         * Timeout: 10 seconds
         *)
        // Implementation here
```

### 4. Consistent Error Handling
```iecst
// Standard error code ranges
// 1000-1999: Initialization errors
// 2000-2999: Operation errors  
// 3000-3999: Communication errors
// 4000-4999: Safety errors
```

## Troubleshooting Common Issues

### Issue 1: State Machine Stuck
**Problem**: State machine doesn't transition between states
**Solution**: 
- Check transition conditions
- Verify timer configurations
- Add debug variables to monitor conditions

```iecst
VAR
    // Debug variables
    bDebugTransition1  : BOOL;
    bDebugTransition2  : BOOL;
    nDebugCounter     : UDINT;
END_VAR

// Add debug monitoring
bDebugTransition1 := (bStartCondition AND NOT bErrorActive);
nDebugCounter := nDebugCounter + 1; // Verify execution
```

### Issue 2: Unexpected State Changes
**Problem**: State changes occur unexpectedly
**Solution**:
- Implement state change logging
- Add validation for state transitions

```iecst
VAR
    aStateHistory : ARRAY[0..9] OF E_States;
    nHistoryIndex : INT;
END_VAR

// Log state changes
IF bStateChanged THEN
    aStateHistory[nHistoryIndex] := eCurrentState;
    nHistoryIndex := (nHistoryIndex + 1) MOD 10;
END_IF
```

### Issue 3: Timer Issues
**Problem**: Timers not resetting properly
**Solution**:
- Always reset timers on state change
- Use consistent timer naming

```iecst
// Centralized timer reset
IF bStateChanged THEN
    tTimer1(IN := FALSE);
    tTimer2(IN := FALSE);
    tTimer3(IN := FALSE);
END_IF
```

## Conclusion

State machine programming in TwinCAT provides a robust foundation for industrial automation projects. By following the patterns and examples in this guide, you'll be able to create maintainable, predictable, and safe automation systems.

### Key Takeaways:
- Always use enumerated types for states
- Implement proper error handling and recovery
- Use timers for safety and timeout protection
- Document state purposes and transitions
- Test all possible state combinations

### Next Steps:
- Practice with the provided examples
- Implement state machines in your current projects
- Explore advanced patterns like hierarchical state machines
- Consider using TwinCAT's UML state chart programming for complex systems

---

**Need Help?** If you're implementing state machines in your TwinCAT project and need assistance, feel free to reach out or check the related articles on error handling and diagnostics.

*Last updated: June 20, 2025*