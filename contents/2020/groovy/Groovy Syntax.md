---
layout: page
title: Groovy Syntax 
permalink: /groovy-syntax/
---



## Groovy Class
- https://groovy-lang.org/syntax.html

~~~groovy
class Person {
    String name

    String greet(String otherPerson) {
       "Hello ${otherPerson}"
    }
}
~~~

### Groovy Map 

~~~groovy
def map = [:]

map."name" = "Groovy"
map.'type' = "Dynamic"

assert map."name" == "Groovy"
assert map.'type' == "Dynamic"

def firstname = "Homer"
map."Simpson-${firstname}" = "Homer Simpson"

assert map.'Simpson-Homer' == "Homer Simpson"
~~~

### Strings

~~~groovy
assert 'ab' == 'a' + 'b'

def aMultilineString = '''line one
line two
line three'''

def strippedFirstNewline = '''\
line one
line two
line three
'''

assert !strippedFirstNewline.startsWith('\n')

def name = 'Groovy' // a plain string
def greeting = "Hello ${name}"

assert greeting.toString() == 'Hello Groovy'


def sum = "The sum of 2 and 3 equals ${2 + 3}"
assert sum.toString() == 'The sum of 2 and 3 equals 5'

def person = [name: 'Guillaume', age: 36]
assert "$person.name is $person.age years old" == 'Guillaume is 36 years old'

def number = 3.14
shouldFail(MissingPropertyException) {
    println "$number.toString()"
}


String thing = 'treasure'
assert 'The x-coordinate of the treasure is represented by treasure.x' ==
    "The x-coordinate of the $thing is represented by $thing.x"   // <= Not allowed: ambiguous!!
assert 'The x-coordinate of the treasure is represented by treasure.x' ==
        "The x-coordinate of the $thing is represented by ${thing}.x"  // <= Curly braces required

assert '$5' == "\$5"
assert '${name}' == "\${name}"

// Interoperability with Java

String takeString(String message) {         
    assert message instanceof String        
    return message
}

def message = "The message is ${'hello'}"   
assert message instanceof GString           

def result = takeString(message)            
assert result instanceof String
assert result == 'The message is hello'
~~~

### Integral literals
~~~groovy
def a = 1
assert a instanceof Integer

// Integer.MAX_VALUE
def b = 2147483647
assert b instanceof Integer

// Integer.MAX_VALUE + 1
def c = 2147483648
assert c instanceof Long

// Long.MAX_VALUE
def d = 9223372036854775807
assert d instanceof Long

// Long.MAX_VALUE + 1
def e = 9223372036854775808
assert e instanceof BigInteger

def na = -1
assert na instanceof Integer

// Integer.MIN_VALUE
def nb = -2147483648
assert nb instanceof Integer

// Integer.MIN_VALUE - 1
def nc = -2147483649
assert nc instanceof Long

// Long.MIN_VALUE
def nd = -9223372036854775808
assert nd instanceof Long

// Long.MIN_VALUE - 1
def ne = -9223372036854775809
assert ne instanceof BigInteger
~~~


### Binary literal
~~~groovy
int xInt = 0b10101111
assert xInt == 175

short xShort = 0b11001001
assert xShort == 201 as short

byte xByte = 0b11
assert xByte == 3 as byte

long xLong = 0b101101101101
assert xLong == 2925l

BigInteger xBigInteger = 0b111100100001
assert xBigInteger == 3873g

int xNegativeInt = -0b10101111
assert xNegativeInt == -175

~~~

### Octal literal
~~~groovy
int xInt = 077
assert xInt == 63

short xShort = 011
assert xShort == 9 as short

byte xByte = 032
assert xByte == 26 as byte

long xLong = 0246
assert xLong == 166l

BigInteger xBigInteger = 01111
assert xBigInteger == 585g

int xNegativeInt = -077
assert xNegativeInt == -63
~~~

### Hexadecimal literal
~~~groovy
int xInt = 0x77
assert xInt == 119

short xShort = 0xaa
assert xShort == 170 as short

byte xByte = 0x3a
assert xByte == 58 as byte

long xLong = 0xffff
assert xLong == 65535l

BigInteger xBigInteger = 0xaaaa
assert xBigInteger == 43690g

Double xDouble = new Double('0x1.0p0')
assert xDouble == 1.0d

int xNegativeInt = -0x77
assert xNegativeInt == -119
~~~

### Decimal literals
~~~groovy
// primitive types
float  f = 1.234
double d = 2.345

// infinite precision
BigDecimal bd =  3.456

assert 1e3  ==  1_000.0
assert 2E4  == 20_000.0
assert 3e+1 ==     30.0
assert 4E-2 ==      0.04
assert 5e-1 ==      0.5
~~~

### Underscore in literals
~~~groovy
long creditCardNumber = 1234_5678_9012_3456L
long socialSecurityNumbers = 999_99_9999L
double monetaryAmount = 12_345_132.12
long hexBytes = 0xFF_EC_DE_5E
long hexWords = 0xFFEC_DE5E
long maxLong = 0x7fff_ffff_ffff_ffffL
long alsoMaxLong = 9_223_372_036_854_775_807L
long bytes = 0b11010010_01101001_10010100_10010010
~~~

### Number type suffixes
- We can force a number (including binary, octals and hexadecimals) to have a specific type by giving a suffix , 
    - either uppercase or lowercase.
    
