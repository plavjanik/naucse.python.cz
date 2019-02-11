# Gane Like Asteroids

Our final project is going to use everything what we know: 
classes, graphics, lists... I hope that you will like it!

We will try to make a clone of the game [Asteroids](https://en.wikipedia.org/wiki/Asteroids_%28video_game%29) that has been released in 1979.

Our version will look like this:

{{ figure(
    img=static('screenshot.png'),
    alt="Asteroids like game screenshot"
) }}

The project is quite complex. It is using few things that were not covered by the course yet. I know that you will be able to look them up.

> [note]
> If you go through the project alone, it is possible that you
> will be stuck at some problem. 
> If it happens to you, let us know.
> We will be glad to help you!

## Spaceship

{# XXX: (asteroids1.py) #}

The first step is to program a spaceship that one can control by keyboard.

* Instance of class `Spaceship` represents the spaceship.
* Every spaceship has two attributes `x` a `y` (position),
  `x_speed` a `y_speed`, `rotation`, and
  `sprite` (2D object in Pyglet wiht position, speed, rotation, and image).
* Spaceship has a method called `tick` that handles the spaceship mechanics – movement, rotation, and control.
* All objects that are in the game are stored in a global list `objects`.
  It contains only the spaceship now.
* All pressed keys should be stored in a *set* (keyword `set`).
  It is a datatype list list but without order. It members can be in
  only once. (Sets are like dictionaries without values.)
  You can use [sets cheatsheet](https://github.com/pyvec/cheatsheets/blob/master/sets/sets-en.pdf) and the official Python documentation contains
  [a tutorial](https://docs.python.org/3/tutorial/datastructures.html#sets)
  and [the reference](https://docs.python.org/3/library/stdtypes.html#set-types-set-frozenset).
  The spaceship using thi set as part of the processing in its `tick` method.
* You can used [the image set](http://opengameart.org/content/space-shooter-redux),
  created by [Kenney Vleugels](http://kenney.nl). He made them public and free. You can draw your own if you want!
* Ve hře později použijeme velké množství
  `Sprite`-ů a vykreslovat je jeden po druhém by trvalo docela dlouho.
  Všechny `Sprite`-y proto přidej do kolekce
  [pyglet.graphics.Batch](https://pythonhosted.org/pyglet/api/pyglet.graphics.Batch-class.html),
  kterou pak Pyglet umí efektivně vykreslit najednou.
  Do „batche” jde přidávat pomocí argumentu při vytváření `Sprite()`
  a odebírat pomocí `sprite.delete()`. Například:

  ```python
  batch = pyglet.graphics.Batch()
  sprite1 = pyglet.sprite.Sprite(obrazek, batch=batch)
  sprite2 = pyglet.sprite.Sprite(obrazek, batch=batch)

  # You can draw all of them at the same time:
  batch.draw()
  ```

  Kolekci `batch` si stejně jako `objects` uchovávej globálně.
* Aby se objekty hýbaly a otáčely podle svých středů, je dobré nastavit „kotvu“
  obrázku na jeho střed (jinak je kotva v levém dolním rohu):

  ```python
  image = pyglet.image.load(...)
  image.anchor_x = image.width // 2
  image.anchor_y = image.height // 2
  self.sprite = pyglet.sprite.Sprite(image, batch=batch)
  ```
* Pro pohyb raketky půjde použít klávesy s šipkami doleva, doprava a rovně.
  Šipky do stran raketu točí, šipka dopředu zrychluje pohyb tím směrem, kam je
  raketka otočená.
  * Základní pohyb raketky je jednoduchý: k <var>x</var>-ové
    souřadnici se přičte <var>x</var>-ová rychlost krát uplynulý čas
    a to samé v <var>y</var>-ové souřadnici i pro úhel otočení:

      ```python
      self.x = self.x + dt * self.x_speed
      self.y = self.y + dt * self.y_speed
      self.rotation = self.rotation + dt * rotation_speed
      ```

      Rychlost otáčení závisí na stisknutých šipkách (doleva nebo doprava).
      V jednom případě je záporná, v druhém kladná. Vhodnou hodnotu zvol
      experimentováním – začni třeba u 4 radiánů za sekundu.
      Všechny podobné „magické hodnoty“ je vhodné definovat
      jako konstanty – tedy proměnné, které na začátku nastavíš a nikdy
      je neměníš. Bývá zvykem je označovat velkými písmeny a dávat je na
      začátek souboru, hned za importy:

      ```python
      ROTATION_SPEED = 4  # radians per second
      ```
  * Zrychlení je trochu složitější: k  <var>x</var>-ové rychlosti
    se přičte kosinus úhlu otočení krát uplynulý čas.
    U <var>y</var>-ové osy se použije sinus.

      ```python
      self.x_speed += dt * ACCELERATION * math.cos(self.rotation)
      self.y_speed += dt * ACCELERATION * math.sin(self.rotation)
      ```

      Všimni si v příkladu konstanty `ACCELERATION`. Tu opět zvol podle uvážení.
  * Když máš hodnoty `self.x`, `self.y` a `self.rotation` spočítané, nezapomeň
    je promítnout do `self.sprite`, jinak se nic zajímavého nestane.

    Pozor na to, že funkce `math.sin` a `math.cos` používají radiány,
    kdežto `pyglet` používá pro `Sprite.rotation` stupně.
    (A k tomu je navíc 0° jinde, a otáčí se na opačnou stranu.)
    Pro sprite je tedy potřeba úhel převést:

      ```python
      self.sprite.rotation = 90 - math.degrees(self.rotation)
      self.sprite.x = self.x
      self.sprite.y = self.y
      ```
  * Když raketka vyletí z okýnka ven, vrať
    ji zpátky do hry na druhé straně okýnka.
    (Zkontroluj si, že to funguje na všech čtyřech stranách.)


* **Bonus 1**: Zkus si přidat několik raketek,
  každou trochu jinak natočenou.

  Každý jednotlivý objekt třídy `Spaceship`
  si udržuje vlastní stav, takže by nemělo být složité
  jich vytvořit víc (a všechny ovládat najednou).
* **Bonus 2**:
  Možná sis všiml{{a}} „skoku”, když
  raketa vyletí z okýnka a vrátí se na druhé straně.
  Tomu se dá zabránit tak, že
  vlevo, vpravo, nahoře i dole vedle naší „scény”
  vykreslíš celou scénu ještě jednou.

  Pyglet na to má speciální nízkoúrovňové funkce,
  kterými můžeš říct „tady kresli všechno posunuté o
  X pixelů vlevo”. Úplné vysvětlení by bylo na dlouho,
  takže si zatím jen zkopíruj kód:

  ```python
  from pyglet import gl

  def draw():
      window.clear()

      for x_offset in (-window.width, 0, window.width):
          for y_offset in (-window.height, 0, window.height):
              # Remember the current state
              gl.glPushMatrix()
              # Move everything drawn from now on by (x_offset, y_offset, 0)
              gl.glTranslatef(x_offset, y_offset, 0)

              # Draw
              batch.draw()

              # Restore remembered state (this cancels the glTranslatef)
              gl.glPopMatrix()
  ```
  Pro přehled, dokumentace k použitým funkcím je tady:
  [glPushMatrix, glPopMatrix](https://www.opengl.org/sdk/docs/man2/xhtml/glPushMatrix.xml),
  [glTranslatef](https://www.opengl.org/sdk/docs/man2/xhtml/glTranslate.xml).

Povedlo se? Můžeš létat vesmírem?
Čas to všechno dát do Gitu!

Projdi si předchozí body, jestli máš opravdu všechno, a můžeš pokračovat dál!

## Asteroids

{# XXX: (asteroids2.py) #}

Přidej druhý typ vesmírného objektu: `Asteroid`.

* Asteroidy a vesmírné lodě mají mnoho společného:
  každý takový vesmírný objekt bude mít polohu,
  rychlost, natočení a pravidla, jak se pohybuje.
  Vytvoř proto třídu `SpaceObject`,
  ve které bude všechno to společné, a z ní poděď
  třídu `Spaceship`, ve které zůstane
  kód specifický pro vesmírnou loď (t.j. ovládání
  klávesnicí, obrázek lodě, začátek v prostředku
  obrazovky).
* Část kódu pro pohyb bude společná pro všechny
  vesmírné objekty (např. věci kolem zrychlení);
  část bude specifická jen pro raketku (ovládání
  pomocí klávesnice).
  Využij funkci `super()` z [lekce o dědičnosti](../../beginners/inheritance/).
* Napiš ještě třídu `Asteroid`,
  která taky dědí ze `SpaceObject`,
  ale má svoje vlastní chování:
  začíná buď na levé nebo spodní straně obrazovky
  s náhodnou rychlostí
  a ke každému asteroidu se přiřadí
  náhodně vybraný obrázek.
  (V Asteroidech je levý a pravý okraj v podstatě
    to samé; a stejně tak horní a spodní.)
* A pak pár asterojdíků různých velikostí přidej
  na začátku do hry.

Povedlo se? Máš dva typy objektů?
Čas to všechno dát do Gitu!

Zase si projdi, jestli máš všechno hotové,
a jdeme na další část!

## Collisions

{# XXX: (asteroids3.py) #}

Naše asteroidy jsou zatím docela neškodné. Pojďme to změnit.

* V této sekci bude tvým úkolem zjistit, kdy
  loď narazila do asteroidu.
  Pro zjednodušení si každý objekt nahradíme
  kolečkem a budeme počítat, kdy se srazí kolečka.
  Každý objekt bude potřebovat mít poloměr – atribut `radius`.
* Aby bylo vidět co si hra o objektech „myslí”,
  nakresli si nad každým objektem příslušné kolečko.
  Nejlepší je to udělat pomocí
  [pyglet.gl](http://pyglet.readthedocs.org/en/latest/programming_guide/gl.html)
  a trochy matematiky; pro teď si jen opiš funkci
  `draw_circle` a pro každý objekt ji zavolej.
  Až to bude všechno fungovat, můžeš funkci dát pryč.

  ```python
  def draw_circle(x, y, radius):
      iterations = 20
      s = math.sin(2*math.pi / iterations)
      c = math.cos(2*math.pi / iterations)

      dx, dy = radius, 0

      gl.glBegin(gl.GL_LINE_STRIP)
      for i in range(iterations+1):
          gl.glVertex2f(x+dx, y+dy)
          dx, dy = (dx*c - dy*s), (dy*c + dx*s)
      gl.glEnd()
  ```
* Když asteroid narazí do lodi, loď exploduje a zmizí.
  Explozi necháme na později, teď je důležité odebrání objektu ze hry.
  Dej ho do metody `SpaceObject.delete`,
  protože vyndávat ze hry se dá jakýkoli objekt.
  V této metodě musíš objekt jednak odstranit
  ze seznamu `objects` a pak zrušit jeho `Sprite`, aby se už v rámci
  `batch` nevykresloval.
* A jak udělat ono narážení?
  V rámci `Spaceship.tick` projdi
  každý objekt, zjisti jestli vzdálenost mezi lodí
  a objektem je menší než součet poloměrů
  (t.j. narazily do sebe) a pokud ano,
  zavolej na objektu metodu `hit_by_spaceship`.

  Zjišťování vzdálenosti ve hře, kde se
  [objekty které vyletí ven vrací na druhé straně](https://en.wikipedia.org/wiki/Wraparound_%28video_games%29),
  není úplně přímočaré, takže si příslušný kód pro teď jen zkopíruj:

  ```python
  def distance(a, b, wrap_size):
      """Distance in one direction (x or y)"""
      result = abs(a - b)
      if result > wrap_size / 2:
          result = wrap_size - result
      return result

  def overlaps(a, b):
      """Returns true iff two space objects overlap"""
      distance_squared = (distance(a.x, b.x, window.width) ** 2 +
                          distance(a.y, b.y, window.height) ** 2)
      max_distance_squared = (a.radius + b.radius) ** 2
      return distance_squared < max_distance_squared
  ```

  Většina objektů v dokončené hře (např. oheň z
  rakety, střela) nebude při kolizi s lodí dělat nic,
  takže metoda `SpaceObject.hit_by_spaceship`
  by neměla dělat nic (musí jen existovat).
  Jen asteroid loď rozbije, takže předefinuj
  `Asteroid.hit_by_spaceship`, aby
  zavolala `delete` lodi.

  Protože lodí může být v naší hře obecně více, musí asteroid
  vědět, se kterou lodí se srazil, aby ji mohl rozbít.
  Metoda `hit_by_spaceship` by tedy na to měla mít argument.

Povedlo se? Konečně se dá prohrát?
Čas to všechno zkontrolovat, dát do Gitu a můžeme pokračovat!

## Attack

{# XXX: (asteroids4.py) #}

Teď zkusíme asteroidy rozbíjet.

* Raketka umí jednou za 0,3 s vystřelit laser.
  Ulož si pro každou raketku (jako atribut) číslo,
  které po každém výstřelu nastav na 0,3
  a pak ho v metodě `tick` nech klesat o 1 za vteřinu.
  Když bude záporné, může hráč vystřelit znovu.
* Když hráč drží mezerník a může vystřelit, vystřelí.
  Ve hře se to projeví tak, že se přidá objekt nové třídy `Laser`.
  Začne na souřadnicích raketky, s natočením raketky
  a s rychlostí raketky plus něco navíc ve směru natočení.
* Každý objekt třídy `Laser` si „pamatuje“,
  jak dlouho ještě bude ve hře.
  Na začátku se tohle číslo nastaví tak, aby přeletěl
  zhruba něco víc než jednu obrazovku.
  Když dojde čas, `Laser` zmizí.
* Ve své metodě `tick` laser projde
  všechny objekty, a pokud se s některým překrývá,
  tak na něm zavolá metodu `hit_by_laser`.
  U většiny objektů tahle metoda nedělá nic,
  jen asteroidy bude rozbíjet.
* Když se laser dotkne asteroidu, asteroid se
  rozdělí na dva menší (nebo, je-li už příliš malý, zmizí úplně).

  Rychlosti nových asteroidů si můžeš nastavit
  podle sebe – důležité je jen, aby každý menší
  asteroid letěl jinam.
  Většinou bývají nové asteroidy rychlejší než ten původní.
* A to je vše! Máš funkční hru!

Povedlo se? Dá se i vyhrát? Čas to všechno dát do Gitu!


## Final Steps and Possible Extentions

{# XXX: (asteroids5.py) #}

Chceš-li ve hře pokračovat, tady jsou další nápady.
Můžeš je dělat v jakémkoli pořadí – nebo si vymysli
vlastní rozšíření!

* Je hra příliš těžká?

  Můžeš přidat životy: na začátku jsou tři,
  a dokud nějaký zbývá, raketka se po zásahu
  asteroidem objeví znovu uprostřed,
  s nulovou rychlostí.
  Hra by taky při tomto „restartu” měla ignorovat
  držené klávesy, dokud je hráč znovu nezmáčkne
  (nejlépe pomocí `pressed_keys.clear()`).

  Počet náhradních lodí můžeš ukázat ikonkami
  na spodku obrazovky.

  **Bonus:** Několik vteřin po
  „restartu” může být raketka nezničitelná,
  aby měla čas odletět, když je zrovna uprostřed
  okýnka asteroid.
* Je hra příliš lehká?

  Přidej úrovně: až hráč vystřílí všechny asteroidy,
  postoupí na další úroveň, kde je asteroidů víc než v té předchozí.

  Číslo úrovně můžeš ukázat pomocí
  [pyglet.text.Label](http://pyglet.readthedocs.org/en/latest/programming_guide/text.html).
* Je pozadí příliš černé?

  V sadě obrázků v adresáři `Backgrounds`
  si vyber pozadí, a vytapetuj s ním celý vesmír.
* Je hra moc strohá?

  Přidej oheň a exploze!
  Chovají se podobně jako `Laser`,
  jen nic neničí a můžou třeba měnit barvu podle
  toho, jak dlouho už jsou ve hře.

  Na efekty můžeš použít obrázky
  [„Smoke particle assets”](http://opengameart.org/content/smoke-particle-assets),
  které nakreslil opět [Kenney Vleugels](http://kenney.nl).
  Doporučuji „White Puff”, které můžeš zmenšit
  (např. `sprite.scale = 1/10`),
  přibarvit
  (např. `sprite.color = 255, 100, 0`)
  nebo částečně zprůhlednit
  (např. `sprite.opacity = 100`).

  Doporučuji si na efekty udělat nový `Batch`
  a vykreslit ho před tím hlavním, aby efekty
  nepřekrývaly herní objekty.
* Nepoznáš, kdy jsi prohrál{{a}} nebo vyhrál{{a}}?

  Na konci můžeš ukázat veliký nápis GAME OVER nebo WINNER.
* Nudíš se?

  V původní hře se občas objeví UFO, které občas
  vystřelí na místo, kde je právě hráčova raketka,
  takže pokud hráč stojí pořád na jednom místě a
  jenom se točí dokola, UFO ho sestřelí.
  Můžeš zkusit dodělat třídu `Ufo`
  a z `Laser` podědit `ShipLaser` a `UfoLaser`.

Povedlo se? Vypadá to a chová se to profesionálně?
Čas to všechno dát do Gitu!