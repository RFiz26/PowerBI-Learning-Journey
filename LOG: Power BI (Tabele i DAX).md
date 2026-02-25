# Cel notatki
Ta notatka to moje uporządkowanie kluczowych zagadnień Power BI i DAX
w trakcie procesu przebranżowienia na stanowisko Data Analyst / BI Developer.
Jeżeli są jakieś błędy z czasem będę go aktualizować

# I. Modele danych – Schemat Gwiazdy

Modele danych to zbiór tabel, które są powiązane między sobą i tworzą **mapy powiązań**. 

## Schemat Gwiazdy (Star Schema)
Jest to optymalny sposób powiązań tabel, oparty na strukturze, w której jedna centralna tabela jest otoczona przez powiązane z nią tabele wymiarów.



### 1. Tabela Faktów (Fact Table)
Jest to centralna tabela w modelu. Zawiera informacje o zdarzeniach (faktach), które wystąpiły w czasie.
* **Zawartość:** Miary liczbowe (cena, koszt, ilość) oraz klucze obce (ID) służące do powiązań z wymiarami.
* **Charakterystyka:** Posiada ogromną liczbę wierszy, ale mało kolumn (tzw. tabela "długa i chuda").

### 2. Tabele Wymiarów (Dimension Tables)
Stanowią "ramiona" modelu. Odpowiadają na pytania: *Kto? Co kupił? Gdzie? Kiedy? Jak?*
* **Przykłady:** Kalendarz (Czas), Produkty, Klienci, Lokalizacje.
* **Charakterystyka:** Mają dużo kolumn opisowych, ale znacznie mniej wierszy niż tabela faktów. Każdy rekord w tabeli wymiarów musi być unikalny (Klucz Główny).

## **Power BI jest zoptymalizowany pod ten układ.**

### Kluczowe zasady:
1. **Relacja (1:*) (Jeden do Wielu):** Filtr zawsze płynie od tabeli wymiaru (strona "1") do tabeli faktów (strona "*").
2. **Unikaj Płatków Śniegu (Snowflake):** To sytuacja, gdzie tabele wymiarów są połączone z innymi tabelami wymiarów zamiast bezpośrednio z faktami. 
    * *Dobra praktyka:* Scalić (rozpłaszczyć) takie tabele w jedną tabelę wymiarów dla lepszej wydajności.

---

# I.a Transformacja tabeli płaskiej w Schemat Gwiazdy

Jeśli dane wejściowe to jedna wielka tabela (**Flat Table**), należy dokonać **dekompozycji**, czyli rozbić ją na Tabelę Faktów i Tabele Wymiarów.

## Proces rozbijania tabeli (Krok po kroku)

### 1. Wydzielenie Tabel Wymiarów (w Power Query)
Z głównej tabeli tworzymy referencje i wyciągamy kolumny opisowe do nowych tabel:
* **Klienci:** Nazwa klienta, adres, e-mail, ID Klienta (można dodać funkcją: *Kolumna Indeksu*).
* **Produkty:** Nazwa produktu, kategoria, cena jednostkowa, ID Produktu.
* **Sposób Płatności:** Rodzaj płatności, ID Płatności.

> **Ważne:** W tabelach wymiarów należy użyć funkcji **"Usuń duplikaty"**. Każdy rekord (np. dany produkt) musi być unikalny.

### 2. Stworzenie Tabeli Faktów
To, co zostaje z tabeli głównej po wyodrębnieniu opisów. Zawiera:
* **Wartości liczbowe:** Ilość, cena całkowita, zysk.
* **Klucze obce (FK):** ID Klienta, ID Produktu, ID Płatności (służą do relacji).
* **Data:** Klucz do połączenia z tabelą kalendarza.

### 3. Tworzenie Tabeli Kalendarz (w Power BI / DAX)
Nie używamy daty bezpośrednio z tabeli faktów do filtrowania czasu. Tworzymy dedykowaną tabelę funkcją DAX:
`Kalendarz = CALENDAR(MIN('Fakty'[Data]), MAX('Fakty'[Data]))`
* Pozwala to na poprawne działanie funkcji **Time Intelligence** (np. porównania rok do roku).

