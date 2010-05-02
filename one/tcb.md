!SLIDE center
# Try Tokyo #

!SLIDE full-page

## 1. What is TCB.

TCB stands for Tokyo Cabinet B+Tree database. Unlike TCH(Tokyo Cabinet Hash database), you can store duplicate keys and also store them in specific order. You can access each record with the cursor in ascending or descending order. Thanks to this mechanism, You can operate range search.

To start up TT in on-memory B+Tree database mode, specify "+".

    $ttserver +

To start up TT in file B+Tree database mode, specify ".tcb" suffix.

    $ttserver test.tcb

!SLIDE full-page

## 2. TCB operations - part 1

## putdup/putdupback/getlist

Store a record with allowing duplication of keys. "putdup" stores downward and "putdupback" stores upward. 

    $ tcrmgr misc localhost putdup bar one
    $ tcrmgr misc localhost putdup bar two
    $ tcrmgr misc localhost getlist bar
    bar
    one
    bar
    two

    $ tcrmgr misc localhost putdupback bar three
    $ tcrmgr misc localhost getlist bar
    bar
    three
    bar
    one
    bar
    two

## 2. TCB operations - part 2

## range

"range hostname upkey rangenum downkey" operation lets you extract collection of records based on the specified range.

    $ tcrmgr put localhost 101 red
    $ tcrmgr put localhost 101 yellow
    $ tcrmgr put localhost 101 red
    $ tcrmgr put localhost 102 yellow
    $ tcrmgr put localhost 103 green
    $ tcrmgr put localhost 104 blue
    $ tcrmgr put localhost 105 orange

    $ tcrmgr misc localhost range 103 2
    103
    green
    104
    blue

    $ tcrmgr misc localhost range 101 0 103
    101
    red
    102
    yellow

This operation is very useful for pagination purpose.
If you want to do this operation, you can also use TCF(Fixed length database) and it will be faster (and the size is smaller). The advantage of using range for TCH is range search against string.

The following example is to use "username:timestamp" as key and tweet as value.

    $  tcrmgr put localhost estraier:1255503845 'Some nasty words'
    $  tcrmgr put localhost mikio:1255503816    'PDF virus'
    $  tcrmgr put localhost mikio:1255503819    'Open Office font support is not good'
    $  tcrmgr put localhost mikio:1255503901    'Where is everybody?'
    $  tcrmgr put localhost jun:1255503818      'Fix bugs'

Assuming the username is all in UTF, you can use "\xff" (this char does not appear in UTF) as lower range and extract all tweets by specific users.

    $ tcrmgr misc localhost range "mikio", 0, "mikio\xff"
    mikio:1255503816
    PDF virus
    mikio:1255503819
    Open Office font support is not good
    mikio:1255503901
    Where is everybody?

For more information about the use of "range", check http://tokyocabinetwiki.pbworks.com/30_range_search_with_btree_abstract_database

## 3. TCB advanced operations

If you use language bindings, you can use other operations such as forward matching key search and tarversing rows up and down.

For more information about language bindings (eg: Ruby), check http://1978th.net/tokyocabinet/rubydoc