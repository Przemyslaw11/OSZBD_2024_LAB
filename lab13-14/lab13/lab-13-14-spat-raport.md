# Raport

# Przetwarzanie i analiza danych przestrzennych 
# Oracle spatial


---

**Imiona i nazwiska:**
- Piotr Urbańczyk
- Przemysław Spyra
--- 

Celem ćwiczenia jest zapoznanie się ze sposobem przechowywania, przetwarzania i analizy danych przestrzennych w bazach danych
(na przykładzie systemu Oracle spatial)

Swoje odpowiedzi wpisuj w miejsca oznaczone jako:

---
> Wyniki, zrzut ekranu, komentarz

```sql
--  ...
```

---

Do wykonania ćwiczenia (zadania 1 – 7) i wizualizacji danych wykorzystaj Oracle SQL Develper. Alternatywnie możesz wykonać analizy w środowisku Python/Jupyter Notebook

Do wykonania zadania 8 wykorzystaj środowisko Python/Jupyter Notebook

Raport należy przesłać w formacie pdf.

Należy też dołączyć raport zawierający kod w formacie źródłowym.

Np.
- plik tekstowy .sql z kodem poleceń
- plik .md zawierający kod wersji tekstowej
- notebook programu jupyter – plik .ipynb

Zamieść kod rozwiązania oraz zrzuty ekranu pokazujące wyniki, (dołącz kod rozwiązania w formie tekstowej/źródłowej)

Zwróć uwagę na formatowanie kodu

<div style="page-break-after: always;"></div>

# Zadanie 1

Zwizualizuj przykładowe dane

US_STATES

![img_3.png](img_3.png)
```sql
SELECT * FROM us_states
```
Stany mają stałą liczbę stanów, tj. 50.

US_INTERSTATES

![img_2.png](img_2.png)
```sql
SELECT * FROM  us_interstates
```
Autostrady międzystanowe są bardziej zagęszczone w regionach o dużej gęstości zaludnienia i w regionach z dużym ruchem tranzytowym. Najwięcej autostrad znajdziemy w północno-wschodnich stanach oraz w Kalifornii, natomiast najmniej w mniej zaludnionych stanach, takich jak Montana czy Wyoming.

US_CITIES

![img.png](img.png)
```sql
SELECT * FROM  us_cities
```
Miasta są najliczniejsze w stanach z dużą populacją, takich jak Kalifornia, Teksas, Nowy Jork, Floryda. Najmniej miast znajdziemy w stanach o małej gęstości zaludnienia, takich jak Alaska, Wyoming, czy Vermont.

US_RIVERS

![img_4.png](img_4.png)
```sql
SELECT * FROM  us_rivers
```
Rzeki są najliczniejsze w regionach o dużej ilości opadów oraz w miejscach z licznymi zbiornikami wodnymi. Przykładowo, najwięcej rzek znajdziemy na wschodnim wybrzeżu, w stanach takich jak Pensylwania czy Nowy Jork, oraz w centralnej części USA. Najmniej rzek jest na suchych obszarach, takich jak pustynie Arizony czy Nevady.

US_COUNTIES

![img_5.png](img_5.png)

```sql
SELECT * FROM  us_counties
```
Hrabstwa są liczniejsze w stanach o dużej powierzchni i historycznie gęsto zaludnionych obszarach, takich jak Georgia czy Wirginia. Najmniej hrabstw znajdziemy w stanach o mniejszej powierzchni lub niskiej gęstości zaludnienia, takich jak Delaware, Rhode Island, czy Alaska.

US_PARKS

![img_6.png](img_6.png)

```sql
SELECT * FROM  us_parks
```
Parki narodowe są najliczniejsze w zachodnich stanach USA, takich jak Kalifornia, Alaska, Utah, Kolorado, i Arizona, gdzie znajduje się wiele terenów chronionych. Najmniej parków narodowych jest w stanach wschodniego wybrzeża oraz w stanach środkowo-zachodnich, poza kilkoma wyjątkami

# Zadanie 2

**Uwaga: Parzyste zadania zostały wykonane w środowisku jupyter notebook.**

Znajdź wszystkie stany (us_states) których obszary mają część wspólną ze wskazaną geometrią (prostokątem)

Pokaż wynik na mapie.

prostokąt

```sql
SELECT  sdo_geometry (2003, 8307, null,
sdo_elem_info_array (1,1003,3),
sdo_ordinate_array ( -117.0, 40.0, -90., 44.0)) g
FROM dual
```



