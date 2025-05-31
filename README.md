WhioDoc - Code Documentation Format
==================================

"WhioDoc" is a documentation format for docstrings and similar comment blocks
for inline documentation of functions / classes / macros / etc. as part of the source 
code for a project.

A quick contrived example:
```python
# Funky blending function
# < settings: (FunkyBlenderSettings) General settings object for this feature
#
# < t: (float :: 0.0 <= v < 1.0) Blending factor, controlling the effects
#      Parameter Value Effects:
#      * 0.0 <= v < 0.2: Idling mode
#      * 0.2 <= v < 0.7: Active mode
#      * 0.7 <= v < 0.9: Volcanic mode
#      * > 0.9:          Extinction state
# < (clip_overflows): (bool) Prevent overflowing values by clamping to min/max
#                            bounds provided by FunkyBlenderSettings
#
# > r_secondarySpectra: ({ float : [float]}) Secondary observations
#                       * Key = Energy Level (KeV)
#                       * Value = List of concentrations (mmol-1)
#
# > returns: ((float, float)) Bipolar coordinates from the defrobbulation 
```

For much of the life of this documentation format, it has largely remained nameless,
with the need for a name only really arising when trying to create a Git repo to
formally write up a spec for this thing. The initial name was "Quack!" in homage to
how this technique was originally developed for documenting "duck-typed" languages.
This quickly became "QuackDoc" (to be more easily searchable), at the expense of
some unfortunate connotations. More recently, it has been renamed "WhioDoc"
(in honour of NZ's "Blue Duck" - https://www.nzbirdsonline.org.nz/species/blue-duck).
(NOTE: "Whio" is apparently pronounced "fee-o" instead of "wee-oh")

This document is Version 1.1 of the spec.


# Motivations, Influences, and Origins

* Designed as a terser, yet more easily scannable + information rich documentation
  format than typical "Doxygen" or "Javadoc" style documentation formats, and less
  clunky / ugly than RST or whatever recent versions of Python now recommend.

* It initially evolved organically over two decades of writing Python code
  and/or Object-Orientated "C" code, where the lack of support for rich type 
  annotations in the core language necessitated the development of supplementary 
  approaches to help fill in the blanks.

* It was always intended to be read in the text editor alongside the code it
  refers to, without requiring syntax highlighting to make sense of it.
  
  It was NOT however intended to be easy to be automatically machine-parsed for 
  producing standalone reference documentation pages / PDF's / etc.  That said, this 
  repo does attempt to provide a reference implementation for some tooling to help 
  bridge the gap for those whose main barrier to adoption of new documentation formats 
  on greenfield projects is the lack of such tooling.

* This documentation format is the default / built-in documentation format used
  by the Kea Programming Language. An old prototype of the syntax for this language
  experimented with having these built into the language in a way similar to pre-ANSI 
  (i.e. "K&R" type) C.


# Key Concepts

## General Structure

Each comment block consists of three broad sections:
* 1) Description of what the function / class / macro / etc. does
  * a) The first line or two contains a short description of what the thing does.
       It should generally be only 1 sentence (or at most ~3-5) long.
       
  * b) This may be then followed by a line break / empty line, followed by a more
       detailed explanation.
       
       General notes may follow here as a series of bulletpoints (`*`)
       
       NOTE: If an extended description is made / bulletpoint-notes are defined,
             blank lines should be added after such description text to visually
             separate them from the parameters section.

* 2) Parameter Definitions / "Inputs and Outputs"
  * Every argument, type parameter, and the return types/formats gets its own
    line / "bullet-point"
  * The order of these should be:
    * 1) Input-Variables / Input-Output Variables / Output-Variables - Appearing in the
         order in which they appear in the arguments list (*)
         (*) For class-wide definitions, these can be the argument types that are 
             provided to the various constructor variants.
         
    * 2) Type-Parameters (i.e. for code using generics or other similar 
         meta-programming techniques)
    
    * 3) Stuff that the function returns

* 3) Important Cautionary Notes + Implementation Annotations
  Each of these starts with an exclamation mark "!" at the start of the first line,
  with subsequent lines indented to line up with the rest of the block.
  
  Blank lines should be left between each cautionary note (especially for the longer 
  ones) so that they do not run together. Again, this is done for readability.


General Notes:
* It is strongly recommended to use blank lines between groups of related
  parameters to improve readability.
  
  Example grouping criteria for related parameters include:
  * Functional groupings
  * Type (i.e. inputs vs outputs vs in-out)


## Parameter Definitions - Format and Structure

Parameter lines take a form resembling one of the following:
```
< mode: (eOperationMode) Type of operation to perform

< influence: (float :: 0.0 <= v < 1.0) Strength of the effect

< brushes: ([Brush]) List of brushes than can be used
< brushes_alt: (list[Brush]) Alternative way of describing a list

< rolesMap: ({ 'ActionNameKey' : PaintRole }) Map examples

<> counterState: (CounterState) State object passed to each node visited
                 to allow accumulation of a counter variable and/or other
                 cross-node states.

> r_secondaryResult: (int) Secondary return value from the function

> returns: (str | None) The name of the matching client, or None otherwise.
```


