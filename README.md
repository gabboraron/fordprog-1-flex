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

# 2
fájl: `2/flex2.l`
> ha az előző programban a reguláris kifejezések helyre  beírnánk, hogy `username    cout << "deva";` akkor a reguláris kifejezés működne, és a szöveg összes `username` szavát lecserélnénk `deva`ra.

# 3
fájl: `3/felx3.l`
A a `c++` kódot `html`re is cserélhetjük így akár:
````flex
\t    cout << "&nbsp;";

\n    cout << "<BR>" << endl;

\<    cout << "&lt;";

\>    cout << "&gt;";
````

# 4
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

# 5
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
