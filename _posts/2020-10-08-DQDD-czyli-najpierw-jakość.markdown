---
layout: post
title:  "DQDD czyli najpierw jakość"
date:   2020-10-08
last_modified_at: 2020-10-08
categories: [DQDD,TDD,Quality,Testing]
tags: [DQDD,TDD,Quality,Testing]
---

Testowanie jest piękne. Daje natychmiastową odpowiedź, czy nasze działania są poprawne. Dobrze napisane testy sprawdzają wszystko od A do Z, od góry do dółu,
bez mozolnego przeglądania wyników naszych funkcji czy procedur i liczenia na łut szczęścia, że akurat trafimy na wadliwy przypadek. 
Koszt napisania testów jest porównywalny do jednorazowego stworzenia i wykonania testów manualnych. Tyle że część wykonawczą testów manualnych realizujemy
w czasie projektu przynajmniej kilkukrotnie. Zysk z automatomatyzacji testów jest więc ogromny, tym większy im dłuższy jest projekt. 
Pozwala też skrócić pętle sprzężenia zwrotnego od developmentu do wykrycia błędów i ich poprawy. Ułatwia to pracę, gdyż nie trzeba po tygodniu, kiedy zrobiło się
już multum innych rzeczy, wracać do zadania o którym już się zapomniało, odświeżać sobie wiedzę i fixować nieszczęsny kod. 
Kolejną rzeczą jest poczucie bezpieczeństwa i pewności siebie programisty, który robiąc zmiany w kodzie, tym bardziej w kodzie który widzi pierwszy raz na oczy
lub wrócił do niego po długim czasie, ma szybką odpowiedź, że nic nie zepsuł i dalej wszystko jest ok. 
Ważnym aspektem testowania jest krótka pętla sprzężenia zwrotnego. Im krótsza, tym lepsza. Tety powinny móc wykonać się w max 5 minut.
Pozwala to od razu uzyskać informację, czy nasze zadanie jest wykonane dobrze i wprowadzić ewentualne poprawki, jeszcze zanim przekażemy je dalej,
a my zmienimy już kontekst w głowie na inne zadanie. 
 
W klasycznym, obiektowym programowaniu najlepszą praktyką testowania własnego kodu jest Test Driven Development (TDD). Zakłada pracę w bardzo krótkich cyklach, 
rzędu 1-2 minut, w których to postępujemy w 3 krokach:
1. piszemy test którego nasz program nie zalicza (np. sprawdza czy da się wywołać konstruktor klasy)
2. piszemy minimalną ilość kodu do zaliczenia testu (np. definiujemy samą klasę, bez żadnej logiki, tylko nazwa)
3. uruchamiamy test który powinien przejść

TDD zakłada, że w danym momencie tylko jeden test może być niezaliczony. Pracujemy więc w cyklu test -> kod -> zaliczenie -> kolejny test -> kod -> zaliczenie -> itd...

W świecie hurtowni danych oraz systemów BI, szczególnie po stronie bazy danych kiedy piszemy customowe agregacje danych trudno jest jeden do jednego 
przełożyć TDD, natomiast można zastosować technikę którą nazwałem Data Quality Driven Development (DQDD). 

DQDD ma zastosowanie wtedy, kiedy procesujemy dane z jakiegoś źródła, do miejsca docelowego. Czyli praktycznie zawsze. Zwracając szczególną 
uwagę na transformacje oraz złączenia z innymi tabelami podczas procesowania danych, możemy spodziewać się kilku błędów jakie
mogą zdażyć się podczas programowania takiego przepływu. Może nam zabraknąć warunku złączenia lub może pojawić się warunek nadmiarowy powodując nadmiarową filtrację.
Możemy zduplikować dane, bądź też wręcz wytworzyć iloczyn kartezjański. Powodów tego może być sporo, zaczynąc od ludzkiego błędu literówki, skopiowanie aliasu tabeli
z lewej i prawej strony warunku złączenia dający zawsze warunek prawdziwy, przez błęd logiczny podczas formułowania warunków złączeń, kończąc na defekcie w danych, a lista i tak nie jest pełna.  

Istotnym aspektem zastosowania metodyki DQDD jest możliwość szybkiego załadowania danych ze źródła do tabeli docelowej.
Dodatkowe skrócenie pętli sprzężenia zwrotnego, gwarantującej nam odpowiedź czy nasz proces jest poprawny bądź nie, może zapewnić wytworzenie odpowiedniego
zestawu danych testowych. Przygotowanie minimalnego zestawu danych testujących wszystkie możliwe przypadki zapewnia minimalny czas przetwarzania, a dodatkowo zapewnia 100% pokrycie
przypadkami. Nawet biorąc dane produkcyjne, nigdy nie mamy pewności, czy pojawią się wszystkie skrajne przypadki. Dodatkowo przetwarzanie pełnych danych jest zawsze bardziej czasochłonne i trudniejsze 
w późniejszym debuggingu, kiedy okazuje się, że wynik nie jest poprawny. Przygotowanie takiego zestawu danych ułatwia również testowanie warstwy UI, kiedy 
wystawiamy już API dla aplikacji frontendowej i wiemy jakich wyników powinniśmy się spodziewać. W przypadku danych produkcyjnych spodziewana odpowiedź API jest inna
za każdym razem, co utrudnia przetestowanie wszystkich metryk. 


Dalsza część tekstu zakłada, że naszym źródłem danych jest model wymiarowych (dimensional model - Kimball), a naszym celem jest stworzenie warstwy raportowej która będzie udostępniana
do aplikacji frontendowej z predefiniowanymi raportami. Aby jak najszybciej zwracać takie dane, najlepiej jest je zagregować do wymaganej postaci, żeby żadna logika nie musiała być procesowana
w czasie od kliknięcia przez użytkownika w aplikacji do czasu zwrócenia danych. Przetestujemy sobie stworzenie jednego takiego agregatu. 


Model źródłowy:


source_png


Zakładamy że naszym faktem jest sprzedaż produktów w aptekach wyrażona w dollarach (sales_dollars). 
Od faktu jest odniesienie do wymiaru czasowego, w którym są dodatkowe indykatory czy dana data należy
do aktualnego miesiąca, czy należy do aktualnego rolującego kwartału oraz czy należy do danego roku (year to date). 
Wymiar produktowy daje informacje o producencie, marce oraz typie. 
Wymiar sklepu (czy apteki) zawiera dane geograficzne gdzie dokonano sprzedaży. 

Wymaganiem jest wyświetlenie sum sprzedaży na poziomie stanów, dla różnych typów produktów w ostatnich 3 miesiącach (rolling quarter). 

Aby szybko uzyskać odpowiedź, czy nasz agregat spełnia wszystkie wymagania, zaczniemy od stworzenia kilku przypadków testowych