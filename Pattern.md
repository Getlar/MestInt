# Pattern

### A játék megtalálható a következő oldalon: [Pattern](https://www.chiark.greenend.org.uk/~sgtatham/puzzles/js/pattern.html).
<br>
A játék egy klasszikus <strong>kényszer-kielégitési</strong> feladat.

<hr>

## A játék szabályai

A Pattern, vagy magyarosabban Minta játék célja, hogy ugy szinezzünk be egy adott méretű négyzetrácsot, hogy megfeleljünk az adott sorhoz/oszlophoz tartozó kényszereknek.

Nehezitésképpen nem latin négyzettel dolgozunk, tehát egy adott szin előfordulhat többször is egy sorban, viszont egy oszlophoz/sorhoz több kényszer is tartozik.

A játékhoz ez esetben két kényszer tartozik, az egyik maga a domain amivel dolgozunk: fehér és fekete szin használható, akár több egy sorban. A mésodik kényszer pedig az, hogy ezek milyen sorrendben és milyen sorrendben helyezkednek el.

A program nagyrészt az <strong>aima-code</strong> 
implementációjához illeszkedik, igy Pythonba lett irva.

Használt keretrendszerek: csp.py, search.py, utils.py

A program 5 konstans rejtvényt tartalmaz, amit meg kell oldjon.

```python
constraints = []

constraints.append([[[1,3],[1],[1],[1,1],[1,1]],[[1,1],[1,1],[1,1],[1,1],[1,1]]])
```

A fenti kódsor egy konstans problémát ir le.

A programhoz először a specifikus kényszereket irtam meg, amelyek magukba foglalják a sorokhoz/oszlopokhoz tartozó kényszereket.
Itt két kényszer lett megfogalmazva:

    - Csak akkor ellenőrizze egy sor/oszlop helyességét, ha azt a sort/oszlopot teljesen feltöltötte szinekkel.
    - Ha teljes a sor/oszlop, akkor nézzen rá a kényszervektorra és állapitsa meg az elhelyezés helyességét.

Azért irtam, hogy sor/oszlop, mert ez a metódus csak vektorokat kap meg a hozzá tartozó kényszerekkel. Nem veszi figyelembe, hogy most melyik oszlopot vagy sort vizsgálja. Ő csak megkap egy vektort majd eldönti róla, hogy helyes-e. 

```python
def frame(a):
    
    ## CHECK IF SEGMENTS IN A VECTOR ARE CORRECT ACCORDING TO THE CONSTRAINTS
    def are_segments_ok(l, constr):
        found = 0
        i = 0
        while i < len(l):
            if l[i]=='b':
                found += 1
                tmpblack = 0
                for j in range(i,len(l)):
                    if l[j]!='b':
                        break
                    tmpblack += 1
                if found <= len(constr) and constr[found-1] != tmpblack:
                    return False
                i = j
            i += 1
        if(found!= len(constr)):
            return False
        return True
    
    ## GETS BOARD INDEXES WITHOUT CONSTRAINTS AND RETURNS ITS STATES (eg. NONE, B, W) 
    def rewrite(l,a):
        return [a.get(i, None) for i in l]
        
    ## CHECKS IF A CONSTRAINT IS SATISFACTORY. IF VECTOR IS FILLED AND SEGMENTS ARE NOT OK RETURN TRUE
    def check(l,a):
        lr = rewrite(l[:-1],a)
        return not None in lr and not are_segments_ok(lr, l[len(l)-1])
    
    return any(check(l,a) for l in frame_constraints)

```
Az <code>a</code> változó egy dictionary lesz ami tartalmazza az indexeket és a hozzá tartozó szineket. A <code>frame_constraints</code> fogja tartalmazni a különböző oszlop/sor kényszerekhez tartozó vektorokat. Például ezek lehetnek a következőek: <code> [1, 2, 3, 4, 5], [1, 6, 11, 16, 21]</code> stb. Végigmegyünk az összesen és keressük azt a vektort amely nem felel meg a kényszereknek. Ha van ilyen akkor a backtracking algoritmusunk visszalép egy lépést, majd csekkolja az egészet megint.

A backtracking algoritmus az <strong>aima-code</strong> által biztositott optimizálókat használja. Ezeket diasorban meg lehet találni.
    
    - MRV - Legkevesebb fennmaradó érték
    - Unordered Domain Value Ordering
    - Forward Checking Inference

```python
def backtracking_search(csp, select_unassigned_variable=csp.mrv,
                        order_domain_values=csp.unordered_domain_values,
                        inference=csp.forward_checking):

    def backtrack(assignment):
        if len(assignment) == len(csp.variables):
            return assignment
        var = select_unassigned_variable(assignment, csp)
        for value in order_domain_values(var, assignment, csp):
            assignment2 = assignment.copy()
            assignment2[var]=value
            if 0 == csp.nconflicts(var, value, assignment) and not frame(assignment2):
                csp.assign(var, value, assignment)
                removals = csp.suppose(var, value)
                if inference(csp, var, value, assignment, removals):
                    result = backtrack(assignment)
                    if result is not None:
                        return result
                csp.restore(removals)
        csp.unassign(var, assignment)
        return None

    result = backtrack({})
    assert result is None or csp.goal_test(result)
    return result
```

