# 16-Bit Timer/Counters #

## Overview over the hardware ##

The ATmega328p has one 16-Bit Timer/Counter (Timer/Counter1), and the 
Atmega2560 has four 16-Bit Timer/Counters (Timer/Counter1, 3, 4, 5).

The heart of a Timer/Counter is a Register holding the actual
count-value (TCNTn). This value is incremented/decremented by
internal clock-pulses. These clock-pulses can be generated by
either

- rising or falling edges on a special pin called Tn (Counter mode), or
- the main-oscillator frequency divived by a prescaler-value of 
  1, 8, 64, 256 or 1024 (Timer mode).
  
The 16-Bit-Tiemr/Counters have the following modes of operation:

- normal mode
- CTC-mode (Clear timer on Compare-match)
- Fast PWM-mode
- Phase correct PWM-mode
- Phase and frequency-correct PWM-mode

The mode of operation determines the count-sequence (only up, or up and down),
and if the count-value is only incremented up to a special TOP-value. 

### Compare-Match-Channels and Interrupts ###

There are two (ATmega328p) or three (Atmega2560) Compare-Match-Channels. Each
of these channels has a Compare-match-Register (OCRnA, OCRnB, OCRnC). If the
actual count-value in the TCNTn-Register is equal to one of the 
Compare-match-registers an interrupt-event occurs.

Another interrupt-event is the overflow of the 16-Bit-TCNTn-Register, which 
occurs when the register contains a value of 65535 and is incremented. The
TCNTn-value then overflows to 0.

### Hardware PWM-outputs ###
A Compare-Match of each Channel can change the voltage-level of a special 
PWM-pin. For example a Compare-Match of Channel B of Timer/Counter1 changes the
state of the pin with the alternative function OC1B (On the ATmega328p, OC1B is
the alternative function of second GPIO-Pin on port B, pin PB2). Depending on
the used mode of operation the pin is set, cleared or toggled on a 
Compare-Match, or a PWM-signal is generated. The PWM-signal can be inverted or
not inverted.

### Further details ###
Input Capture is not yet supported by this library.

For further details on the Timer/Counter-hardware, refer to the datatsheet of 
the microcontroller.


## Using the 16-Bit-Timer/Counter module ##

Add the files `GpioPinMacros.h`, `Timer16Bit.h` and `Timer16Bit.cpp` to your 
project, and `#include Timer16Bit.h`.

First, a local or global Timer/Counter-object must be created:
```C
TimerCounter16Bit tc1 = makeTimerCounter16BitObject( 1 );
```
Replace 1 with the number of the Timer/Counter to be used.

Set the mode of the Timer/Counter. Replace `T16_NORMAL` with the appropriate 
constant from the enum `Timer16_mode`, to use another mode of operation:
```C
tc1.setMode( T16_NORMAL );
```

Select a the clock-source (CPU-oscillator-frequency divided by a 
prescaler-value, or rising/falling edges on the Tn-pin). Use one of the 
constants of the enum `Timer16_ClockSource`. After selecting the clock-source, 
the Timer/Counter starts. Therefore this method is best called at the end of
the Timer/Counter-Initialization:
```C
tc1.selectClockSource( T16_PRESC_1024 );
```

To get the actual count-Value (content of the TCNTn-Register) use:
```C
uint16_t i = tc1.getActualCountValue();
```

In some modes of operation a TOP-count-value can be set. If the 
TOP-count-value can be set, it is stored either in the 
special-function-register OCRnA or in ICRn. The `setTopValue`-method chooses
the correct register, according to the previously set mode of operation.

```C
tc1.setTopValue(39999);
```

Compare-Match-Values for the two/three Compare-Match-channels can be set. These 
values are stored in the registers OCRnA, OCRnB or OCRnC, respectively. For the 
first parameter of `setCompareMatchValue` use one of the constants defined in 
enum `Timer16_CompChannel`. The second parameter is the unsigned 16-bit-wide 
Compare-Match-value:
```C
tc1.setCompareMatchValue( T16_COMP_B, 39999 );
```

To enable interrupts use the mehtod `enableInterrupts`. Interrupts must also 
globally be enabled with `sei();`, and an ISR must be implemented for each
interrupt. Use a bitwise-or of contants of the enum `Timer16_Interrupts` as 
argument. For example, to enable the Compare-Match-A and the 
Compare-Match-B-Interrupt of the Timer/Counter use:
```C
tc1.enableInterrupts( T16_INT_COMP_MATCH_A | T16_INT_COMP_MATCH_B );
```
For interrupts also the methods `disableInterrupts` and 
`clearPendingInterruptEvents` can be useful.

For each compare-match-channel there is a PWM-Pin. To generate 
hardware-PWM-signals on the PWM-pins OCnA, OCnB and OCnC the method
`setPwmPinMode` can be used. The first parameter defines the 
Output-Compare-Channel A, B or C, the second parameter defines, how the 
PWM-Signal is generated. For example, to create an inverted PWM-Signal
on Pin OC1B, use:
```C
tc1.setPwmPinMode(T16_COMP_B, T16_PIN_PWM_INVERTED);
```
Use a constant of the enum `Timer16_CompChannel` for the first parameter, and 
one of the constants of the enum `Timer16_PwmPinMode` for the sedond parameter.

After power-up the PWM-Pins are normal GPIO-Pins, configured as inputs. 
A PWM-Pin must also be configured as output, for example by using 
the macro `setGpioPinModeOutput` from `GpioPinMacros.h`. Before doing so, the
initial state (initial High- or Low-voltage-level of the PWM-pin) can be set by
forcing a Compare-Match using the method `forceOutputCompareMatch`. (See
datasheet and example `exampleTimer16Bit_PWM.cpp` for details).