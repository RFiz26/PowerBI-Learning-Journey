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

