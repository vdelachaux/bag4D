# bag4D v1.0 (12/2013)

When I make my shopping, I buy a pack of six bottles of milk, bread and salt. I put everything in my shopping bag. When arriving at home I retrieves each article in the bag without thinking to how to do more than necessary.  With bag4D you can do the same, in your code.

***bag4D***  is a component which provides a set of commands that lets you create data bags where you can store and retrieve data like you want. You must keep a value? put it in a bag, then another method or a form can test if the element exists and recover the stored value.

* Value bags are inter-process. 
* Value bags are shared between host database and components. 
* Value bags can be exported to or imported from a document. 
* Value bags can be stored as blob allowing to transfer  data, through web-services, to another 4D
* Value bags are handled by a long integer reference. But you can also name the bag and then you will get the reference associated with this name.
* Value bags can be embedded in another one and ***bag4D*** will auto-create embedded bags according to the given pathname.

_Example_:

```
$bag := bag_new ( "parameters" )
bag_putBoolean ( $bag ; "options.general.close window" ; True )
```

In this example, after the call to ***bag\_putBoolean***, *$bag* contains the embedded bag "options", which contains the embedded bag "general", which contains the embedded bag "close window" that contains the value **True**.

* The elements of a bag​​, shares [properties](#properties) and [methods](#methods) to simplify the code. For example:

```
$bag := bag_new ( "TEST" )
bag_putLong ( $bag ; "long_1++")
bag_putLong ( $bag ; "long_2" ; 2 )
...
bag_putLong ( $bag ; "long_9" ; 9 )

bag ( "TEST.sum()" ; -> $sum ) 	//45
bag ( TEST.max()" ; -> $max) 	//9
bag ( "TEST.min()" ; -> $min )	//1
bag ( "TEST.squareSum()" ; -> $squareSum )	//285
bag ( "TEST.average()" ; -> $average )	//5

While ( bag ( "TEST.forEach()" ) )
	bag ( "TEST.this.name" ; -> $name)  //get the name of the current element
	bag ( "TEST.this.value" ; -> $value )  //get the value
	...
	bag ( "TEST.this.index" ; -> $i )  //get the index
	If ( ( $i % 2 ) = 0)
		bag ( "TEST.this.empty()" )  //clear the value
	End if		
End while 

bag ( "TEST.sum()" ; -> $sum ) 	//25

$result := 0
While ( bag_forEach ( $bag ; -> $value ) )
	$result := $result + $value
End while 
```		

## Creation & Destruction Routines

### **bag\_new**  {( name )} -> *bagHandle*

Creates a new bag and returns its reference

If the name is passed, the reference may be retrieved from the name, by the [**bag\_findByName**](#bag_findbyname--name---baghandles----baghandle) command.

```
$bag := bag_new ( "myBag" )
```

### bag\_copy ( *bagHandle* {; *name* }) -> *bagHandle*

Creates a copy of the bag bagHandle and returns its reference

If the name is passed, the reference may be retrieved from the name, by the [**bag\_findByName**](#bag_findbyname--name---baghandles----baghandle) command.

```
$bag2 := bag_copy ( $bag ; "myBag" )
```
### **bag\_clear** {( *bagHandle* {; *elementName* })}

Deletes the element elementName of the bag bagHandle. 

If the second parameter is omitted, it's the entire bag bagHandle that is deleted. If no parameter is passed all bags are deleted.

```
bag_clear ( $bag ; "element 1" )	//delete element 1 form the value bag $Bag
bag_clear ( $bag  )			        //delete the value bag $Bag
bag_clear 				            //delete all created bags
```

If *bagHandle* is not a valid bag reference, a warning assertion* is displayed.

**Note**: By default, assertions are enabled but they may have been disabled using the [**bag\_SET_OPTION**](#bag_set_option--option--on-) command.

## Put Value Routines

This set of commands allow to store data in a value bag.

Each of them expect a valid bag handle as first parameter and as second parameter a name or a pathname. If it's a pathname, the embedded bags will be created if necessary.

If no item in the bag has the given name or pathname, a new item is created.

If an item with the given name or pathname exists and has the same type, its value is replaced.

If an item with the given name or pathname exists and has a different type, if the option "variant types" is off, the value isn't stored and an error assertion is displayed (if the option "assertions" is on).

When relevant, the name may refer to an array element by adding the index of the item in square brackets. For example the line below, sets to 100 the tenth element of the array stored in the item "counter".

```
bag_putLong ( $bag ; "counter[10]" ; 100 )
```

If it is an array element, ***bag4D*** ensures, if possible, the conversion of the value before storing. For example the value 1 will be stored as a long in a longint array, as the textual value "1" in a text array and as "true" in a boolean array.

Some commands accept an operator as suffix. This operator allows  to modify the data stored. For example the first line below, adds the picture passed to the right of the stored picture in the element "myPicture". The next line, increments by one the value of the item "myCounter".

```
bag_putPicture ( $bag ; "myPicture+=" ; $picture ) 
bag_putLong ( $bag ; "myCounter++" )
```

### **bag_putArray** ( *bagHandle* ; *pathname* ; -> *array* {; *internal format* })

This command allows to store a one-dimensional array pointed to by the third parameter in the bag referenced by the parameter bagHandle.

All type of arrays are managed except 2-dimensional arrays.

The fourth optional parameter is a boolean. If the parameter "internal format" is **False**, the array is stored so that you can get or set more quickly an element. If this parameter is **True**  (default value), the array is compacted before to be stored. This mode is faster for storage and loading large arrays but will be less rapid to get or set an element. Choose the best mode according to your needs.

The type of the array, the count of elements and the current element are stored as properties and can be retrieved without loading the array.

### **bag_putLong** ( *bagHandle* ; *pathname* {; *longint* })

This command allow to store a long integer in the bag referenced by the parameter bagHandle.

The optional third parameter is the value to store in the bag. If this parameter is missing, the value is set to 0.

```
bag_putLong ( $bag ; "ID" ; 123456789 )
```

The managed operators for this command are:

| Operators | Actions   [^ignored]                                                        |
|:---------:|:--------------------------------------------------------------------------- |
| ++        |increment by one the current value                                           |
| --        |decrement by one the current value                                           |
| +=        |add the passed value to the current value                                    |
| -=        |subtract the passed value from the current value                             |
| *=        |multiplie the current value with the value passed                            |
| /=        |divide the current value with the value passed                               |
| \=        |perform a longint division of the current value with the passed value        |
| %=        |do the remainder of the division of the current value with the passed value  |
| ^=        |for an exponentiation of the current value with the passed value             |
| !         |the stored value will be 1 if it was equal to 0 and 0 if not                 |
| &=        |store a bitwise AND between the stored value and the passed value            |
| \|=       |store a bitwise OR (inclusive) between the stored value and the passed value |

[^ignored]: For increment et decrement, the second parameter is ignored

_Examples_:

```
bag_putLong ( $bag ; "number" ; 100 )         //value is 100
bag_putLong ( $bag ; "number++" )	             // value is 101
bag_putLong ( $bag ; "number*=" ; 2 )         // value is 202
bag_putLong ( $bag ; "number%=" ; 100 )       // value is 0,02

bag_putLong ( $Bag ; "arrayLong[3]++")
```		

### **bag_putReal** ( *bagHandle* ; *pathname* {; *real* })

This command allow to store a real in the bag referenced by the parameter bagHandle.

The optional third parameter is the value to store in the bag. If this parameter is missing, the value is set to 0.

```
bag_putReal ( $bag ; "Pi" ; 3.141592653589793239 )
```

The managed operators for this command are:

| Operators | Actions                                                                    |
|:---------:|:-------------------------------------------------------------------------- |
| ++        |increment by one the current value                                          |
| --        |decrement by one the current value                                          |
| +=        |add the passed value to the current value                                   |
| -=        |subtract the passed value from the current value                            |
| *=        |multiplie the current value with the value passed                           |
| /=        |divide the current value with the value passed                              |
| \=        |perform a longint division of the current value with the passed value       |
| %=        |do the remainder of the division of the current value with the passed value |
| ^=        |for an exponentiation of the current value with the passed value            |


### **bag_putText** ( *bagHandle* ; *pathname* {; *text* })

This command allow to store a text in the bag referenced by the parameter bagHandle.

The optional third parameter is the value to store in the bag. If this parameter is missing, the value is set to "".

```
bag_putText ( $bag ; "welcomeMessage" ; "Hello world!" )
```

The concatenation operator is managed as name suffix to modify an existing text element.

```
bag_putText ( $bag ; "welcomeMessage" ; "Hello" )       //value is Hello
bag_putText ( $bag ; "welcomeMessage+=" ; " world!" )   //value is Hello world!
```

### **bag_putBoolean** ( *bagHandle* ; *pathname* {; *boolean* })

This command allow to store a boolean in the bag referenced by the parameter bagHandle.

The optional third parameter is the value to store in the bag. If this parameter is missing, the value is set to false.

		bag_putBolean ( $bag ; "bag4D is incredible" ; True )

### **bag_putDate** ( *bagHandle* ; *pathname* {; *date* })

This command allow to store a date in the bag referenced by the parameter bagHandle.

The optional third parameter is the value to store in the bag. If this parameter is missing, the value is set to 00/00/00.

```
bag_putDate ( $bag ; "today" ; Current date )
```

### **bag_putTime** ( *bagHandle* ; *pathname* {; *time* })

This command allow to store a time value in the bag referenced by the parameter bagHandle.

The optional third parameter is the value to store in the bag. If this parameter is missing, the value is set to 00:00:00.

```
bag_putTime ( $bag ; "time" ; Current time )
```

### **bag_putPicture** ( *bagHandle* ; *pathname* {; *picture* {; *codec*{; *hOffset*{; *vOffset* }}}})

This command allow to store a picture in the bag referenced by the parameter bagHandle.

The optional third parameter is the picture to store in the bag. If this parameter is missing, the stored picture will be an empty picture.

The fourth optional parameter is the codec to use for the stored image. The default value is ".png"

```
bag_putPicture ( $bag ; "picture" ; $picture ; ".tiff")  // Picture is stored as tiff
bag_putPicture ( $bag ; "picture" ; $picture )           // Picture is stored as png
```

The name may be suffixed with an operator that will be used to modify an existing element. The managed operators are:

| Operators | Actions                                                                    |
|:---------:|:-------------------------------------------------------------------------- |
| +=        |adds the picture passed to the right of the current picture                 |
| /=        |adds the picture passed to the bottom of the current picture                |
| &=        |puts the picture passed over the current picture.                           |

If the optional hOffset and vOffset parameters are used, a translation is applied to picture passed before superimposition.

The *codec*, the *width*, the *height* and the *size* of the picture are stored as properties and can be retrieved without loading the picture.

### **bag_putBlob** ( *bagHandle* ; *pathname* ; -> *blob* )

This command allow to store a blob in the bag referenced by the parameter bagHandle.

The required third parameter is a pointer to the blob to store in the bag.

```
CONVERT FROM TEXT ( "Hello world!" ; "utf-8" ; $Blob )
bag_putBlob ( $bag ; "myBlob" ; $Blob )
```

### **bag_putPointer** ( *bagHandle* ; *pathname* ; -> *something* )

This command allows to store a pointer in the bag referenced by the parameter bagHandle.

The required third parameter is the pointer to store in the bag.

The pointer can point a variable or a field. All pointers are managed except a pointer to a local variable.

```
bag_putPointer ( $bag ; "myPointer" ; -> [table] )
```

### **bag_putVariable** ( *bagHandle* ; *pathname* ; -> *value* )

This command allows to store the content of the variable or the field pointed to by the third required parameter in the bag referenced by the parameter bagHandle.

The value type must be in: boolean, integer or longint, real, string, text, time, date, picture, blob.

The value can either be retrieved with the command bag_getVariable or bag_get(type) according to the original value's type.

```
bag_putVariable ( $bag ; "myVar" ;  -> vTest )
```

### **bag_putBag** ( *bagHandle* ; *pathname* ; *bagHandle* )

This command allows to embed an existing bag in the bag referenced by the parameter bagHandle.

```
bag_putVariable ( $bag ; "myVar" ;  $bag2 )
```

## Get Value Routines

This set of commands allow to get the value of an item stored into the value bag.

Each of them expect a valid bag handle as first parameter and a name or a pathname as second parameter. 

An error assertion is displayed if the bag handle is not valid or the item is not found. In these cases an empty value is returned.

If possible, bag4D performs the conversion between the type of the stored data and the type requested. If it's not possible, a warning assertion is generated.

When relevant, the name may refer to an array element by adding the index of the item in square brackets. For example the line below, gets the tenth element of the array stored in the item "counter".

```
$count10 := bag_getLong ( $bag ; "counter[10]" )
```

### bag_getLong ( *bagHandle* ; *pathname* ) -> *longint* 

This command gets a longint value from an item stored in the bag referenced by the parameter bagHandle.

### **bag_getReal** ( *bagHandle* ; *pathname* ) -> *real* 
 
This command gets a real value from an item stored in the bag referenced by the parameter bagHandle.

### bag_getText ( bagHandle ; pathname ) -> text 

This command gets a text value from an item stored in the bag referenced by the parameter bagHandle.

### **bag_getBoolean** ( *bagHandle* ; *pathname* ) -> *boolean* 

This command gets a boolean value from an item stored in the bag referenced by the parameter bagHandle.

### **bag_getDate** ( *bagHandle* ; *pathname* ) -> *date* 

This command gets a date value from an item stored in the bag referenced by the parameter bagHandle.

### **bag_getTime** ( *bagHandle* ; *pathname* ) -> *time* 

This command gets a time value from an item stored in the bag referenced by the parameter bagHandle.

### **bag_getPicture** ( *bagHandle* ; *pathname* {; *codec* }) -> *picture* 

This command gets a real value from an item stored in the bag referenced by the parameter bagHandle. 

If the third parameter is omitted, the codec used will be ".png".

### **bag_getBlob** ( *bagHandle* ; *pathname* ; -> *blob* )

This command sets the object pointed to by the 3rd parameter with the blob value from an item stored in the bag referenced by the parameter bagHandle.

### **bag_getPointer** ( *bagHandle* ; *pathname* ) -> *pointer* 

This command gets a pointer value from an item stored in the bag referenced by the parameter bagHandle.

### **bag_getBag** ( *bagHandle* ; *pathname* ) -> *bagHandle* 

This command gets a bagHandle from the embedded bag stored in the bag referenced by the parameter bagHandle.

### **bag_getVariable** ( *bagHandle* ; *pathname* ; -> *blob* )

This command sets the object pointed to by the 3rd parameter with the value of an item stored in the bag referenced by the parameter bagHandle.

This command doesn't allow to retrieve the value of an array element.

## Item info routines

### **bag_itemExist** ( *bagHandles*  ; *name* ) -> *boolean*

Returns true if the value bag, referenced by bagHandle, contains an item named name.

### **bag_itemType**( *bagHandles*  ; *name* ) -> *longint*

Returns the type of the data stored in the item name in the value bag bagHandle.

The types are the values of the corresponding types in 4D plus one: 8858 is returned if the item is an embedded bag.

### **bag_arraySize**( *bagHandles*  ; *name* ) -> *longint*

Returns the size of the array stored in the item name in the value bag bagHandle.

## The command bag

This command is a shortcut to manage items in a value bag.

The syntax is very simple: give it, in the first parameter, the name of a bag and an action to perform, it will return true if the action was completed and, where appropriate, a value in the object pointed to by the second parameter. Like this:

```
bag ( 'BAGNAME.action'  ; -> object ) -> boolean
```

The first parameter should always start with a name of an existing value bag followed by a dot, optionally followed by a path to an item, then the name of a method or of a property.

```
bag ( 'BAGNAME.level_1.level_2action'  ; -> object ) -> boolean
```

For example, after execution of the line below, the variable 'result' will contain the sum of all the top-level items of the bag 'myBag'.

```
bag ( 'myBag.sum()'  ; -> result )
```

### methods 

### **forEach()**
This method is used to iterate through each element of an array or of a bag. Exp:

```
While (bag ( "myBag.forEach()" ))
	bag ( "myBag.this.name" ; ->$name )
	bag ( "myBag.this.value" ; ->$value )
End while 
```

### **empty()**


### **min()**


### **max()**


### **sum()**


### **squareSum()**


### **average()**


### **join( {separator} )**


### **toJson()**


### **items()**


### **itemCount()**


## properties

**this** refers to the current element in a foreach() loop

**name** returns the name of the current element as a text

**index** returns the index of the current element as a longint

**type** returns the type of the current element as a longint. 

The values ​​are the same as those used in 4D and described in the constant theme "Field and Variable Types". One specific value is added for the value bag, it's 8858.

**codec** returns, for an image item, the codec used at the time of the storage. It's a text value.

**width** returns, for an image item, the width of the image. It's a numeric value.

**height** returns, for an image item, the height of the image. It's a numeric value.

**size** fills the result with the size of an array item, or the weight of a picture or a BLOB before the storage. It's a numeric value.

For a BLOB or an image, if a second pointer is passed as 3rd parameter, the variable pointed to will be filled with the compressed size of the item. This is a numeric value.

**value** the variable pointed to by the second parameter is populated with the data of the stored element. If possible, a data conversion is performed according to the variable pointed.

## Utility Routines

### **bag_findByName** ( *name* {; ->*bagHandles* }) -> *bagHandle*

Returns the reference of the first created bag whose name is equivalent to the string passed in parameter name. If several bags are found, the command can also fill the array passed by pointer in the second parameter with the reference of each bag.

```
$BagHandle := bag_findByName ( "myBag" )
$BagHandle := bag_findByName ( "myMultipleBags" ; -> bagHandleArray )
```

If you know that there are several bags with the same name, you can retrieve the reference of one of them with this syntax for name : "name[n]" where n is the order of creation of the bag named name.

```
$BagHandle_1 := bag_new ( "options" )
$BagHandle_2 := bag_new ( "options" )
$BagHandle_3 := bag_new ( "options" )
…
$BagHandle := bag_findByName ( "options[2]" ) // $BagHandle  = $BagHandle_2
```

### **bag_forEach** ( *bagHandles* {; ->*itemValue* })

This command allow to go through all, first level items, of a bag in a "While…End while" control flow structure. The command can also fill the second parameter passed by pointer with the value of the item.

```
C_REAL ( $sum ; $value)
While( bag_forEach ( $Lon_bag ; -> $value ))
	$sum := $sum + $value
End while
```
	
See also the "forEach()" method of a bag

### **bag_rename** ( *bagHandles* ; *name* )

Rename the bag whose reference is bagHandle with the sting passed in parameter name. 

### **bag\_SET_OPTION** ( *option* {; *on* })

This command allow to turn on/off some options of the component according to the boolean passed in the second parameter. If the second parameter is omitted, the option is turned off.

Option value can be:

|options         |meaning                                                             |default  |
|:---------------|:-------------------------------------------------------------------|:--------|
|"default"       |All options are restored to their default value                     |         |
|"assertions"    |to turn on/off assertion [^assertion]                               |**True** |
|"variant types" |turns on/off the modification of the type of an element [^variant]  |**False**|


[^assertion]: An assertion is a message to inform the developer about something that is (error) or seems to be (warning) an anomaly. When the assertions are turned "On", the code isn't completed if it's an error. When the assertions are turned "Off", the code is completed if possible without message. Usually, you set assertions "On" in development mode or test mode and "Off" for the final user.

[^variant]: By default, if you try to put a value into an item of a different type, an error is generated. However if you want to change a type of an item you can change the behavior by this option.

 