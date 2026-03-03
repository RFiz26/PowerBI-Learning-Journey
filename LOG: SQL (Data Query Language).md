# SQL: Od Wybierania do Zarządzania Danymi
## 1: Podział komend:
  * Podstawy pobierania danych (**SELECT**)
  * Manipulacja danymi (DML: **INSERT, UPDATE, DELETE**)
  * Zarządzanie strukturą bazy (DDL: **ALTER, DROP, TRUNCATE**)
  * Filtrowanie i operatory logiczne (**WHERE, AND, OR, NULL**)

## 2. Kluczowe koncepty:

 **Zasada "Pudełek i Regałów"**: Tabela to regał, a każdy wiersz to pudełko z danymi. Wybierasz całe pudełka lub zaglądasz do środka.
 
 **Bezpieczne usuwanie**: Zawsze używaj klauzuli WHERE przy DELETE i UPDATE. Brak tego słowa to jak użycie wielkiej gumki do mazania na całej stronie naraz.
 
 **Struktura vs Zawartość**: Rozróżniaj operacje na danych (to, co jest w środku pudełek – INSERT, UPDATE) od operacji na samej tabeli (budowa regału – ALTER, DROP).
 
 **Praca z "Pustką" (NULL)**: W SQL brak danych to nie jest zero ani spacja, to stan **NULL**. Sprawdzamy go specjalnym zwrotem **IS NULL**.
 
 **Częściowe wstawianie danych (INSERT)**: Jeśli tabela ma np. 10 kolumn, a podajesz tylko 3 wartości, pozostałe 7 otrzyma wartość NULL lub wartość domyślną (o ile tabela na to pozwala). Jeśli kolumna wymaga danych (NOT NULL), baza wyrzuci błąd.

## 3. Opisy funckji:

* **SELECT**: Komenda "pokaż mi". Wyciąga dane na ekran.
  
     Przykład: `SELECT imie, nazwisko FROM Klienci;`
  
* **INSERT INTO**: Komenda "dodaj". Wstawia nowy, świeży wiersz do tabeli.

    Przykład: `INSERT INTO Klienci (imie) VALUES ('Jan');`
  
* **UPDATE**: Komenda "popraw". Zmienia treść w istniejącym już wierszu.

    Przykład: `UPDATE Klienci SET miasto = 'Gdańsk' WHERE id = 1;`
  
* **SET**: Wspólnik UPDATE. Wskazuje, co dokładnie chcemy zmienić (np. SET cena = 10).

    Przykład: `SET status = 'Aktywny'`

* **DELETE**: Komenda "usuń wiersz". Wymazuje konkretne wpisy.

    Przykład: `DELETE FROM Klienci WHERE id = 5;`
  
* **TRUNCATE**: Komenda "wyczyść szufladę". Usuwa wszystkie dane z tabeli, ale zostawia pustą tabelę.

    Przykład: `TRUNCATE TABLE Klienci;`

* **DROP**: Komenda "zniszcz". Usuwa bezpowrotnie całą tabelę wraz z jej strukturą (ewentualnie można znaleźć ją w koszu)

    Przykład: `DROP TABLE Stare_Dane;`

* **WHERE**: Najważniejszy filtr. Mówi bazie: "Zrób to TYLKO TAM, GDZIE spełniony jest ten warunek".

    Przykład: `WHERE wiek >= 18`

* **LIKE**: Szukanie danych po nie dokładnej nazwie z wykorzystaniem **%**
    * 'G%'-szuka wszystkich danych w kolumnie zaczynających się na G (Grzebień, garnek itp.)
    * '%ów'-szuka wszystkich danych w kolumnie kończących się na ów (Kraków,Gdów)
    * '%mok%-szuka wszystkich dancyh w kolumnie który zawiera ciag 'mok' (smok,mokradła, domokrązca)

    Przykład: `WHERE państwo LIKE 'A%'`

  **UWAGA!!** Wielkość liter może mieć znaczenie (MySQL lub SQL Server nie ma). W przypadku **Oracle/SQLite** nalezy zastosowac funkcje **LOWER** lub **UPPER** 

  Przykład: `WHERE LOWER(owoc) LIKE '%jabłko%'`

* **ORDER BY**: Komenda "posortuj". Pozwala ułożyć wyniki:
    * Domyślnie (A-Z, 1-100): Wystarczy napisać **ORDER BY** kolumna **ASC**.
    * Odwrotnie (Z-A, 100-1): Należy dopisać DESC na końcu, np. **ORDER BY** cena **DESC**$

* **DISTINCT**: Słowo, które usuwa powtórki z wyników wyszukiwania.

    Przykład: `SELECT DISTINCT miasto FROM Klienci;`

* **ALTER**: Komenda "przebuduj". Służy do zmiany struktury (np. dodania nowej kolumny).

    Przykład: `ALTER TABLE Pracownicy ADD Email VARCHAR(255);`

## 4. SQL: Gotowe wzory:
### 1. Dodawanie 

