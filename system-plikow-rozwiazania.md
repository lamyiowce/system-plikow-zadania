# System plików - ćwiczenia
### B3
1. Załóżmy, że blok na dysku ma rozmiar 1024 B. Jaki rozmiar ma grupa?
2. Załóżmy, że grupa bloków ma rozmiar 128 MiB. Ile wynosi rozmiar bloku na dysku?
3. Załóżmy, że blok na dysku ma rozmiar 1024 B. Grup na dysku jest dokładnie 400. Deskryptor grupy zajmuje 24 B. Który bit w mapie bitowej zajętości bloków w grupie zawierającej wszystkie deskryptory grup odpowiada blokowi zawierającemu tę mapę?
4. Dlaczego funkcja kontrolująca spójność mapy bitowej dla każdej z tych map zakończy się błędem?
```
0111 1111 1101 1010 0110 ...
1011 0000 0000 0000 0000 ...
1101 1111 1111 1111 1111 ...
```

1. Mapa zajętości bloków musi się mieścić w jednym bloku, więc bloków jest 1024 * 8. Czyli bloki zajmują 8 MiB.
2. x - rozmiar bloku w bajtach,  to wtedy rozmiar grupy to 8x^2. Skoro rozmiar grupy to 128 MiB, to x = 4KiB 
3. Superblock to 1 blok, Deskryptory zajmują 10 bloków. Czyli bitmapa to 12. blok (licząc od 1).
4. * superblok jest wolny!
* deskryptory grup są wolne
* trzeci bit to albo część dalsza deskryptorów grup, albo bitmapa bloków, czyli musi być zajęte.

### B4
Jaki jest maksymalny rozmiar partycji dla systemu plików, w którym pierwsza grupa przechowuje wszystkie deskryptory grup danej partycji. Przyjmijmy, że deskryptor grupy ma rozmiar 64 B, a blok dyskowy – 4 KiB. Jaki byłby w takiej sytuacji rozmiar metagrupy? 

> W późniejszych wersjach systemu (ext3 i ext4) wprowadzono tak zwane metagrupy. Metagrupa jest zbiorem grup, o tak dobranym rozmiarze, aby deskryptory grup w obrębie jednej metagrupy mieściły się w pojedynczym bloku dyskowym. Deskryptory grup danej metagrupy są przechowywane w jej pierwszej grupie i dodatkowo w drugiej i ostatniej. Metagrupa to następny poziom w hierarchii, pomiedzy grupą a partycją.

W jednej grupie zmieści się 128 MiB / 64 B = 2^21 deskryptorów. Rozmiar grupy to 128 MiB, więc maksymalny rozmiar partycji to 2^21 * 128MiB = 256 TiB. Taka partycja byłaby niepraktyczna, bo całe grupy byłyby zajmowane przez metadane.  

## Operacje na plikach
> I-węzeł pliku w systemach ext2 i ext3 zawiera numery pierwszych 12 bloków z danymi oraz numery jednego bloku pośredniego, jednego bloku podwójnie pośredniego i jednego bloku potrójnie pośredniego (tablica i-block).
### C1
Plik o rozmiarze logicznym 15000 B został umieszczony na partycji, dla której rozmiar bloku ustalono na 1 KiB. 
Policz, ile operacji dyskowych wymaga
* wczytanie,
* zapisanie
fragmentu pliku o wielkości 1000 B (uwaga: nie 1 KiB) położonego
* począwszy od adresu logicznego 6000,
* począwszy od adresu logicznego 14000.
W obu przypadkach plik jest otwarty, ale żaden blok danych nie został jeszcze sprowadzony do pamięci. W pamięci znajduje się jedynie i-węzeł pliku. Możemy założyć, że wszystkie bloki pliku są zapisane (plik nie jest dziurawy).
* Jak zmienią się odpowiedzi bez ostatniego założenia?
* W jakiej sytuacji uzyskamy najmniejszą liczbę operacji dyskowych?

Plik zajmuje 15000 / 1024 = 15 bloków. Gdy fragment ma 1000B i zaczyna się na adresie 6000, więc aby go wczytać musimy przeczytać dwa bloki, czyli 2 operacje dyskowe. Przy zapisaniu musimy najpierw wczytać oba bloki i potem je zapisać, czyli 4 operacje. 
Gdy zaczyna się na 14000, to do 12 bloków mamy bezpośrednie wskaźniki, a do 3 pośrednie. Plik znajduje się w węzłach pośrednich, więc musimy jeszcze dodatkowo wczytać jeden block ze wskaźnikami. Najmniej operacji dyskowych będzie, jeśli plik będzie pusty przy zapisywaniu. 

## C2 
> Jeden wskaźnik to 4 B. Więc dla bloku 1 KiB w węźle pośrednim jest 256 dowiązań. Wskaźnikiem dwupośrednim da się zaadresować 256 * 256 bloków, w trójpośrednim 256 ^ 3
Załóżmy, że rozmiar bloku w systemie plików ext3 został ustalony na 1 KiB. Ile maksymalnie, a ile minimalnie bloków dyskowych może być potrzebne do zapamiętania pliku o rozmiarze 2 GiB?
__Wskazówka: pliki mogą być rozrzedzone (dziurawe).__

2 GiB / 1 KiB = 2^21 bloków. 
* Minimalnie: 12 bloków bezpośrednio + 


## C3
> Gdy rozmiar pliku w i-węźle systemu ext2 był opisywany liczbą 32-bitową i jej najstarszy bit był zarezerwowany, plik mógł mieć wielkość co najwyżej 2 GiB. We współczesnych systemach ext2 i ext3 32-bitowe pole i-węzła o nazwie i-blocks przechowuje rozmiar pliku wyrażony w liczbie 512 bajtowych porcji. Rozmiar bloków na dysku zależy od konfiguracji. Dopuszczalne są bloki o wielkości 512 B, 1 KiB, 2 KiB, 4 KiB, a nawet 8 KiB (o ile architektura na to pozwala). Numery bloków fizycznych są 32-bitowe.

Wiedząc to wszystko i biorąc pod uwagę sposób adresowania bloków, uzupełnij poniższą tabelkę dla systemów ext2 i ext3:

Z jednej strony mamy ograniczenie 2^41 B (wynika z pola przechowującego rozmiar: 512 * 2^32).
b - rozmiar bloku
maks rozmiar pliku: __min__ {b*(12 + b/4 * (b/4)^2 + (b/4)^3, 2^41}
maks. rozmiar partycji: 
rozmiar bloku | maksymalny rozmiar pliku | maksymalny rozmiar partycji
--------------|--------------------------|----------------------------
1 KiB 		  | 16GiB                    | 4 TiB
--------------|--------------------------|----------------------------
2 KiB 		  | 256 GiB                  | 8 TiB
--------------|--------------------------|----------------------------
4 KiB 		  | 2 TiB                    | 16 TiB
--------------|--------------------------|----------------------------
8 KiB         | 2 TiB                    | 32 TiB
--------------|--------------------------|----------------------------


