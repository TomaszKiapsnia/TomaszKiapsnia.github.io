---
layout: post
title:  "DQDD czyli najpierw jakość"
date:   2020-10-08
last_modified_at: 2020-10-08
categories: [DQDD,TDD,Quality,Testing]
tags: [DQDD,TDD,Quality,Testing]
---

Testowanie jest piękne. Daje natychmiastową odpowiedź, czy nasze działania są 
poprawne. Dobrze napisane testy sprawdzają wszystko od A do Z, od góry do dołu,
bez mozolnego przeglądania wyników naszych funkcji czy procedur i liczenia na
łut szczęścia, że akurat trafimy na wadliwy przypadek. 

# Wszystko ma swoją cenę
Koszt napisania zautomatyzowanych testów jest porównywalny do jednorazowego stworzenia i wykonania testów manualnych. Tyle, że część wykonawczą testów manualnych realizujemy
w czasie projektu przynajmniej kilkukrotnie. Zysk z automatyzacji testów jest więc ogromny, tym większy, im dłuższy jest projekt. 
Pozwala też cykl developmentu, wykrycia błędów i ich poprawy. Ułatwia to pracę, gdyż nie trzeba po tygodniu, kiedy zrobiło się
już multum innych rzeczy, wracać do zadania o którym już się zapomniało, odświeżać sobie wiedzę i fixować nieszczęsny kod. 
Kolejną rzeczą jest poczucie bezpieczeństwa i pewności siebie programisty, który robiąc zmiany w kodzie, tym bardziej w kodzie który widzi pierwszy raz na oczy
lub wrócił do niego po długim czasie, ma szybką odpowiedź, że nic nie zepsuł i dalej wszystko jest ok.
# Akcja reakcja 
Ważnym aspektem testowania jest czas ich wykonania. Im krócej testy się wykonują, tym lepiej.
Pozwala to od razu uzyskać informację, czy nasze zadanie jest wykonane dobrze i wprowadzić ewentualne poprawki jeszcze zanim przekażemy je dalej,
a my nie zmienimy jeszcze kontekst w głowie na inne zadanie. Natomiast zbyt długo trwające testy na tym etapie mogą spowodować, że 
się do nich zniechęcamy i istnieje duże prawdopodobieństwo, że zostaną zarzucone. 
 
# Skąd pomysł?
W klasycznym, obiektowym programowaniu najlepszą praktyką testowania własnego kodu jest Test Driven Development (TDD). Zakłada pracę w bardzo krótkich cyklach, 
rzędu 1-2 minuty, w których to postępujemy w 3 krokach:
1. piszemy test którego nasz program nie zalicza (np. sprawdza czy da się wywołać konstruktor klasy) (faza red)
2. piszemy minimalną ilość kodu do zaliczenia testu (np. definiujemy samą klasę, bez żadnej logiki, tylko pusty konstruktor) (faza green)
3. refactorujemy nasz kod (faza refactor)

TDD zakłada, że w danym momencie tylko jeden test może być niezaliczony, więc nie możemy przejść do kolejnego testu, a i co za tym idzie kolejnej części kodu, dopóki nie
zaliczmy aktualnego testu.


# DWH i BI
W świecie hurtowni danych oraz systemów BI, szczególnie po stronie bazy danych kiedy piszemy customowe agregacje danych, trudno jest jeden do jednego 
przełożyć TDD. Można natomiast zastosować technikę, którą nazwałem Data Quality Driven Development (DQDD). 

