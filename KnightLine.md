# Knight Line
### A játék megtalálható a következő oldalon: [Knight Line](http://mrraow.com/index.php/aiai-home/).
<br>
A játék egy klasszikus kétszemélyes logikai játék.
<hr>

## Játék szabályai

A Knight Line nagyon hasonlit egy talán más ismert kétszemélyes játékra, amit Grundy-nak hivnak. 

[Grundy](https://en.wikipedia.org/wiki/Grundy%27s_game).

Ezt keverjük sakkból vett elemekkel és kapunk is egy olyan játékot, ami egyszerű de élvezhető is.

Minden játékos egy stack húszárral kezd, amik egymáson helyezkednek a pálya közepén. A kezdő játékos ezek után elmozdithat annyi lovat a stack-ről amennyit akar egy olyan mezőre, amely már rendelkezik szomszédokkal. A huszárokkal csak L alakba lehet lépni. Ezután jön a másik játékos és igy tovább. Az nyer, aki hamarabb kirak 4 bábut átlósan.

A program nagyrészt az <strong>aima-code</strong> implementációjához illeszkedik, igy Pythonba lett irva.

Használt keretrendszerek: games.py, search.py, utils.py

A program egy 13x13-as griden valósul meg és minden játékos 30 lóval kezd. Azért válaszottam ezt a számot mert ezzel nagy valószinűséggel lesz végállapot.

A program a pályafeltöltéssel kezdődik, ami a következő formában néz ki. <code>[X, Y, Lovak Száma, Játékos]</code>, ahol a játékos lehet 'X' vagy 'O'.

Ezt a játékmezőt átadjuk a KnightLine osztálynak, ami a következőt csinálja:
    
    - Kap a kereséstől egy valid lépést, azt implementálja
    - Megnézi, hogy a következő játékos milyen lépésekkel fog rendelkezni
    - Megváltoztatja az aktuális állapotot.

Az aktuális állapot hasznosságot a következő heurisztikával fogalmaztam meg. Azt vettem észre egy csomó példajáték után, hogy a táblán terjeszkedni kell, igy egy jó lépés az lenne ha a huszáraink felét mozditanánk. Egy huszár vagy az összes huszár mozgatása egyenértékűen rossz, igy ennek a két faktornak köszönhetően a binomális együtthatókat vettem segitségül.

A binomiális együtthatók segitségével olyan hasznossági mércét tudtam meghatározni, ami arra ösztönzi a játékost, hogy terjeszkedjen. Sajnos nagy tábla esetén, ahol sok lóval játszunk egy nagy csomó felének való mozgatása nagyon nagy értéket vonna maga után igy ezeket az értékeket mind normalizáltam 0 és 1 közé. 

Plusz heurisztikaként azt is felvetettem, hogy sokkal jobb lépés olyan helyre bábut tenni, ahol már átlósan több mint egy lóval rendelkeznénk, igy minden egy átlóban lévő saját ló után egy pontot irunk hózzá a hasznossághoz. 

Ennek köszönhetően akkor kerülünk végállapotba, ha az aktuális állapot hasznossága nagyobb lesz mint 4, vagy kisebb mint -4 (MIN és MAX), vagy kifogytunk a lépésekből, akkor viszont döntetlen a helyzet.

Normalizalas: Xnorm = (X-Xmin)/(Xmax-Xmin)

A játék egy while ciklusban megy egész addig, amig valaki nem nyer vagy elfogynak a szabad lépések. Minden MinMax keresés előtt lemásoljuk az aktuális állapotot, hiszen a keresések megváltoztatják, igy ha nem mentjük el, a keresés lezárja a játékunkat.

A játék játszható MinMax, Alpha-Beta és random játékossal is. 

Alpha-Beta keresés esetén korlátozni tudjuk, hogy a játékfába milyen mélyre menjen le a keresés. Minél lejjebb annál pontosabb, de lassabb is.