~~~groovy
assert 42I == new Integer('42')
assert 42i == new Integer('42') // lowercase i more readable
assert 123L == new Long("123") // uppercase L more readable
assert 2147483648 == new Long('2147483648') // Long type used, value too large for an Integer
assert 456G == new BigInteger('456')
assert 456g == new BigInteger('456')
assert 123.45 == new BigDecimal('123.45') // default BigDecimal type used
assert 1.200065D == new Double('1.200065')
assert 1.234F == new Float('1.234')
assert 1.23E23D == new Double('1.23E23')
assert 0b1111L.class == Long // binary
assert 0xFFi.class == Integer // hexadecimal
assert 034G.class == BigInteger // octal
~~~

### Math operations
- binary operations between byte, char, short and int result in int
- binary operations involving long with byte, char, short and int result in long
- binary operations involving BigInteger and any other integral type result in BigInteger
- binary operations involving BigDecimal with byte, char, short, int and BigInteger result in BigDecimal
- binary operations between float, double and BigDecimal result in double
- binary operations between two BigDecimal result in BigDecimal

~~~groovy
// base and exponent are ints and the result can be represented by an Integer
assert    2    **   3    instanceof Integer    //  8
assert   10    **   9    instanceof Integer    //  1_000_000_000

// the base is a long, so fit the result in a Long
// (although it could have fit in an Integer)
assert    5L   **   2    instanceof Long       //  25

// the result can't be represented as an Integer or Long, so return a BigInteger
assert  100    **  10    instanceof BigInteger //  10e20
assert 1234    ** 123    instanceof BigInteger //  170515806212727042875...

// the base is a BigDecimal and the exponent a negative int
// but the result can be represented as an Integer
assert    0.5  **  -2    instanceof Integer    //  4

// the base is an int, and the exponent a negative float
// but again, the result can be represented as an Integer
assert    1    **  -0.3f instanceof Integer    //  1

// the base is an int, and the exponent a negative int
// but the result will be calculated as a Double
// (both base and exponent are actually converted to doubles)
assert   10    **  -1    instanceof Double     //  0.1

// the base is a BigDecimal, and the exponent is an int, so return a BigDecimal
assert    1.2  **  10    instanceof BigDecimal //  6.1917364224

// the base is a float or double, and the exponent is an int
// but the result can only be represented as a Double value
assert    3.4f **   5    instanceof Double     //  454.35430372146965
assert    5.6d **   2    instanceof Double     //  31.359999999999996

// the exponent is a decimal value
// and the result can only be represented as a Double value
assert    7.8  **   1.9  instanceof Double     //  49.542708423868476
assert    2    **   0.1f instanceof Double     //  1.0717734636432956

def myBooleanVariable = true
boolean untypedBooleanVar = false
booleanField = true

~~~

### Lists

~~~groovy
def numbers = [1, 2, 3]         

assert numbers instanceof List  
assert numbers.size() == 3

def heterogeneous = [1, "a", true] 

def arrayList = [1, 2, 3]
assert arrayList instanceof java.util.ArrayList

def linkedList = [2, 3, 4] as LinkedList    
assert linkedList instanceof java.util.LinkedList

LinkedList otherLinked = [3, 4, 5]          
assert otherLinked instanceof java.util.LinkedList


def letters = ['a', 'b', 'c', 'd']

assert letters[0] == 'a'     
assert letters[1] == 'b'

assert letters[-1] == 'd'    
assert letters[-2] == 'c'

letters[2] = 'C'             
assert letters[2] == 'C'

letters << 'e'               
assert letters[ 4] == 'e'
assert letters[-1] == 'e'

assert letters[1, 3] == ['b', 'd']         
assert letters[2..4] == ['C', 'd', 'e']  

def multi = [[0, 1], [2, 3]]     
assert multi[1][0] == 2 
~~~

### Arrays
~~~groovy
String[] arrStr = ['Ananas', 'Banana', 'Kiwi']  

assert arrStr instanceof String[]    
assert !(arrStr instanceof List)

def numArr = [1, 2, 3] as int[]      

assert numArr instanceof int[]       
assert numArr.size() == 3

def matrix3 = new Integer[3][3]         
assert matrix3.size() == 3

Integer[][] matrix2                     
matrix2 = [[1, 2], [3, 4]]
assert matrix2 instanceof Integer[][]


String[] names = ['Cédric', 'Guillaume', 'Jochen', 'Paul']
assert names[0] == 'Cédric'     

names[2] = 'Blackdrag'          
assert names[2] == 'Blackdrag'

def primes = new int[] {2, 3, 5, 7, 11}
assert primes.size() == 5 && primes.sum() == 28
assert primes.class.name == '[I'

def pets = new String[] {'cat', 'dog'}
assert pets.size() == 2 && pets.sum() == 'catdog'
assert pets.class.name == '[Ljava.lang.String;'

// traditional Groovy alternative still supported
String[] groovyBooks = [ 'Groovy in Action', 'Making Java Groovy' ]
assert groovyBooks.every{ it.contains('Groovy') }

~~~

### Maps
~~~groovy
def colors = [red: '#FF0000', green: '#00FF00', blue: '#0000FF']   

assert colors['red'] == '#FF0000'    
assert colors.green  == '#00FF00'    

colors['pink'] = '#FF00FF'           
colors.yellow  = '#FFFF00'           

assert colors.pink == '#FF00FF'
assert colors['yellow'] == '#FFFF00'

assert colors instanceof java.util.LinkedHashMap

def numbers = [1: 'one', 2: 'two']

assert numbers[1] == 'one'

def key = 'name'
def person = [key: 'Guillaume']      

assert !person.containsKey('name')   
assert person.containsKey('key')  

~~~