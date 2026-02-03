Problem 1:
Problem: Dane wejściowe używały kropki jako separatora, co skutkowało błędem DataFormat.Error przy zmiane formatu z tekstu na liczby całkowite w Power Query.

Próba 1: Bezpośrednia zmiana typu na "Liczba dziesiętna" powodowała błąd DataFormat.Error, ponieważ program nie potrafił zinterpretować kropki wewnątrz liczby.
Próba 2: Ręczna zmiana kropek na przecinki w notatniku ( dużo dodatkowej roboty)

Rozwiązanie: Zastosowałam funkcję Zmień typ z użyciem ustawień regionalnych (Change Type using Locale).
<img width="836" height="765" alt="image" src="https://github.com/user-attachments/assets/47bfcd13-bda3-476c-8399-47c94f1edac3" />
<img width="693" height="349" alt="image" src="https://github.com/user-attachments/assets/e62854a0-63ed-473d-af1a-0b35ba882c6e" />


