# MiniTest & MiniMock

This is a library designed for unit testing & mocking in the [MiniScript](https://miniscript.org) programming language.

## Global behaviours

If you are running within GreyHack, set `globals.isGH = true` to get coloured printouts.

If you are running on a version of MiniScript with `stackTrace` set `globals.__Intrinsics = {"stackTrace": @stackTrace}` prior to loading MiniTest/MiniMock - otherwise, it is advised to use the additional `note` parameter on your assert/expect calls.

## MiniTest

Tests are defined using `TEST(string, function)` - the preferred way to declare a test is like so:

```Lua
TEST "Foo does Bar", function()
    assertTrue true, "my failure note"
    assertTrue true // failure note is optional
    // more test code here
end function
```

### Setup & Teardown

You can pass a function to both `TEST_SETUP(function)` and `TEST_TEARDOWN(function)` which will respectively be called before and after each test. calling these is entirely optional.

### Running Tests

In order for your tests to execute, call `RUN_ALL_TESTS(verbose = true, retainTests = false)`. If you disable verbose mode, only failures are printed. If you enable `retainTests` they will not be evicted from the queue and will execute again if you call `RUN_ALL_TESTS` - additionally, your setup and teardown functions will remain defined.

### Assertions/Expectations

Every check comes in a pair - one is prefixed `assert` and the other is prefixed `expect`. Using `assert` will terminate whereas expect will not. An error is printed in either case. For brevity, only the `assert` functions are listed.

All these functions accept an optional final argument which should be a string. This will be printed along with an error if the check fails.

`assertEqual actual, expected` - asserts that the two arguments match by equality (`==`)<br/>
`assertNotEq actual, expected` - asserts that the two arguments match by inequality (`!=`)<br/>
`assertIsa actual, expected` - asserts that `actual isa expected`<br/>
`assertIsnt actual, expected` - asserts that `not actual isa expected`<br/>
`assertTrue actual` - asserts that `actual` is exactly equal to `true` - in MiniScript, this includes the number `1`<br/>
`assertTruthy actual` - asserts that `actual` is non-`null` and not equal to `false`/`0`<br/>
`assertFalse actual` - asserts that `actual` is exactly equal to `false` - in MiniScript, this includes the number `0`<br/>
`assertNull actual` - asserts that `actual` is exactly equal to `null`

## MiniMock

MiniMock enables you to generate mocks from existing `map` objects, define your own from scratch, or a mixture of both. A Mock requires that functions be called in the order you set them up. MiniMock will terminate with an error if calls occur out of order or if a call is made that is unexpected.

MiniMock currently supports functions taking up to **7** parameters. Modifying the source code to handle more than this is trivial, should you need to do so.

### Creating and Using an Instance of a Mock

`mock = MiniMock.build(object)` returns a new mock object based on `object`. To start with nothing, simply pass `{}`. Subsequent examples will act as a continuation of this line (i.e. you named the mock object `mock`)

`mock.define("function", paramCount)` - manually defines a function with the string name you pass and parameter count `paramCount`

`mock.expectCall("function").withParams([paramOne, paramTwo, MiniMock.Any]).thenInvoke(@myFunc).andReturn(123)` - sets up an expectation for a call to `function` with 3 parameters. The third parameter is permitted to be anything. After params are validated, your function `myFunc` will be invoked. If your function returns any non-`null`, this will be passed back to the caller. Otherwise, `123` would be returned.

`withParams` is optional if the function doesn't have any parameters. Additionally, if the function takes only one parameter, you can omit `[]`<br/>
`thenInvoke` is always optional. It will be passed the arguments the caller sent in a list (`[]`)<br/>
`andReturn` is always optional, though the caller will receive `null` as for any call to a function with no return statement in MiniScript.<br/>

At this point, you can pass `mock` to whatever you are testing.

### Mocking Globals

This can be achieved through creating an empty mock and using `.define` to declare the functions - for example: `globals.print = mock.define("print", 1)`.

If needed, you can of course backup and restore the globals. If you are overriding any intrinsics, since they don't exist in `globals` to begin with, just use `globals.remove` to delete the key you created.

### Validating a Mock

An important final step is to do something like `assertTrue mock.getResult` - this will print an error if any expectations were unfulfilled. Generally recommended to place these somewhere like your `TEST_TEARDOWN` as soon as possible. Of course, if you don't need or want to check if the mocks were actually called, don't.

### Caveats

If you mock something that defines either `expectCall`, `define` or `getResult` then those functions themselves will become mock functions. In this case, use something like `expectCall = @MiniMock.expectCall` and you can then use `expectCall(myMock, "function")`.