### 1) Initial Bulletpoint

Each line starts with a symbol which indicates what type of IO role it performs:
* **`<`** - A less-than symbol indicates that the variable is an input-only (read-only)
            variable that is being used to pass information into the function.

* **`>`** - A greater-than symbol indicates that the variable is used to return a value
            from the function (i.e. often a secondary return type of the function,
            when it's inconvenient to pack this into the return arg)

* **`<>`** - A pair of "less-than and greater-than" symbols indicates that the variable
             acts as both an input (e.g. providing current state), as well as an output
             (i.e. the updated / merged state gets written into that object)


For completeness, the other symbols that may be used at the start of other lines
in the same position (as the first symbol / token on a new line, when not inside
a existing block)
* **`!`** - As described above, a line starting with an exclamation mark indicates
            an important cautionary note.
            
  * **`! throws`** - When accompanied by the "throws" keyword, this indicates that
                     the function may emit an exception that needs to be caught/
                     handled.
                     
                     The name of the exception(s) being thrown should immediately 
                     follow the "throws" keyword.

* **`!!`** - A double/triple exclamation mark may be used to indicate a point that 
             needs particular care and attention. It should generally not be used
             except in really exceptional circumstances.
            
* **`#`** - A hash is typically used for type parameters

* **`@`** - An "AT" symbol may be used to indicate some reference material or related
            type / documentation that should be consulted for additional clarifications.


### 2) Variable Name

This part is relatively self explanatory.

The only things to mention here:
* For "optional" arguments (i.e. mostly those that have a default argument so don't
  need to be passed in, OR perhaps unused ones whose values we don't care about),
  they can be marked as optional here by putting parentheses around the name
  
  e.g.
  ```python
  # Example of an optional parameter
  # < (superflouousArg): (bool) Do not do anything?
  def example_optional_arg_fn(superflouousArg = True):
  	print("...Do something not involving that arg...")
  ```

* There **must** always be a space left between the "bulletpoint" symbol and
  the name of the variable

* There **must** always be a colon immediately following the variable name, with a
  space following that before any further text appears on the line.
  
  This colon should in general sit right up against the variable name without a space.
  i.e.
  ```
  GOOD:
  < varNameDynamicType: (int) For dynamically typed languages (e.g. Python)
  < varNameTyped: For c-like "explicitly-typed" languages, where the types are redundant
  
  BAD:
  < varName : (eFooBar) There shouldn't be a space after the variable name
  
  <varName:(ContextState)Everything compact like this is also really bad
  <varName: (ContextState) Better but still also bad
  <varName : (ContextState) Nah! Don't do that...
  <varName :(ContextState) Uggh! What is this?!
  ```
  
  NOTE: Whether this still applies in the case of variable-names ending in underscores
        is something a future update may provide revised guidance on, as in practice,
        such variable names have not come up in practical usage of this approach so far.

* If not specified by your codebase's existing coding standards, it is highly
  recommended that all output variables (i.e. those with ">") have some kind
  of prefix / suffix that indicates the type of role they play.
  
  As a starting point, it is recommended to adopt the style used by Blender
  (i.e. using the "`r_`" prefix). Other alternatives sometimes seen include:
  "`_out`". 

* When using ">" to document the return types of a function, the variable name
  must be set to "returns"
  
  NOTE: No other argument should ever get that identifier to avoid confusion.


### 3) Type Specifiers

This part is OPTIONAL when working in a typed language where the types are
explicitly being listed again in the function signature immediately below
the function definition. Otherwise, it MUST be included.

The mini-language within the parentheses following the colon is used to specify
the expected types of the values that can be passed into that argument.

For working with dynamically-typed languages where there is no other type-info 
available, this is critical in ensuring that the codebase stays easily understandable 
+ maintainable by humans. This approach initially came about when Python didn't
have any type annotations at all, and is helpful even after it gained these
(i.e. to be more expressive in cases where the builtin approach is too restricted
 - e.g. such as when referencing a type might involve creating a circular import,
   or when it's harder to accurately express the actual intent)/


**Basic Principles:**
* In general, just put the desired type's name into this space
  * "Num" can be used when an int / float type number can be used, with caring
    about the details too much.

* When the value can take one of several different values (e.g. for defined vs  
  undefined variants that may be handled via `std::option<A>` in Rust), define
  the allowed variants separated by Pipe / Bar symbols.
  e.g.
  ```
  (str | None)
  (QNH<uint> | FlightLevel<uint> | FlightLevelMeters<uint>)
  ```

* For lists / arrays / vectors / etc.:
  * Variant 1 (Preferred):  Surround the name of the type used for each element
    with the type 
    ```
    ([ElementTypename])
    ```
  * Variant 2: Use Python annotation syntax, e.g.
    ```
    (list[ElementTypename])
    ```
  * It is permissible to use this syntax recursively, e.g.
    ```
    (list[dict[str, ToolFlags]])
    ({ str : { str : StateHandler } })
    ```
    NOTE: However, in such cases, you're almost better explicitly declaring those
          nested / inline types separately and using that alias (see "Complex Type
          Specifier" notes below).
  
* For mappings / dicts:
  ```
  ({ KeyType : ValueType })
  ```


**Range / Unit Specifiers:**
When dealing with numeric types, it is useful to be able to specify the range
of values that they apply to, and/or maybe the type of units that they use.

* When there's just a single value needing a range, use the following constructions:
  * **Variant 1:** Range of values, where the value ("v") should always be between a number "A" and another number "C"
    ```
    (Num :: A <= v <= C)
    ```
    
    Examples:
    * `(float :: 0.0 <= v <= 1.0)`
    * `(int :: 1 <= v < 256)`
    * `(Num :: -5 < v < 5)`
    
    Notes:
    * `<` and `<=` can be used as needed to make either endpoint inclusive or not
    * The order *MUST* always be from low to high.
  
  * **Variant 2:** Lower Bound ("A") on Value ("v").  (NOTE: Use "uint" for "v >= 0" cases when only positive-integers are needed)
    ```
    (Num :: v >= A)
    ```
    
    Notes:
    * The equality sign can be either `>` or `>=` as necessary
    * The `v` *MUST* appear on the LHS of the expression
    * This is mostly used for either specifying 0/1 indexing, and/or other similar types of signed-values
    
  * **Variant 3:** Upper Bound ("B") on Value ("v").
    ```
    (Num :: v <= B)
    ```
    Notes:
    * This is rarely needed / used in isolation (i.e. usually you'll need to use a range instead anyway). It is only included here for completeness.
    * The same rules apply as for Variant 2.

* When the ranged-value is part of a more complex description (e.g. key:value pairs, or inside list definition / etc.),
  the range specifier should NOT be part of the signature, but should instead be supplied in the description.
  
  > **Aside - Code Quality / Maintainability Tip:**  
  > This area has traditionally not been well explored / covered in historical usage of this documentation format,
  > as the need for complex specifiers here is usually an indication of a "bad" API definition that will cause
  > problems down the track. Specifically, in many cases, it's better to make proper type defines for these things,
  > which means that the simpler type alias can just be used in the type specifier, thus avoiding most of the
  > problems in the first place.
  
  See notes on "Description" text below for more details.

* **TODO**: Units Support


### 4) Description

The descriptive text for each parameter follows a similar pattern to what is
done for the comment-block's description: i.e. 
* 1) Short description of what the variable is for
* 2) More detailed descriptions of the valid values OR the interpretation of
     the parameter. In particular, it is critical when dealing with map/dict
     types (especially when using generic int/float ID's) to specify the 
     interpretation of the Key vs Value types.

Formatting Advice:
* Descriptions should start with a capital letter, unless referring to some
  other identifier / variable / keyword that uses a lowercase name instead.

* For multi-line descriptions, the following lines should be intended to line up
  with either the type-specifier on the first line OR the start of the description.
  
  Considerations:
  * When there are multiple parameters in a block, aligning with the start of the
    descriptions helps aid scannability.
  * However, if the variable names / type descriptions are all quite verbose,
    it can be ok to align with the start of the type-ID to make better use of
    available line space.

* If type specifiers get really long (or variable-name + type specifier get
  sufficiently long), it is ok to have the description start on a new line following 
  the type-specifier (and in line with the opening paren).


**Descriptions for Complex Type Specifiers:**
This section covers what should happen for more complex type specifiers with sub-types (e.g. key:value pairs, list/collection definitions with nested types / alternatives, etc.)

> **TODO**: There is currently no established standard for this. More details will be added to a future version of this spec when best-practice is established.
>
> Draft Rules:
> * When there's just a single value/type, just include a "Range:" tag on a new line/paragraph after the normal text-descriptions, and put the range-specifier beside that
> * When there are multiple values/types:
>   * 1) Break down each part of the complex specifier into a bullet-point tree (one bullet-point per item), and breaking down further within that tree
>   * 2) The key idea here is to maintain the general principles we are using here:
>        i.e. 
>        * i) **WHAT** info we are dealing with,
>        * ii) **WHY** it's useful / necessary to supply, and 
>        * iii) **WHAT** constraints / limits apply to them.


# Versioning and Attribution

The original creator of this standard is Dr Joshua Leung (@Aligorith).


This standard is free to use for all projects (personal or commercial),
provided that the following conditions are met (where "You" = anyone
making use of this directly or indirectly):
* You may NOT rebrand this as your own creation, and must acknowledge / attribute
  the original source upon which any derivatives are made.

* Derivative versions may NOT remove any of these conditions.

* You may NOT sell copies of this standard (or related tooling defined in this repo)
  as a standalone product, OR part of "wrapper" product where these constitute a
  significant majority of the product being sold.

* No warranties / liability / etc. are implied from your use of this standard.
  (i.e. You CANNOT sue the original author(s) for damages / etc. - To be applied in
   the BROADEST POSSIBLE INTERPRETATION POSSIBLE ACROSS ALL JURISDICTIONS).
   

Version History:
* Evolution + Development Period: 2009 - 2018

* 1.0 - This marks the first attempt to define a formal definition of this
        spec / standard to encourage wider adoption.