> Wyniki, zrzut ekranu, komentarz

```python
rectangle = """SELECT sdo_util.to_wktgeometry(
    sdo_geometry (2003, 8307, null,
    sdo_elem_info_array (1,1003,3),
    sdo_ordinate_array ( -117.0, 40.0, -90., 44.0))
) g FROM dual"""

rectangle_result = cursor.execute(rectangle).fetchall()

# Tu zaczyna się definicja nowej mapy,
# poniżej dokładane są do niej nowe "warstwy".

m = folium.Map()

rectangle_result = loads(cursor.execute(rectangle).fetchall())

st = {'fillColor': 'blue', 'color': 'red'}

l = []
for row in rectangle_result:
    g = geojson.Feature(geometry=row[0], properties={})
    l.append(g)

feature_collection = geojson.FeatureCollection(l)
folium.GeoJson(feature_collection, style_function=lambda x:st).add_to(m)  

m
```

Zrzut ekranu:

![alt text](image.png)

Komentarz:
Prostkąt na mapie Stanów Zjednoczonych.


Użyj funkcji SDO_FILTER

```sql
SELECT state, geom FROM us_states
WHERE sdo_filter (geom,
sdo_geometry (2003, 8307, null,
sdo_elem_info_array (1,1003,3),
sdo_ordinate_array ( -117.0, 40.0, -90., 44.0))
) = 'TRUE';
```

Zwróć uwagę na liczbę zwróconych wierszy (16)


> Wyniki, zrzut ekranu, komentarz

```python
filter_query = """SELECT state, sdo_util.to_wktgeometry(geom) FROM us_states
WHERE sdo_filter (geom,
sdo_geometry (2003, 8307, null,
sdo_elem_info_array (1,1003,3),
sdo_ordinate_array ( -117.0, 40.0, -90., 44.0))
) = 'TRUE'"""

filter_result = cursor.execute(filter_query).fetchall()

filter_geom = [loads(res[1]) for res in filter_result]

st2 = {'fillColor': 'white', 'color': 'blue'}

l2 = []
for row in filter_geom:
    g = geojson.Feature(geometry=row, properties={})
    l2.append(g)

feature_collection2 = geojson.FeatureCollection(l2)

folium.GeoJson(feature_collection2, style_function=lambda x:st2).add_to(m)  

m 
```

Zrzut ekranu:

![alt text](image-1.png)


Komentarz: Stany (us_states), których obszary mają część wspólną ze wskazaną geometrią (prostokątem) uzyskane zapytaniem wykorzystującym
`SDO_FILTER`.

Wydaje się, jakby zapytanie wykorzystujące funkcję `SDO_FILTER` błędnie zwracało dwa stany nieprzecinające się z prostkątem. Takie działanie `SDO_FILTER` wyjaśnia dokumentacja Oracla, wg któej funkcja `SDO_FILTER` wykonuje tylko operację filtracji pierwotnej. Jest to szybka operacja, która może zwrócić niektóre fałszywe pozytywy (obiekty, które są identyfikowane jako interakcje, ale które faktycznie nie oddziałują). Aby uzyskać dokładne wyniki, można użyć operacji filtracji wtórnej, takiej jak SDO_RELATE.

Użyj funkcji  SDO_ANYINTERACT

```sql
SELECT state, geom FROM us_states
WHERE sdo_anyinteract (geom,
sdo_geometry (2003, 8307, null,
sdo_elem_info_array (1,1003,3),
sdo_ordinate_array ( -117.0, 40.0, -90., 44.0))
) = 'TRUE';
```

Porównaj wyniki sdo_filter i sdo_anyinteract

Pokaż wynik na mapie


> Wyniki, zrzut ekranu, komentarz

```python
anyinteract_query = """SELECT state, sdo_util.to_wktgeometry(geom) FROM us_states
WHERE sdo_anyinteract (geom,
sdo_geometry (2003, 8307, null,
sdo_elem_info_array (1,1003,3),
sdo_ordinate_array ( -117.0, 40.0, -90., 44.0))
) = 'TRUE'"""

anyinteract_result = cursor.execute(anyinteract_query).fetchall()

anyinteract_geom = [loads(res[1]) for res in anyinteract_result]

st3 = {'fillColor': 'green', 'color': 'green'}

l3 = []
for row in anyinteract_geom:
    g = geojson.Feature(geometry=row, properties={})
    l3.append(g)

feature_collection3 = geojson.FeatureCollection(l3)

folium.GeoJson(feature_collection3, style_function=lambda x:st3).add_to(m)  

m 
```

