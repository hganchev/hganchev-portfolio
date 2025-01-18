# How to round a REAL/LREAL in TwinCAT
### Autor: Hristo Ganchev (https://github.com/hganchev)
#### Date: 2023-29-08

![Alt text](REAL-LREAL.png)

## Introduction

TwinCAT is a PLC programming software from Beckhoff. It is based on CODESYS and **IEC 61131-3** standard and it is used to program Beckhoff PLCs. 

>The five programming languages for application programming defined in the IEC 61131-3 are available in the CODESYS development environment.

>* IL (instruction list) is an assembler-like programming language. The IEC 61131-3 user organization PLCopen has declared this language as “deprecated”, which means it shall not be used for new projects anymore.
>* ST (structured text) is similar to programming in Pascal or C
>* LD (ladder diagram) enables programmers to virtually combine relay contacts and coils
>* FBD (function block diagram) enables users to rapidly program both Boolean and analog expressions
>* SFC (sequential function chart) is convenient for programming sequential processes and flows

The logic of the PLC is written in POUs (Program Organization Units). There is several types of POUs, but the most used are:
- Function
- Function Block(FB)
- Program(PRG)

The Function and Function Block POUs are used to write reusable code. The difference between them is that the Function Block can have instance variables it can be represented as a class in C#.

The Program POUs are used to write the logic of the PLC.

## How TwinCAT REAL works

Since TwinCAT Structured Text is a pascal based when you are defining a REAL, you need to do it like:

```pascal 
VAR
	rVariable : REAL; 
END_VAR
``` 
The description given from Beckhoff site is that:

>The data types REAL and LREAL are floating-point types according to IEEE 754. They are necessary for the use of decimal numbers and floating-point numbers in point representation or exponential representation.

The size of the types is given in the following table:

| Data Type| Lower limit  				|  Upper Limit  				| Storage Space |
|----------|:--------------------------:|:-----------------------------:|:-------------:|
| REAL 	   | -3.402823e+38 				| 3.402823e+38 					|32-bit			|
| LREAL    | -1.7976931348623158e+308   | 1.7976931348623158e+308 		|64-bit			|

## Functions we will need

To round a REAL value we will need to use the following functions:
```pascal
TRUNC
```

>This IEC operator converts from the data type REAL to the data type DINT. TwinCAT only uses the integer part of the number.

To use the TRUNC function we need to do it like this:
```pascal
rDINT := TRUNC(3.14);
```
where rDINT is the variable that will hold the integer part of the REAL value in our case it will be equal to 3.

```pascal
EXPT(base,exponent)
```

>The IEC operator exponentiates one number with another and returns the base to the power of the exponent: power = baseexponent. Both base and exponent are input values (parameters). The power function is not defined if the base is 0 and the exponent is negative at the same time. However, the behavior in this case depends on the platform.

To use the EXPT function we need to do it like this:
```pascal
rDINT := EXPT(2,3);
```
where rDINT is the variable that will hold the result of the exponentiation in our case it will be equal to 8.


## Why we need to round REAL/LREAL

REAL and LREAL values are used to store decimal numbers. The problem is that the PLC can't store all decimal numbers. For example if we want to store the number 0.1 in a REAL variable the value that will be stored will be 0.10000000149011612. This is because the PLC can't store all decimal numbers and it stores the closest value to the number that we want to store.

Let's make a test:

1. First step is to define a test REAL variable. 
```pascal
rTestReal : REAL;
```

2. Second step is to define a dividend REAL variable and assign it the value of 22. 
```pascal
rDividend : REAL := 22.00;
```

3. Third step is to define a devisor REAL variable and assign it the value of 7. 
```pascal
rDevisor : REAL := 7.00;
```

4. Fourth step is to assign the value of the dividend divided by the devisor to the test REAL variable. 
```pascal
rTestReal := rDividend / rDevisor;
```

The result of the division will be **3.142857**.

![Alt text](REAL_Devision.PNG)

>:attention: So if we want to show the value of the test REAL variable, let's say on a HMI, or we want to export it to a text file, the value will be too long. That's one way why we need to round the value of the REAL variable.

## What needs to be done?

