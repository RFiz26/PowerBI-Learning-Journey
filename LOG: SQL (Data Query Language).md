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