A program törzsének az <strong>aima-code</strong> által megirt MapColoringCSP keretet használtam. Ez a metódus visszaad egy olyan CSP példányt, ami inkább egy szinesző kényszerkielégitési problémára hasonlit, mint például egy Towers játékra. Argumentumokba a szin domain-t kell megadni, esetünkben a <code> ['b', 'w']</code> meg is teszi és egy szomszédsági listát. A CSP a következőkkel foglalkozik:

    - változók - az esetünkben ezek lesznek az indexek
    - domains - jelen esetben ezek lesznek a szinek
    - neighbours - ez lesz a szomszédsági lista
    - constraints - a program elején megfogalmazott konstans probléma

Ezt a példányt majd át is adhatjuk az <code>aima-code</code> által nyújtott backtracking algoritmusnak is és már kész is vagyunk.

```python
def PatternCSP(fcs):
    
    ## GETS NEIGHBOURS OF A GRID ELEMENT. (eg. 1,1 has 1,1...5, and 1...5,1)
    def neighbour(i,j):
        return [i*size+k for k in range(1,size+1) if j!=k]+\
               [k*size+j for k in range(size) if i!=k]
        
    ## GETS CONSTRAINTS AND CONSTRUCTS THE CURRENT FRAME FOR THE GAME
    def frame(fcs):
        l,d = fcs
        cs = []
        for i in range(size):
            c = [i*size+k for k in range(1,size+1)]
            c.append(l[i])
            cs.append(c)
            
            c=[k*size+i+1 for k in range(size)]
            c.append(d[i])
            cs.append(c)
            
        return cs
    
    ## SETS UP VARIABLES FOR MAP COLORING CSP PROBLEM
    global frame_constraints
    global average
    size = len(fcs[1])
    vars = [i for i in range(1,size*size + 1)]
    domain = ['b','w']
    neighbours = {i*size+j:neighbour(i,j) for i in range(size) for j in range(1,size+1)}
    frame_constraints = frame(fcs)
    
    ## CREATES A MAP COLORING CSP PROBLEM
    t = csp.MapColoringCSP(domain,neighbours)
    
    ## STARTS TIMER AND BEGINS THE BACKTRACKING ALGORITHM
    start_time = time.time()
    z = backtracking_search(t)
    for i in range(size):
        for j in range(size):
            print(z.get(i*size+j+1), end=" ")
        print()
    print('Time to solve: {:.3f}'.format(time.time()-start_time))
    average += time.time()-start_time
    

for constraint in constraints:
    PatternCSP(constraint)
    print()

print('Average time to solve was: {:.3f}'.format(average/len(constraints)))
```

Kényszerkielégités: 

    - állapot egy fekete doboz
    - állapotátmenetek, heurisztika és célállapot kell legyen implementálva

A feladat magasabb rendű kemény kényszert fogalmaz meg.

<strong>Kezdeti állapot:</strong> egyik változónak sincs értéke.

<strong>Rákövetkező függvény:</strong> rendeljünk egy olyan értéket valamely még értékkel nem rendelkező változóhoz, mely nem okoz konfliktust


A backtracking algoritmus a keresőfában megy és minden csúcsban egy változóhoz értéket rendel. Ezt mélységi keresésnek nevezzük.


Mivel lehetett volna még megoldani? 
 
    - mrv alapból gyors
    - fokszám heurisztika: válasszuk azt a változót, amely legtöbbször szerepel a hozzárendeletlen változókra vonatkozó kényszerekben
    - legkevésbe korlátozó érték: olyan értéket válasszunk amely a legkevesebb értéket zárja ki
    - előrenéző ellenőrzés: keresés leáll ha valamilyen változónak nem maradt értéke
    - előrenéző ellenőrzés helyett lehetne használni élkonzisztenciát: X->Y akkor konzisztens ha X minden x értékéhez van y eleme Y megengedett érték.
    - program esetén a teljes értékadás O(d^n) - 2^25 = 160
    - egy részproblémának 5 változója van, 25 a változóink száma és domain az 2, igy élkonzisztenciával lecsökkenthető - n/c * d^c = 5 * 2^5 = 
    - Hurok nélküli kényszergráfban megoldható O(nd^2) időben, ami - 25 * 2^2 = 125
    - Ebből a gráfból csináljunk fát egyes csúcsok értékelésével igy a futási idő O(d^2*(n-c)*d^2). Kis c esetén nagyon gyors. - 4 * 20 * 4 = 160
    - Lokális kereséssel, hegymászás és szimulált hűtés: ezek teljes állapottal dolgoznak, minden változó értékelt. 
    - Ezek átirják a változók értékeit.
    - Változó kiválasztása konfliktusos változók közül véletlenszerűen
    - Érték választása legkevesebb kényszert megszegő érték. Heurisztika a megszegett kényszerek száma.


Mivel lehetett volna még gyorsitani?

    - puha kényszerek implementálásával, minden értékeléshez költség tartozik, például ha a kényszervektorban található számok összege már nagyobb mint ahány fekete van lerakva, akkor ne tegye tovább a sort, vegyen vissza a feketékből.
    - Egy vektort nem csak akkor lehetne vizsgálni, ha már teljesen megtelte, hanem feltöltés közben is.

