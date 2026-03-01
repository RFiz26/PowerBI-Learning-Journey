#Title: SQL: Od Wybierania do Zarządzania Danymi
Subject: Bazy Danych (SQL)
Topics:

  * Podstawy pobierania danych (**SELECT**)
  * Manipulacja danymi (DML: INSERT, UPDATE, DELETE)
  * Zarządzanie strukturą bazy (DDL: ALTER, DROP, TRUNCATE)
  * Filtrowanie i operatory logiczne (WHERE, AND, OR, NULL)

Summary: Ten przewodnik tłumaczy, jak rozmawiać z bazą danych, aby wyciągać z niej informacje, dodawać nowe wpisy oraz bezpiecznie zmieniać lub usuwać dane bez niszczenia całej bazy.

#Key Concepts:

Zasada "Pudełek i Regałów": Tabela to regał, a każdy wiersz to pudełko z danymi. Wybierasz całe pudełka lub zaglądasz do środka.

Bezpieczne usuwanie: Zawsze używaj klauzuli WHERE przy DELETE i UPDATE. Brak tego słowa to jak użycie wielkiej gumki do mazania na całej stronie naraz.

Struktura vs Zawartość: Rozróżniaj operacje na danych (to, co jest w środku pudełek – INSERT, UPDATE) od operacji na samej tabeli (budowa regału – ALTER, DROP).

Praca z "Pustką" (NULL): W SQL brak danych to nie jest zero ani spacja, to stan NULL. Sprawdzamy go specjalnym zwrotem IS NULL.

Vocabulary List:

SELECT: Komenda "pokaż mi". Wyciąga dane na ekran.

INSERT INTO: Komenda "dodaj". Wstawia nowy, świeży wiersz do tabeli.

UPDATE: Komenda "popraw". Zmienia treść w istniejącym już wierszu.

SET: Wspólnik UPDATE. Wskazuje, co dokładnie chcemy zmienić (np. SET cena = 10).

DELETE: Komenda "usuń wiersz". Wymazuje konkretne wpisy.

TRUNCATE: Komenda "wyczyść szufladę". Usuwa wszystkie dane z tabeli, ale zostawia pustą tabelę.

DROP: Komenda "zniszcz". Usuwa bezpowrotnie całą tabelę wraz z jej strukturą.

WHERE: Najważniejszy filtr. Mówi bazie: "Zrób to TYLKO TAM, GDZIE spełniony jest ten warunek".

DISTINCT: Słowo, które usuwa powtórki z wyników wyszukiwania.

ALTER: Komenda "przebuduj". Służy do zmiany struktury (np. dodania nowej kolumny).

Key Questions:

Jaka jest różnica między DELETE a TRUNCATE? (Podpowiedź: jedno wymazuje po kolei, drugie wyrzuca wszystko naraz).

Co się stanie, jeśli zapomnisz o WHERE w zapytaniu UPDATE?

Jakiego słowa użyjesz, aby posortować wyniki od A do Z?

Dlaczego do zmiany typu danych w kolumnie używamy ALTER, a nie UPDATE?

Jak sprawdzić, czy w danej komórce w ogóle nie ma wpisanych danych?
