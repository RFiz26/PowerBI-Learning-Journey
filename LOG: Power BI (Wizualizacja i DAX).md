# I. Modele danych – Schemat Gwiazdy

Modele danych to zbiór tabel, które są powiązane między sobą i tworzą **mapy powiązań**. 

## Schemat Gwiazdy (Star Schema)
Jest to optymalny sposób powiązań tabel, oparty na strukturze, w której jedna centralna tabela jest otoczona przez tabele powiązane.

### 1. Tabela Faktów (Fact Table)
Jest to centralna tabela w modelu. Zawiera informacje o zdarzeniach, które wystąpiły w czasie.
* **Zawartość:** Liczby (cena, koszt, ilość) oraz klucze ID służące do powiązań.
* **Charakterystyka:** Posiada ogromną liczbę wierszy, ale mało kolumn (tzw. tabela "długa i chuda").



### 2. Tabele Wymiarów (Dimension Tables)
Stanowią "ramiona" modelu. Odpowiadają na pytania: *Kto? Co kupił? Gdzie? Kiedy? Jak?*
* **Przykłady:** Kalendarz, opis produktów, dane klientów.
* **Charakterystyka:** Mają dużo kolumn (opisy), ale znacznie mniej wierszy niż tabela faktów. Każdy element w tabeli wymiarów jest unikalny.

---

## **Power BI jest zoptymalizowany pod ten układ.**

---
### Kluczowe zasady:
1. **Relacja (1:*)(Jeden do Wielu)**  Filtr zawsze płynie od tabeli wymiaru (strona "1") do tabeli faktów (strona "*").
2. **Unikaj Płatków Śniegu (Snowflake):** To sytuacja, gdzie aby połączyć tabelę faktów z wymiarem, trzeba przejść przez inną tabelę wymiaru. 
    * *Dobra praktyka:* Połączyć te tabele wymiarów w jedną ( "rozpłaszczyć") 


# I.1. Transformacja tabeli płaskiej w Schemat Gwiazdy

Jeśli dane wejściowe to jedna wielka tabela (**Flat Table**), należy dokonać **dekompozycji**, czyli rozbić ją na Tabelę Faktów i Tabele Wymiarów.

## Proces rozbijania tabeli (Krok po kroku)

### 1. Wydzielenie Tabel Wymiarów (w Power Query)
Z głównej tabeli tworzymy referencje i wyciągamy kolumny opisowe do nowych tabel:
* **Klienci:** Nazwa klienta, adres, e-mail, ID Klienta(<-funkcja Kolumna Indeks).
* **Produkty:** Nazwa produktu, kategoria, cena jednostkowa, ID Produktu.
* **Sposób Płatności:** Rodzaj płatności, ID Płatności.

> **Ważne:** W tabelach wymiarów musisz użyć funkcji **"Usuń duplikaty"**. Każdy rekord (np. dany produkt) musi być unikalny.

### 2. Stworzenie Tabeli Faktów
To, co zostaje z tabeli głównej po procesie dekompozycji. Zawiera:
* **Wartości liczbowe:** Ilość, cena całkowita, zysk.
* **Klucze obce (FK):** ID Klienta, ID Produktu, ID Płatności (służą do relacji).
* **Datę:** Klucz do połączenia z kalendarzem.

### 3. Tworzenie Tabeli Kalendarz (w Power BI / DAX)
Nie używamy daty bezpośrednio z tabeli faktów do filtrowania. Tworzymy dedykowaną tabelę funkcją DAX:
`Kalendarz = CALENDAR(MIN('Fakty'[Data]), MAX('Fakty'[Data]))`
* Pozwala to na używanie funkcji **Time Intelligence** (np. porównanie rok do roku).

### 4. Budowa relacji (Widok Modelu)
Łączymy tabele za pomocą kluczy ID:
* Przeciągamy klucz z Tabeli Wymiarów (strona **1**) do Tabeli Faktów (strona **\***).
* **Zasada:** Filtr płynie zawsze od wymiaru do faktów.

