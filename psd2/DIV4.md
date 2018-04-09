# System *Sygnet* służący do skalowania procesu KYC poprzez bankowe poświadczenie tożsamości klienta

Wersja 4.1 (09/04/2018)

**Proponujemy rozwiązanie w zakresie KYC, które bazując na legislacji PSD2 umożliwia wykorzystanie, będącej w posiadaniu banku, informacji o tożsamości klienta. Skalowanie procesu KYC uzyskujemy poprzez wykorzystanie kryptograficznego poświadczenia tożsamości klienta uzyskane od jego banku. Dzięki temu bank, oprócz świadczenia usług finansowych, jest w stanie radykalnie ułatwić swoim klientom proces weryfikacji KYC w sytuacji, gdy chcą oni skorzystać z usług innych podmiotów w branży FinTech (zarówno bankowych jak i niebankowych), a w dalszej konsekwencji, staje się dla nich generatorem ich cyfrowej tożsamości w Internecie.**

## 1. Wprowadzenie

#### 1.1 Jak weryfikować tożsamość online?

Internetowa weryfikacja tożsamości to próba rozwiązania następującego problemu: w jaki sposób w warunkach online uzyskać pewność, że osoba (klient) podająca się jako K rzeczywiście jest tą osobą K?

Istotne jest tu to, że jest to sytuacja online, czyli *nie* w realu. W sytuacji kontaktu w realu dokument ze zdjęciem można uznać za wystarczająco dobry sposób weryfikacji tożsamości (mimo swoich oczywistych wad: jest kosztowny w produkcji i relatywnie łatwo może być podrobiony, zwłaszcza w przypadku, gdy nie wiemy jak dokładnie powinien wyglądać oryginalny dokument, np. dowód tożsamości obcokrajowca).

Z oczywistych powodów w warunkach online zdjęcie, które jest integralną częścią dokumentu tożsamości, przestaje być użyteczne, bo nie ma go z czym porównać. 

#### 1.2 Problem: nieskalowalność KYC

Wymogi KYC narzucają na firmy konieczność ustalenia tożsamości każdego nowego klienta.

Wszystkie obecne procedury KYC mają jedną istotną wadę: nie skalują się. Przy pozyskaniu nowego klienta każda firma, która podlega wymogom KYC, musi samodzielnie dokonać ustalenia jego tożsamości, co jest kosztowne i czasochłonne (dla obu stron: firmy i jej klienta). Tak więc w obecnie istniejącym paradygmacie ten sam kosztowny i czasochłonny proces musi być wykonywany wielokrotnie przez kolejne firmy.

Jedynym znanym nam sposobem na skalowanie KYC jest wykorzystanie procedury tzw. testowego przelewu, który polega na tym, że klient poświadcza swoją tożsamość poprzez wykonanie przelewu ze swojego rachunku w innym banku do banku, który chce dokonać weryfikacji KYC. Oczywiste są wady tego rozwiązania:

* nie rozwiązuje to problemu dla podmiotów niebankowych,
* nie skaluje się więcej niż raz (bo nie można w ten sposób potwierdzić tożsamości w kolejnym banku),
* wymaga to od klienta dodatkowego wysiłku.

Niemniej tego rodzaju kombinowanie (tj. używanie przelewu bankowego do celów niefinansowych) pokazuje, że problem nieskalowalności KYC rzeczywiście istnieje.

Warty podkreślenia jest fakt, że outsourcing procesu KYC do specjalistycznej firmy, która się tym zajmuje, nie rozwiązuje problemu skalowania KYC. Nawet jeśli podmiot specjalizujący się w KYC dostanie zlecenie weryfikacji klienta K, którego wcześniej weryfikował dla innej firmy, to i tak cały proces KYC będzie musiał być uruchomiony od nowa, bo nie ma żadnego formalnego dowodu, że w obu przypadkach jest to rzeczywiście ten sam klient K.

#### 1.3 Co chcemy osiągnąć?

Szukamy rozwiązania dla procesu KYC, które:

- umożliwi skalowanie procedury KYC, czyli wyeliminuje konieczność powtarzania tego procesu przez kolejne firmy,
- będzie działać dla wszystkich firm, które podlegają wymogom KYC (tj. nie tylko dla podmiotów bankowych),
- będzie miało realną szansę na masową adopcję, zarówno po stronie biznesów jak i ich klientów,
- otworzy drogę na inne niż KYC zastosowania, w szczególności do rozpowszechnienia koncepcji cyfrowej tożsamości.

