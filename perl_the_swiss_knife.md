# <a name="perl-one-liners"></a>Perl one liners

**Table of Contents**

* [Executing Perl code](#executing-perl-code)
* [Simple search and replace](#simple-search-and-replace)
* [Line filtering](#line-filtering)
    * [Regular expressions based filtering](#regular-expressions-based-filtering)
    * [Fixed string matching](#fixed-string-matching)
    * [Line number based filtering](#line-number-based-filtering)
* [Field processing](#field-processing)
    * [Field comparison](#field-comparison)
    * [Specifying different input field separator](#specifying-different-input-field-separator)
    * [Specifying different output field separator](#specifying-different-output-field-separator)
* [Changing record separators](#changing-record-separators)
    * [Input record separator](#input-record-separator)
    * [Output record separator](#output-record-separator)
* [Multiline processing](#multiline-processing)
* [Perl regular expressions](#perl-regular-expressions)
    * [sed vs perl subtle differences](#sed-vs-perl-subtle-differences)
    * [Backslash sequences](#backslash-sequences)
    * [Non-greedy quantifier](#non-greedy-quantifier)
    * [Lookarounds](#lookarounds)
    * [Ignoring specific matches](#ignoring-specific-matches)
    * [Special capture groups](#special-capture-groups)
    * [Modifiers](#modifiers)
    * [Quoting metacharacters](#quoting-metacharacters)
* [Two file processing](#two-file-processing)
    * [Comparing whole lines](#comparing-whole-lines)
    * [Comparing specific fields](#comparing-specific-fields)
    * [Line number matching](#line-number-matching)
* [Creating new fields](#creating-new-fields)
* [Dealing with duplicates](#dealing-with-duplicates)
* [Lines between two REGEXPs](#lines-between-two-regexps)
    * [All unbroken blocks](#all-unbroken-blocks)
    * [Specific blocks](#specific-blocks)
    * [Broken blocks](#broken-blocks)

<br>

```bash
$ perl -le 'print $^V'
v5.22.1

$ man perl
PERL(1)                Perl Programmers Reference Guide                PERL(1)

NAME
       perl - The Perl 5 language interpreter

SYNOPSIS
       perl [ -sTtuUWX ]      [ -hv ] [ -V[:configvar] ]
            [ -cw ] [ -d[t][:debugger] ] [ -D[number/list] ]
            [ -pna ] [ -Fpattern ] [ -l[octal] ] [ -0[octal/hexadecimal] ]
            [ -Idir ] [ -m[-]module ] [ -M[-]'module...' ] [ -f ]
            [ -C [number/list] ]      [ -S ]      [ -x[dir] ]
            [ -i[extension] ]
            [ [-e|-E] 'command' ] [ -- ] [ programfile ] [ argument ]...

       For more information on these options, you can run "perldoc perlrun".
...
```

**Prerequisites and notes**

* familiarity with programming concepts like variables, printing, control structures, arrays, etc
* Perl borrows syntax/features from **C, shell scripting, awk, sed** etc. Prior experience working with them would help a lot
* familiarity with regular expression basics
    * if not, check out **ERE** portion of [GNU sed regular expressions](./gnu_sed.md#regular-expressions)
    * examples for non-greedy, lookarounds, etc will be covered here
* this tutorial is primarily focussed on short programs that are easily usable from command line, similar to using `grep`, `sed`, `awk` etc
    * do NOT use style/syntax presented here when writing full fledged Perl programs which should use **strict, warnings** etc
    * see [learnxinyminutes - perl](https://learnxinyminutes.com/docs/perl/) for quick intro to using Perl for full fledged programs
* links to Perl documentation will be added as necessary
* unless otherwise specified, consider input as ASCII encoded text only

<br>

## <a name="executing-perl-code"></a>Executing Perl code

* One way is to put code in a file and use `perl` command with filename as argument
* Another is to use [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) at beginning of script, make the file executable and directly run it

```bash
$ cat code.pl
print "Hello Perl\n"
$ perl code.pl
Hello Perl

$ # similar to bash
$ cat code.sh
echo 'Hello Bash'
$ bash code.sh
Hello Bash
```

* For short programs, one can use `-e` commandline option to provide code from command line itself
* This entire chapter is about using `perl` this way from commandline

```bash
$ perl -e 'print "Hello Perl\n"'
Hello Perl

$ # similar to
$ bash -c 'echo "Hello Bash"'
Hello Bash

$ # multiple commands can be issued separated by ;
$ # -l will be covered later, here used to append newline to print
$ perl -le '$a=25; $b=12; print $a**$b'
59604644775390625
```

**Further Reading**

* `perl -h` for summary of options
* [perldoc - Command Switches](https://perldoc.perl.org/perlrun.html#Command-Switches)
* [explainshell](https://explainshell.com/explain?cmd=perl+-F+-l+-anpeE+-i+-0+-M) - to quickly get information without having to traverse through the docs
* See [Changing record separators](#changing-record-separators) section for more details on `-l` option

<br>

## <a name="simple-search-and-replace"></a>Simple search and replace

* **substitution** command syntax is very similar to `sed` for search and replace
    * syntax is `variable =~ s/REGEXP/REPLACEMENT/FLAGS` and by default acts on `$_` if variable is not specified
    * see [perldoc - SPECIAL VARIABLES](https://perldoc.perl.org/perlvar.html#SPECIAL-VARIABLES) for explanation on `$_` and other such special variables
    * more detailed examples will be covered in later sections
* Just like other text processing commands, `perl` will automatically loop over input line by line when `-n` or `-p` option is used
    * like `sed`, the `-n` option won't print the record
    * `-p` will print the record, including any changes made
    * newline character being default record separator
    * `$_` will contain the input record content, including the record separator (unlike `sed` and `awk`)
* and similar to other commands, `perl` will work with both stdin and file input

```bash
$ # change only first ',' to ' : '
$ # same as: sed 's/,/ : /'
$ seq 10 | paste -sd, | perl -pe 's/,/ : /'
1 : 2,3,4,5,6,7,8,9,10

$ # change all ',' to ' : ' by using 'g' modifier
$ # same as: sed 's/,/ : /g'
$ seq 10 | paste -sd, | perl -pe 's/,/ : /g'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10

$ cat greeting.txt
Hi there
Have a nice day
$ # same as: sed 's/nice day/safe journey/' greeting.txt
$ perl -pe 's/nice day/safe journey/' greeting.txt
Hi there
Have a safe journey
```

* inplace editing
* similar to [GNU sed - using * with inplace option](./gnu_sed.md#prefix-backup-name), one can also use `*` to either prefix the backup name or place the backup files in another existing directory

```bash
$ # same as: sed -i.bkp 's/Hi/Hello/' greeting.txt
$ perl -i.bkp -pe 's/Hi/Hello/' greeting.txt
$ # original file gets preserved in 'greeting.txt.bkp'
$ cat greeting.txt
Hello there
Have a nice day

$ # use this with caution, changes made cannot be undone
$ perl -i -pe 's/nice day/safe journey/' greeting.txt
$ cat greeting.txt
Hello there
Have a safe journey
```

* Multiple input files are treated individually and changes are written back to respective files

```bash
$ cat f1
I ate 3 apples
$ cat f2
I bought two bananas and 3 mangoes

$ # -i can be used with or without backup
$ perl -i -pe 's/3/three/' f1 f2
$ cat f1
I ate three apples
$ cat f2
I bought two bananas and three mangoes
```

<br>

## <a name="line-filtering"></a>Line filtering

<br>

#### <a name="regular-expressions-based-filtering"></a>Regular expressions based filtering

* syntax is `variable =~ m/REGEXP/FLAGS` to check for a match
    * `variable !~ m/REGEXP/FLAGS` for negated match
    * by default acts on `$_` if variable is not specified
* as we need to print only selective lines, use `-n` option
    * by default, contents of `$_` will be printed if no argument is passed to `print`

```bash
$ cat poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # same as: grep '^[RS]' or sed -n '/^[RS]/p' or awk '/^[RS]/'
$ # /^[RS]/ is shortcut for $_ =~ m/^[RS]/
$ perl -ne 'print if /^[RS]/' poem.txt
Roses are red,
Sugar is sweet,

$ # same as: grep -i 'and' poem.txt
$ perl -ne 'print if /and/i' poem.txt
And so are you.

$ # same as: grep -v 'are' poem.txt
$ # !/are/ is shortcut for $_ !~ m/are/
$ perl -ne 'print if !/are/' poem.txt
Sugar is sweet,

$ # same as: awk '/are/ && !/so/' poem.txt
$ perl -ne 'print if /are/ && !/so/' poem.txt
Roses are red,
Violets are blue,
```

* using different delimiter
* quoting from [perldoc - Regexp Quote-Like Operators](https://perldoc.perl.org/perlop.html#Regexp-Quote-Like-Operators)

> With the m you can use any pair of non-alphanumeric, non-whitespace characters as delimiters

```bash
$ cat paths.txt
/foo/a/report.log
/foo/y/power.log
/foo/abc/errors.log

$ perl -ne 'print if /\/foo\/a\//' paths.txt
/foo/a/report.log

$ perl -ne 'print if m#/foo/a/#' paths.txt
/foo/a/report.log

$ perl -ne 'print if !m#/foo/a/#' paths.txt
/foo/y/power.log
/foo/abc/errors.log
```

<br>

#### <a name="fixed-string-matching"></a>Fixed string matching

* similar to `grep -F` and `awk index`
* See also
    * [perldoc - index function](https://perldoc.perl.org/functions/index.html)
    * [perldoc - Quote and Quote-like Operators](https://perldoc.perl.org/5.8.8/perlop.html#Quote-and-Quote-like-Operators)
    * [Quoting metacharacters](#quoting-metacharacters) section

```bash
$ # same as: grep -F 'a[5]' or awk 'index($0, "a[5]")'
$ # index returns matching position(starts at 0) and -1 if not found
$ echo 'int a[5]' | perl -ne 'print if index($_, "a[5]") != -1'
int a[5]

$ # however, string within double quotes gets interpolated, for ex
$ a='123'; echo "$a"
123
$ perl -e '$a=123; print "$a\n"'
123

$ # so, for commandline usage, better to pass string as environment variable
$ # they are accessible via the %ENV hash variable
$ perl -le 'print $ENV{PWD}'
/home/learnbyexample
$ perl -le 'print $ENV{SHELL}'
/bin/bash

$ echo 'a#$%d' | perl -ne 'print if index($_, "#$%") != -1'
$ echo 'a#$%d' | s='#$%' perl -ne 'print if index($_, $ENV{s}) != -1'
a#$%d
```

* return value is useful to match at specific position
* for ex: at start/end of line

```bash
$ cat eqns.txt
a=b,a-b=c,c*d
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # start of line
$ # same as: s='a+b' awk 'index($0, ENVIRON["s"])==1' eqns.txt
$ s='a+b' perl -ne 'print if index($_, $ENV{s})==0' eqns.txt
a+b,pi=3.14,5e12

$ # end of line
$ # length function returns number of characters, by default acts on $_
$ s='a+b' perl -ne '$pos = length() - length($ENV{s}) - 1;
                    print if index($_, $ENV{s}) == $pos' eqns.txt
i*(t+9-g)/8,4-a+b
```

<br>

#### <a name="line-number-based-filtering"></a>Line number based filtering

* special variable `$.` contains total records read so far, similar to `NR` in `awk`
    * But no equivalent of awk's `FNR`, [see this stackoverflow Q&A for workaround](https://stackoverflow.com/questions/12384692/line-number-of-a-file-in-perl)
* See also [perldoc - eof](https://perldoc.perl.org/perlfunc.html#eof)

```bash
$ # same as: head -n2 poem.txt | tail -n1
$ # or sed -n '2p' or awk 'NR==2'
$ perl -ne 'print if $.==2' poem.txt
Violets are blue,

$ # print 2nd and 4th line
$ # same as: sed -n '2p; 4p' or awk 'NR==2 || NR==4'
$ perl -ne 'print if $.==2 || $.==4' poem.txt
Violets are blue,
And so are you.

$ # same as: tail -n1 poem.txt
$ # or sed -n '$p' or awk 'END{print}'
$ perl -ne 'print if eof' poem.txt
And so are you.
```

* for large input, use `exit` to avoid unnecessary record processing

```bash
$ seq 14323 14563435 | perl -ne 'if($.==234){print; exit}'
14556

$ # sample time comparison
$ time seq 14323 14563435 | perl -ne 'if($.==234){print; exit}' > /dev/null
real    0m0.005s
$ time seq 14323 14563435 | perl -ne 'print if $.==234' > /dev/null
real    0m2.439s

$ # mimicking head command, same as: head -n3 or sed '3q'
$ seq 14 25 | perl -pe 'exit if $.>3'
14
15
16

$ # same as: sed '3Q'
$ seq 14 25 | perl -pe 'exit if $.==3'
14
15
```

* selecting range of lines
* `..` is [perldoc - range operator](https://perldoc.perl.org/perlop.html#Range-Operators)

```bash
$ # same as: sed -n '3,5p' or awk 'NR>=3 && NR<=5'
$ # in this context, the range is compared against $.
$ seq 14 25 | perl -ne 'print if 3..5'
16
17
18

$ # selecting from particular line number to end of input
$ # same as: sed -n '10,$p' or awk 'NR>=10'
$ seq 14 25 | perl -ne 'print if $.>=10'
23
24
25
```

<br>

## <a name="field-processing"></a>Field processing

* `-a` option will auto-split each input record based on one or more continuous white-space, similar to default behavior in `awk`
* Special variable array `@F` will contain all the elements, index starting from `0`
* See also [perldoc - split function](https://perldoc.perl.org/functions/split.html)

```bash
$ cat fruits.txt
fruit   qty
apple   42
banana  31
fig     90
guava   6

$ # print only first field, index starting from 0
$ # same as: awk '{print $1}' fruits.txt 
$ perl -lane 'print $F[0]' fruits.txt
fruit
apple
banana
fig
guava

$ # print only second field
$ # same as: awk '{print $2}' fruits.txt 
$ perl -lane 'print $F[1]' fruits.txt
qty
42
31
90
6
```

* by default, leading and trailing whitespaces won't be considered when splitting the input record
    * mimicking `awk`'s default behavior

```bash
$ printf ' a    ate b\tc   \n'
 a    ate b	c   
$ printf ' a    ate b\tc   \n' | perl -lane 'print $F[0]'
a
$ printf ' a    ate b\tc   \n' | perl -lane 'print $F[-1]'
c

$ # number of fields, $#F gives index of last element - so add 1
$ echo '1 a 7' | perl -lane 'print $#F+1'
3
$ printf ' a    ate b\tc   \n' | perl -lane 'print $#F+1'
4
```

<br>

#### <a name="field-comparison"></a>Field comparison

* for numeric context, Perl automatically tries to convert the string to number, ignoring white-space
* for string comparison, use `eq` for `==`, `ne` for `!=` and so on

```bash
$ # if first field exactly matches the string 'apple'
$ # same as: awk '$1=="apple"{print $2}' fruits.txt
$ perl -lane 'print $F[1] if $F[0] eq "apple"' fruits.txt
42

$ # print first field if second field > 35 (excluding header)
$ # same as: awk 'NR>1 && $2>35{print $1}' fruits.txt 
$ perl -lane 'print $F[0] if $F[1]>35 && $.>1' fruits.txt
apple
fig

$ # print header and lines with qty < 35
$ # same as: awk 'NR==1 || $2<35' fruits.txt
$ perl -ane 'print if $F[1]<35 || $.==1' fruits.txt
fruit   qty
banana  31
guava   6

$ # if first field does NOT contain 'a'
$ # same as: awk '$1 !~ /a/' fruits.txt 
$ perl -ane 'print if $F[0] !~ /a/' fruits.txt
fruit   qty
fig     90
```

<br>

#### <a name="specifying-different-input-field-separator"></a>Specifying different input field separator

* by using `-F` command line option

```bash
$ # second field where input field separator is :
$ # same as: awk -F: '{print $2}'
$ echo 'foo:123:bar:789' | perl -F: -lane 'print $F[1]'
123

$ # last field, same as: awk -F: '{print $NF}'
$ echo 'foo:123:bar:789' | perl -F: -lane 'print $F[-1]'
789
$ # second last field, same as: awk -F: '{print $(NF-1)}'
$ echo 'foo:123:bar:789' | perl -F: -lane 'print $F[-2]'
bar

$ # second and last field
$ # other ways to print more than 1 element will be covered later
$ echo 'foo:123:bar:789' | perl -F: -lane 'print "$F[1] $F[-1]"'
123 789

$ # use quotes to avoid clashes with shell special characters
$ echo 'one;two;three;four' | perl -F';' -lane 'print $F[2]'
three
```

* Regular expressions based input field separator

```bash
$ # same as: awk -F'[0-9]+' '{print $2}'
$ echo 'Sample123string54with908numbers' | perl -F'\d+' -lane 'print $F[1]'
string

$ # first field will be empty as there is nothing before '{'
$ # same as: awk -F'[{}= ]+' '{print $1}'
$ # \x20 is space character, can't use literal space within [] when using -F
$ echo '{foo}   bar=baz' | perl -F'[{}=\x20]+' -lane 'print $F[0]'

$ echo '{foo}   bar=baz' | perl -F'[{}=\x20]+' -lane 'print $F[1]'
foo
$ echo '{foo}   bar=baz' | perl -F'[{}=\x20]+' -lane 'print $F[2]'
bar
```

* empty argument to `-F` will split the input record character wise

```bash
$ # same as: gawk -v FS= '{print $1}'
$ echo 'apple' | perl -F -lane 'print $F[0]'
a
$ echo 'apple' | perl -F -lane 'print $F[1]'
p
$ echo 'apple' | perl -F -lane 'print $F[-1]'
e

$ # use -C option when dealing with unicode characters
$ # S will turn on UTF-8 for stdin/stdout/stderr streams
$ printf 'hi👍 how are you?' | perl -CS -F -lane 'print $F[2]'
👍
```

<br>

#### <a name="specifying-different-output-field-separator"></a>Specifying different output field separator

* Method 1: use `$,` to change separator between `print` arguments
    * could be remembered easily by noting that `,` is used to separate `print` arguments

```bash
$ # by default, the various arguments are concatenated
$ echo 'foo:123:bar:789' | perl -F: -lane 'print $F[1], $F[-1]'
123789

$ # change $, if different separator is needed
$ echo 'foo:123:bar:789' | perl -F: -lane '$,=" "; print $F[1], $F[-1]'
123 789
$ echo 'foo:123:bar:789' | perl -F: -lane '$,="-"; print $F[1], $F[-1]'
123-789

$ # argument can be array too
$ echo 'foo:123:bar:789' | perl -F: -lane '$,="-"; print @F[1,-1]'
123-789
$ echo 'foo:123:bar:789' | perl -F: -lane '$,=" - "; print @F'
foo - 123 - bar - 789
```

* Method 2: use `join`

```bash
$ echo 'foo:123:bar:789' | perl -F: -lane 'print join "-", $F[1], $F[-1]'
123-789

$ echo 'foo:123:bar:789' | perl -F: -lane 'print join "-", @F[1,-1]'
123-789

$ echo 'foo:123:bar:789' | perl -F: -lane 'print join " - ", @F'
foo - 123 - bar - 789
```

* Method 3: use `$"` to change separator when array is interpolated, default is space character
    * could be remembered easily by noting that interpolation happens within double quotes

```bash
$ # default is space
$ echo 'foo:123:bar:789' | perl -F: -lane 'print "@F[1,-1]"'
123 789

$ echo 'foo:123:bar:789' | perl -F: -lane '$"="-"; print "@F[1,-1]"'
123-789

$ echo 'foo:123:bar:789' | perl -F: -lane '$"=","; print "@F"'
foo,123,bar,789
```

* use `BEGIN` if same separator is to be used for all lines
    * statements inside `BEGIN` are executed before processing any input text

```bash
$ # can also use: perl -lane 'BEGIN{$"=","} print "@F"' fruits.txt
$ perl -lane 'BEGIN{$,=","} print @F' fruits.txt
fruit,qty
apple,42
banana,31
fig,90
guava,6
```

## <a name="changing-record-separators"></a>Changing record separators

* Before seeing examples for changing record separators, let's cover a detail about contents of input record and use of `-l` option
* See also [perldoc - chomp](https://perldoc.perl.org/functions/chomp.html)

```bash
$ # input record includes the record separator as well
$ # can also use: perl -pe 's/$/ 123/'
$ echo 'foo' | perl -pe 's/\n/ 123\n/'
foo 123

$ # this example shows better use case
$ # similar to paste -sd but with ability to use multi-character delimiter
$ seq 5 | perl -pe 's/\n/ : / if !eof'
1 : 2 : 3 : 4 : 5

$ # -l option will chomp off the record separator (among other things)
$ echo 'foo' | perl -l -pe 's/\n/ 123\n/'
foo

$ # -l also sets output record separator which gets added to print statements
$ # ORS gets input record separator value if no argument is passed to -l
$ # hence the newline automatically getting added for print in this example
$ perl -lane 'print $F[0] if $F[1]<35 && $.>1' fruits.txt
banana
guava
```

<br>

#### <a name="input-record-separator"></a>Input record separator

* by default, newline character is used as input record separator
* use `$/` to specify a different input record separator
    * unlike `awk`, only string can be used, no regular expressions
* for single character separator, can also use `-0` command line option which accepts octal/hexadecimal value as argument
* if `-l` option is also used
    * input record separator will be chomped from input record
    * in addition, if argument is not passed to `-l`, output record separator will get whatever is current value of input record separator
    * so, order of `-l`, `-0` and/or `$/` usage becomes important

```bash
$ s='this is a sample string'

$ # space as input record separator, printing all records
$ # same as: awk -v RS=' ' '{print NR, $0}'
$ # ORS is newline as -l is used before $/ gets changed
$ printf "$s" | perl -lne 'BEGIN{$/=" "} print "$. $_"'
1 this
2 is
3 a
4 sample
5 string

$ # print all records containing 'a'
$ # same as: awk -v RS=' ' '/a/'
$ printf "$s" | perl -l -0040 -ne 'print if /a/'
a
sample

$ # if the order is changed, ORS will be space, not newline
$ printf "$s" | perl -0040 -l -ne 'print if /a/'
a sample 
```

* `-0` option used without argument will use the ASCII NUL character as input record separator 

```bash
$ printf 'foo\0bar\0' | cat -A
foo^@bar^@$ 
$ printf 'foo\0bar\0' | perl -l -0 -ne 'print'
foo
bar

$ # could be golfed to: perl -l -0pe ''
$ # but dont use `-l0` as `0` will be treated as argument to `-l`
```

* values `-0400` to `-0777` will cause entire file to be slurped
    * idiomatically, `-0777` is used

```bash
$ # s modifier allows . to match newline as well
$ perl -0777 -pe 's/red.*are //s' poem.txt
Roses are you.

$ # replace first newline with '. '
$ perl -0777 -pe 's/\n/. /' greeting.txt
Hello there. Have a safe journey
```

* for paragraph mode (two more more consecutive newline characters), use `-00` or assign empty string to `$/`

Consider the below sample file

```bash
$ cat sample.txt
Hello World

Good day
How are you

Just do-it
Believe it

Today is sunny
Not a bit funny
No doubt you like it too

Much ado about nothing
He he he
```

* again, input record will have the separator too and using `-l` will chomp it
* however, if more than two consecutive newline characters separate the paragraphs, only two newlines will be preserved and the rest discarded
    * use `$/="\n\n"` to avoid this behavior

```bash
$ # print all paragraphs containing 'it'
$ # same as: awk -v RS= -v ORS='\n\n' '/it/' sample.txt
$ perl -00 -ne 'print if /it/' sample.txt
Just do-it
Believe it

Today is sunny
Not a bit funny
No doubt you like it too

$ # based on number of lines in each paragraph
$ perl -F'\n' -00 -ane 'print if $#F==0' sample.txt
Hello World

$ # unlike awk -F'\n' -v RS= -v ORS='\n\n' 'NF==2 && /do/' sample.txt
$ # there wont be empty line at end because input file didn't have it
$ perl -F'\n' -00 -ane 'print if $#F==1 && /do/' sample.txt
Just do-it
Believe it

Much ado about nothing
He he he
```

* Re-structuring paragraphs

```bash
$ # same as: awk 'BEGIN{FS="\n"; OFS=". "; RS=""; ORS="\n\n"} {$1=$1} 1'
$ perl -F'\n' -00 -ane 'print join ". ", @F; print "\n\n"' sample.txt
Hello World

Good day. How are you

Just do-it. Believe it

Today is sunny. Not a bit funny. No doubt you like it too

Much ado about nothing. He he he

```

* multi-character separator

```bash
$ cat report.log 
blah blah
Error: something went wrong
more blah
whatever
Error: something surely went wrong
some text
some more text
blah blah blah

$ # number of records, same as: awk -v RS='Error:' 'END{print NR}'
$ perl -lne 'BEGIN{$/="Error:"} print $. if eof' report.log
3
$ # print first record
$ perl -lne 'BEGIN{$/="Error:"} print if $.==1' report.log
blah blah

$ # same as: awk -F'\n' -v RS='Error:' '/surely/{print RS $0}' report.log
$ perl -F'\n' -lane 'BEGIN{$/="Error:"} print "$/$_" if /surely/' report.log
Error: something surely went wrong
some text
some more text
blah blah blah

```

* Joining lines based on specific end of line condition

```bash
$ cat msg.txt
Hello there.
It will rain to-
day. Have a safe
and pleasant jou-
rney.

$ # same as: awk -v RS='-\n' -v ORS= '1' msg.txt
$ # can also use: perl -pe 's/-\n//' msg.txt
$ perl -pe 'BEGIN{$/="-\n"} chomp' msg.txt
Hello there.
It will rain today. Have a safe
and pleasant journey.
```

<br>

#### <a name="output-record-separator"></a>Output record separator

* one way is to use `$\` to specify a different output record separator
    * by default it doesn't have a value

```bash
$ # note that despite $\ not having a value, output has newlines
$ # because the input record still has the input record separator
$ seq 3 | perl -ne 'print'
1
2
3
$ # same as: awk -v ORS='\n\n' '{print $0}'
$ seq 3 | perl -ne 'BEGIN{$\="\n"} print'
1

2

3

```

* dynamically changing output record separator

```bash
$ # same as: awk '{ORS = NR%2 ? " " : "\n"} 1'
$ # note the use of -l to chomp the input record separator
$ seq 6 | perl -lpe '$\ = $.%2 ? " " : "\n"'
1 2
3 4
5 6
$ # -l also sets the output record separator
$ # but gets overridden by $\
$ seq 6 | perl -lpe '$\ = $.%3 ? "-" : "\n"'
1-2-3
4-5-6
```

* passing argument to `-l` to set output record separator

```bash
$ seq 8 | perl -ne 'print if /[24]/'
2
4

$ # null separator, note how -l also chomps input record separator
$ seq 8 | perl -l0 -ne 'print if /[24]/' | cat -A
2^@4^@

$ # comma separator
$ seq 8 | perl -l054 -ne 'print if /[24]/'
2,4, 

$ # to add a final newline to output, use END and printf
$ seq 8 | perl -l054 -ne 'print if /[24]/; END{printf "\n"}'
2,4,
```

<br>

## <a name="multiline-processing"></a>Multiline processing

* Processing consecutive lines

```bash
$ cat poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # match two consecutive lines
$ # same as: awk 'p~/are/ && /is/{print p ORS $0} {p=$0}' poem.txt
$ perl -ne 'print "$p$_" if /is/ && $p=~/are/; $p=$_' poem.txt
Violets are blue,
Sugar is sweet,
$ # if only the second line is needed, same as: awk 'p~/are/ && /is/; {p=$0}'
$ perl -ne 'print if /is/ && $p=~/are/; $p=$_' poem.txt
Sugar is sweet,

$ # print if line matches a condition as well as condition for next 2 lines
$ # same as: awk 'p2~/red/ && p1~/blue/ && /is/{print p2} {p2=p1; p1=$0}'
$ perl -ne 'print $p2 if /is/ && $p1=~/blue/ && $p2=~/red/;
            $p2=$p1; $p1=$_' poem.txt
Roses are red,
```

Consider this sample input file

```bash
$ cat range.txt 
foo
BEGIN
1234
6789
END
bar
BEGIN
a
b
c
END
baz
```

* extracting lines around matching line
* how `$n && $n--` works:
    * need to note that right hand side of `&&` is processed only if left hand side is `true`
    * so for example, if initially `$n=2`, then we get
        * `2 && 2; $n=1` - evaluates to `true`
        * `1 && 1; $n=0` - evaluates to `true`
        * `0 && ` - evaluates to `false` ... no decrementing `$n` and hence will be `false` until `$n` is re-assigned non-zero value

```bash
$ # similar to: grep --no-group-separator -A1 'BEGIN' range.txt 
$ # same as: awk '/BEGIN/{n=2} n && n--' range.txt
$ perl -ne '$n=2 if /BEGIN/; print if $n && $n--' range.txt
BEGIN
1234
BEGIN
a

$ # print only line after matching line, same as: awk 'n && n--; /BEGIN/{n=1}'
$ perl -ne 'print if $n && $n--; $n=1 if /BEGIN/' range.txt
1234
a

$ # generic case: print nth line after match, awk 'n && !--n; /BEGIN/{n=3}'
$ perl -ne 'print if $n && !--$n; $n=3 if /BEGIN/' range.txt
END
c

$ # print second line prior to matched line
$ # same as: awk '/END/{print p2} {p2=p1; p1=$0}' range.txt
$ perl -ne 'print $p2 if /END/; $p2=$p1; $p1=$_' range.txt
1234
b

$ # use reversing trick for generic case of nth line before match
$ # same as: tac range.txt | awk 'n && !--n; /END/{n=3}' | tac
$ tac range.txt | perl -ne 'print if $n && !--$n; $n=3 if /END/' | tac
BEGIN
a
```

<br>

## <a name="perl-regular-expressions"></a>Perl regular expressions

* examples to showcase some of the features not present in ERE and modifiers not available in `sed`'s substitute command
* many features of Perl regular expressions will NOT be covered, but external links will be provided wherever relevant
    * See [perldoc - perlre](https://perldoc.perl.org/perlre.html) for complete reference
* examples/descriptions based only on ASCII encoding

<br>

#### <a name="sed-vs-perl-subtle-differences"></a>sed vs perl subtle differences

* input record separator being part of input record

```bash
$ echo 'foo:123:bar:789' | sed -E 's/[^:]+$/xyz/'
foo:123:bar:xyz
$ # newline character gets replaced too as shown by shell prompt
$ echo 'foo:123:bar:789' | perl -pe 's/[^:]+$/xyz/'
foo:123:bar:xyz$ 
$ # simple workaround is to use -l option
$ echo 'foo:123:bar:789' | perl -lpe 's/[^:]+$/xyz/'
foo:123:bar:xyz

$ # of course it has uses too
$ seq 10 | paste -sd, | sed 's/,/ : /g'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10
$ seq 10 | perl -pe 's/\n/ : / if !eof'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10
```

* how much does `*` match?

```bash
$ # sed will choose biggest match
$ echo ',baz,,xyz,,,' | sed 's/[^,]*/A/g'
A,A,A,A,A,A,A
$ echo 'foo,baz,,xyz,,,123' | sed 's/[^,]*/A/g'
A,A,A,A,A,A,A

$ # but perl will match both empty and non-empty strings
$ echo ',baz,,xyz,,,' | perl -lpe 's/[^,]*/A/g'
A,AA,A,AA,A,A,A
$ echo 'foo,baz,,xyz,,,123' | perl -lpe 's/[^,]*/A/g'
AA,AA,A,AA,A,A,AA
```

* backslash sequences inside character classes

```bash
$ # \w would simply match w
$ echo 'w=y-x+9*3' | sed 's/[\w=]//g'
y-x+9*3

$ # \w would match any word character
$ echo 'w=y-x+9*3' | perl -pe 's/[\w=]//g'
-+*
```

* replacing specific occurrence
* See [stackoverflow - substitute the nth occurrence of a match in a Perl regex](https://stackoverflow.com/questions/2555662/how-can-i-substitute-the-nth-occurrence-of-a-match-in-a-perl-regex) for workarounds

```bash
$ echo 'foo:123:bar:baz' | sed 's/:/-/2'
foo:123-bar:baz

$ echo 'foo:123:bar:baz' | perl -pe 's/:/-/2'
Unknown regexp modifier "/2" at -e line 1, at end of line
Execution of -e aborted due to compilation errors.
```

* variable interpolation when `$` or `@` is used
* See also [perldoc - Quote and Quote-like Operators](https://perldoc.perl.org/5.8.8/perlop.html#Quote-and-Quote-like-Operators)

```bash
$ seq 2 | sed 's/$a/xyz/'
1
2

$ # uninitialized variable, same applies for: perl -pe 's/@a/xyz/'
$ seq 2 | perl -pe 's/$a/xyz/'
xyz1
xyz2
$ # initialized variable
$ seq 2 | perl -pe '$a=2; s/$a/xyz/'
1
xyz

$ # using single quotes as delimiter won't interpolate
$ # not usable for one-liners given shell's own single/double quotes behavior
$ cat sub_sq.pl
s'$a'xyz'
$ seq 2 | perl -p sub_sq.pl
1
2
```

* back reference
* See also [perldoc - Warning on \1 Instead of $1](https://perldoc.perl.org/perlre.html#Warning-on-%5c1-Instead-of-%241)

```bash
$ # use $& to refer entire matched string in replacement section
$ echo 'hello world' | sed 's/.*/"&"/'
"hello world"
$ echo 'hello world' | perl -pe 's/.*/"&"/'
"&"
$ echo 'hello world' | perl -pe 's/.*/"$&"/'
"hello world"

$ # use \1, \2, etc or \g1, \g2 etc for back referencing in search section
$ # use $1, $2, etc in replacement section
$ echo 'a a a walking for for a cause' | perl -pe 's/\b(\w+)( \1)+\b/$1/g'
a walking for a cause
```

<br>

#### <a name="backslash-sequences"></a>Backslash sequences

* `\d` for `[0-9]`
* `\s` for `[ \t\r\n\f]`
* `\h` for `[ \t]`
* `\n` for newline character
* `\D`, `\S`, `\H`, `\N` respectively for their opposites
* See [perldoc - perlrecharclass](https://perldoc.perl.org/perlrecharclass.html#Backslash-sequences) for full list and details

```bash
$ # same as: sed -E 's/[0-9]+/xxx/g'
$ echo 'like 42 and 37' | perl -pe 's/\d+/xxx/g'
like xxx and xxx

$ # same as: sed -E 's/[^0-9]+/xxx/g'
$ # note again the use of -l because of newline in input record
$ echo 'like 42 and 37' | perl -lpe 's/\D+/xxx/g'
xxx42xxx37

$ # no need -l here as \h won't match newline
$ echo 'a b c  ' | perl -pe 's/\h*$//'
a b c
```

<br>

#### <a name="non-greedy-quantifier"></a>Non-greedy quantifier

* adding a `?` to `?` or `*` or `+` or `{}` quantifiers will change matching from greedy to non-greedy. In other words, to match as minimally as possible
    * also known as lazy quantifier
* See also [regular-expressions.info - Possessive Quantifiers](http://www.regular-expressions.info/possessive.html)

```bash
$ # greedy matching
$ echo 'foo and bar and baz land good' | perl -pe 's/foo.*and//'
 good
$ # non-greedy matching
$ echo 'foo and bar and baz land good' | perl -pe 's/foo.*?and//'
 bar and baz land good

$ echo '12342789' | perl -pe 's/\d{2,5}//'
789
$ echo '12342789' | perl -pe 's/\d{2,5}?//'
342789

$ # for single character, non-greedy is not always needed
$ echo '123:42:789:good:5:bad' | perl -pe 's/:.*?:/:/'
123:789:good:5:bad
$ echo '123:42:789:good:5:bad' | perl -pe 's/:[^:]*:/:/'
123:789:good:5:bad

$ # just like greedy, overall matching is considered, as minimal as possible
$ echo '123:42:789:good:5:bad' | perl -pe 's/:.*?:[a-z]/:/'
123:ood:5:bad
$ echo '123:42:789:good:5:bad' | perl -pe 's/:.*:[a-z]/:/'
123:ad
```

<br>

#### <a name="lookarounds"></a>Lookarounds

* Ability to add if conditions to match before/after required pattern
* There are four types
    * positive lookahead `(?=`
    * negative lookahead `(?!`
    * positive lookbehind `(?<=`
    * negative lookbehind `(?<!`
* One way to remember is that **behind** uses `<` and **negative** uses `!` instead of `=`

The string matched by lookarounds are like word boundaries and anchors, do not constitute as part of matched string. They are termed as **zero-width patterns**

* positive lookbehind `(?<=`

```bash
$ s='foo=5, bar=3; x=83, y=120'

$ # extract all digit sequences
$ echo "$s" | perl -lne 'print join " ", /\d+/g'
5 3 83 120

$ # extract digits only if preceded by two lowercase alphabets and =
$ # note how the characters matched by lookbehind isn't part of output
$ echo "$s" | perl -lne 'print join " ", /(?<=[a-z]{2}=)\d+/g'
5 3

$ # this can be done without lookbehind too
$ # taking advantage of behavior of //g when () is used
$ echo "$s" | perl -lne 'print join " ", /[a-z]{2}=(\d+)/g'
5 3

$ # change all digits preceded by single lowercase alphabet and =
$ echo "$s" | perl -pe 's/(?<=\b[a-z]=)\d+/42/g'
foo=5, bar=3; x=42, y=42
$ # alternate, without lookbehind
$ echo "$s" | perl -pe 's/(\b[a-z]=)\d+/${1}42/g'
foo=5, bar=3; x=42, y=42
```

* positive lookahead `(?=`

```bash
$ s='foo=5, bar=3; x=83, y=120'

$ # extract digits that end with ,
$ # can also use: perl -lne 'print join ":", /(\d+),/g'
$ echo "$s" | perl -lne 'print join ":", /\d+(?=,)/g'
5:83

$ # change all digits ending with ,
$ # can also use: perl -pe 's/\d+,/42,/g'
$ echo "$s" | perl -pe 's/\d+(?=,)/42/g'
foo=42, bar=3; x=42, y=120

$ # both lookbehind and lookahead
$ echo 'foo,,baz,,,xyz' | perl -pe 's/,,/,NA,/g'
foo,NA,baz,NA,,xyz
$ echo 'foo,,baz,,,xyz' | perl -pe 's/(?<=,)(?=,)/NA/g'
foo,NA,baz,NA,NA,xyz
```

* negative lookbehind `(?<!` and negative lookahead `(?!`

```bash
$ # change foo if not preceded by _
$ # note how 'foo' at start of line is matched as well
$ echo 'foo _foo 1foo' | perl -pe 's/(?<!_)foo/baz/g'
baz _foo 1baz

$ # join each line in paragraph by replacing newline character
$ # expect the one at end of paragraph
$ perl -00 -pe 's/\n(?!$)/. /g' sample.txt
Hello World

Good day. How are you

Just do-it. Believe it

Today is sunny. Not a bit funny. No doubt you like it too

Much ado about nothing. He he he
```

* variable lookbehind with `\K`
* useful when positive lookbehind is not a constant length of characters to look up
    * for ex: quantifiers that can match different number of characters

```bash
$ # lookbehind is checking start of line (0 characters) and comma(1 character)
$ echo ',baz,,,xyz,,' | perl -pe 's/(?<=^|,)(?=,|$)/NA/g'
Variable length lookbehind not implemented in regex m/(?<=^|,)(?=,|$)/ at -e line 1.

$ # \K helps in such cases
$ echo ',baz,,,xyz,,' | perl -pe 's/(^|,)\K(?=,|$)/NA/g'
NA,baz,NA,NA,xyz,NA,NA
```

**Further Reading**

* [stackoverflow - reverse four letter words](https://stackoverflow.com/questions/46870285/reverse-four-length-of-letters-with-sed-in-unix)
* [stackoverflow - lookarounds and possessive quantifier](https://stackoverflow.com/questions/42437747/pcre-negative-lookahead-gives-unexpected-match)

<br>

#### <a name="ignoring-specific-matches"></a>Ignoring specific matches

* A useful construct is `(*SKIP)(*F)` which allows to discard matches not needed
    * regular expression which should be discarded is written first, `(*SKIP)(*F)` is appended and then required regular expression is added after `|`

```bash
$ s='Car Bat cod12 Map foo_bar'
$ # all words except those starting with 'c' or 'C'
$ echo "$s" | perl -lne 'print join "\n", /\bc\w+(*SKIP)(*F)|\w+/gi'
Bat
Map
foo_bar

$ s='I like "mango" and "guava"'
$ # all words except those surrounded by double quotes
$ echo "$s" | perl -lne 'print join "\n", /"[^"]+"(*SKIP)(*F)|\w+/g'
I
like
and
$ # change words except those surrounded by double quotes
$ echo "$s" | perl -pe 's/"[^"]+"(*SKIP)(*F)|\w+/\U$&/g'
I LIKE "mango" AND "guava"
```

**Further Reading**

* [perldoc - Special Backtracking Control Verbs](https://perldoc.perl.org/perlre.html#Special-Backtracking-Control-Verbs)
* [rexegg - Excluding Unwanted Matches](http://www.rexegg.com/backtracking-control-verbs.html#skipfail)

<br>

#### <a name="special-capture-groups"></a>Special capture groups

* `\1`, `\2` etc only matches exact string
* `(?1)`, `(?2)` etc re-uses the regular expression itself

```bash
$ s='baz 2008-03-24 and 2012-08-12 foo 2016-03-25'
$ # (?1) refers to first capture group (\d{4}-\d{2}-\d{2})
$ echo "$s" | perl -pe 's/(\d{4}-\d{2}-\d{2}) and (?1)//'
baz  foo 2016-03-25

$ # using \1 won't work as the two dates are different
$ echo "$s" | perl -pe 's/(\d{4}-\d{2}-\d{2}) and \1//'
baz 2008-03-24 and 2012-08-12 foo 2016-03-25
```

* use `(?:` to group regular expressions without capturing it, so this won't be counted for backreference
* See also [stackoverflow - what is non-capturing group](https://stackoverflow.com/questions/3512471/what-is-a-non-capturing-group-what-does-a-question-mark-followed-by-a-colon)

```bash
$ s='Car Bat cod12 Map foo_bar'
$ # check what happens if ?: is not used
$ echo "$s" | perl -lne 'print join "\n", /(?:Bat|Map)(*SKIP)(*F)|\w+/gi'
Car
cod12
foo_bar

$ # using ?: helps to focus only on required capture groups
$ echo 'cod1 foo_bar' | perl -pe 's/(?:co|fo)\K(\w)(\w)/$2$1/g'
co1d fo_obar
$ # without ?: you'd need to remember all the other groups as well
$ echo 'cod1 foo_bar' | perl -pe 's/(co|fo)\K(\w)(\w)/$3$2/g'
co1d fo_obar
```

* named capture groups `(?<name>`
    * for backreference, use `\k<name>`
    * accessible via `%+` hash in replacement section

```bash
$ s='baz 2008-03-24 and 2012-08-12 foo 2016-03-25'
$ echo "$s" | perl -pe 's/(\d{4})-(\d{2})-(\d{2})/$3-$2-$1/g'
baz 24-03-2008 and 12-08-2012 foo 25-03-2016

$ # naming the capture groups might offer clarity
$ echo "$s" | perl -pe 's/(?<y>\d{4})-(?<m>\d{2})-(?<d>\d{2})/$+{d}-$+{m}-$+{y}/g'
baz 24-03-2008 and 12-08-2012 foo 25-03-2016
$ echo "$s" | perl -pe 's/(?<y>\d{4})-(?<m>\d{2})-(?<d>\d{2})/$+{m}-$+{d}-$+{y}/g'
baz 03-24-2008 and 08-12-2012 foo 03-25-2016

$ # and useful to transform different capture groups
$ s='"foo,bar",123,"x,y,z",42'
$ echo "$s" | perl -lpe 's/"(?<a>[^"]+)",|(?<a>[^,]+),/$+{a}|/g'
foo,bar|123|x,y,z|42
```

**Further Reading**

* [perldoc - Extended Patterns](https://perldoc.perl.org/perlre.html#Extended-Patterns)
* [rexegg - all the (? usages](http://www.rexegg.com/regex-disambiguation.html)
* [regular-expressions - recursion](http://www.regular-expressions.info/recurse.html#balanced)

<br>

#### <a name="modifiers"></a>Modifiers

* some are already seen, like the `g` (global match) and `i` (case insensitive matching)
* first up, the `r` modifier which returns the substitution result instead of modifying the variable it is acting upon

```bash
$ perl -e '$x="feed"; $y=$x=~s/e/E/gr; print "x=$x\ny=$y\n"'
x=feed
y=fEEd

$ # the r modifier is available for transliteration operator too
$ perl -e '$x="food"; $y=$x=~tr/a-z/A-Z/r; print "x=$x\ny=$y\n"'
x=food
y=FOOD
```

* `e` modifier allows to use Perl code in replacement section instead of string
* use `ee` if you need to construct a string and then apply evaluation

```bash
$ # replace numbers with their squares
$ echo '4 and 10' | perl -pe 's/\d+/$&*$&/ge'
16 and 100

$ # replace matched string with incremental value
$ echo '4 and 10 foo 57' | perl -pe 's/\d+/++$c/ge'
1 and 2 foo 3
$ # passing initial value
$ echo '4 and 10 foo 57' | c=100 perl -pe 's/\d+/$ENV{c}++/ge'
100 and 101 foo 102

$ # formatting string
$ echo '104 and 10 foo 57' | perl -pe 's/\d+/sprintf "%03d", $&/ge'
104 and 010 foo 057

$ # calling a function
$ echo 'food:12:explain:789' | perl -pe 's/\w+/length($&)/ge'
4:2:7:3

$ # applying another substitution to matched string
$ echo '"mango" and "guava"' | perl -pe 's/"[^"]+"/$&=~s|a|A|gr/ge'
"mAngo" and "guAvA"
```

* multiline modifiers

```bash
$ # m modifier to match beginning/end of each line within multiline string
$ perl -00 -ne 'print if /^Believe/' sample.txt
$ perl -00 -ne 'print if /^Believe/m' sample.txt
Just do-it
Believe it

$ perl -00 -ne 'print if /funny$/' sample.txt
$ perl -00 -ne 'print if /funny$/m' sample.txt
Today is sunny
Not a bit funny
No doubt you like it too

$ # s modifier to allow . meta character to match newlines as well
$ perl -00 -ne 'print if /do.*he/' sample.txt
$ perl -00 -ne 'print if /do.*he/s' sample.txt
Much ado about nothing
He he he
```

**Further Reading**

* [perldoc - perlre Modifiers](https://perldoc.perl.org/perlre.html#Modifiers)
* [stackoverflow - replacement within matched string](https://stackoverflow.com/questions/40458639/replacement-within-the-matched-string-with-sed)

<br>

#### <a name="quoting-metacharacters"></a>Quoting metacharacters

* part of regular expression can be surrounded within `\Q` and `\E` to prevent matching meta characters within that portion
    * however, `\` , `$` and `@` would still be interpolated
    * `\E` is optional if applying `\Q` till end of search expression
* typical use case is string to be protected is already present in a variable, for ex: user input or result of another command
* See also [perldoc - Quoting metacharacters](https://perldoc.perl.org/perlre.html#Quoting-metacharacters)


```bash
$ # same as: s='a+b' perl -ne 'print if index($_, $ENV{s})==0' eqns.txt
$ s='a+b' perl -ne 'print if /^\Q$ENV{s}/' eqns.txt
a+b,pi=3.14,5e12

$ s='a+b' perl -pe 's/^\Q$ENV{s}/ABC/' eqns.txt
a=b,a-b=c,c*d
ABC,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ s='a+b' perl -pe 's/\Q$ENV{s}\E.*,/ABC,/' eqns.txt
a=b,a-b=c,c*d
ABC,5e12
i*(t+9-g)/8,4-a+b
```

<br>

## <a name="two-file-processing"></a>Two file processing

First, a bit about `$#ARGV` and hash variables

```bash
$ # $#ARGV can be used to know which file is being processed
$ perl -lne 'print $#ARGV' <(seq 2) <(seq 3) <(seq 1)
1
1
0
0
0
-1

$ # creating hash variable
$ # checking if a key is present using exists
$ # or if value is known to evaluate to true
$ perl -le '$h{"a"}=5; $h{"b"}=0; $h{1}="abc";
            print "key:a value=", $h{"a"};
            print "key:b present" if exists $h{"b"};
            print "key:1 present" if $h{1}'
key:a value=5
key:b present
key:1 present
```

<br>

#### <a name="comparing-whole-lines"></a>Comparing whole lines

Consider the following test files

```bash
$ cat colors_1.txt
Blue
Brown
Purple
Red
Teal
Yellow

$ cat colors_2.txt
Black
Blue
Green
Red
White
```

* For two files as input, `$#ARGV` will be `0` only when first file is being processed
* Using `next` will skip rest of code
* entire line is used as key

```bash
$ # common lines
$ # same as: grep -Fxf colors_1.txt colors_2.txt
$ # same as: awk 'NR==FNR{a[$0]; next} $0 in a' colors_1.txt colors_2.txt
$ perl -ne 'if(!$#ARGV){$h{$_}=1; next}
            print if $h{$_}' colors_1.txt colors_2.txt
Blue
Red

$ # lines from colors_2.txt not present in colors_1.txt
$ # same as: grep -vFxf colors_1.txt colors_2.txt
$ # same as: awk 'NR==FNR{a[$0]; next} !($0 in a)' colors_1.txt colors_2.txt
$ perl -ne 'if(!$#ARGV){$h{$_}=1; next}
            print if !$h{$_}' colors_1.txt colors_2.txt
Black
Green
White
```

* alternative constructs

```bash
$ # using if-else instead of next
$ perl -ne 'if(!$#ARGV){ $h{$_}=1 }
            else{ print if $h{$_} }' colors_1.txt colors_2.txt
Blue
Red

$ # read all lines of first file in BEGIN block
$ # <> reads a line from current file argument
$ # eof will ensure only first file is read
$ perl -ne 'BEGIN{ $h{<>}=1 while !eof; }
            print if $h{$_}' colors_1.txt colors_2.txt
Blue
Red
$ # this method also allows to easily reset line number
$ # close ARGV is similar to calling nextfile in GNU awk
$ perl -ne 'BEGIN{ $h{<>}=1 while !eof; close ARGV}
            print "$.\n" if $h{$_}' colors_1.txt colors_2.txt
2
4

$ # or pass 1st file content as STDIN, $. will be automatically reset as well
$ perl -ne 'BEGIN{ $h{$_}=1 while <STDIN> }
            print if $h{$_}' <colors_1.txt colors_2.txt
Blue
Red
```

<br>

#### <a name="comparing-specific-fields"></a>Comparing specific fields

Consider the sample input file

```bash
$ cat marks.txt
Dept    Name    Marks
ECE     Raj     53
ECE     Joel    72
EEE     Moi     68
CSE     Surya   81
EEE     Tia     59
ECE     Om      92
CSE     Amy     67
```

* single field
* For ex: only first field comparison instead of entire line as key

```bash
$ cat list1
ECE
CSE

$ # extract only lines matching first field specified in list1
$ # same as: awk 'NR==FNR{a[$1]; next} $1 in a' list1 marks.txt
$ perl -ane 'if(!$#ARGV){ $h{$F[0]}=1 }
             else{ print if $h{$F[0]} }' list1 marks.txt
ECE     Raj     53
ECE     Joel    72
CSE     Surya   81
ECE     Om      92
CSE     Amy     67

$ # if header is needed as well
$ # same as: awk 'NR==FNR{a[$1]; next} FNR==1 || $1 in a' list1 marks.txt
$ perl -ane 'if(!$#ARGV){ $h{$F[0]}=1 }
             else{ print if $h{$F[0]} || !$c++ }' list1 marks.txt
Dept    Name    Marks
ECE     Raj     53
ECE     Joel    72
CSE     Surya   81
ECE     Om      92
CSE     Amy     67
```

* for multiple field comparison, use multidimensional hash

```bash
$ cat list2
EEE Moi
CSE Amy
ECE Raj

$ # extract only lines matching both fields specified in list2
$ # same as: awk 'NR==FNR{a[$1,$2]; next} ($1,$2) in a' list2 marks.txt
$ perl -ane 'if(!$#ARGV){ $h{$F[0]}{$F[1]}=1 }
             else{ print if $h{$F[0]}{$F[1]} }' list2 marks.txt
ECE     Raj     53
EEE     Moi     68
CSE     Amy     67
```

* field and value comparison

```bash
$ cat list3
ECE 70
EEE 65
CSE 80

$ # extract line matching Dept and minimum marks specified in list3
$ # same as: awk 'NR==FNR{d[$1]; m[$1]=$2; next} $1 in d && $3 >= m[$1]'
$ perl -ane 'if(!$#ARGV){ $d{$F[0]}=1; $m{$F[0]}=$F[1] }
             else{ print if $d{$F[0]} && $F[2]>=$m{$F[0]} }' list3 marks.txt
ECE     Joel    72
EEE     Moi     68
CSE     Surya   81
ECE     Om      92
```

* See also [stackoverflow - Fastest way to find lines of a text file from another larger text file](https://stackoverflow.com/questions/42239179/fastest-way-to-find-lines-of-a-text-file-from-another-larger-text-file-in-bash)

<br>

#### <a name="line-number-matching"></a>Line number matching

```bash
$ # replace mth line in poem.txt with nth line from nums.txt
$ # same as: awk -v m=3 -v n=2 'BEGIN{while(n-- > 0) getline s < "nums.txt"}
$ #                             FNR==m{$0=s} 1' poem.txt
$ m=3 n=2 perl -pe 'BEGIN{ $s=<> while $ENV{n}-- > 0; close ARGV}
                    $_=$s if $.==$ENV{m}' nums.txt poem.txt
Roses are red,
Violets are blue,
-2
And so are you.

$ # print line from fruits.txt if corresponding line from nums.txt is +ve number
$ # same as: awk -v file='nums.txt' '{getline num < file; if(num>0) print}'
$ file='nums.txt' perl -ne 'BEGIN{open($f,$ENV{file})}
                            $num=<$f>; print if $num>0' fruits.txt
fruit   qty
banana  31
```

<br>

## <a name="creating-new-fields"></a>Creating new fields

* Number of fields in input record can be changed by simply manipulating `$#F`

```bash
$ s='foo,bar,123,baz'

$ # reducing fields
$ # same as: awk -F, -v OFS=, '{NF=2} 1'
$ echo "$s" | perl -F, -lane '$,=","; $#F=1; print @F'
foo,bar

$ # creating new empty field(s)
$ # same as: awk -F, -v OFS=, '{NF=5} 1'
$ echo "$s" | perl -F, -lane '$,=","; $#F=4; print @F'
foo,bar,123,baz,

$ # assigning to field greater than $#F will create empty fields as needed
$ # same as: awk -F, -v OFS=, '{$7=42} 1'
$ echo "$s" | perl -F, -lane '$,=","; $F[6]=42; print @F'
foo,bar,123,baz,,,42
```

* adding a field based on existing fields

```bash
$ # adding a new 'Grade' field, split will be covered later
$ # same as: awk 'BEGIN{OFS="\t"; split("DCBAS",g,//)}
$ #          {NF++; $NF = NR==1 ? "Grade" : g[int($(NF-1)/10)-4]} 1' marks.txt
$ perl -lane 'BEGIN{$,="\t"; @g = split //, "DCBAS"} $#F++;
              $F[-1] = $.==1 ? "Grade" : $g[$F[-2]/10 - 5]; print @F' marks.txt
Dept    Name    Marks   Grade
ECE     Raj     53      D
ECE     Joel    72      B
EEE     Moi     68      C
CSE     Surya   81      A
EEE     Tia     59      D
ECE     Om      92      S
CSE     Amy     67      C
```

* two file example

```bash
$ cat list4
Raj class_rep
Amy sports_rep
Tia placement_rep

$ # same as: awk -v OFS='\t' 'NR==FNR{r[$1]=$2; next}
$ #          {NF++; $NF = FNR==1 ? "Role" : $NF=r[$2]} 1' list4 marks.txt
$ perl -lane 'if(!$#ARGV){ $r{$F[0]}=$F[1] }
              else{ $#F++; $F[-1] = !$c++ ? "Role" : $r{$F[1]};
                    print join "\t", @F }' list4 marks.txt
Dept    Name    Marks   Role
ECE     Raj     53      class_rep
ECE     Joel    72
EEE     Moi     68
CSE     Surya   81
EEE     Tia     59      placement_rep
ECE     Om      92
CSE     Amy     67      sports_rep
```

<br>

## <a name="dealing-with-duplicates"></a>Dealing with duplicates

* retain only first copy of duplicates

```bash
$ cat duplicates.txt
abc  7   4
food toy ****
abc  7   4
test toy 123
good toy ****

$ # whole line, same as: awk '!seen[$0]++' duplicates.txt
$ perl -ne 'print if !$seen{$_}++' duplicates.txt
abc  7   4
food toy ****
test toy 123
good toy ****

$ # particular column, same as: awk '!seen[$2]++' duplicates.txt
$ perl -ane 'print if !$seen{$F[1]}++' duplicates.txt
abc  7   4
food toy ****

$ # total count, same as: awk '!seen[$2]++{c++} END{print +c}' duplicates.txt
$ perl -lane '$c++ if !$seen{$F[1]}++; END{print $c+0}' duplicates.txt
2
```

* if input is so large that integer numbers can overflow
* See also [perldoc - bignum](https://perldoc.perl.org/bignum.html)

```bash
$ # avoid unnecessary counting altogether
$ # same as: awk '!($2 in seen); {seen[$2]}' duplicates.txt
$ perl -ane 'print if !$seen{$F[1]}; $seen{$F[1]}=1' duplicates.txt
abc  7   4
food toy ****

$ # use arbitrary-precision integers, limited only by available memory
$ # same as: awk -M '!($2 in seen){c++} {seen[$2]} END{print +c}' duplicates.txt
$ # -M option here enables the use of bignum module
$ perl -Mbignum -lane '$c++ if !$seen{$F[1]}; $seen{$F[1]}=1;
                       END{print $c+0}' duplicates.txt
2
```

* multiple fields

```bash
$ # same as: awk '!seen[$2,$3]++' duplicates.txt
$ perl -ane 'print if !$seen{$F[1]}{$F[2]}++' duplicates.txt
abc  7   4
food toy ****
test toy 123
```

* retaining specific copy

```bash
$ # second occurrence of duplicate
$ # same as: awk '++seen[$2]==2' duplicates.txt
$ perl -ane 'print if ++$seen{$F[1]}==2' duplicates.txt
abc  7   4
test toy 123

$ # third occurrence of duplicate
$ # same as: awk '++seen[$2]==3' duplicates.txt
$ perl -ane 'print if ++$seen{$F[1]}==3' duplicates.txt
good toy ****

$ # retaining only last copy of duplicate
$ # reverse the input line-wise, retain first copy and then reverse again
$ # same as: tac duplicates.txt | awk '!seen[$2]++' | tac
$ tac duplicates.txt | perl -ane 'print if !$seen{$F[1]}++' | tac
abc  7   4
good toy ****
```

* filtering based on duplicate count
* allows to emulate [uniq](./sorting_stuff.md#uniq) command for specific fields

```bash
$ # all duplicates based on 1st column
$ # same as: awk 'NR==FNR{a[$1]++; next} a[$1]>1' duplicates.txt duplicates.txt
$ perl -ane 'if(!$#ARGV){ $a{$F[0]}++ }
             else{ print if $a{$F[0]}>1 }' duplicates.txt duplicates.txt
abc  7   4
abc  7   4

$ # more than 2 duplicates based on 2nd column
$ # same as: awk 'NR==FNR{a[$2]++; next} a[$2]>2' duplicates.txt duplicates.txt
$ perl -ane 'if(!$#ARGV){ $a{$F[1]}++ }
             else{ print if $a{$F[1]}>2 }' duplicates.txt duplicates.txt
food toy ****
test toy 123
good toy ****

$ # only unique lines based on 3rd column
$ # same as: awk 'NR==FNR{a[$3]++; next} a[$3]==1' duplicates.txt duplicates.txt
$ perl -ane 'if(!$#ARGV){ $a{$F[2]}++ }
             else{ print if $a{$F[2]}==1 }' duplicates.txt duplicates.txt
test toy 123
```

<br>

## <a name="lines-between-two-regexps"></a>Lines between two REGEXPs

* This section deals with filtering lines bound by two *REGEXP*s (referred to as blocks)
* For simplicity the two *REGEXP*s usually used in below examples are the strings **BEGIN** and **END**

<br>

#### <a name="all-unbroken-blocks"></a>All unbroken blocks

Consider the below sample input file, which doesn't have any unbroken blocks (i.e **BEGIN** and **END** are always present in pairs)

```bash
$ cat range.txt 
foo
BEGIN
1234
6789
END
bar
BEGIN
a
b
c
END
baz
```

* Extracting lines between starting and ending *REGEXP*

```bash
$ # include both starting/ending REGEXP
$ # same as: awk '/BEGIN/{f=1} f; /END/{f=0}' range.txt
$ perl -ne '$f=1 if /BEGIN/; print if $f; $f=0 if /END/' range.txt
BEGIN
1234
6789
END
BEGIN
a
b
c
END

$ # can also use: perl -ne 'print if /BEGIN/../END/' range.txt
$ # which is similar to sed -n '/BEGIN/,/END/p'
$ # but not suitable to extend for other cases
```

* other variations

```bash
$ # same as: awk '/END/{f=0} f; /BEGIN/{f=1}' range.txt
$ perl -ne '$f=0 if /END/; print if $f; $f=1 if /BEGIN/' range.txt
1234
6789
a
b
c

$ # check out what these do:
$ perl -ne '$f=1 if /BEGIN/; $f=0 if /END/; print if $f' range.txt
$ perl -ne 'print if $f; $f=0 if /END/; $f=1 if /BEGIN/' range.txt
```

* Extracting lines other than lines between the two *REGEXP*s

```bash
$ # same as: awk '/BEGIN/{f=1} !f; /END/{f=0}' range.txt
$ perl -ne '$f=1 if /BEGIN/; print if !$f; $f=0 if /END/' range.txt
foo
bar
baz

$ # the other three cases would be
$ perl -ne '$f=0 if /END/; print if !$f; $f=1 if /BEGIN/' range.txt
$ perl -ne 'print if !$f; $f=1 if /BEGIN/; $f=0 if /END/' range.txt
$ perl -ne '$f=1 if /BEGIN/; $f=0 if /END/; print if !$f' range.txt
```

<br>

#### <a name="specific-blocks"></a>Specific blocks

* Getting first block

```bash
$ # same as: awk '/BEGIN/{f=1} f; /END/{exit}' range.txt 
$ perl -ne '$f=1 if /BEGIN/; print if $f; exit if /END/' range.txt
BEGIN
1234
6789
END

$ # use other tricks discussed in previous section as needed
$ # same as: awk '/END/{exit} f; /BEGIN/{f=1}' range.txt
$ perl -ne 'exit if /END/; print if $f; $f=1 if /BEGIN/' range.txt
1234
6789
```

* Getting last block

```bash
$ # reverse input linewise, change the order of REGEXPs, finally reverse again
$ # same as: tac range.txt | awk '/END/{f=1} f; /BEGIN/{exit}' | tac
$ tac range.txt | perl -ne '$f=1 if /END/; print if $f; exit if /BEGIN/' | tac
BEGIN
a
b
c
END

$ # or, save the blocks in a buffer and print the last one alone
$ # same as: awk '/4/{f=1; b=$0; next} f{b=b ORS $0} /6/{f=0} END{print b}'
$ seq 30 | perl -ne 'if(/4/){$f=1; $b=$_; next}
                     $b.=$_ if $f; $f=0 if /6/; END{print $b}'
24
25
26
```

* Getting blocks based on a counter

```bash
$ # get only 2nd block
$ # same as: seq 30 | awk -v b=2 '/4/{c++} c==b{print; if(/6/) exit}'
$ seq 30 | b=2 perl -ne '$c++ if /4/; if($c==$ENV{b}){print; exit if /6/}'
14
15
16

$ # to get all blocks greater than 'b' blocks
$ # same as: seq 30 | awk -v b=1 '/4/{f=1; c++} f && c>b; /6/{f=0}'
$ seq 30 | b=1 perl -ne 'if(/4/){$f=1; $c++}
                         print if $f && $c>$ENV{b}; $f=0 if /6/'
14
15
16
24
25
26
```

* excluding a particular block

```bash
$ # excludes 2nd block
$ # same as: seq 30 | awk -v b=2 '/4/{f=1; c++} f && c!=b; /6/{f=0}'
$ seq 30 | b=2 perl -ne 'if(/4/){$f=1; $c++}
                         print if $f && $c!=$ENV{b}; $f=0 if /6/'
4
5
6
24
25
26
```

* extract block only if matches another string as well

```bash
$ perl -ne 'if(/BEGIN/){$f=1; $m=0; $b=""}; $m=1 if $f && /23/;
            $b.=$_ if $f; if(/END/){print $b if $m; $f=0}' range.txt 
BEGIN
1234
6789
END

$ seq 30 | perl -ne 'if(/4/){$f=1; $m=0; $b=""}; $m=1 if $f && /^(5|25)$/;
                     $b.=$_ if $f; if(/6/){print $b if $m; $f=0}'
4
5
6
24
25
26
```

<br>

#### <a name="broken-blocks"></a>Broken blocks

* If there are blocks with ending *REGEXP* but without corresponding start, earlier techniques used will suffice
* Consider the modified input file where starting *REGEXP* doesn't have corresponding ending

```bash
$ cat broken_range.txt
foo
BEGIN
1234
6789
END
bar
BEGIN
a
b
c
baz

$ # the file reversing trick comes in handy here as well
$ # same as: tac broken_range.txt | awk '/END/{f=1} f; /BEGIN/{f=0}' | tac
$ tac broken_range.txt | perl -ne '$f=1 if /END/;
                         print if $f; $f=0 if /BEGIN/' | tac
BEGIN
1234
6789
END
```

* But if both kinds of broken blocks are present, for ex:

```bash
$ cat multiple_broken.txt 
qqqqqqq
BEGIN
foo
BEGIN
1234
6789
END
bar
END
0-42-1
BEGIN
a
BEGIN
b
END
;as;s;sd;
```

then use buffers to accumulate the records and print accordingly

```bash
$ # same as: awk '/BEGIN/{f=1; buf=$0; next} f{buf=buf ORS $0}
$ #          /END/{f=0; if(buf) print buf; buf=""}' multiple_broken.txt
$ perl -ne 'if(/BEGIN/){$f=1; $b=$_; next} $b.=$_ if $f;
            if(/END/){$f=0; print $b if $b; $b=""}' multiple_broken.txt
BEGIN
1234
6789
END
BEGIN
b
END

$ # note how buffer is initialized as well as cleared
$ # on matching beginning/end REGEXPs respectively
```




<br>

<br>

<br>

*More to follow*