Zrzut ekranu:

![alt text](image-2.png)

```python
print(f"Liczba stanów zwróconych przez SDO_FILTER: {len(filter_result)}")
print(f"Liczba stanów zwróconych przez SDO_ANYINTERACT: {len(anyinteract_result)}")
```
Liczba stanów zwróconych przez SDO_FILTER: 16\
Liczba stanów zwróconych przez SDO_ANYINTERACT: 14

Komentarz: Stany (us_states), których obszary mają część wspólną ze wskazaną geometrią (prostokątem) uzyskane zapytaniem wykorzystującym
`SDO_ANYINTERACT` nałożone na mapę stanów, których obszary mają część wspólną ze wskazaną geometrią (prostokątem) uzyskane zapytaniem wykorzystującym `SDO_FILTER`.

Wygląda na to, że zapytanie wykorzystujące funkcję `SDO_ANYINTERACT` poprawnie zwraca stany mające część wspólną z prostokątem.


# Zadanie 3

Znajdź wszystkie parki (us_parks) których obszary znajdują się wewnątrz stanu Wyoming

Użyj funkcji SDO_INSIDE

```sql
SELECT p.name, p.geom
FROM us_parks p, us_states s
WHERE s.state = 'Wyoming'
AND SDO_INSIDE (p.geom, s.geom ) = 'TRUE';
```

W przypadku wykorzystywania narzędzia SQL Developer, w celu wizualizacji na mapie użyj podzapytania

```sql
SELECT pp.name, pp.geom  FROM us_parks pp
WHERE id IN
(
    SELECT p.id
    FROM us_parks p, us_states s
    WHERE s.state = 'Wyoming'
    and SDO_INSIDE (p.geom, s.geom ) = 'TRUE'
)
```

![img_7.png](img_7.png)

```sql
SELECT p.name, p.geom
FROM us_parks p
WHERE SDO_INSIDE (p.geom, (SELECT geom FROM us_states WHERE state = 'Wyoming')) = 'TRUE';
```

```sql
SELECT state, geom FROM us_states
WHERE state = 'Wyoming'
```

```sql
SELECT p.name, p.geom
FROM us_parks p, us_states s
WHERE s.state = 'Wyoming'
AND SDO_ANYINTERACT (p.geom, s.geom ) = 'TRUE';
```

W celu wizualizacji użyj podzapytania


```sql
SELECT p.name, p.geom
FROM us_parks p
WHERE SDO_ANYINTERACT (p.geom, (SELECT geom FROM us_states WHERE state = 'Wyoming')) = 'TRUE';

```
![img_8.png](img_8.png)

Stan Wyoming jest domem dla kilku znanych i dużych parków narodowych. Jednym z największych i najbardziej znanych parków narodowych w Wyoming jest ```Yellowstone National Park```, który jest również jednym z najstarszych parków narodowych na świecie. Innymi znanymi parkami narodowymi w Wyoming są ``Grand Teton National Park`` oraz ``Devils Tower National Monument``.

# Zadanie 4

Znajdź wszystkie jednostki administracyjne (us_counties) wewnątrz stanu New Hampshire

```sql
SELECT c.county, c.state_abrv, c.geom
FROM us_counties c, us_states s
WHERE s.state = 'New Hampshire'
AND SDO_RELATE ( c.geom,s.geom, 'mask=INSIDE+COVEREDBY') = 'TRUE';

SELECT c.county, c.state_abrv, c.geom
FROM us_counties c, us_states s
WHERE s.state = 'New Hampshire'
AND SDO_RELATE ( c.geom,s.geom, 'mask=INSIDE') = 'TRUE';

SELECT c.county, c.state_abrv, c.geom
FROM us_counties c, us_states s
WHERE s.state = 'New Hampshire'
AND SDO_RELATE ( c.geom,s.geom, 'mask=COVEREDBY') = 'TRUE';
```

W przypadku wykorzystywania narzędzia SQL Developer, w celu wizualizacji danych na mapie należy użyć podzapytania (podobnie jak w poprzednim zadaniu)



> Wyniki, zrzut ekranu, komentarz

```python

```

