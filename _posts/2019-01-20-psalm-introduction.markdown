---
layout: post
title:  "Better Type Checking with Psalm"
date:   2019-01-20 00:00:00 +0930
---

In 2019, strong and static types are being talked about more and more, especially in most of the programming communtiies that I'm part of.

PHP has had typehints for some time now, and now with things like typed function returns avaialble, scalar typehints and typed properties coming in 7.4 we're moving to a world where strong types are everywhere.

Unfortunatley, there are some shortfalls with PHP and its type system, due to the nature of the language, being dynamically typed, strong typing, via typehints is something not at the core of the language there are other shortcomings like not being able to tell the language you have an array of X, or generics. These features have come up in numerous RFC in the core over the years to give us this, some have been accepted, some have not.

Forunatly for us, there are some tools that allow us to get some 'better' static typing in PHP, PHPStan, Phan, and the one I'm going dive into, (Psalm)[https://github.com/vimeo/psalm].

Psalm's tag line is "Life is complicated. PHP can be, too. Psalm is designed to understand that complexity, allowing it to quickly find common programmer errors like null references and misspelled variable names."

The biggest save that we've found, is that Psalm forces us to really handle those error cases, null checks, type changes, and things that make your code do unexpected things.

To get started with Psalm you first need to install it into your project using composer:

```
    composer require --dev vimeo/psalm
```

Once composer has done it's thing, you can setup a Psalm configuration file, there are multiple run levels for how strict you want your checks to be, if you have a legacy project, you may way to start at a lower level and increase strictness slowly, so you are not overwhelmed with 1000s of errors at once. By default, Psalm sets all warning to the 'info' level.

```
    ./vendor/bin/psalm --init src
```

Take the following contrived example of a UserRepositories with User objects, with no type information.

```
class User
{
    private $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function name()
    {
        return $this->name;
    }
}

class UserRepository
{
    private $users = [];

    public function __construct()
    {
        $this->users = [
            100 => new User('Sam'),
            103 => new User('Michael'),
            104 => new User('Damo'),
            200 => new User('Tom')
        ];
    }

    public function find($id)
    {
        if (array_key_exists($id, $this->users)) {
            return $this->users[$id];
        }

        return null;
    }
}

$userRepository = new UserRepository();

$user = $userRepository->find(100);

echo $user->name();
```

We're not using any of the languages type system, and Psalm tells us all about this, and even tell us what we should do to make things better.


```
-> % ./vendor/bin/psalm
Scanning files...
Analyzing files...

ERROR: MissingPropertyType - src/Demo.php:5:13 - Property User::$name does not have a declared type
    private $name;


ERROR: MissingParamType - src/Demo.php:7:33 - Parameter $name has no provided type
    public function __construct($name)


ERROR: MissingReturnType - src/Demo.php:12:21 - Method User::name does not have a return type
    public function name()


ERROR: MissingPropertyType - src/Demo.php:20:13 - Property UserRepository::$users does not have a declared type - consider array{100?:User, 103?:User, 104?:User, 200?:User}
    private $users = [];


ERROR: MissingReturnType - src/Demo.php:32:21 - Method UserRepository::find does not have a return type
    public function find($id)


ERROR: MissingParamType - src/Demo.php:32:26 - Parameter $id has no provided type
    public function find($id)


------------------------------
6 errors found
------------------------------

Checks took 0.13 seconds and used 28.878MB of memory
```

In the next post, I'll go through adding types, what we can do at the language level, and what we may need to go outside of those bounds for, and what the future for PHP might hold.
