php commands
==============

    file_put_contents($file, $contents)   write $contents to $file
    file_get_contents($file)              read $file into a variable
    Insert end-of-line code:              echo PHP_EOL;

    uniqid              Generate a unique id string based on timestamp
    serialize           Generate a (storable) string from an object/array
    unserialize         Load (stored) string data into an object/array
    end(array)          Return value of last element in array
    compact('items')    shorthand for array('items' => $items);
    strip_tags($str)    strips all html tags from $str
    get_class           Returns the name of a class from an object
    get_class_methods   Returns all public methods of a class
    
    is_callable($c, $s) Returns true if the class $c has (public) method $s
    method_exists       Similar to is_callable, but for all methods (including protected/private) 

    var_export()        Like var_dump, but writes data in a way it can be captured. For instance:

    \Log::debug(var_export(DB::getQueryLog(), true));  // writes debug info to the log file

    array_merge         merge two simple arrays
    array_replace_recursive  merge two complex arrays; replace matching items


Replace a newline with a <br> code:
return str_replace(array("\r\n", "\n", "\r"), '<br>', e($model->field));



Reflection <a name="reflection">
-----------------------------------

This is a standard PHP class, used to report information about a class. Official documentation at:
http://www.php.net/manual/en/class.reflectionclass.php

Laravel uses it internally for several things. Externally, we can use it to see a list of available methods for a class. Like this:

```php
    $r = new ReflectionClass('ClassFacadeOrAliasName');
    $methods = $r->getMethods();

    $methodArray = array();
    foreach($methods as $method) {
        $methodArray[$method->getName()] = $method->getDocComment();
    }
    ksort($methodArray);
    return $methodArray;
```php

It has several interesting things to choose from:

    getMethods          // gets a list of public methods for the class
    getProperties       // public properties
    getConstants        // constants defined by the class
    getDocComment       // DocBlock comments for a class (or method, or property)
    getFileName         // Gets the name of the file the class is defined in
    getInterfaceNames   // List all interfaces the class instanciates
    getNamespaceName    // Return the full namespace of the class
    getParentClass      // Return the name of the parent class for this class
    getMethod()->getParameters()

We can also use it in testing, to test private and protected methods. There's some disagreement about whether private and protected methods should be tested; doing so makes the tests more brittle, but it will point out precisely where things are failing, if there's an error. For instance:

```php
    /**
     * @dataProvider getSearchStrings
     */
    public function testSplitIntoWords($input, $expected)
    {
        // use a reflection class to make this method testable
        $class = new \ReflectionClass('KBase\Repositories\Searcher');
        $method = $class->getMethod('splitStringIntoWords');
        $method->setAccessible(true);
        $output = $method->invokeArgs($class, array($input));
        $this->assertEquals($expected, $output);
    }
```



Passing by Reference
-------------------------
Use `&` to pass a reference, rather than a value. For instance, to modify an array in place:

``` php 
    protected function injectTagIntoArrayForId(&$array, $id, $tag=' selected="selected"')
    {
        foreach($array as &$item) {
            if($item['id']==$id) {
                $item['extra'] = $tag;
                return;
            }
        }
    }
```



Static vs Non-static
----------------------

    Non-static:  $this->function(x);
    Static:      self::function(x);

In a static function, you can only call other static functions, unless you create an instance, like so: 

    $instance = new static;
    $instance->function(x);

There are some interesting variations on a couple of static calls:

    self::$items       items in this class
    static::$items     items in this class, or descendents of this class


Magic functions
-----------------
Magic functions run at (given) times. For instance, __call runs whenever a function can not be found, so you can do something else:

``` php
    public static function __call($name, $args)
    {
        $fn = strtolower($name);

        if (substr($fn, 0, 8) == 'gettable'){
            $table = $this->getTable();
            return $table[substr($fn, 8)];
        } elseif (substr($fn, 0, 8) == 'getfield') {
            $fields = $this->getFields();
            return $fields[substr($fn, 8)];
        }
        return strtolower($name);
    }
```

    
Namespaces
------------

Based off the folder structure.

    My project: package/src/Vendor/Package.php
        github: jijoel/Package/src/Vendor/Package.php

    namespace Vendor\Package

You can call methods from a namespaced class like this:

    $foo = (new Foo\Bar)->method();

You can also run tests that call standard functions based on your namespace. For instance, time. If there is no time() function in your namespace, php will just use the default time() function. If you have it declared in your namespace, though, it will use that. eg,

Foo.php:

```php
    namespace Fizz;
    class Foo {
        function bar { return time(); }
    }
```

FooTest.php:

```php
    namespace Fizz;
    function time() { return 'buzz'; }

    class FooTest extends \PHPUnit_Framework_TestCase {
        function testGetBar() {
            $result = (new Foo)->bar();
            $this->assertEquals('buzz', $result);
        }
    }
```