# II. Miary vs. Kolumny Obliczeniowe

## **Miary (Measures)**- parametry (obliczenia) tworzone za pomocą języka **DAX**. Nie zajmują miejsca w pamięci, dopóki nie zostaną wykorzystane na wykresie, reagują na filtry (Slicer) zastosowane przez użytkownika
###  Podstawowe pojęcia DAX
**DAX (Data Analysis Expressions)** to język formuł. Większość obliczeń opiera się na agregacjach:
* `SUM(Tabela[Kolumna])` – suma wartości.
* `AVERAGE(Tabela[Kolumna])` – średnia.
* `DISTINCTCOUNT(Tabela[Kolumna])` – liczy unikalne wystąpienia (np. ilu różnych klientów zrobiło zakupy).


W Power BI miary (Measures) to obliczenia wykonywane dynamicznie. W przeciwieństwie do kolumn obliczeniowych, miary nie zwiększają rozmiaru pliku i reagują na każdy filtr (Slicer), który kliknie użytkownik.

## 1. Podstawowe pojęcia DAX
**DAX (Data Analysis Expressions)** to język formuł. Większość obliczeń opiera się na agregacjach:
* `SUM(Tabela[Kolumna])` – suma wartości.
* `AVERAGE(Tabela[Kolumna])` – średnia.
* `DISTINCTCOUNT(Tabela[Kolumna])` – liczy unikalne wystąpienia (np. ilu różnych klientów zrobiło zakupy).

---

## 2. Funkcja CALCULATE – "Królowa Funkcji"
`CALCULATE` to najważniejsza funkcja w Power BI. Pozwala ona wykonać obliczenie, zmieniając jednocześnie filtry w modelu.

**Składnia:**
$$CALCULATE(Expression, Filter1, Filter2, ...)$$

* **Expression:** Miara lub agregacja (np. `SUM(Sprzedaż[Wartość])`).
* **Filter:** Warunek, który ma zostać narzucony (np. `Produkty[Kolor] = "Czerwony"`).

### Przykład użycia:
Chcesz stworzyć miarę, która zawsze pokazuje sprzedaż tylko dla kategorii "Elektronika", niezależnie od tego, co użytkownik kliknie w filtrach:
`Sprzedaż Elektroniki = CALCULATE([Suma Sprzedaży], Produkty[Kategoria] = "Elektronika")`

---

## 3. Kontekst Obliczeń (Context)
To najważniejsza koncepcja, która odróżnia Power BI od Excela:

1. **Kontekst Filtra (Filter Context):** Wszystko, co ogranicza dane w raporcie (slicery, kliknięcie w słupek na wykresie, filtry strony). Miary zawsze "patrzą" na ten kontekst.
2. **Kontekst Wiersza (Row Context):** Występuje w kolumnach obliczeniowych. Power BI patrzy na dane wiersz po wierszu.

---

## 4. Time Intelligence (Analiza Czasu)
Dzięki DAX i dobrze zbudowanej tabeli kalendarza, możemy łatwo porównywać okresy:

* **Total YTD (Year To Date):** Suma od początku roku do dzisiaj.
  `Sprzedaż YTD = TOTALYTD([Suma Sprzedaży], 'Kalendarz'[Data])`
* **Same Period Last Year:** Porównanie do analogicznego okresu rok temu.
  `Sprzedaż Rok Temu = CALCULATE([Suma Sprzedaży], SAMEPERIODLASTYEAR('Kalendarz'[Data]))`



---

## 5. Dobra Praktyka: Tabela na Miary
Nie trzymaj miar rozproszonych po różnych tabelach. 
* Stwórz pustą tabelę (np. o nazwie "### MIARY").
* Przechowuj tam wszystkie formuły DAX.
* Dzięki temu Twój model jest uporządkowany i łatwiejszy w zarządzaniu.
