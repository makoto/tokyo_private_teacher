!SLIDE center
# Try Tokyo #

!SLIDE full-page

## 1. Tokyo Cabinet and Tokyo Tyrant

!SLIDE full-page

* Tokyo Cabinet(TC) is a high performance DBM(Data Base Manager). TC supports hash(TCH) , B+ tree(TCB), fixed-length array(TCF) and table database type(TCT). Each database supports various functionalities with some performance tradeoffs. Each database can be accessed through various programming language bindings(Java, Lua, Perl, Ruby are officially supported) or through Tokyo Tyrant.

* Tokyo Tyrant(TT) is a network interface to Tokyo Cabinet. TT offers multiple process client access (such as web server). TT uses the abstract API of Tokyo Cabinet(TCA) which offers common interface to various TC database types. You can access TT via http client, memcached client, various programming language bindings, or command line tools which we use for this tutorial

!SLIDE full-page

## 2. Command Line tools

## ttserver

The command `ttserver' is a command to startup server.

    ttserver [-host name] [-port num] [options] [dbname]

If you run it without any parameter, it will start as on-memory hash database.

    ttserver

## tcrmgr

The command `tcrmgr' is a utility for test and debugging TC through TT.

    tcrmgr put [-port num] [-options] host key value

In this tutorial, you specify "localhost" at "host"

    tcrmgr put localhost foo bar

  
For the full command list, refer to http://1978th.net/tokyotyrant/spex.html#clientprog

!SLIDE full-page

## 3. What is TCH.

Hash (TCH) provides basic functionality for key value stores. Key can be binary or text, but  must be unique. In adition to basic operations (get/put/out), you can do various operations, such as appending values to a value to use like array (putcat), traversing values(iterinit/iternext), and storing multiple records(putlist/getlist), 

To start up TT in on-memory hash database mode, specify either "*" or no options.

    $ttserver *

To start up TT in file hash database mode, specify ".tch" suffix.

    $ttserver test.tch


!SLIDE full-page

## 4. TCH basic operations - part 1

## put/get

"put hostname key value" lets you store key/value record.

"get hostname key" returns a value of the key you looked up.

    $  tcrmgr put localhost one first
    $  tcrmgr put localhost two second
    $  tcrmgr get localhost one
    first

!SLIDE full-page

## 5. TCH basic operations - part 2

## list/mget

What if you want to extract multiple keys?

"list hostname" shows all keys in the database.

"mget hostname key key ..." shows values of the keys you looked up.

      $ tcrmgr list localhost
      one
      two
      $ tcrmgr mget localhost one two
      one	first
      two	first

!SLIDE full-page

## 6. TCH basic operations - part 3

## out

"out hostname key" deletes a record of the key you specified.

      $ tcrmgr out localhost one
      $ tcrmgr list localhost 
      two

!SLIDE full-page

## 7. TCH command options - part 1

## -dc

There are various options on tcrmgr"

One example is "-dc". This option lets you append values

      $ tcrmgr put -dc localhost two ',dos'
      $ tcrmgr get localhost two
      second,dos

!SLIDE full-page

## 8. TCH command options - part 2

## -ai

There are various options on tcrmgr"

Another example is "-ai". This option lets you increment values and return the incremented value.

      $ tcrmgr put -dai localhost counter 1
      1
      $ tcrmgr put -dai localhost counter 1
      2
      $ tcrmgr put -dai localhost counter 1
      3

If you only want to extract the result without incrementing, just specify 0.

      $ tcrmgr put -dai localhost counter 0
      3

!SLIDE full-page

## 9. TCH command options - part 3

## -pv

In earlier example, you used "list" and "mget" to print out all values in a database.
You can use "-pv" options instead to list all the results with values.

      $ tcrmgr list -pv localhost 
      counter	
      two	second,dos

!SLIDE full-page

## 10. Lua extensions 

## exec

TT supports Lua extensions to enhance TC/TT capability.
To include lua extension, you have to startup TT with -ext options.

      $ ttserver -ext eval.lua test.tch

(Example file from http://github.com/igrigorik/tokyo-recipes/blob/master/dynamic-extension/eval.lua)

Once started up, use "exec" command to execute the lua functions.
 
      $ tcrmgr ext localhost eval 'function hello() return "hello" end'
      $ tcrmgr ext localhost hello
        hello
      $ tcrmgr ext localhost eval 'function hello() return "bye" end'
      $ tcrmgr ext localhost hello
        bye

For more information , check http://www.igvita.com/2009/07/13/extending-tokyo-cabinet-db-with-lua

!SLIDE full-page

## 11. TCH advanced operations - part 1

## misc iternext

If you want to use more advanced operations, you need to specify them as subcomands of misc command.

Few examples are "iterinit/iternext" which traverse the database. This is quite useful when your database size is relatively big and won't to avoid getting all the result with list, though the order is arbitrary.

      $ tcrmgr misc localhost iterinit
      $ tcrmgr misc localhost iternext
      bar
      uno
      $ tcrmgr misc localhost iternext
      counter

      $ tcrmgr misc localhost iternext
      two
      second,dos

You will see more example of "misc" when you use specific features of other database types, such as B+Tree and Table.
For more information, check http://tokyocabinetwiki.pbworks.com/37_hidden_features_of_the_misc_method

!SLIDE full-page

## 12. TCH advanced operations - part 2

## misc  

You can also add multiple key/values in one operation.

      $ tcrmgr misc localhost putlist one uno two dos three tres
      $ tcrmgr misc localhost getlist one two three
      one
      uno
      two
      dos
      three
