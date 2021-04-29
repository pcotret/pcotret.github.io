# Arduino vs. embedded C - AVR reversing


> As I was teaching some embedded C basics, I was asked what are some benefits of embedded C over the classic Arduino language for an Arduino-based board. This article tries to see what we can do by reversing a **really** simple program compiled with both methods for the Arduino Uno.

## Prerequisites
- [`arduino-cli`](https://github.com/arduino/arduino-cli): Command Line Interface for Arduino
- AVR cross compiler: `sudo apt install gcc-avr`
- Optional: an Arduino simulator such as [SimulIDE](https://simulide.blogspot.com/)

## Sample program

![](./img/arduino_blink.jpg)

We want to create the most simple program which goal is to light on the built-in LED, located at port 13 (or PORT PB5) on the Arduino Uno.

### Arduino

```c
void setup()
{
    pinMode(LED_BUILTIN, OUTPUT);
}
void loop()
{
	digitalWrite(LED_BUILTIN, HIGH);
}
```
### Embedded C
```c
#include <avr/io.h>
int main()
{
    DDRB = 1 << PB5;
    PORTB = 1 << PB5;
    return 0;
}
```

### Codes and compilation

```bash
$ ls -lR
./led-arduino:
total 4
-rw-r--r-- 1 pascal pascal 104 mai    1 12:58 led-arduino.ino
./led-embedded:
total 4
-rw-r--r-- 1 pascal pascal 81 mai    1 13:00 led-embedded.ino
$ arduino-cli compile --fqbn arduino:avr:uno --output-dir led-arduino led-arduino
$ arduino-cli compile --fqbn arduino:avr:uno --output-dir led-embedded led-embedded
```

## Binary comparison

|             | Arduino   | Embedded C |
| ----------- | --------- | ---------- |
| Storage use | 724 bytes | 144 bytes  |

The embedded C code is 5 times smaller than the Arduino one which is  a bit "weird" as both codes do the same thing! Let's find why by reversing binaries and analyzing assembly codes.

`arduino-cli` produces the `*.elf` binary and the `*.hex` file which is just a series of bytes to be loaded in the Arduino.

## Reversing binaries

```bash
$ avr-objdump -S led-arduino/led-arduino.ino.elf > led-arduino.asm
$ avr-objdump -S led-embedded/led-embedded.ino.elf > led-embedded.asm
```

### Arduino binary

```assembly
void init()
{
	// this needs to be called before setup() or some functions won't
	// work there
	sei();
 174:	78 94       	sei
	
	// on the ATmega168, timer 0 is also used for fast hardware pwm
	// (using phase-correct PWM would mean that timer 0 overflowed half as often
	// resulting in different millis() behavior on the ATmega8 and ATmega168)
#if defined(TCCR0A) && defined(WGM01)
	sbi(TCCR0A, WGM01);
 176:	84 b5       	in	r24, 0x24	; 36
 178:	82 60       	ori	r24, 0x02	; 2
 17a:	84 bd       	out	0x24, r24	; 36
	sbi(TCCR0A, WGM00);
 17c:	84 b5       	in	r24, 0x24	; 36
 17e:	81 60       	ori	r24, 0x01	; 1
 180:	84 bd       	out	0x24, r24	; 36
	// this combination is for the standard atmega8
[...]
```

Here is a part of the Arduino objdump. The assembly code is really long for such a program... It is easy to understand when we look at the source code of `pinMode()` and `digitalWrite()` (https://github.com/arduino/ArduinoCore-avr/blob/master/cores/arduino/wiring_digital.c). It does not only write a value into a register...

Garretlab made an analysis of both functions:

- https://garretlab.web.fc2.com/en/arduino/inside/hardware/arduino/avr/cores/arduino/wiring_digital.c/digitalWrite.html
- https://garretlab.web.fc2.com/en/arduino/inside/hardware/arduino/avr/cores/arduino/wiring_digital.c/pinMode.html

### Embedded C binary

```assembly
48 00000080 <main>:
49 #include <avr/io.h>
50 int main()
51 {
52   DDRB = 1<<PB5;
53   80:	80 e2       	ldi	r24, 0x20	; 32
54   82:	84 b9       	out	0x04, r24	; 4
55  PORTB = 1<<PB5;
56  84:	85 b9       	out	0x05, r24	; 5
57  return 0;
58  86:	90 e0       	ldi	r25, 0x00	; 0
59  88:	80 e0       	ldi	r24, 0x00	; 0
60  8a:	08 95       	ret
```

At first sight, it's clearly easier. Step-by-step instruction decoding:

**ldi r24, 0x20:** according to the AVR ISA [1], `ldi` load the value `0x20` in `r24` which is a General Purpose Working Register of the Arduino.

![registers](./img/registers.jpg)

**out 0x04, r24:** this instruction write the value stored in `r24` at the address `0x04` (the address of `DDRB` according to [2]). `0x20=0b100000`: `DDRB5` is set to 1 :wink:

![mapping](./img/mapping.jpg)

**out 0x05, r24:** same thing, for `PORTB`.

## Conclusion

Embedded C code is usually quicker and smaller as long as we make some effort to study the datasheet. But it's worth it when we're working with embedded systems! :wink:

## References

1. [AVR ISA](http://ww1.microchip.com/downloads/en/devicedoc/atmel-0856-avr-instruction-set-manual.pdf)
2. [ATMega328 datasheet](http://ww1.microchip.com/downloads/en/DeviceDoc/ATmega48A-PA-88A-PA-168A-PA-328-P-DS-DS40002061A.pdf)