`INSERT INTO NazwaTabeli (kolumna1, kolumna2) VALUES ('tekst', 100);`

### 2. Zmiana

`UPDATE NazwaTabeli SET kolumna = 'nowa_wartosc' WHERE id = 1;`

### 3. Usuwanie

`DELETE FROM NazwaTabeli WHERE id = 1;` (W DELETE nie używamy gwiazdki (*)!!)

### 4. Wyświetlanie (Raport)

`SELECT * FROM NazwaTabeli ORDER BY cena DESC;`

### 5. Przebudowa (Nowa cecha, np. kolor)

`ALTER TABLE NazwaTabeli ADD nazwa_kolumny typ_danych;`

## 5. Agregacje: SQL kalkulator:

 * **COUNT()** – liczy ile jest wierszy
   
 Przykład: `SELECT COUNT(*) FROM Produkty;`
 Jeżeli chce się policzyć tylko unikalne wartości należy użyć zwrotu  `COUNT(DISTINCT kolumna)`

 * **SUM()** – sumuje wartości
    
 Przykład: `SELECT SUM(cena * ilosc) FROM Produkty;`
 
 * **AVG()** – wyciąga średnią
   
 Przykład: `SELECT AVG(cena) FROM Produkty;`

 * **MIN()/MAX()** – znajduję najmniejszą/największa wartość
 
 Przykład: `SELECT MIN(cena), MAX(cena) FROM Produkty;`

### **GROUP BY** – Agregacja z grupowaniem

Pozwala na tworzenie raportów z podziałem na kategorie i wynikami obliczeń. Przykład: Jak wyświetlić całkowitą ilość produktów w zależności od kategorii?
 
 Przykład:`SELECT kategoria, SUM(ilosc) FROM Produkty GROUP BY kategoria;`
 
### **HAVING** – Warunkowanie agregacji
Jeżeli chcemy przefiltrować wyniki, które są efektem agregacji (a nie są surowymi danymi w tabeli), stosujemy klauzulę HAVING.

Przykład 1: `SELECT dostawca, SUM(ilosc) FROM Magazyn GROUP BY dostawca HAVING SUM(ilosc) > 100;`

Przykład 2 (Połączenie WHERE i HAVING): `SELECT produkt, SUM(ilosc) FROM Magazyn WHERE dostawca = 'Warzywex' GROUP BY produkt HAVING SUM(ilosc) > 10;`

## 6. Łączenie tabel

Często baza składa się z wielu tabel, które są ze sobą powiązane za pomocą kluczy. 

Klucze dzielimy na:

  -**Klucz główny (Primary Key – PK)**: Jednoznacznie identyfikuje każdy wiersz w tabeli. Musi być unikalny i nie może być pusty (NULL), np. kolumna id.

  -**Klucz obcy (Foreign Key – FK)**: Kolumna wskazująca na klucz główny w innej tabeli. Tworzy relację między tabelami i zapewnia tzw. integralność referencyjną danych, np. kolumna id_klient.
  
  
  ### **JOIN (...) ON**
  
  Służy do łączenia tabel w taki sposób, aby wynikiem była ich część wspólna (tylko dopasowane pary). Schemat zapisu wygląda następująco:

  **(...) FROM Tab_1 JOIN Tab_2 ON Tab_1.id_obcy = Tab_2.id (...)**(Zasada: Najpierw nazwa tabeli, potem nazwa kolumny z ID!)

  Łatwiejsza forma (Aliasy):**(...) FROM Tab_1 t1 JOIN Tab_2 t2 ON t1.id_obcy = t2.id (...)**
  
 **Ważne**: Kolejność tabel w INNER JOIN nie ma wpływu na wynik, jednak dla czytelności najlepiej jako pierwszą podać tabelę główną, a później dodatkową.
           
Przykład: `SELECT p.nazwisko, d.nazwa_dzialu FROM Pracownicy p JOIN Dzialy d ON p.dzial_id = d.id;`

### LEFT JOIN (lub RIGHT JOIN) (...) ON

Jest to połączenie jednostronne. Tabela po lewej (lub prawej) stronie jest brana w całości, a z drugiej tabeli dołączane są tylko te wiersze, które mają powiązanie z pierwszą. W miejscach, gdzie brakuje dopasowania, w kolumnach pojawi się wartość `NULL`.

Schemat zapisu:**(...) FROM Tab_1 LEFT JOIN Tab_2 ON Tab_1.id_obcy = Tab_2.id (...)**

Przykład (Szukanie braków):`SELECT Goscie.imie FROM Goscie LEFT JOIN Opinie ON Goscie.id = Opinie.gosc_id WHERE Opinie.id IS NULL;`


## 7. **Podzapytania (Subqueries)**
 To zapytanie wewnątrz innego zapytania. Najczęściej używamy ich w klauzuli `WHERE`
Przykładowo chcemy znaleźć nazwy dań, które mają wyższą cene niż średnia cena całego menu:

Przykład: `SELECT nazwa FROM Dania WHERE cena > (SELECT AVG(cena) FROM Dania);`