# Zadanie 5

Znajdź wszystkie miasta w odległości 50 mili od drogi (us_interstates) I4

Pokaż wyniki na mapie

```sql
SELECT c.city, c.state_abrv, c.location 
FROM us_cities c
WHERE ROWID IN 
( 
    SELECT c.rowid
    FROM us_interstates i, us_cities c 
    WHERE i.interstate = 'I4'
    AND SDO_WITHIN_DISTANCE (c.location, i.geom, 'distance=50 unit=mile') = 'TRUE'
);
```
![img_9.png](img_9.png)

Ta autostrada biegnie przez stan Floryda, przechodząc przez miasta takie jak Tampa, Orlando i St. Petersburg.

Dodatkowo:

a)     Znajdz wszystkie jednostki administracyjne przez które przechodzi droga I4

b)    Znajdz wszystkie jednostki administracyjne w pewnej odległości od I4

c)     Znajdz rzeki które przecina droga I4

d)    Znajdz wszystkie drogi które przecinają rzekę Mississippi

e)    Znajdz wszystkie miasta w odlegości od 15 do 30 mil od drogi 'I275'

f)      Itp. (własne przykłady)


```sql
--- Znajdź wszystkie jednostki administracyjne przez które przechodzi droga I4

SELECT s.*
FROM us_states s, us_interstates i
WHERE SDO_RELATE(s.geom, i.geom, 'mask=INSIDE') = 'TRUE' AND i.interstate = 'I4';

--- Znajdź wszystkie jednostki administracyjne w pewnej odległości od I4

SELECT s.*
FROM us_states s, us_interstates i
WHERE SDO_WITHIN_DISTANCE(s.geom, i.geom, 'distance=50 unit=mile') = 'TRUE' AND i.interstate = 'I4';

--- Znajdź rzeki które przecina droga I4

SELECT r.*
FROM us_rivers r, us_interstates i
WHERE SDO_RELATE(r.geom, i.geom, 'mask=ANYINTERACT') = 'TRUE' AND i.interstate = 'I4';

--- Znajdź wszystkie drogi które przecinają rzekę Mississippi

SELECT i.*
FROM us_interstates i, us_rivers r
WHERE SDO_RELATE(i.geom, r.geom, 'mask=ANYINTERACT') = 'TRUE' AND r.river_name = 'Mississippi';

--- Znajdź wszystkie miasta w odległości od 15 do 30 mil od drogi 'I275'

SELECT c.*
FROM us_cities c, us_interstates i
WHERE SDO_WITHIN_DISTANCE(c.location, i.geom, 'distance=15 unit=mile') = 'TRUE' 
AND SDO_WITHIN_DISTANCE(c.location, i.geom, 'distance=30 unit=mile') = 'TRUE'
AND i.interstate = 'I275';

--- Wlasne przykłady
--- Znajdź wszystkie parki narodowe w odległości 25 mil od drogi krajowej nr 1.

SELECT p.*
FROM us_parks p, us_highways h
WHERE SDO_WITHIN_DISTANCE(p.geom, h.geom, 'distance=25 unit=mile') = 'TRUE' AND h.highway_name = 'US-1';

--- Znajdź wszystkie lotniska w obrębie stanu Kalifornia.

SELECT a.*
FROM us_airports a, us_states s
WHERE SDO_CONTAINS(s.geom, a.location) = 'TRUE' AND s.state_name = 'California';

```


# Zadanie 6

Znajdz 5 miast najbliższych drogi I4

```sql
SELECT c.city, c.state_abrv, c.location
FROM us_interstates i, us_cities c 
WHERE i.interstate = 'I4'
AND sdo_nn(c.location, i.geom, 'sdo_num_res=5') = 'TRUE';
```

>Wyniki, zrzut ekranu, komentarz

```sql
--  ...
```


Dodatkowo:

a)     Znajdz kilka miast najbliższych rzece Mississippi

b)    Znajdz 3 miasta najbliżej Nowego Jorku

c)     Znajdz kilka jednostek administracyjnych (us_counties) z których jest najbliżej do Nowego Jorku

d)    Znajdz 5 najbliższych miast od drogi  'I170', podaj odległość do tych miast

e)    Znajdz 5 najbliższych dużych miast (o populacji powyżej 300 tys) od drogi  'I170'

f)      Itp. (własne przykłady)


> Wyniki, zrzut ekranu, komentarz
> (dla każdego z podpunktów)

