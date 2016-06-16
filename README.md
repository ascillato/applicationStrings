## Application Strings
This is a minimal arduino sketch type example that demonstrates a method to maintain strings within an arduino core esp8266 project. This code has been tested on an esp8266 however it should also work on any arduino board with minimal change.

There are many small pieces of sample code available all over the internet for esp8266 boards and many arduino type boards that show strings embedded directly within C/C++ and arduino sketch code.

Quite often (almost always?) no passing comment about how bad an idea this can be is mentioned. For very small projects this might not pose a problem at all however as a project grows over time from small to medium and onto large, many directly embedded string literals can start to become an issue in two primary ways:

1. Strings become sprawling and scattered through project code and as a result are difficult to find and maintain.
2. String literals directly embedded in code are mapped directly into heap memory on a device and eventual heap memory depletion == unexpected application crash and device reboot.

## What does this thing do?
This minimal example tackles both of the previously identified problems by:

1. It centralizes all applications strings into a single point of maintenance consolidated string table so that all strings are easily maintainable in one location.
2. It solves the heap memory problem by using [PROGMEM](https://www.arduino.cc/en/Reference/PROGMEM) to store all strings defined within the string table in flash memory.

As an added benefit this example also demostrates how to use the compiler pre-processor to generate code from the defined string table so that:

1. Each string defined in the string table gets an individually code generated PROGMEM string
2. All strings defined in the string table get defined in an array of all strings correctly setup for PROGMEM storage
3. An enumeration is code generated from the string table so that each string defined gets an enum value assigned. This feature facilitates pulling individual strings from PROGMEM flash using an easy to use string helper function that takes a corresponding string specific enum value.

## Pre-processor generated code output
As a bonus and to see what code the pre-processor is actually generating, you can add the -E option to gcc which will tell gcc to "Stop after the preprocessing stage; do not run the compiler proper. The output is in the form of preprocessed source code, which is sent to the standard output." In real terms with the esp8266, this means that .o object files will be created but not compiled to byte code which means you can then look at the code within the .o file(s) to see what the pre-processor did. 

To enable option this with the esp8266 arduino core, edit your /.arduino15/packages/esp8266/hardware/esp8266/2.1.0/platform.txt file and insert the -E option on the compiler.cpreprocessor.flags line e.g. make it look like this:

`compiler.cpreprocessor.flags=-E -D__ets__ -DICACHE_FLASH -U__STRICT_ANSI__ "-I{compiler.sdk.path}/include"`

When the code in this example is compiled in eclipse with the gcc xtensa tool chain for the esp8266, the code that gets generated by the pre-processor looks like this:

```cpp
enum appStringType {
 uniqueStrEnumVal1, uniqueStrEnumVal2, uniqueStrEnumVal3,
};

extern const char APP_STR_uniqueStrEnumVal1[]; extern const char APP_STR_uniqueStrEnumVal2[]; extern const char APP_STR_uniqueStrEnumVal3[];

extern const char* const appStrings[];

const char APP_STR_uniqueStrEnumVal1[] __attribute__((section(".irom.text"))) = "String1"; const char APP_STR_uniqueStrEnumVal2[] __attribute__((section(".irom.text"))) = "String2"; const char APP_STR_uniqueStrEnumVal3[] __attribute__((section(".irom.text"))) = "String3";

const char* const appStrings[] __attribute__((section(".irom.text"))) = {
  APP_STR_uniqueStrEnumVal1, APP_STR_uniqueStrEnumVal2, APP_STR_uniqueStrEnumVal3,
};
```
