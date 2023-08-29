# How to split a STRING in TwinCATIntroduction
### Autor: Hristo Ganchev (https://github.com/hganchev)
#### Date: 2023-23-08

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

Spliting a string in PLC can be a challenging if you don't know what functions to use.There is several ways to do it, but I will show you one of it. 

## How TwinCAT STRING works

Since TwinCAT Structured Text is a pascal based when you are defining a string, you need to do it like:

```pascal 
VAR
	sVariable : STRING(255); 
END_VAR
``` 
where 255 is the length of the string.The description given from Beckhoff site is that:

> A variable of data type STRING can accept any string. The size specification for the storage space allocation in the declaration refers to characters and is enclosed by round or square brackets. If no size is specified, TwinCAT assumes 80 characters by default.

> As a rule, TwinCAT does not limit the string length, but the string function only 
processes lengths between 1 and 255! If a variable is initialized with a string that is too long for the data type of the variable, TwinCAT truncates the string from the right.

## STRING functions

TwinCAT has several functions that can be used to manipulate strings. The functions are:
```pascal
 FIND(STR1, STR2)
``` 
>The function FIND searches for a partial string within a string.
FIND (STR1, STR2) means: Find the position of the first character where STR2 appears in STR1 for the first time. If STR2 is not found in STR1, then OUT:=0

```pascal
 LEFT(STR, SIZE)
``` 
>The function LEFT returns the left, initial string for a given string.
LEFT (STR, SIZE) means: Take the first SIZE character from the left in the string STR.

```pascal
 LEN(STR)
``` 
>The function LEN returns the length of a string

```pascal
 MID(STR, LEN, POS)
``` 
>The function MID returns a partial string from within a string.
MID (STR, LEN, POS) means: Retrieve LEN characters from the STR string beginning with the character at position POS.

```pascal
 RIGHT(STR, LEN)
``` 
>The function RIGHT returns the right, initial string for a given string.
RIGHT (STR, SIZE) means: Take the first SIZE character from the right in the string STR.

```pascal
 REPLACE(STR1, STR2, L, P)
``` 
>The function REPLACE replaces a partial string from a larger string with a third string.
REPLACE (STR1, STR2, L, P) means: Replace L characters from STR1 with STR2 beginning with the character in the P position.

```pascal 
DELETE(STR, LEN, POS)
``` 
>The function DELETE removes a partial string from a larger string at a defined position. The input variable STR is type STRING, LEN and POS are type INT, the return value of the function is type STRING.
DELETE (STR, LEN, POS) means: Delete LEN characters from STR beginning with the character in the POS.

```pascal
INSERT(STR1, STR2, POS)
```
>The function INSERT inserts a string into another string at a defined point.
INSERT (STR1, STR2, POS) means: Insert STR2 into STR1 after position POS.

```pascal
CONCAT(STR1, STR2)
```
>Concatenation (combination) of two strings.

## What needs to be done

Lets make a function that will split a string to an array of strings.
To define a function in TwinCAT we need to do it like this:
```pascal
FUNCTION F_SplitString : ARRAY[0..9] OF STRING(100)
```
where F_SplitString is the name of the function, ARRAY[0..9] is the array of strings that will be returned and STRING(100) is the length of the string.

1. First step is to define a string variable with the length of the string you want to split. Lets say we want to split a string with the length of 100 characters. 
```pascal
VAR_INPUT
	sInput 		: STRING(100);
END_VAR
```
2. Second step is to define an input value that will be used to split the string. Lets say we want to split the string with a character of comma. 
```pascal
VAR_INPUT
	sInput 		: STRING(100);
	sSplitChar	: STRING(1);
END_VAR
```
We define the input value as a string with the length of 1 character.

3. Third step is to define a variable that will hold the copy of the input string. 
```pascal
sInputCopy : STRING(100);
```

4. Fourth step is to define a variable that will hold the split value. 
```pascal
sSplitValue : STRING(100);
```

5. Fifth step is to define a variable that will hold the length of the split value. 
```pascal
iSplitLength : INT := 0;
```

## Spliting the string

Lets say we want to split the string 'Hello,World' with the split character of comma.

1. First step is to copy the input string to the copy variable. 
```pascal
sInputCopy := sInput;
```
The value of the copy variable will be **'Hello,World'**.

2. Second step is to count the number of split values. 
```pascal
FOR i := 0 TO 9 DO

END_FOR
```
3. Third step is to check if the split character is found. 
```pascal
IF FIND(sInputCopy, sSplitChar) > 0 THEN
```
The value of the FIND function will be **6**.

4. Fourth step is to get the split value. 
```pascal
sSplitValue := LEFT(sInputCopy, FIND(sInputCopy, sSplitChar) - 1);
```
We use the LEFT function to copy the split value to the variable and we subtract 1 from the FIND function because we don't want to copy the split character.

The value of the split value will be **'Hello'**.

5. Fifth step is to take the length of the split value and add the length of the split character. 
```pascal
iSplitLength := LEN(sSplitValue) + 1;
```
The value of the split length will be **6**.

6. Sixth step is to define condition when the split value is not found. 
```pascal
ELSE
	sSplitValue := sInputCopy;
	iSplitLength := LEN(sSplitValue);
END_IF
```

7. Seventh step is to delete the split value from the copy variable. 
```pascal
sInputCopy := DELETE(sInputCopy, iSplitLength, 1);
```
The value of the copy variable will be **'World'**.

8. Eighth step is to copy the split value to the array. 
```pascal
F_SplitString[i] := sSplitValue;
```

9. Ninth step is to check if the copy variable is empty and if is to stop spliting. 
```pascal
IF LEN(sInputCopy) = 0 THEN
	EXIT;
END_IF
```

## The whole function

```pascal
FUNCTION F_SplitString : ARRAY[0..9] OF STRING(100)
VAR_INPUT
	sInput 		: STRING(100);
	sSplitChar	: STRING(1);
END_VAR
VAR
	sInputCopy 	: STRING(100);
	sSplitValue : STRING(100);
	iSplitLength : INT := 0;
	i : INT;
END_VAR

sInputCopy := sInput;
FOR i := 0 TO 9 DO
	IF FIND(sInputCopy, sSplitChar) > 0 THEN
		sSplitValue := LEFT(sInputCopy, FIND(sInputCopy, sSplitChar) - 1);
		iSplitLength := LEN(sSplitValue) + 1;
	ELSE
		sSplitValue := sInputCopy;
		iSplitLength := LEN(sSplitValue);
	END_IF

	sInputCopy := DELETE(sInputCopy, iSplitLength, 1);
	
	F_SplitString[i] := sSplitValue;

	IF LEN(sInputCopy) = 0 THEN
		EXIT;
	END_IF
END_FOR
```

## How to use the function

1. First step is to define a variable that will hold the array of strings. 
```pascal
sArray : ARRAY[0..9] OF STRING(100);
```

2. Second step is to call the function. 
```pascal
sArray := F_SplitString(sInput := 'Hello,World', sSplitChar := ',');
```

The result will be:
```pascal
sArray[0] = 'Hello'
sArray[1] = 'World'
```

## Conclusion

We have made a function that will split a string to an array of strings.
We used the predefined STRING functions from Beckhoff TwinCAT to do it.

The function does not have any error handling, so if you want to use it in your project you need to add it.


## Example

This function is used in my [twincat-tools](https://github.com/hganchev/twincat-tools/tree/main/TwinCAT%20Tools/twincat-tools/POUs/String) library.