Lets make a function that will round a REAL value.
To define a function in TwinCAT we need to do it like this:
```pascal
FUNCTION F_ROUND_REAL : REAL
```
where F_ROUND_REAL is the name of the function, : REAL is the return value of the function.

1. First step is to declare the input parameter of the REAL value that we want to round.
```pascal
VAR_INPUT
	rInput : REAL;
END_VAR
```

2. Second step is to declare the input parameter of the number of decimal places that we want to round.
```pascal
VAR_INPUT
	rInput : REAL;
	iDecimalPlaces : INT;
END_VAR
```

3. Third step is to declare the local variables that we will use in the function.
```pascal
VAR
	rOutput : REAL;
	rLeftPart : REAL;
END_VAR
```
**rOutput** is the variable that will hold the rounded value.
**rLeftPart** is the variable that will hold the left part of the REAL value.


## Round the REAL value

Let's say we want to round a variable with value 1000,222222 to the second place.

1. First step is to multiply the REAL value with 10 to the power of the number of decimal places that we want to round. In our case we want to round to the second place, so we need to multiply the REAL value with 100.
```pascal
rOutput := rInput * EXPT(10,iDecimalPlaces);
```
The EXPT function will take the first parameter and will raise it to the power of the second parameter. In our case the result will be 100022,2222.

2. Second step is to get the left part of the REAL value. We can do it with the function **TRUNC**. This function will take only the integer part of the REAL value. In our case the integer part of the REAL value is 1000. Then we want to subtract the integer part from the REAL value.
```pascal
rLeftPart := rOutput - TRUNC(rOutput);
```
In our case the result will be 0,222222.

3. Third step is to check if the left part is greater than 0.5. If it is greater than 0.5 we need to add 1 to the integer part of the REAL value. If it is less than 0.5 we don't need to do anything.
If the value is negative we need to subtract 1 if it is less or equal to -0.5.
```pascal
IF rLeftPart >= 0.5 THEN
	rOutput := rOutput + 1;
ELSIF rLeftPart <= -0.5 THEN
	rOutput := rOutput - 1;
ELSE
	rOutput := rOutput;
END_IF
```

This is done because we want to round the value to the nearest integer. If the left part is greater than 0.5 we need to add 1 to the integer part of the REAL value. If the left part is less than 0.5 we don't need to do anything.

4. Fourth step is to divide the REAL value with 10 to the power of the number of decimal places that we want to round. In our case we want to round to the second place, so we need to divide the REAL value with 100.
```pascal
rOutput := rOutput / EXPT(10,iDecimalPlaces);
F_ROUND_REAL := rOutput;
```

## The whole function

```pascal
FUNCTION F_ROUND_REAL : REAL
VAR_INPUT
	rInput : REAL;
	iDecimalPlaces : INT;
END_VAR
VAR
	rOutput : REAL;
	rLeftPart : REAL;
END_VAR

rOutput := rInput * EXPT(10,iDecimalPlaces);
rLeftPart := rOutput - TRUNC(rOutput);
IF rLeftPart >= 0.5 THEN
	rOutput := rOutput + 1;
ELSIF rLeftPart <= -0.5 THEN
	rOutput := rOutput - 1;
ELSE
	rOutput := rOutput;
END_IF
rOutput := rOutput / EXPT(10,iDecimalPlaces);
F_ROUND_REAL := rOutput;
```

## How to use the function

To use the function we need to do it like this:
```pascal
rRoundedValue := F_ROUND_REAL(1000.2222, 2);
```
where rRoundedValue is the variable that will hold the rounded value, F_ROUND_REAL is the name of the function, 1000.2222 is the REAL value that we want to round, 2 is the number of decimal places that we want to round.

The result of the function will be **1000.22**.

## Conclusion

We have made a function that will round a REAL value.
We used the predefined TRUNC and EXPT functions from Beckhoff TwinCAT to do it.

The function does not have any error handling, so if you want to use it in your project you need to add it.


## Example

This function is used in my [twincat-tools](https://github.com/hganchev/twincat-tools/tree/main/TwinCAT%20Tools/twincat-tools/POUs/Round) library.
