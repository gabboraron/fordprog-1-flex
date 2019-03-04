# flex
## 1


fájl: `1/flex1.l`
- fájl mködése:
`noyywrap` a forrásfájl végén az elmzésnek is vége
`c++` a generált program C++ nyelvű
````flex
%option noyywrap c++
````
A szükséges C++ includeok:
````flex
%{
#include <iostream>
#include <fstream>

using namespace std;
%}
````
jelenleg nincsennek reguláris kifejezések:
````flex
%%

%%
````
> Ha volna akkor a Flex által generált reguláris elmező ezeket a szabályokat ilesztené rá a megadott kifejezésekre, ha egyik szabály sem illeszthető akkor hibát dob, azaz az aktuális karaktert a megadott kimenetre továbbítja, ebben az esetben mivel egyetlen szabály sincs, ezért minden bemenet hiba, hisz nincs mit illeszteni, azaz mindent kidob a kimenetre.

a maradék a C++ kód amire szükségünk van, main-nel együtt:
````c++
int main()
{
    ifstream in("input.txt");
    yyFlexLexer fl( &in, &cout );
    fl.yylex();
    return 0;
}
````
egyben:
````flex
%option noyywrap c++

%{
#include <iostream>
#include <fstream>

using namespace std;
%}

%%

%%

int main()
{
    ifstream in("input.txt");
    yyFlexLexer fl( &in, &cout );
    fl.yylex();
    return 0;
}
````
- futtassuk: `flex flex1.l`
- keletkezik egy `lex.yy.cc` fájl benne C++ kóddal
- fordítsuk: `g++ -o flex1 lex.yy.cc`
- a fordítás eredménye `flex1` futtatható állomány
- futtassuk:ekkor az `input.txt` tartalma jelenik meg standar kimeneten

## 2
fájl: `2/flex2.l`
> ha az előző programban a reguláris kifejezések helyre  beírnánk, hogy `username    cout << "deva";` akkor a reguláris kifejezés működne, és a szöveg összes `username` szavát lecserélnénk `deva`ra.

## 3
fájl: `3/felx3.l`
A a `c++` kódot `html`re is cserélhetjük így akár:
````flex
\t    cout << "&nbsp;";

\n    cout << "<BR>" << endl;

\<    cout << "&lt;";

\>    cout << "&gt;";
````

## 4
fájl: `4/felx4.l` és `4/flex4v2.l`

Reguláris kifejezésekhez változókat is használhatunk, az első részben kell definiálni őket, akár kezdőértékkel együtt, így:
````flex
%{
#include <iostream>
#include <fstream>

using namespace std;

int sor=1,oszlop=1;
%}
````
Mintaillesztésígy sor vége jelre, __ki kell írni a két változó aktuális értékét,
majd inkrementálni kell a sorok számát, és alaphelyzetbe kell állítani az oszlopok
számát __:
````flex
\n        {
            cout << sor << ' ' << oszlop << endl;
            ++sor;
            oszlop=1;
        }
````
Minden egyéb karakterre növeljük az oszlopzámot:
````flex
.            ++oszlop;
````

De mindez megoldható az `YYLeng()` fügvény használatával is, í]y viszont a teljes sorokra kell regexet illeszteni:
````flex
[^\n]*\n    {
                cout << sor << ' ' << YYLeng() << endl;
                ++sor;
              }
````

## 5
Ha szavakra tördeljük a szöveget úgy, hogy kiíratjuk a kezdőkarakter sorát és oszlopát, azt így lehet:
fájl: `5/flex5.l`
````flex
[^\n\t ]+    {
                 cout << sor << ' ' << oszlop << ' ' << YYText() << endl;
                 oszlop += YYLeng();
                }

\n        {
            ++sor;
             oszlop=1;
          }

.            ++oszlop;
````
- Ebből az új sort, tabot, szóközt nem tartalmazó üres sorozat: `[\n\t ]+`
- a szót magát így tudjuk kiítratni: `YYText()`
- a hosszát így: `YYLeng()`

**Másképp:**
fájl: `5/flex5v2.l`
Az `yylineno` opcióval az elmező számontartja a sorok számát.
A sor változót meg a `lineno()` függvény váltja ki:
````flex
[^\n\t ]+    {
                    cout << lineno() << ' ' << oszlop << ' ' << YYText() << endl;
                    oszlop += YYLeng();
               }

\n            oszlop=1;

.            ++oszlop;
````
# Flex regex
> eredeti itt: http://people.cs.aau.dk/~marius/sw/flex/Flex-Regular-Expressions.html

**Flex Regular Expressions**

The characters and literals may be described by:

`x`
    the character x.
    
`.`
    any character except newline.
    
`[xyz]`
    Any characters amongst x, y or z. You may use a dash for character intervals: [a-z] denotes any letter from a through z. You may use a leading hat to negate the class: [0-9] stands for any character which is not a decimal digit, including new-line.
    
`\x`
    if x is an a, b, f, n, r, t, or v, then the ANSI-C interpretation of \x. Otherwise, a literal x (used to escape operators such as *).
    
`\0`
    a NUL character.
    
`\num`
    the character with octal value num.
    
`\xnum`
    the character with hexadecimal value num.
    
`"string"`
    Match the literal string. For instance "/*" denotes the character / and then the character *, as opposed to /* denoting any number of slashes.
    
`<<EOF>>`
    Match the end-of-file. 


 _The basic operators to make more complex regular expressions are, with r and s being two regular expressions:_

`(r)`
    Match an r; parentheses are used to override precedence.
    
`rs`
    Match the regular expression r followed by the regular expression s. This is called concatenation.
    
`r|s`
    Match either an r or an s. This is called alternation.
    
`{abbreviation}`
    Match the expansion of the abbreviation definition. Instead of writing
    
````flex
    %%
    [a-zA-Z_][a-zA-Z0-9_]*   return IDENTIFIER;
    %%

    you may write

    id  [a-zA-Z_][a-zA-Z0-9_]*
    %%
    {id}   return IDENTIFIER;
    %%
````

The quantifiers allow to specify the number of times a pattern must be repeated:

`r*`
    zero or more r's.
    
`r+`
    one or more r's.
    
`r?`
    zero or one r's.

`r{[num]}`
    num times r

`r{min,[max]}`
    anywhere from min to max (defaulting to no bound) r's. 


For instance `-?([0-9]+|[0-9]*\.[0-9]+([eE][-+]?[0-9]+)?)` matches `C integer` and floating point numbers.

One may also depend upon the context:

`r/s`
    Match an r but only if it is followed by an s. This type of pattern is called trailing context. The text matched by s is included when determining whether this rule is the "longest match", but is then returned to the input before the action is executed. So the action only sees the text matched by r. Using trailing contexts can have a negative impact on the scanner, in particular the input buffer can no longer grow upon demand. In addition, it can produce correct but surprising errors. Fortunately it is seldom needed, and only to process pathologic languages such as Fortran. For instance to recognize its loop keyword, do, one needs: `DO/[A-Z0-9]*=[A-Z0-9]*`,to distinguish `DO1I=1,5`, a `for` loop where I runs from `1` to `5`, from `DO1I=1.5`, a `definition/assignment` of the floating variable `DO1I` to `1.5`. Voir Fortran and Satellites, for more on Fortran loops and traps.

`^r`
    Match an r at the beginning of a line.

`r$`
    Match an r at the end of a line. This is rigorously equivalent to r/\n, and therefore suffers the same problems, see http://people.cs.aau.dk/~marius/sw/flex/Simple-Uses-of-Flex.html