### 4. Budowa relacji (Widok Modelu)
Łączymy tabele za pomocą kluczy ID:
* Przeciągamy klucz z Tabeli Wymiarów (strona **1**) do Tabeli Faktów (strona **\***).
* **Zasada:** Kierunek filtrowania powinien być jednokierunkowy (od wymiaru do faktu).

---

# II. Miary vs. Kolumny Obliczeniowe

## **Miary (Measures)**
Obliczenia tworzone za pomocą języka **DAX**. 
* Nie zajmują miejsca w pamięci modelu (obliczane "w locie").
* Są dynamiczne: reagują na filtry (Slicery) i kontekst raportu.

## **Kolumny obliczeniowe (Calculated Columns)**
Nowa kolumna dodana na stałe do tabeli (zajmuje pamięć RAM po odświeżeniu danych). Stosuje się je głównie do:
* Kategoryzowania danych (np. grup wiekowych: "18-25", "26-40").
* Tworzenia kolumn pomocniczych do sortowania.
* Wykonywania operacji wiersz po wierszu, gdy wynik musi być użyty jako filtr (Slicer) lub legenda.
* Przykład: `Kolumna_Zysk = 'Sprzedaż'[CENA] - 'Sprzedaż'[Koszt]`

---

# III. Język DAX i funkcja CALCULATE

## **DAX (Data Analysis Expressions)**
Język formuł Power BI. Większość obliczeń opiera się na **agregacjach**, czyli zamianie wielu wierszy w jedną wartość (np. suma, średnia).

### Podstawowe pojęcia DAX:
* `SUM(Tabela[Kolumna])` – suma wartości.
* `AVERAGE(Tabela[Kolumna])` – średnia.
* `DISTINCTCOUNT(Tabela[Kolumna])` – liczba unikalnych wystąpień.
* `COUNT(Tabela[Kolumna])` – liczba wszystkich rekordów.

### Funkcje Iteracyjne (X-functions)
Funkcje kończące się na **X** (np. **SUMX**, **AVERAGEX**) to tzw. **iteratory**. Pozwalają one wykonać obliczenia wiersz po wierszu na poziomie tabeli, a następnie zagregować wynik.
* Podobnie jak miary, obliczane są w momencie użycia na wykresie.
* Używamy ich, gdy musimy wykonać operację matematyczną przed sumowaniem (np. suma iloczynów).

**Przykład:**
`Całkowity Przychód = SUMX('Sprzedaż', 'Sprzedaż'[Cena] * 'Sprzedaż'[Ilość])`

### Funkcja IF
Działa analogicznie do funkcji w Excelu: `IF(Warunek, Prawda, Fałsz)`.
* W miarach służy głównie do sprawdzania warunków opartych na zagregowanych wynikach.
* **Uwaga:** Stosowanie `IF` w kolumnach obliczeniowych przy bardzo dużych zbiorach danych może obniżyć wydajność.

## Funkcja CALCULATE – "Królowa Funkcji"
`CALCULATE` to najważniejsza funkcja w DAX. Pozwala wykonać obliczenie, zmieniając (nadpisując) istniejący kontekst filtrów.

**Składnia:**
$$CALCULATE(Expression, Filter1, Filter2, ...)$$

* **Expression:** Miara (stworzona wcześniej) lub agregacja. 
    * *Wskazówka:* Lepiej używać miar – jeśli zmienisz definicję miary bazowej, wszystkie `CALCULATE`, które z niej korzystają, zaktualizują się automatycznie.
* **Filter:** Warunek narzucony na obliczenie. Można dodać wiele filtrów (działają jak operator AND).
* **Funkcja ALL():** Często używana wewnątrz `CALCULATE`, aby zignorować filtry (Slicery) i policzyć np. sumę całkowitą dla całego roku.

