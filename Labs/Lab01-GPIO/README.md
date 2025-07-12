# Lab 01: GPIO Control - LED Sequencing with Push Button

## 📊 Lab Info
- **Date Completed**: 13 July 2025
- **Time Invested**: 2 hours
- **Difficulty**: ⭐⭐⭐ (3/5)
- **Status**: ✅ Completed

## 🎯 Objective
Learn basic GPIO operations by controlling onboard LEDs using a push button with proper software debouncing.

## 🔧 Hardware Used
- **Board**: STM32F4 Discovery Kit
- **LEDs**: PD12 (Green), PD13 (Orange), PD14 (Red), PD15 (Blue)
- **Button**: PA0 (User Button - Blue)
- **IDE**: STM32CubeIDE

## 💻 Implementation Overview

### GPIO Configuration
```c
// Output Pins (LEDs)
PD12, PD13, PD14, PD15 → GPIO_MODE_OUTPUT_PP

// Input Pin (Button)  
PA0 → GPIO_MODE_INPUT with GPIO_NOPULL
```

### Algorithm Flow
1. Initialize GPIO ports and pins
2. Read button state continuously
3. Detect button press (rising edge detection)
4. Implement 50ms software debouncing
5. Increment counter and control LEDs sequentially
6. Reset counter after reaching maximum (4)

### LED Sequence
| Counter | LED State | GPIO Pin | Color |
|---------|-----------|----------|-------|
| 0 | All OFF | All RESET | None |
| 1 | LED1 ON | PD13 SET | Orange |
| 2 | LED2 ON | PD12 SET | Green |
| 3 | LED3 ON | PD14 SET | Red |
| 4 | LED4 ON | PD15 SET | Blue |

## 🐛 Problems Encountered & Solutions

### Problem 1: All LEDs Lighting Simultaneously ❌
**Issue**: All four LEDs were turning on at once instead of sequentially.

**Investigation**:
- ✅ GPIO pin assignments correct
- ✅ Counter logic working
- ❌ Switch-case structure had issues

**Root Cause**: Missing `break;` statements causing fall-through behavior.

**Before (Broken)**:
```c
switch (counter) {
    case 1: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_13, GPIO_PIN_SET); counter++;
    case 2: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET); counter++;
    case 3: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_14, GPIO_PIN_SET); counter++;
    // All cases execute due to fall-through!
}
```

**After (Fixed)** ✅:
```c
switch (counter) {
    case 1: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_13, GPIO_PIN_SET); counter++; break;
    case 2: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET); counter++; break;
    case 3: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_14, GPIO_PIN_SET); counter++; break;
    case 4: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_15, GPIO_PIN_SET); counter = 0; break;
}
```

**💡 Learning**: Always use `break;` statements in switch-case unless fall-through is intentionally required.

---

### Problem 2: Button Press Not Detected ❌
**Issue**: `time_pressed` variable remained zero, button presses weren't being registered.

**Root Cause**: Used assignment operator (`=`) instead of comparison operator (`==`).

**Before (Broken)**:
```c
if (pushButton_status = 1 && previous_pushButton_status == 0) {
    time_pressed = HAL_GetTick();  // Never executed!
}
```

**After (Fixed)** ✅:
```c
if (pushButton_status == 1 && previous_pushButton_status == 0) {
    time_pressed = HAL_GetTick();  // Now works correctly
}
```

**💡 Learning**: C compiler allows assignment in conditions, but it's usually unintended. Always use `==` for comparison, `=` for assignment.

---

### Problem 3: Inconsistent Debouncing ❌
**Issue**: Debounce mechanism wasn't working reliably.

**Root Cause**: `previous_pushButton_status` was being updated in multiple places, causing state confusion.

**Before (Broken)**:
```c
while (1) {
    pushButton_status = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0);
    previous_pushButton_status = pushButton_status;  // ❌ Too early!
    
    // Button logic here...
    
    previous_pushButton_status = pushButton_status;  // ❌ Duplicated!
}
```

**After (Fixed)** ✅:
```c
while (1) {
    pushButton_status = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0);
    
    // All button logic here...
    
    previous_pushButton_status = pushButton_status;  // ✅ Single update at end
}
```

**💡 Learning**: State variables should be updated once per loop iteration, typically at the end of the loop.

## 🔍 Key Code Sections

### Main Control Logic
```c
uint8_t counter = 0;
uint32_t time_pressed = 0;
uint8_t pushButton_status = 0;
uint8_t previous_pushButton_status = 0;
uint32_t debounce_threshold = 50;

while (1) {
    pushButton_status = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0);
    
    // Rising edge detection (button press)
    if (pushButton_status == 1 && previous_pushButton_status == 0) {
        time_pressed = HAL_GetTick();
    }
    
    // Falling edge detection with debouncing (button release)
    if (pushButton_status == 0 && previous_pushButton_status == 1) {
        if (HAL_GetTick() - time_pressed >= debounce_threshold) {
            switch (counter) {
                case 0: reset_gpio(); counter++; break;
                case 1: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_13, GPIO_PIN_SET); counter++; break;
                case 2: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_SET); counter++; break;
                case 3: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_14, GPIO_PIN_SET); counter++; break;
                case 4: HAL_GPIO_WritePin(GPIOD, GPIO_PIN_15, GPIO_PIN_SET); counter = 0; break;
                default: counter = 0;
            }
        }
    }
    
    previous_pushButton_status = pushButton_status;
}
```

### GPIO Reset Function
```c
void reset_gpio(){
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_12, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_13, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_14, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(GPIOD, GPIO_PIN_15, GPIO_PIN_RESET);
}
```

## 🧪 Testing & Validation

**Test Cases Performed**:
- ✅ Single button press advances to next LED
- ✅ Rapid button presses are properly debounced
- ✅ Counter resets to 0 after reaching maximum (4)
- ✅ All LEDs turn off when counter = 0
- ✅ System responds within debounce threshold (50ms)


## 📚 Key Learning Points

1. **GPIO Configuration**: Understanding input/output pin setup using HAL functions
2. **Edge Detection**: Implementing rising/falling edge detection for user input
3. **Software Debouncing**: Timer-based debouncing to eliminate mechanical switch bounce
4. **State Management**: Proper handling of previous states for reliable edge detection
5. **C Programming Pitfalls**:
   - Switch-case fall-through behavior
   - Assignment vs comparison operators
   - Variable scope and update timing

## 🔧 Hardware Configuration

**Pin Mapping**:
```
PA0  → User Button (Blue) - Input
PD12 → LED4 (Green)       - Output  
PD13 → LED3 (Orange)      - Output
PD14 → LED5 (Red)         - Output
PD15 → LED6 (Blue)        - Output
```



*Completed: 13 July 2025 | Time: 2 hours | Status: ✅ Working*
