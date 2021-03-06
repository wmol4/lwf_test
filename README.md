#lwf_test -- LightWeight Function Test

*Created on Fri Dec  2 15:05:45 2016*

*@author: Tom Blanchet*

##Description:

This is a simple module for doing quick and basic functional testing of python 3.5 code. Built with many helpers, including
an inheritable Timer and print-outs for tests
    
##Installation:

To install lwf_test, simply go to the root of this repository, and execute the following in your command line:

    >>> python setup.py install

To test the installation, use:

    >>> lwf_test -t

*(Only pay attention to the final message to see if the installation succeeded with the "All Tests Passed!" message.
The "tests" that are printed are pre-defined tests for the package that include those designed to fail)*


##Basic Usage:##

The main tools in this module are the "makeTester" wrapper function,
the "Tester" method that binds to the function, and the "printFinalResults"
helper function. 

The simplist use case is the following:
```python
from lwf_test import makeTester, printFinalResults

@makeTester()
def myFunction(a, b, c):
    ...
    return ret

myFunction.Tester(ret_I_expect, test_a, test_b, test_c)

printFinalResults()
```
Tester will either return "success", "failure", or "error" by default, naturally depending on the outcome of the test, as well as format and print the result and information of the individual test. Then, printFinalResults() will format and print a summary of all the tests conducted. The printed output for the above will look something like this:
```
Result for myFunction test # 1:
::::::SUCCESS::::::
    Details:
    Args: (<test_a>, <test_b>, <test_c>)
    Kwargs: {}
    Expected output: <ret_I_expect>
    True output: <ret_I_expect>

======= Final Test Summary =======
Total Tests: 1
Successes: 1
Failures: 0
Errors: 0
```

Below are a few more ways to use lwf_test and some examples in 
executable code:

```python
from lwf_test import *

disableVerboseTests() #Will not print individual outcomes this way

#All keword arguments for makeTester are optional.
#   catchErrors modifies what are considered "acceptable" errors
#   durring testing
@makeTester(catchErrors = ZeroDivisionError)
def myFunction(a, b):
    return a/b

myFunction.Tester(5, 15, 3) #Will return "success"
myFunction.Tester(1, 4, 2)  #Will return "failure"
myFunction.Tester(20, 4, 0) #Will return "error"

try:
    #This try block will raise an error that will not be caught
    #and the test will not be registered in TestResultsHelper

    myFunction.Tester(3, "hi", "there")

except TypeError as te: 
    print("fatal error:", te)
    #raise

    #Ideally, you would want to raise these errors
    #so that testing stops. 

#Gets the TestResultsHelper instance for the named function. 
#Can also use TestResultsHelper.getInstanceForFunc(myFunction)

testHelper = TestResultsHelper['myFunction']

#Will print "3", ignoring the unexpected error
print("Total number of tests:", testHelper.getTotalTests())

#Will print (in some order):
#>>> successes : 1
#>>> failures : 1
#>>> errors : 1
for k, v in testHelper.getOutcomeTotals().items():
    print(k, ":", v)

    from lwf_test import makeTester, printFinalResults,\
                SUCCESS_KEY, FAILURE_KEY, ERROR_KEY, TestResultHelper

#Since we disabled printing for each of the tests above, we can print out
#all the tests that happened before using the "verbose" option in
#printFinalResults.
printFinalResults(verbose = True)

enableVerboseTests() #You can re-enable verbose tests at any time

#You can bind testers to many different kinds of functions
#As long as the wrapped object is callable, a tester can be bound to it!
@makeTester()
def nothing():
    return

@makeTester()
def add(a, b):
    return a + b

lambda_add = makeTester()(lambda a, b: a + b)

class simpleObject(object):
    number = 0
    total = 0
    def __init__(self):
        simpleObject.total += 1
        self.index = simpleObject.total
    @makeTester()
    def doSomething(self):
        simpleObject.number += 1
        return str(self.index) + "hi" + str(simpleObject.number)

#The tester returns the associated global key for each outcome

assert nothing.Tester()         == SUCCESS_KEY
assert nothing.Tester(None)     == SUCCESS_KEY
assert nothing.Tester(None, 2)  == ERROR_KEY
assert add.Tester()             == ERROR_KEY
assert add.Tester(5, 1, 2)      == FAILURE_KEY
assert add.Tester(5, 2, 3)      == SUCCESS_KEY
assert lambda_add.Tester(5, 1, 2)      == FAILURE_KEY
assert lambda_add.Tester(5, 2, 3)      == SUCCESS_KEY

objA = simpleObject()
objB = simpleObject()

assert simpleObject.doSomething.Tester("GIVE ERROR") == ERROR_KEY

assert simpleObject.doSomething.Tester("1hi1", objA) == SUCCESS_KEY
assert simpleObject.doSomething.Tester("1hi2", objA) == SUCCESS_KEY
assert simpleObject.doSomething.Tester("2hi3", objB) == SUCCESS_KEY
assert simpleObject.doSomething.Tester("2hi4", objB) == SUCCESS_KEY

assert objA.doSomething() == "1hi5"

printFinalResults()
```