```sql
--  ...
```


# Zadanie 7

Oblicz długość drogi I4

```sql
SELECT SDO_GEOM.SDO_LENGTH (geom, 0.5,'unit=kilometer') length
FROM us_interstates
WHERE interstate = 'I4';
```
![img_10.png](img_10.png)

Długość drogi to 212.26 kilometrów.

Dodatkowo:

a)     Oblicz długość rzeki Mississippi

b)    Która droga jest najdłuższa/najkrótsza

c)     Która rzeka jest najdłuższa/najkrótsza

d)    Które stany mają najdłuższą granicę

e)    Itp. (własne przykłady)



```sql
--- Oblicz długość rzeki Mississippi
SELECT SUM(SDO_GEOM.SDO_LENGTH(geom, 0.5, 'unit=kilometer')) AS total_length
FROM us_rivers
WHERE SYSTEM = 'Mississippi';
```

![img_11.png](img_11.png)

```sql
--- Oblicz długość rzeki Mississippi
SELECT SUM(SDO_GEOM.SDO_LENGTH(geom, 0.5, 'unit=kilometer')) AS total_length
FROM us_rivers
WHERE SYSTEM = 'Mississippi';

```

```sql
-- Która droga jest najdłuższa/najkrótsza

-- Najdłuższa droga
SELECT interstate, SDO_GEOM.SDO_LENGTH(geom, 0.5, 'unit=kilometer') AS length
FROM us_interstates
ORDER BY length DESC
FETCH FIRST 1 ROW ONLY;

-- Najkrótsza droga
SELECT interstate, SDO_GEOM.SDO_LENGTH(geom, 0.5, 'unit=kilometer') AS length
FROM us_interstates
ORDER BY length
FETCH FIRST 1 ROW ONLY;

```
- Najdłuższa droga:

![img_14.png](img_14.png)

to I90, jej długość to 4290.64 km.

- Najkrótsza droga:

![img_13.png](img_13.png)

to I564, jej długość to 0.462km


![img_12.png](img_12.png)

```sql
-- Najdłuższa rzeka
SELECT river_name, SDO_GEOM.SDO_LENGTH(geom, 0.5, 'unit=kilometer') AS length
FROM us_rivers
ORDER BY length DESC
FETCH FIRST 1 ROW ONLY;

-- Najkrótsza rzeka
SELECT river_name, SDO_GEOM.SDO_LENGTH(geom, 0.5, 'unit=kilometer') AS length
FROM us_rivers
ORDER BY length
FETCH FIRST 1 ROW ONLY;
```

- Najdłuższa rzeka:

![img_15.png](img_15.png)

to St. Lawrence, jej długość to 6950.91 km.

- Najkrótsza rzeka:

![img_16.png](img_16.png)

to również St. Lawrence, jej długość to 1,16 km.

```sql
--- Które stany mają najdłuższą granicę
SELECT state, SDO_GEOM.SDO_LENGTH(geom, 0.5, 'unit=kilometer') AS length
FROM us_states
ORDER BY length DESC
FETCH FIRST 1 ROW ONLY;
```
![img_17.png](img_17.png)

Stan Alaska ma najdłuższą granicę, któa wynosi 26138.37 km.


Oblicz odległość między miastami Buffalo i Syracuse

```sql
SELECT SDO_GEOM.SDO_DISTANCE ( c1.location, c2.location, 0.5) distance
FROM us_cities c1, us_cities c2
WHERE c1.city = 'Buffalo' and c2.city = 'Syracuse';
```

Odległość między miastami Buffalo i Syracuse to 222184,61 km


Dodatkowo:

a)     Oblicz odległość między miastem Tampa a drogą I4

b)    Jaka jest odległość z między stanem Nowy Jork a  Florydą

c)     Jaka jest odległość z między miastem Nowy Jork a  Florydą

d)    Podaj 3 parki narodowe do których jest najbliżej z Nowego Jorku, oblicz odległości do tych parków

e)    Przetestuj działanie funkcji

a.     sdo_intersection, sdo_union, sdo_difference

b.     sdo_buffer

c.     sdo_centroid, sdo_mbr, sdo_convexhull, sdo_simplify

f)      Itp. (własne przykłady)


```sql
--- Oblicz odległość między miastem Tampa a drogą I4
SELECT SDO_GEOM.SDO_DISTANCE(c.location, i.geom, 0.5) AS distance_km
FROM us_cities c, us_interstates i
WHERE c.city = 'Tampa' AND i.interstate = 'I4';

```