## 2. Skalowalny proces KYC

#### 2.1 Założenia

Najogólniej mówiąc, skalowalny KYC polega na tym, że jeden podmiot (w naszym przypadku bank) przeprowadza weryfikację tożsamości danej osoby (klienta) K, a następnie wynik tej weryfikacji jest udostępniany innym podmiotom (bankowym i niebankowym).

Do konstrukcji skalowalnego KYC wykorzystujemy następujący zestaw założeń:

- (niemal) każdy dorosły człowiek ma konto w banku,
- każdy bank zna tożsamość każdego swojego klienta,
- w ramach wprowadzonego przez PSD2 API możliwe jest wydobywanie z banku informacji dotyczących tożsamości danego klienta,
- każdy bank dysponuje [kwalifikowanym podpisem elektronicznym](https://pl.wikipedia.org/wiki/Podpis_kwalifikowany)  i może go użyć do kryptograficznego podpisywania informacji przesyłanych poprzez API.

Naturalną konsekwencją wydaje się zatem istnienie możliwości zrobienia użytku z informacji o tożsamości klientów bankowych (tj. informacji, które banki i tak posiadają) w celu radykalnego usprawnienia procesu KYC.

#### 2.2 System Sygnet

W dalszej części niniejszego dokumentu opisujemy propozycję systemu o nazwie *Sygnet*, który realizuje powyższe założenia. Nazwa *Sygnet* w zamierzeniu kojarzyć się ma (zarówno w języku polskim jak i angielskim) z własnoręcznym podpisem i ma nawiązywać do czegoś, co służy do potwierdzania tożsamości użytkownika i autoryzacji różnego rodzaju działań.

> Signet - a small seal, especially one set in a ring, used instead of or with a signature to give authentication to an official document.

#### 2.3 Koncepcja

System Sygnet umożliwia klientowi K dostarczenie dowolnej firmie F podpisanego elektronicznie przez wiarygodny podmiot (w naszym przypadku bank B) certyfikatu poświadczającego jego (tj. klienta K) tożsamość.

**W systemie Sygnet skalowanie procesu KYC uzyskujemy poprzez wykorzystanie kryptograficznego poświadczenia tożsamości klienta uzyskane od jego banku.**

Oznacza to, że nasz pomysł w zakresie KYC sprowadza się *de facto* do tego: w kontrolowany sposób wyprowadzamy na zewnątrz informacje (tj. dane osobowe klienta), które do tej pory leżały bezużytecznie (z perspektywy świata zewnętrznego) w systemie bankowym. Dzięki temu bank, oprócz świadczenia usług finansowych, staje się generatorem cyfrowej tożsamości swoich klientów.

#### 2.4 Wymagania

Żeby powyższy mechanizm KYC mógł funkcjonować potrzebne będzie spełnienie następujących warunków:

1. Wymagane jest żeby nasz partner bankowy zgodził się kryptograficznie podpisywać informacje udostępniane w ramach wprowadzonego przez PSD2 API (w szczególności chodzi nam o informacje dotyczące tożsamości danego klienta). Tak więc wymaganą inwestycją ze strony banku jest dostarczenie dodatkowej, tj. nie wymaganej przez PSD2, funkcjonalności w swoim API.
2. Aby system mógł być użyteczny docelowo partnerów bankowych musi co najmniej kliku. Potrzebne więc będzie wystandaryzowanie procesu tak, żeby firma F otrzymywała dane KYC w tym samym formacie niezależnie od instancji banku, który jest ich źródłem.
3. Integralną częścią naszej aplikacji musi być możliwość weryfikacji certyfikowanego podpisu elektronicznego banku, tak żeby firma F mogła w łatwy sposób uzyskać pewność, że otrzymane od banku B informacje na temat klienta K rzeczywiście zostały przez ten bank wygenerowane i nie zostały zmodyfikowane po ich podpisaniu. Jest to funkcjonalność podobna do tej, która jest oferowana przez powszechnie dostępne serwisy internetowe, np. [MadKom](https://madkom.pl/weryfikacja-podpisu-elektronicznego/).
4. Ponieważ my, jako twórcy i operatorzy aplikacji mobilnej będącej w posiadaniu klienta K, pełnimy rolę TPP (Third Party Provider), musimy być podmiotem zarejestrowanym w KNF zgodnie z wymaganiami PSD2. Wydaje się, że licencja AIS (Account Information Service) w tym przypadku będzie wystarczająca.



#### 2.5 Proces podstawowy

Załóżmy, że firma F potrzebuje dokonać weryfikacji KYC klienta K i ma zaufanie do banku B, tj. podpisane elektronicznie oświadczenie banku B w zakresie tożsamości klienta K uznaje za prawdziwe.

Wtedy proces KYC w systemie Sygnet może wyglądać następująco:

1. Klient K zainteresowany skorzystaniem z usług firmy F, potwierdza, że ma konto w banku, który jest wspierany przez system Sygnet i wybiera ten system jako mechanizm weryfikacji tożsamości w procesie KYC.
2. Klient K jest przekierowany na stronę webową systemu Sygnet. Przekierowanie zawiera wygenerowany przez firmę F unikalny identyfikator ID, którego rolą jest uwiarygodnienie niniejszego procesu z punktu widzenia firmy F: uzyska ona w ten sposób pewność, że odpowiedź banku B zostanie wygenerowana specjalnie dla tej konkretnej sytuacji.
3. W ramach systemu Sygnet klient K loguje się do swojego banku B, a następnie autoryzuje wygenerowane przez system Sygnet zapytanie do bankowego API. Zapytanie to dotyczy danych osobowych klienta K, które są wymagane w procesie weryfikacji KYC, a także zawiera wyżej opisany identyfikator ID.
4. W odpowiedzi na wyżej opisane zapytanie, bank B zwraca podpisany elektronicznie pakiet zawierający i łączący w jedną całość wymagane dane osobowe klienta oraz wyżej opisany identyfikator ID.
5. Po weryfikacji podpisu banku B i identyfikatora ID firma F uznaje uzyskane od banku dane osobowe klienta K za prawdziwe i aktualne, i tym samym spełniające kryteria KYC.

#### 2.6 Proces rozszerzony 

Nasuwa się pytanie, czy firma F po uzyskaniu kryptograficznie podpisanego pakietu informacji KYC od banku B nie przekaże go kolejnej firmie, podważając tym samym wartość dodaną naszego systemu. 

Możliwe jest zmodyfikowanie powyższego schematu działania w taki sposób, aby zapobiec tego rodzaju problemowi. Zmodyfikowany proces mógłby wyglądać następująco:

1. Oprócz unikalnego identyfikatora ID, firma F wysyła do systemu Sygnet swój klucz publiczny.
2. Zarówno identyfikator ID jak i klucz publiczny firmy F są przekazywane do banku B, który generuje pakiet KYC w sposób opisany powyżej, ale przed podpisaniem go własnym kluczem, szyfruje go kluczem publicznym firmy F.
3. Firma F po otrzymaniu pakietu jest w stanie go odkodować i zweryfikować prawdziwość zawartych w nim danych. 

Tym samym przekazanie przez firmę F otrzymanego od banku pakietu KYC innej firmie przestaje mieć sens, bo wymagałoby to także ujawnienia swojego klucza prywatnego umożliwiającego jego rozkodowanie, do czego firma F raczej nie będzie skłonna.

## 3. Uogólnienie systemu

Łatwo jest zauważyć, że zamiast dostarczanego przez firmę F identyfikatora ID (który jest niczym innym niż losowym, i przez to unikalnym, ciągiem znaków) moglibyśmy użyć ciągu znaków, który jest nielosowy i przez to może być nośnikiem jakiegoś znaczenia, np. hash dowolnego dokumentu.

Idąc tym tropem widzimy, że system Sygnet może nie tylko służyć do skalowania procesu KYC, ale także funkcjonować jako ogólny mechanizm podpisywania przez bank w imieniu swojego klienta (i na jego żądanie) dowolnego dokumentu.

Wówczas ogólny schemat systemu Sygnet można opisać w sposób następujący: klient K przekazuje do banku B dowolny ciąg znaków, a bank B zwraca, podpisany kryptograficznie swoim kluczem prywatnym, pakiet łączący w jedną całość dwa elementy:

- dane osobowe klienta K, które pozwalają na jego jednoznaczną identyfikację (np. imię, nazwisko, nr dowodu osobistego albo PESEL),
- wyżej wspomniany ciąg znaków w formie nienaruszonej.

Jeśli powyższy ciąg znaków to hash jakiegoś dokumentu, np. oświadczenia woli albo umowy cywilno-prawnej, to pod względem kryptograficznej wiarygodności uzyskujemy *de facto* ten sam efekt, który mielibyśmy, gdyby klient K używał swojego własnego certyfikowanego podpisu elektronicznego, a podmiotem certyfikującym ten podpis byłby jego bank.

Jednak z punktu widzenia *User* *Experience* uzyskujemy istotną korzyść: w naszym podejściu nie wymagamy, aby użytkownik posiadał i dbał o klucz prywatny (co jest istotą podpisu elektronicznego), bo w naszym schemacie to **bank podpisuje się swoim kluczem prywatnym w imieniu i na żądanie klienta**.

Oczywistym niebezpieczeństwem jest to, że bank może uzurpować sobie tożsamość klienta (tj. podpisać się pod czymkolwiek za niego bez jego wiedzy), ale taki jest nieusuwalny *trade-off* tego rodzaju rozwiązania. Zdejmujemy z użytkownika obowiązek troski o klucz prywatny, ale odbywa się to kosztem konieczności zwiększonego zaufania do banku.

Gdybyśmy chcieli usunąć ten *trade-off*, system Sygnet w wersji "profesjonalnej" mógłby mieć funkcjonalność pozwalającą na:

* wygenerowanie dla klienta K unikalnego klucza prywatnego i odpowiadającego mu klucza publicznego, 
* uzyskanie od banku B elektronicznie podpisanego certyfikatu, który połączyłby w jedną całość tożsamość klienta K z jego kluczem publicznym.

Posiadając taki bankowy certyfikat, klient K w ramach systemu Sygnet mógłby posługiwać się swoim kluczem prywatnym niczym certyfikowanym podpisem elektronicznym - oczywiście przy założeniu, że uznamy bank B za wiarygodne źródło certyfikacji dla klienta K.

## 4. Czym nasze rozwiązanie różni się od profilu zaufanego?

Wedle [dokumentacji](https://www.gov.pl/cyfryzacja/profil-zaufany-ego-) Ministerstwa Cyfryzacji profil zaufany to bezpłatna metoda potwierdzania tożsamości obywatela w systemach elektronicznej administracji.

> Profil zaufany działa jak odręczny podpis. Możesz dzięki niemu wysyłać przez Internet dokumenty i wnioski do różnych urzędów (np. wnieść podanie, odwołanie, skargę). Profil zaufany potwierdza tożsamość obywatela. Podpis potwierdzony profilem zaufanym, podobnie jak kwalifikowany podpis elektroniczny, skutecznie zastępuje w kontaktach z podmiotami publicznymi podpis własnoręczny.

Na pierwszy rzut oka profil zaufany wygląda na rozwiązanie bardzo podobne do naszego. Co więcej, podobnie jak w naszym podejściu jednym z podmiotów zdolnych do wygenerowania takiego profilu dla użytkownika jest serwis bankowy. Kilka banków w Polsce oferuje taką usługę.

Jest jednak istotna różnica: zastosowanie profilu zaufanego jest ograniczone do serwisów państwowych. Ograniczenie to wynika ze sposobu działania tego systemu: polega on na tym, że po założeniu profilu zaufanego użytkownik uzyskuje dostęp do wspólnego dla systemów państwowych mechanizmu logowania, lecz nie jest to równoważne z uzyskaniem uniwersalnej cyfrowej tożsamości. Tak więc profil zaufany z założenia nie może być rozszerzony na sferę niepaństwową i tym samym nie rozwiązuje problemu KYC dla firm.

Niemniej porównanie naszego rozwiązania do profilu zaufanego wydaje się jak najbardziej uzasadnione. Można nawet powiedzieć, że nasz system oferuje biznesom rozwiązanie w zakresie KYC analogiczne do tego, jakie profil zaufany oferuje urzędom państwowym w zakresie mechanizmu logowania, tj. weryfikacji tożsamości obywatela w warunkach online.

## 5. Monetyzacja systemu

System Sygnet ma szansę być postrzeganym jako sytuacja typu *win-win* dla wszystkich interesowanych, ponieważ:

1. Dostarczamy bankom możliwość monetyzowania informacji o tożsamości ich klientów, z wykorzystaniem mechanizmów autoryzacji, które muszą zostać dostarczone na potrzeby TPP (wymaganie PSD2). Dodatkową korzyścią dla banków jest możliwość dostarczenia nowego typu usługi dla klientów: oprócz oferowania usług stricte finansowych bank może stać się generatorem cyfrowej tożsamości dla swoich klientów.
2. Dostarczamy wszystkim podmiotom, które podlegają wymogom KYC, mechanizm szybkiej weryfikacji tożsamości klientów. Jego użycie przede wszystkim radyklanie obniży koszty procesu KYC po stronie tych podmiotów, a także istotnie skróci czas, jaki upływa od momentu zainteresowania klienta ofertą do momentu faktycznej sprzedaży towaru lub usługi, eliminując tym samym okazję do rozmyślenia się przed zakupem.
3. Konsument nie jest rozpraszany formalnymi wymogami i może skoncentrować się na tym, co dla niego ważne, czyli na konsumpcji.

System Sygnet występuje więc w roli pośrednika pomiędzy:

- firmą F, która potrzebuje zweryfikować nowego klienta pod kątem KYC,
- a bankiem B, który już dokonał takiej weryfikacji KYC dla danego klienta.

Nasza rola pośrednika jest tu dość mocno uzasadniona, bo dzięki systemowi Sygnet firma F unika dwóch relatywnie trudno wykonalnych kwestii, które bierze na siebie nasz system:

- firma F nie musi uzyskać statusu TPP (Third Party Provider) w PSD2,
- firma F nie musi integrować się z wieloma bankami.

Zakładamy, że korzyści dla firmy F będą na tyle istotne, że uzasadniony stanie się  przepływ finansowy między firmą F i bankiem B, i tym samym powstanie okazja dla nas do pobierania prowizji z tytułu tego przepływu.

## 6. Disclaimer

Niniejszy dokument jest tylko wstępnym zarysem pomysłu (można go potraktować jako tekst wizjonerski). W swojej obecnej formie nie wyczerpuje on wszystkich tematów, które będą wymagać analizy, zanim zdecydujemy się na pójście z propozycją do potencjalnego partnera bankowego i ostatecznie uznamy, że opisane rozwiązanie jest warte wdrożenia.

#### 6.1 Aspekty rynkowe

* Jakie podmioty w Polsce (i Europie) potrzebują weryfikować swoich klientów w zakresie KYC? Jak dużo ich jest?
* Jakie koszty ponoszą podmioty (zarówno bankowe jak i niebankowe) w związku z KYC?
* W jakim kierunku będzie się zmieniać zapotrzebowanie na KYC w przyszłości?

#### 6.3 Aspekty prawne - PSD2

* Czy wedle PSD2 zapytania do bankowego API mogą dotyczyć danych osobowych klienta?
* Jakie są wymagania dotyczące prywatności danych osobowych zgromadzonych w bankach? Banki mogą nie być skore do udostępniania danych klientów, ale mogą zgodzić się na podpisanie tych danych, jeśli będą one w formie zahashowanej.
* Czy PSD2 standaryzuje API, które banki muszą udostępnić, czy nakłada jedynie wymagania funkcjonalne, a techniczne aspekty pozostawia do decyzji poszczególnych banków?

#### 6.4 Aspekty prawne - KYC

* Jakie dokładnie są wymogi KYC finansowego w Polsce (i Europie)? Jakie są wymogi w przypadku stosowania outsourcingu KYC?
* Czy uzyskanie kryptograficznie poświadczonych przez bank danych osobowych klienta jest dopuszczalną formą weryfikacji KYC?

#### 6.5 Aspekty biznesowe

* Czy banki będą skore do wdrożenia dodatkowego API? Wiadomo, że i tak muszą przygotować API spełniające wymagania PSD2, więc potencjalnie dodatkowy *endpoint* nie powinien być kłopotliwy, niemniej jednak przed przystąpieniem do prac warto zbadać ich zainteresowanie, w tym także jakie są ich oczekiwane przychody z udziału w tym przedsięwzięciu.
* Jakie techniczne wymagania nałożą banki na TPP? Na ile podobne będą mechanizmy autoryzacji dostępu do API w różnych bankach? Od tego zależy ile pracy trzeba będzie włożyć w opracowanie systemu Sygnet, i tym samym determinują jego opłacalność.
* Ile banków musiałoby wejść do systemu, aby firmy widziały sens w integracji  z systemem Sygnet. Może się okazać, że dopiero udział dwóch lub trzech dużych banków da nam możliwość weryfikacji tożsamości wystarczająco dużej liczby potencjalnych klientów, żeby taka integracja przyniosła wymierne oszczędności pozwalające na wystarczająco szybkie odzyskanie poniesionych nakładów.
* Jakie są odpowiedzi na te pytania jeżeli chcemy przenieść ten mechanizm do innych krajów UE?