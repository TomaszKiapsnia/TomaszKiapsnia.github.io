---
layout: post
title:  "[PL] Wersjonowanie wymiarów"
date:   2020-11-27
last_modified_at: 2020-11-27
categories: [DWH,Dimensions,Versioning]
tags: [DWH,Dimensions,Versioning]
---

#PROTEST!
Filmy mają swoje kolejne części, książki - wydania, oprogramowanie - wersje.
Zmieniając kod źródłowy oprogramowania, wersjonujemy go. Tworzymy nową wersję,
czyli zmieniamy jakąś część już istniejącego tworu. Natomiast jeśli zmieniamy dane w wymiarach,
to znaczy, że one się wolno zmieniają. Brzmi sensownie? Dla mnie nie dlatego tytuł tego postu
jest trochę moim sprzeciwem do nazwy Slowly Changing Dimensions (wolno zmieniające się wymiary). 
Jeśli ktoś widzi większy sens tej nazwy, zapraszam do kontaktu. Chętnie przyjmę każdy logiczny argument
za stosowaniem słowa Slowly (wolno) w tej nazwie poza argumentem, że tak się przyjeło.
Bo co to znaczy wolno? Czy dane mogą się zmieniać tylko raz na rok? Na kwartał? 
Nie mogą się zmieniać codziennie? Albo częściej? 

Ten post to trochę moja pokuta, bo stosowałem rozwiązania opisane w definicji, a w życiu nie połączyłem tego z hasłem WOLNO
zmieniających się wymiarów.

#Wymiar
Wymiar sam z siebie daje nam tłumaczenie danych faktowych. Dodaje nam kolejnych jego, właśnie, wymiarów. W tabeli faktowej
trzymamy odniesienie do wymiaru, a wszystkie dane niemetryczne, opisowe, odczytujemy z tabeli wymiarowej.
Dzięki wersjonowaniu wymiarów dodajemy sobie możliwości analizowania danych w różnych punktach historii.
Przykładem może być analiza historii sprzedaży w wymiarze geografii. Jeśli sklep w regionie A zrealizował część sprzedaży,
a później został przeniesiony do regionu B, to bez historii wymiarowej gdy wyciągniemy stan aktualny, dowiemy się tylko,
że sklep jest w regionie B. Czy analizując historię sprzedaży w regionach, powinniśmy zaliczyć całą sprzedaż sklepu do regionu B? 
Jak zawsze odpowiedź jest jedna - to zależy. Ale moja intuicja podpowiada, że nie. 

#Co z tym zrobić? 
Odpowiedź jest prosta - to zależy. A tak serio, to trzymać historię rekordów. Tę historię można trzymać na różne sposoby
i jest to dobrze zdefiniowane właśnie pod hasłem wolno zmieniających się wymiarów. Każdy sposób przetrzymywania
historii (albo jej braku) ma przypisany numer.


#Typ0
Zerowy typ nie definiuje żadnych akcji związanych z historią i przeważnie do tego typu należą dane statyczne. 
Dobrym przykładem jest wymiar czasu. Rok 2021 jest zawsze po 2020 (oby), a grudzień zawsze w grudniu. 
 
#Typ1
W typie pierwszym historia nie jest trzymana. Dane są po prostu nadpisane, a przeszłość zapomniana. 
Książka telefoniczna jest dobrym przykładem, kiedy w kolejnym wydaniu książki Pan Iksiński ma swój nowy numer, ale o starym 
już wszyscy zapomnieli.


#Typ2
W typie drugim trzymana jest cała historia. Jeśli rekord ma swoją kolejną wersję, dodawany jest nowy wiersz, oznaczany jako aktualny,
a stara wersja jest dezaktualizowana, ale nadal trzymana. Najczęściej stosuję się dwie kolumny z datami od kiedy i do kiedy
wiersz był aktualny. W przypadku wiersza aktualnego data do jest nullowa i tak możemy łatwo określić, które wiersze są aktualne. 

#Typ3
Kontrowersyjna sprawa. Jest to taki trochę pivot typu drugiego. Zamiast tworzyć nowy wiersz, tworzymy nową kolumnę, aby trzymać poprzednią
wartość. Co jeśli chcemy trzymać nie tylko poprzednią, a 2 wartości w tył? Co jeśli chcemy wersjonować wszystkie 200 kolumn w wymiarze? 
Na 10 wpisów do tyłu? Absolutnie odradzam. Powinno być zastosowane tylko jeśli są naprawdę dobre powody, aby nie stosować innych typów.

#Typ4 
Typ 4 to kombinacja typu 1 i typu 2. Tworzymy dwie tabele: jedną na wiersze aktualne, a drugą na wiersze historyczne
z datami od i do kiedy wiersze były aktualne i znajdowały się w pierwszej tabeli.
Mój faworyt, gdyż dba o czytelność kodu. Pracując na danych aktualnych, nie martwimy się o filtracje wierszy nieaktualnych.
Pracując z danymi historycznymi, filtrujemy odpowiednią datę. Zysk w stosunku do typu 2 jest przy pracy na danych aktualnych, a przy
historii i tak i tak musimy filtrować.

#Typ5
Typ 5 to metoda podziału bazowego wymiaru na te kolumny, które zmieniają się wolno (sic!) i te, które zmieniają się szybko.
Jeśli szybko zmieniające się dane można łatwo odseparować, oszczędzamy masę miejsca na trzymaniu w historii zawsze lub prawie zawsze 
tych samych danych w kolumnach z wolno zmieniającymi się danymi. Wtedy wolno zmieniające się dane utrzymujemy w typie 1, a szybko w typie 4.

#Typ6
Kombinacja (typ1) na kombinacji (typ2) na kombinacji (typ3). Zmieniając rekord, dodajemy nowy wiersz z datą od i datą do (typ2) i aktualną
wartość zmienionej kolumny wpisujemy też do kolumny historycznej (typ3). Aktualizujemy również wartość zmienianej kolumny we wszystkich historycznych wpisach (typ1).
Aby odzyskać więc jaka była wartość w punkcie w historii, wybieramy wiersz po dacie od i do (typ2) i sprawdzamy wartość
w kolumnie historycznej (typ3). Dodatkowo w każdym wierszu mamy stan aktualny (typ1).

#Podsumowując
Definicja jest dość kompletna. Koncept dość prosty, jednak warto to znać. Daje gotowe przepisy na wszelkie potrzeby. 
Klasyczny wzorzec projektowy w technikach modelowania danych. 
Polecam, zachęcam, pozdrawiam.

Tomek