### Przykład użycia:
Pokazanie sprzedaży tylko dla kategorii "Elektronika", niezależnie od innych wybranych kategorii:
`Sprzedaż Elektroniki = CALCULATE([Suma Sprzedaży], Produkty[Kategoria] = "Elektronika")`

> **Ważne:** `CALCULATE` modyfikuje obszar danych (kontekst) przed wykonaniem obliczenia. Dzięki temu jest znacznie szybszy i wydajniejszy niż funkcja `IF` sprawdzająca warunek wiersz po wierszu!

# IV. Tabela CALENDAR

W Power BI dobrą praktyką jest stworzenie dedykowanej tabeli kalendarzowej, która obejmuje cały zakres dat występujących w modelu danych. Pozwala to na poprawną analitykę czasu (**Time Intelligence**).

## Tworzenie tabeli (DAX)
Korzystając z funkcji `CALENDAR` oraz `ADDCOLUMNS`, możemy dynamicznie wygenerować tabelę na podstawie dat z tabeli źródłowej.

**Przykład kodu:**

```dax
Calendar_table = 
VAR start_data = MIN(original_table[Data])
VAR stop_data = MAX(original_table[Data])

RETURN 
ADDCOLUMNS(
    CALENDAR(start_data, stop_data),
    "Rok",            YEAR([Date]),
    "Miesiąc_nr",     MONTH([Date]),
    "Miesiąc",        FORMAT([Date], "MMMM"),
    "Kwartał",        "K" & FORMAT([Date], "Q"),
    "Dzień",          DAY([Date]),
    "Dzień tygodnia", FORMAT([Date], "dddd"),
    "Rok-miesiąc",    FORMAT([Date], "YYYY-MM")
)
```
## Analiza kodu krok po kroku

### 1. Definicja zmiennych (VAR)
Zmienne pozwalają na jednorazowe obliczenie wartości i ich wielokrotne użycie w kodzie.

* `VAR start_data = MIN(original_table[Data])`  
* `VAR stop_data = MAX(original_table[Data])`  
Wyznaczają one najstarszą (minimalną) oraz najmłodszą (maksymalną) datę w danych źródłowych. Stanowią one ramy czasowe – pierwszy i ostatni dzień Twojego kalendarza.

### 2. Funkcja bazowa CALENDAR
`CALENDAR(start_data, stop_data)`  
Ta funkcja tworzy tabelę z jedną kolumną o nazwie **[Date]**. Generuje ona kompletną listę wszystkich dat dzień po dniu, bez żadnych przerw, od wyznaczonej daty startowej do końcowej.

### 3. Rozszerzanie tabeli (ADDCOLUMNS)
Funkcja `ADDCOLUMNS` przyjmuje naszą listę dat i „dokleja” do niej dodatkowe kolumny zdefiniowane przez użytkownika. Składnia to zawsze: **"Nazwa Kolumny", Formuła**.

* `"Rok", YEAR([Date])` – Wyciąga rok z daty (np. 2023).
* `"Miesiąc_nr", MONTH([Date])` – Zwraca numer miesiąca (1–12). **Bardzo ważne do poprawnego sortowania nazw miesięcy!**
* `"Miesiąc", FORMAT([Date], "MMMM")` – Zamienia datę na pełną nazwę miesiąca (np. „Styczeń”).
* `"Kwartał", "K" & FORMAT([Date], "Q")` – Pobiera numer kwartału i dodaje literę „K” (np. „K1”, „K2”).
* `"Dzień", DAY([Date])` – Wyciąga numer dnia miesiąca (1–31).
* `"Dzień tygodnia", FORMAT([Date], "dddd")` – Zwraca pełną nazwę dnia tygodnia (np. „Poniedziałek”).
* `"Rok-miesiąc", FORMAT([Date], "YYYY-MM")` – Tworzy format tekstowy idealny do osi wykresów, zachowując chronologię.