DQDD ma zastosowanie wtedy, kiedy procesujemy dane ze źródła do miejsca docelowego. Czyli praktycznie zawsze. Zwracając szczególną 
uwagę na transformacje oraz złączenia z innymi tabelami podczas procesowania danych, możemy spodziewać się kilku błędów jakie
mogą zdarzyć się podczas programowania takiego przepływu. Może nam zabraknąć warunku złączenia lub może pojawić się warunek nadmiarowy powodując nadmiarową filtrację.
Możemy zduplikować dane, bądź też wręcz wytworzyć iloczyn kartezjański. Powodów tego może być sporo, zaczynąc od ludzkiego błędu literówki, skopiowanie aliasu tabeli
z lewej i prawej strony warunku złączenia dający zawsze warunek prawdziwy, przez błąd logiczny podczas formułowania warunków złączeń, kończąc na defekcie w danych, a lista i tak nie jest pełna.  

Istotnym wymogiem do zastosowania metodyki DQDD jest możliwość szybkiego załadowania danych ze źródła do tabeli docelowej.
Dodatkowe skrócenie czasu od napisania kodu do informacji czy jest poprawny, może zapewnić wytworzenie odpowiedniego
zestawu danych testowych. Przygotowanie minimalnego zestawu danych testujących wszystkie możliwe przypadki zapewnia minimalny czas przetwarzania, a dodatkowo pozwala na 100% pokrycie
przypadkami testowymi (analogia do pokrycia kodu). Nawet biorąc dane produkcyjne, nigdy nie mamy pewności, czy pojawią się wszystkie skrajne sytuacje. Przetwarzanie pełnych danych jest zawsze bardziej czasochłonne i trudniejsze 
w późniejszym debugowaniu, kiedy okazuje się, że wynik nie jest poprawny. Przygotowanie odpowiedniego zestawu danych ułatwia również testowanie warstwy UI. Gdy 
wystawiamy już API dla aplikacji frontendowej, to wiemy jakich wyników powinniśmy się spodziewać. W przypadku danych produkcyjnych spodziewana odpowiedź API jest inna
za każdym razem, co utrudnia przetestowanie wszystkich metryk. 

