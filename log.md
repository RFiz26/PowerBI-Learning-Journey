## [2024-05-22] Rozwiązanie problemu z separatorem dziesiętnym

**Problem:** Kolumna z wartościami dziesiętnymi była rozpoznawana jako tekst z powodu kropki (format US), co uniemożliwiało obliczenia w Power BI (ustawienia PL).

**Rozwiązanie:** Zamiast ręcznej zamiany znaków, użyłam funkcji: 
`Zmień typ` -> `Użyj ustawień regionalnych` -> `Angielski (Stany Zjednoczone)`.
<img width="1711" height="833" alt="image" src="https://github.com/user-attachments/assets/31521e2d-5bf5-4478-a97e-26f42296791c" />



**Kod M (Power Query):**
```powerquery
#"Zmieniono typ z ustawieniami regionalnymi" = Table.TransformColumnTypes(#"Changed Type", {{"Physical_Activity", type number}}, "en-US")