![img_18.png](img_18.png)

Odległość między miastem Tampa a drogą I4 to 3103.91 km
```sql
--- Jaka jest odległość z między stanem Nowy Jork a  Florydą
SELECT SDO_GEOM.SDO_DISTANCE(s1.geom, s2.geom, 0.5) AS distance_km
FROM us_states s1, us_states s2
WHERE s1.state_name = 'New York' AND s2.state_name = 'Florida';

```

![img_19.png](img_19.png)

Odległość między stanem Nowy Jork a Florydą to 1256583.87 km

```sql
--- Jaka jest odległość z między miastem Nowy Jork a  Florydą
SELECT SDO_GEOM.SDO_DISTANCE(c1.location, c2.location, 0.5) AS distance_km
FROM us_cities c1, us_cities c2
WHERE c1.city = 'New York' AND c2.city = 'Florida';

```

Odległość między Nowym Jorkiem a Florydą zależy od konkretnych miast w tych stanach. Na przykład, odległość między Nowym Jorkiem a Miami wynosi około 2,060 kilometrów w linii prostej. Jednak jeśli chodzi o odległość drogową, zazwyczaj jest to około około 2,060-2,090 kilometrów.

```sql
--- Podaj 3 parki narodowe do których jest najbliżej z Nowego Jorku, oblicz odległości do tych parków
SELECT p.name, SDO_GEOM.SDO_DISTANCE(c.location, p.geom, 0.5) AS distance_km
FROM us_cities c, us_parks p
WHERE c.city = 'New York'
ORDER BY distance_km
FETCH FIRST 3 ROWS ONLY;

```

![img_20.png](img_20.png)

3 parki narodowe do których jest najbliżej z Nowego Jorku to ```Institute Park```, ```Prospect Park``` oraz ```Thompkins Park```.

```sql
-- SDO_INTERSECTION: Zwraca część wspólną dwóch geometrii
SELECT SDO_GEOM.SDO_INTERSECTION(a.geom, b.geom, 0.005) AS intersection_geom
FROM geometry_table a, geometry_table b
WHERE a.id = 1 AND b.id = 2;

-- SDO_UNION: Zwraca sumę dwóch geometrii
SELECT SDO_GEOM.SDO_UNION(a.geom, b.geom, 0.005) AS union_geom
FROM geometry_table a, geometry_table b
WHERE a.id = 1 AND b.id = 2;

-- SDO_DIFFERENCE: Zwraca różnicę między dwoma geometriami
SELECT SDO_GEOM.SDO_DIFFERENCE(a.geom, b.geom, 0.005) AS difference_geom
FROM geometry_table a, geometry_table b
WHERE a.id = 1 AND b.id = 2;

```


```sql
-- SDO_BUFFER: Tworzy bufor wokół geometrii
SELECT SDO_GEOM.SDO_BUFFER(geom, 0.1, 0.005) AS buffer_geom
FROM geometry_table
WHERE id = 1;
```

```sql
-- SDO_CENTROID: Oblicza centroid dla geometrii
SELECT SDO_GEOM.SDO_CENTROID(geom, 0.005) AS centroid_geom
FROM geometry_table
WHERE id = 1;

-- SDO_MBR: Zwraca minimalny zewnętrzny prostokąt dla geometrii
SELECT SDO_GEOM.SDO_MBR(geom) AS mbr_geom
FROM geometry_table
WHERE id = 1;

-- SDO_CONVEXHULL: Tworzy otoczkę wypukłą wokół geometrii
SELECT SDO_GEOM.SDO_CONVEXHULL(geom, 0.005) AS convexhull_geom
FROM geometry_table
WHERE id = 1;

-- SDO_SIMPLIFY: Uproszcza geometrię
SELECT SDO_GEOM.SDO_SIMPLIFY(geom, 0.005) AS simplified_geom
FROM geometry_table
WHERE id = 1;

```


# Zadanie 8

Wykonaj kilka własnych przykładów/analiz


>Wyniki, zrzut ekranu, komentarz

```sql
--  ...
```

Punktacja

|   |   |
|---|---|
|zad|pkt|
|1|0,5|
|2|1|
|3|1|
|4|1|
|5|3|
|6|3|
|7|6|
|8|4|
|razem|20|