# Przykład
Dalsza część tekstu zakłada, że naszym źródłem danych jest model wymiarowy (dimensional model - Kimball), a naszym celem jest stworzenie warstwy raportowej, która będzie udostępniana
do aplikacji frontendowej z predefiniowanymi raportami. Aby jak najszybciej zwracać takie dane, najlepiej jest je zagregować do postaci w jakiej będą prezentowane w UI, żeby żadna logika nie musiała być procesowana
w czasie od kliknięcia przez użytkownika w aplikacji do czasu zwrócenia danych. Przetestujemy sobie stworzenie jednego takiego agregatu.
Jako narzędzie do automatyzacji testów użyjemy FitNesse (http://docs.fitnesse.org/)


Model źródłowy:
![Model źródłowy](/assets/images/dqdd/source_model.png)


W fakcie f_pharma_sales jest sprzedaż produktów w aptekach wyrażona w dollarach (sales_dollars). 
Z faktu jest odniesienie do wymiaru czasowego d_date, w którym są dodatkowe indykatory czy dana data należy
do aktualnego miesiąca (curr_mth_ind), czy należy do ostatnich 3 miesięcy (roll_qtr_ind) oraz czy należy do danego roku (ytd_ind). 
Wymiar produktowy d_product daje informacje o producencie, marce oraz typie. 
Wymiar sklepu (czy apteki) d_shop zawiera dane geograficzne gdzie dokonano sprzedaży. 


Wymaganiem jest wyświetlenie sum sprzedaży na poziomie stanów, dla różnych typów produktów w ostatnich 3 miesiącach (rolling quarter). Dla stanów i typów 
z zerową sprzedażą należy zwrócić 0.

Tworzymy więc tabelę docelową r_states_prod_types

![Model docelowy](/assets/images/dqdd/target_model.png)

# Przypadki testowe

Aby szybko uzyskać odpowiedź, czy nasz agregat spełnia wszystkie wymagania, zdefiniujemy sobie najpierw przypadki testowe:

1. Istnieje każda kombinacja stanu i typu produktu
2. Każda kombinacja stanu i typu produktu może wystąpić tylko raz
3. Suma całkowita w tabeli docelowej r_states_prod_types musi być zgodna z kwartalną sumą całkowitą w tabeli źródłowej f_pharma_sales
4. Sumy poszczególnych typów produktów w r_states_prod_types muszą być zgodne z kwartalną sumą typów w f_pharma_sales
5. Sumy poszczególnych stanów w r_states_prod_types muszą być zgodne z kwartalną sumą stanów w f_pharma_sales


Do takich wymagań tworzymy sobie również zestaw danych testowych, które pozwolą nam przetestować każdy możliwy przypadek:

### d_date 
![dane d_date](/assets/images/dqdd/d_date.png)
Wstawiamy tylko dwie daty, jedna wpadająca w nasz rolujący kwartał, druga nie.

### d_product 
![dane d_product](/assets/images/dqdd/d_product.png)
Mamy 3 typy poruktów: płyny, tabletki, spray.

### d_shop
![dane d_shop](/assets/images/dqdd/d_shop.png)
Dwa sklepy w różnych stanach.


### f_pharm_sales
Do f_pharma_sales wstawiamy dane tak, aby uzyskać wszystkie przypadki sprzedaży dodatniej i negatywnej w każdym typie produktu oraz stanie. 
Dodatkowo dodajemy testową wartość z poza naszej wymaganej daty, aby sprawdzić czy tylko kwartalne dane są załadowane. 

![dane f_pharma_sales](/assets/images/dqdd/f_pharma_sales.png )
![dane f_pharma_sales przetłumaczone](/assets/images/dqdd/f_pharma_sales_trans.png )

# komora maszyny testującej jest pusta...
Przechodząc od słów do czynów, zaczynamy pracę z naszym zapytaniem ładującym dane oraz testami. 

### test#1
Pierwszy test który definiujemy, to czy istnieje każda kombinacja typu produktu i stanu. 

zapytanie na danych źródłowych:
```
    SELECT prd.type, shp.state
      FROM d_product prd
CROSS JOIN d_shop shp;
```
Zapytanie na danych docelowych:
```
    SELECT prod_type, shop_state
      FROM r_states_prod_types;
```	  
Definicja testu w FitNesse:
![definicja w fitness](/assets/images/dqdd/1st_test_def.png)

Pierwsze uruchomienie:
![pierwszy test bez kodu](/assets/images/dqdd/1st_test_fail.png)
Oczywiście test oblany, gdyż w tabelce docelowej nie ma danych. 

Tworzymy najmniejszy możliwy kod ładujący tabelę docelową, aby spełnić ten test. 
```
CREATE OR REPLACE PROCEDURE load_r_states_prod_types IS

BEGIN
    EXECUTE IMMEDIATE 'TRUNCATE TABLE r_states_prod_types';
   
    INSERT INTO r_states_prod_types (prod_type,shop_state)
        SELECT prd.type AS prod_type
             , shp.state AS shop_state
          FROM d_product prd
    CROSS JOIN d_shop shp;
    
    COMMIT;
END load_r_states_prod_types;
```
Uruchamiam procedurę:
```
EXEC load_r_states_prod_types;
```
uruchamiam test:
![pierwszy zielony test](/assets/images/dqdd/1st_test_pass.png)

BRAWO!

### test#2
Piszemy drugi test, który jest w zasadzie uzupełnieniem pierwszego i nie będziemy musieli dodatkowo pisać kodu, natomiast dla pełności
DQ sprawdzamy, czy nie generujemy duplikatów. Poniższe zapytanie powinno nam zwrócić wartość 0:
```
SELECT COUNT(*)
  FROM (SELECT COUNT(*)
          FROM r_states_prod_types
      GROUP BY prod_type, shop_state
        HAVING COUNT(*) > 1)
```		
Odpalamy test w fitnesse:
2nd test pass.png
![drugi zielony test](/assets/images/dqdd/2nd_test_pass.png)

### test#3
Kolejny test to sprawdzenie, czy suma całkowita w tabeli docelowej jest równa sumie kwartalnej w tabeli źródłowej. Test na tabeli źródłowej:
```
    SELECT SUM(sales_dollars) AS sales_dollars
      FROM f_pharma_sales fhs
INNER JOIN d_date dat
        ON dat.date_id = fhs.date_id
       AND dat.roll_qtr_ind = 'Y';
```	   
Test na tabeli docelowej:
```
SELECT SUM(sales_dollars) AS sales_dollars
  FROM r_states_prod_types;
```
![trzeci czerwony test](/assets/images/dqdd/3rd_test_fail.png)

Dopisujemy minimalną ilość kodu (a w zasadzie modyfikujemy nasze zapytanie), aby spełnić wymaganie:
```
         SELECT prd.type AS prod_type
              , shp.state AS shop_state
              , SUM(fph.sales_dollars) AS sales_dollars
           FROM d_product prd
     CROSS JOIN d_shop shp
LEFT OUTER JOIN f_pharma_sales fph
             ON fph.product_id = prd.product_id
            AND fph.shop_id = shp.shop_id
     INNER JOIN d_date dat
             ON dat.date_id = dat.date_id
            AND dat.roll_qtr_ind = 'Y'
       GROUP BY prd.type
              , shp.state; 
```			  
Odpalamy test:
![trzeci czerwony test#2](/assets/images/dqdd/3rd_test_second_fail.png)

I okazuje się, że testu nie zaliczyliśmy, a otrzymana wartość jest bardzo mocno na minusie. 
Mając teraz w głowie nasze przygotowane dane testowe wiemy, że z jakiegoś powodu największa ujemna wartość która nie jest w naszym kwartale, została zawarta w obliczeniach.
Wracamy do naszego kodu i okazuje się, że przez przypadek daliśmy alias z tej samej tabeli zarówno z lewej jak i z prawej strony:
```
     INNER JOIN d_date dat
             ON dat.date_id = dat.date_id
```			 
Szybka poprawka:
```
 ON dat.date_id = fph.date_id
```
Przeładowanie danych i test jest zaliczony:
![trzeci zielony test](/assets/images/dqdd/3rd_test_pass.png)

Ostatnie dwa testy DQ do spełnienia zostawiam jako ćwiczenie dla chętnych. 

# Zawsze testować!
Okazało się, że nawet dla tak prostego przypadku, sumiennie pisząc test przed napisaniem kodu mamy 
natychmiastową odpowiedź, czy nasz kod jest poprawny czy nie. Za każdym razem również należy uruchamiać wszystkie testy dla 
tworzonej części tak, aby mieć pewność, że nie zepsuło się poprzednich kroków:
![all good](/assets/images/dqdd/all_tests.png)

Część z tych testów może być również później wykorzystana jako testy akceptacyjne. Dodatkowo możemy ich użyć jako sprawdzenia DQ już na samych danych produkcyjnych,
kiedy przed wypchnięciem danych do aplikacji klienta mamy jeszcze czas na finale testy.

Stosowanie DQDD daje ogromny zysk w porównaniu do skrajnego przypadku nietestowania lub przypadku testowania manualnego.
I to zysk potrójny. Raz: sprawdzając od razu swój kod, co redukuje czas na poprawianie błędów w późniejszych fazach. Dwa: pisząc od razu testy,
które mogą być wykorzystane w fazie produkcyjnej jako finalne DQ. Trzy: redukując czas powtarzanych w kółko testów manualnych. 
Należy jednak pamiętać o tym, aby stworzenie testów i ich wykonanie było szybkie, łatwie i przyjemne. Nie możemy czekać godzinami, aż testy dadzą
nam odpowiedź, bo mają testować względnie małe zmiany. Pomogą w tym minimalne dane testowe, które spełnią każdy przypadek testowy, oraz względnie proste 
narzędzie do uruchamiania testów, jak FitNesse. 
Polecam, zachęcam, pozdrawiam. 

Tomek