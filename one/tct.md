!SLIDE center
# Try Tokyo #

!SLIDE full-page

## 1. What is TCT.

TCT stands for Tokyo Cabinet Table database. As the name suggests, you can store multiple key value pair within a value.

To start up TT in file table database mode, specify ".tct" suffix.

    $ttserver test.tct

!SLIDE full-page

## 2. TCT operations - part 1

## Creating a record.

    $tcrmgr put -sep "|" localhost 1 \
      "name|Jotaro Kujo|sex|male|hdate|20050321|div|brd,dev"
    $tcrmgr put -sep "|" localhost 81 \
      "name|Josuke Higashikata|sex|male|hdate|20060601|div|dev"
    $tcrmgr put -sep "|" localhost 92 \
      "name|Giorno Giovanna|sex|male|hdate|20070311|div|hr"
    $tcrmgr put -sep "|" localhost 127 \
      "name|Jorin Kujo |sex|female|hdate|20070523|div|brd,hr"
  
    $tcrmgr list -pv -sep '|' localhost
      1	name|Jotaro Kujo|sex|male|hdate|20050321|div|brd,dev
      81	name|Josuke Higashikata|sex|male|hdate|20060601|div|dev
      92	name|Giorno Giovanna|sex|male|hdate|20070311|div|hr
      127	name|Jorin Kujo |sex|female|hdate|20070523|div|brd,hr


A little Trivia: Lots of Mikio's code example data are from ["JoJo's Bizarre Adventure"](http://jjba.wikia.com/wiki/Main_Page)

!SLIDE full-page

## 3. TCT operations - part 2

## search.

Searching the query result is very similar to sql.

* addcond = specify various operators
* columns = specify whether the result shows key  only or key and value
* setorder = equivalent to "order by" of mysql
* setmax = equivalent to "limit" of mysql

Here are some examples.

      $tcrmgr misc -sep '|' localhost search \
        "addcond|name|STRBW|Jo" "columns"

        |1|name|Jotaro Kujo|sex|male|hdate|20050321|div|brd,dev
        |81|name|Josuke Higashikata|sex|male|hdate|20060601|div|dev
        |127|name|Jorin Kujo |sex|female|hdate|20070523|div|brd,hr

      $tcrmgr misc -sep '|' localhost search \
        "addcond|hdate|NUMGE|20060101" \
        "addcond|sex|STREQ|male" "columns"

        |81|name|Josuke Higashikata|sex|male|hdate|20060601|div|dev
        |92|name|Giorno Giovanna|sex|male|hdate|20070311|div|hr

      $tcrmgr misc -sep '|' localhost search \
        "addcond|div|STROR|brd,hr" \
        "setorder|hdate|NUMASC" "setmax|1" "columns"

For more information, check http://1978th.net/tokyocabinet/spex-en.html#tctdbapi  


!SLIDE full-page

## 4. TCT operations - part 3

## Full text search.


!SLIDE full-page

## 5. TCT operations - part 4

## metasearch.

Metasearch provides UNION, INTERSECTION, and DIFF capability.

!SLIDE full-page

## 6. TCT operations - part 5


## Setting Index.


## 7 TCT operations - part 6

## genid

How do you generate unique keys?  You can use genuid to generate unique id.

      $ tcrmgr misc localhost genuid
      1
      $ tcrmgr misc localhost genuid
      2
      $ tcrmgr misc localhost genuid
      3

## 8. TCT bindings - part 1

Did you find tct syntax are a bit too complicating?  Some third party language binding have nice abstractions.

      require 'tokyo_tyrant'
      t = TokyoTyrant::Table.new('127.0.0.1', 1978)

      100.times do |i|
        t[i] = { 'name' => "Pat #{i}", 'sex' => i % 2 > 0 ? 'male' : 'female' }
      end
      # Get records for a query
      t.find{ |q|
        q.condition('sex', :streq, 'male')
        q.limit(5)
      }
      # => [{:sex=>"male", :name=>"Pat 1", :__id=>"1"}, {:sex=>"male", :name=>"Pat 3", :__id=>"3"},   {:sex=>"male", :name=>"Pat 5", :__id=>"5"}, {:sex=>"male", :name=>"Pat 7", :__id=>"7"}, {:sex=>"male", :name=>"Pat 9", :__id=>"9"}]

For more information, check http://github.com/actsasflinn/ruby-tokyotyrant

## 8. TCT bindings - part 2

## proc

Some operations are only available from Language bindings (because "misc" does not support all operations). One useful example is "proc". "proc" is equivalent to "update" of sql and let you update rows which matches the condition in atomic way.

    qry.proc() do |pkey, cols|
        cols["sex"] = cols["sex"] == "male" ? "female" : "male"
        TDBQRY::QPPUT
    end