## Łączenie z tabelą faktów (Fact Table)
Tabela `Calendar_table` pełni rolę **tabeli wymiarów** (Dimension Table). Łączymy ją z tabelą faktów relacją **1:* (jeden do wielu)** za pomocą kolumn dat:

* **Z tabeli faktów:** kolumna, która posłużyła jako źródło dla funkcji `MIN`/`MAX` (np. `original_table[Data]`).
* **Z tabeli Calendar_table:** kolumna `[Date]` (podstawowa kolumna kalendarza).

> [!IMPORTANT]
> Aby zachować ciągłość okresów na raportach, należy zawsze używać kolumn czasowych z tabeli `Calendar_table`. Tabela faktów może posiadać luki (np. dni wolne, w których nie było sprzedaży), co bez kalendarza mogłoby generować błędy w obliczeniach Time Intelligence.

### **WAŻNE! Problemy z formatem czasu (DateTime)**
Jeżeli w tabeli głównej data zawiera również godzinę, funkcja `CALENDAR` stworzy kolumnę w formacie `DD-MM-YYYY 00:00:00`. Relacja nie zadziała poprawnie, jeśli godziny w obu tabelach nie będą identyczne (np. sprzedaż o 14:00 nie połączy się z kalendarzem ustawionym na 00:00).

**Rozwiązanie:** W Power Query, w tabeli faktów, należy zmienić typ danych kolumny z „Data/Godzina” na „Data” (sam dzień bez czasu). Dopiero na podstawie tak przygotowanej kolumny należy budować `Calendar_table` i tworzyć relacje.

##. Sortowanie chronologiczne 
Power BI domyślnie sortuje tekstowe nazwy miesięcy alfabetycznie (np. wrzesień będzie przed styczniem). Aby wykresy wyglądały poprawnie:
1. Kliknij w kolumnę `Miesiąc` w widoku danych.
2. Wybierz z górnego menu opcję **Sort by column** (Sortuj według kolumny).
3. Wybierz kolumnę `Miesiąc_nr`.


## Funkcje Analizy Czasu (Time Intelligence)

Dzięki stworzeniu tabeli `Calendar_table` i połączeniu jej z faktami, można używać zaawansowanych funkcji DAX do porównywania okresów.

###  Porównanie do analogicznego okresu rok wcześniej **(SAMEPERIODLASTYEAR)**
Funkcja ta zwraca zbiór dat przesunięty dokładnie o jeden rok wstecz w stosunku do dat wybranych w filtrze (np. na raporcie).

**Przykład miary:**
```dax
Zysk(Last Year) = 
CALCULATE(
    [Suma Sprzedaży], 
    SAMEPERIODLASTYEAR('Calendar_table'[Date])
)
```
### Uniwersalne przesunięcia czasu (DATEADD)
Do porównania danych innych niż do zeszłego roku, np zaszłego miesiąca czy kwartału stosuje się funkcji ``DATEADD`

**Składnia:**

`DATEADD(<daty>, <liczba_interwałów>, <interwał>)`
#### Przykład użycia:
```
Sprzedaż PM (Previous Month) = 
CALCULATE(
    [Suma Sprzedaży], 
    DATEADD('Calendar_table'[Date], -1, MONTH)
)
```
###  Narastająco od początku roku ***(TOTALYTD)***
Funkcja oblicza wartość od 1 stycznia danego roku do aktualnie wybranej daty.

**Przykład miary:**
```Sprzedaż YTD (Year To Date) = 
TOTALYTD(
    [Suma Sprzedaży], 
    'Calendar_table'[Date]
)
```

### Oznaczanie jako Tabela Dat
Aby funkcje takie jak `TOTALYTD` czy `SAMEPERIODLASTYEAR` działały bezbłędnie, należy poinformować Power BI, że ta tabela to Twój główny kalendarz:
1. Należy kliknac prawym przyciskiem myszy na `Calendar_table` w panelu danych.
2. Wybrać **Mark as calander table** (Oznacz jako tabelę dat)
3. Wskaż kolumnę `Date` jako kluczową.
