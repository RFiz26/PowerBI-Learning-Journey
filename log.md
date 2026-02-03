## [2024-05-22] Rozwiązanie problemu z separatorem dziesiętnym

**Problem:** Kolumna z wartościami dziesiętnymi była rozpoznawana jako tekst z powodu kropki (format US), co uniemożliwiało obliczenia w Power BI (ustawienia PL).

**Rozwiązanie:** Zamiast ręcznej zamiany znaków, użyłam funkcji: 
`Zmień typ` -> `Użyj ustawień regionalnych` -> `Angielski (Stany Zjednoczone)`.
<img width="1711" height="833" alt="image" src="https://github.com/user-attachments/assets/31521e2d-5bf5-4478-a97e-26f42296791c" />



**Kod M (Power Query):**
```powerquery
#"Zmieniono typ z ustawieniami regionalnymi" = Table.TransformColumnTypes(#"Changed Type", {{"Physical_Activity", type number}}, "en-US")
```
## [2024-05-22] Weryfikacja jakości danych (Data Profiling) w Power Query

**Problem:** Chciałam sprawdzić jakość danych (puste wartości błędy,duplikaty, rozkład danych, wartości ekstremalne)

**Rozwiązanie:** Zastosowanie narzędzi profilowania danych w Power Query (Zakładka **Widok**), aby szybko ocenić stan zbioru danych przed dalszą obróbką.

**Użyte funkcje:**
* **Jakość kolumn (Column Quality):** Pozwala szybko sprawdzić procent danych prawidłowych (Valid), błędów (Errors) oraz pustych (Empty).
* **Rozkład kolumn (Column Distribution):** Wyświetla wykres słupkowy pokazujący liczbę **unikatowych** (Unique) (liczba elementów które wystąpiły tylko raz) i **odrębnych** (Distinct) ( liczba elemetnów które wystąpiły chociaż raz) wartości w kolumnie. Pomaga wykryć anomalie w ID lub kategoriach. (**Jeżeli Unique=Distinct <- nie ma dublikatów**)
* **Profil kolumn (Column Profile):** Wyświetla szczegółowe statystyki (Minimum, Maximum, Średnia, Odchylenie standardowe) oraz histogram dla zaznaczonej kolumny. <- tutaj można popatrzeć na **ekstrema** czy mają sens

<img width="1491" height="987" alt="image" src="https://github.com/user-attachments/assets/9d41d2ba-27f3-4443-94b8-c10decf65f7b" />

## [2024-05-22] Załadowanie danych z Google BigQuery

**Problem** Podczas próby ściągnięcia danych z systemu Google BigQuery (Pobierz dane->Więcej...->Google BigQuery-> Połącz) pojawił się problem
<img width="975" height="609" alt="image" src="https://github.com/user-attachments/assets/91f7e60d-cd13-40e8-b74b-2d4f9cc4b0b2" />
**Próba 1** Zainstalowanie 64-bitowowego sterownika ODBC dla BigQuery.
    **Rezultat** Błąd dalej występował
**Próba 2** Wyłączenie Google BigQuery Storage API (Sugestia Google Gemini)
  Plik -> Opcje i ustawienia -> Opcje
<img width="1001" height="800" alt="image" src="https://github.com/user-attachments/assets/405c11a4-a1eb-42db-846d-02e76faed85e" />
<img width="998" height="793" alt="image" src="https://github.com/user-attachments/assets/45644cd1-fd58-40ef-a590-9df7fd121658" />

  **Rezultat** Problem zniknął, dane zostały załadowane
