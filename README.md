# Easy Lua

Easy Lua is the simplest way to add [Lua](http://www.lua.org/) scripts to your Adobe Flash or ActionScript project.
It builds on top of the excellent Lua port that ships with Adobe's open source [CrossBridge](https://github.com/adobe-flash/crossbridge) project.

#### Features

- Hides all the low level Lua C API details in one easy to use class.  No Lua C API knowledge required.
- Works with all ActionScript platforms (web, desktop, and mobile).
- Automatically converts Lua return values to ActionScript variables for easy interoperability.
- Supports multiple instances of the Lua interpretter so that you can sandbox separate scripts.
- Simplifies the process of loading embedded Lua scripts in your project.

#### Todo

- Currently ActionScript can call Lua code (and receive a response), but Lua code can't call ActionScript directly.

## Installation

Installation is simple.
First, copy the files in the `src` and `lib` directories to your project.
Second, tell the compiler to link your code to `lib/lua.swc`.

## Basic Usage

Create an instance of the `EasyLua` class:

    var easyLua:EasyLua = new EasyLua();

Call the `eval` method to execute some Lua code.  This code defines a `helloWorld` function that returns a string:

    easyLua.eval("function helloWorld() return 'hello world' end");

Call the function that we defined above and return the result back to ActionScript:

    var result:String = easyLua.eval("return helloWorld()");

You must explicity add the `return` keyword to tell Lua to return the value back to ActionScript.
Failure to do so will result in no value being returned.
This way you get to decide if you want to deal with the conversion overhead.
Most basic Lua types (nil, numbers, strings, booleans, and tables) are supported.
An exception will be raised if you try to return an unsupported type.

Easy Lua provides a helper method called `evalFunction` to simplify all of the above.
Unlike `eval`, this method will automatically include a `return` along with convert any arguments to their Lua equivalents.

    easyLua.eval("function helloWorldImproved(arg) return arg end");
    var result:* = easyLua.evalFunction('helloWorld', { foo: 'bar' });

In Lua, tables can act as integer indexed arrays or as hashes (associative arrays).
Easy Lua tries to automatically convert tables to the correct actionscript type (`Array` or `Object`).
The `autoConvertArrays` setter can disable the automatic conversion (and save some CPU cycles).
Tables will then always convert to `Object`:

    easyLua.autoConvertArrays = false;

You must manually `dispose` instances of Easy Lua when you are finished with them.
This ensures that the resources used by Lua are released.

    easyLua.dispose()


## Embedding Scripts

Most non-trivial Lua programs store their code in files.
Easy Lua provides the `evalEmbedded` method so that you can embed your Lua code files and load them at runtime.
It is designed to be used with ActionScript's embedded asset feature where each asset file becomes its own class.

First you will need to embed your scripts just like you would embed any other asset.
Below is an example `asset/hello.lua` that you would create in your project's root folder:

    function helloWorld()
      return "hello world"
    end

Below is an example `src/MyAssets.as` where you define all of your asset classes.

    package
    {
      public class MyAssets
      {
        [Embed(source="../asset/hello.lua", mimeType="application/octet-stream"))]
        public static var helloLuaScript:Class;
      }
    }

You then load each of your assets files using the `evalEmbedded` method in your application code:

    var easyLua:EasyLua = new EasyLua();
    easyLua.evalEmbedded(MyAssets.helloLuaScript);
    var result:String = easyLua.eval("return helloWorld()");

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
