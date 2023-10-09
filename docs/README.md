# Coding Convention

> Clean code is simple and direct. Clean code reads like well-written prose. Clean code never obscures the designer’s intent but rather is full of crisp abstractions and straightforward lines of control. - Grady Booch author of Object Oriented Analysis and Design with Applications”

## Opening Statement

This coding convention has been created to help keep our code base clean and tidy. It will act as a guide for those of you who have not settled on a standard, and will impose rules on those of you who wish to contribute.

## Naming

### Intent

Use names which both reveal your intent and removes the need for supplementing your code with comments.

```example
// Put the UPS in to full capacity mode if the Power Supply 10 is in fault
IF SUP_10.flt THEN
	UPS.smode := 1;
END_IF
```

The example above has been renamed and refactored to remove the comment. This allows us to read the code without placing any mental obstacles in the way of our understanding.

```example
IF PowerSupply10.HasFault THEN
	UPS.mode := FULL_CAPACITY_MODE;
END_IF
```

### Avoid abbreviations

Where possible, do not abbreviate. This forces us to think, decode and remember a term. Every time we give our brain something to do it forces it to work harder. Instead favour whole words and phrases so that our mind can stay on auto-pilot when reading our code.

```example
   VAR strBtn : BOOL;
```

It may seem only a small step, but the brain power that we have saved not decoding an abbreviation can be used for other more creative or fault finding tasks.

```example
   VAR startButton : BOOL;
```

## Comments

Ask yourself 'why do I need a comment?'

If you need them to convey your code's true meaning then you should refactor and rename your variables to make your [Intent](?id=intent) clear.

Do not use them for auditing, changes, author. This adds extra work to the coders following you and they will be missed and become out of date. Favour source control instead.

**Comments must not be used to,**

- explain how your code works (Rename / Refactor)
- comment out old code (Delete it and use Source Control)
- add author information (use Source Control)

**Comments may be use**

- as place holders or for general information.

```example
IF PowerSupply10.HasFault THEN
	// Custom fault reaction code may go here...
END_IF
```

Remember, no comments are the best form of comments!

## Enumeration

### Global Enumeration

Global Enumeration should be kept to a minimum as this will cause tight coupling between objects which use them. Tight coupling is a bad thing. Try instead to replace global enumeration with other objects which can be passed around.

### Local Enumeration

Enumeration inside of a class is good and will make CASE statements easier to use and extend. Use inline enumeration as shown below. This keeps internal state locked away inside a class and prevents us needing to manage this enumeration in a second file.

Enumeration must be ALL_CAPITALS with underscore word separation.

```declaration
FUNCTION_BLOCK Cylinder
VAR
	state : (RETRACTED, EXTENDING, EXTENDED, RETRACTING, JAMMED_EXTENDING, JAMMED_RETRACTING);
END_VAR
```

```body
CASE state OF
	RETRACTED:
		//
	EXTENDING:
		//
//...
END_CASE
```

## Constants

Constants must be ALL_CAPITALS with underscore word separation.

Replace "Magic numbers" with constants to assist with readablity and understanding.

```example
VAR CONSTANT

	MAXIMUM_BUFFER_SIZE_IN_BYTES : UDINT := 200;

END_VAR
```

## Class

In the IEC standard, there is a keyword for "Class". Favour the use of CLASS POUs whenever possible. If you are unable to define a CLASS POU, consider using a Function Block with the attributes outlined below. Ensure you decorate your classes with the no_explicit_call attribute to prevent them from being used as standard IEC Function Blocks.

```example
{attribute 'no_explicit_call' := 'This FB is a CLASS and must be accessed using methods or properties'}
```

!>You must not place code in the FUNCTION_BLOCK body. Use a public method instead, such as .CyclicCall();

!>You must not allow VAR_INPUT, VAR_OUTPUT and VAR_IN_OUT to exsit in a class declaration.

### Naming

Always use **PascalCase** for class names. You should use noun or noun phrase for class names. Do not use prefixes. An underscore may be used if the class is typed and it assists with overall the readability.

```example
{attribute 'no_explicit_call' := 'This FB is a CLASS and must be accessed using methods or properties'}
FUNCTION_BLOCK AnalogValue_LREAL EXTENDS AnalogValue
// VAR_INPUT, VAR_OUTPUT and VAR_IN_OUT is not permitted here.
VAR

END_VAR
```

### Private Variables

Private variables must be **camelCase**.

If a private variable shares the same name as a Property, prefix it with an underscore. However, try to avoid such name clashes whenever possible.

### Example

```declaration
{attribute 'no_explicit_call' := 'This FB is a CLASS and must be accessed using methods or properties'}
FUNCTION_BLOCK Cylinder EXTENDS ComponentBase IMPLEMENTS I_Move
VAR
	_isBusy : BOOL; // underscore required if there is an IsBusy property
    state : (RETRACTED, EXTENDING, EXTENDED, RETRACTING, JAMMED_EXTENDING, JAMMED_RETRACTING);
	retractedSensor : I_Sensor;
	extendedSensor : I_Sensor;
END_VAR
```

```body
// The body of a class should not contain code.
// Cyclic code, if necessary, should be executed from a method.
```

## Methods

Only Classes or Function Blocks defined with ```no_explicit_call``` should contain methods. Refrain from defining methods within a Function Block or Program.

Methods, like classes should have one reason to exist. They should do one job and do it well.

### Naming

Always use **PascalCase** for method names. You should use verb or verb phrase for method names. Do not use prefixes.

```example
cylinder.Retract();
persistentData.Save();
```

!>Methods should not have the word 'and' in them as this is a sure sign that they are doing more than one job.

### Arguments

Methods should have as few arguments as possible. Zero arguments is best. One argument is ok. Two arguments is almost too many. Try to pass objects in to methods in order to reduce argument count. Avoid Boolean arguments as this is typically a sign that a method is dual purpose.

```example
// example of a bad method
csvReader.LoadFileAndParseResultsToArrayIfLoggingIsEnabled('file.csv',resultsArray,Logging.IsEnabled);
```

The example above has been refactored to divide the method in to smaller methods with smaller number of arguments.

```example
IF logging.IsEnabled THEN
	csvReader.LoadFile('file.csv');
	csvReader.ParseToArray(resultsArray);
END_IF
```
