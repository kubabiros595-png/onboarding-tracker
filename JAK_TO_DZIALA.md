# Jak to działa: Onboarding Tracker

Notatka dla mnie, prostym językiem. Wszystkie firmy i osoby w projekcie są wymyślone.

## O co chodzi w jednym zdaniu

Klient podpisuje umowę, system sam układa plan wdrożenia z 8 etapów, a potem codziennie sprawdza, czy coś się nie spóźnia, i od razu mówi, które wdrożenie jest zagrożone.

## 8 etapów wdrożenia i po co każdy

| Etap | Kto odpowiada | Po co |
| --- | --- | --- |
| 1. Kickoff | my | Start: ustalamy zakres, kontakty i plan. Klient wie, co go czeka. |
| 2. Dostęp techniczny | klient | Klient daje nam to, czego potrzebujemy: klucze API, dostęp do środowiska testowego, dane do SSO. Bez tego nic dalej nie ruszy. |
| 3. Integracja zainstalowana | klient (przy samym imporcie CSV: my) | Deweloper klienta podłącza nasz produkt: API, webhook, logowanie SSO. Przy imporcie CSV nie ma nic do instalowania po stronie klienta, więc ten etap robimy my. |
| 4. Pierwsze zdarzenie testowe (przy CSV: pierwszy import testowy) | my | Sprawdzamy, że dane naprawdę płyną. To pierwszy techniczny dowód, że integracja działa. |
| 5. Pierwszy efekt dla klienta (first value) | klient | Klient pierwszy raz widzi korzyść z produktu na swoich danych. Od tego momentu wie, za co płaci. |
| 6. Szkolenie | my | Zespół klienta umie sam korzystać z produktu. |
| 7. Uruchomienie | my | Produkt działa „na prawdziwo” u klienta. |
| 8. Przekazanie do CSM-a | my | Wdrożenie się kończy, klient przechodzi do opiekuna (Customer Success Managera) z notatką: stan, otwarte sprawy, kontakty, następny krok. |

Każdy etap ma termin w **dniach roboczych** (weekend się nie liczy). Integracje dokładają dni do etapu 3: API +4, SSO +3, webhook +2, CSV +1. SSO dokłada też 1 dzień do dostępu technicznego, bo potrzebny jest administrator logowania po stronie klienta.

## Czym jest time-to-first-value

Time-to-first-value (TTFV) to **liczba dni od podpisania umowy do etapu 5, czyli pierwszego efektu dla klienta**. Liczę dni kalendarzowe, bo klient też tak to odczuwa.

Dlaczego to ważne: im dłużej klient płaci i nie widzi efektu, tym większe ryzyko, że się zniechęci. W testach: wdrożenie, które skończyło się przed terminem, miało TTFV 16 dni, a to spóźnione 36 dni. Jeśli pierwszego efektu jeszcze nie było, system pokazuje puste pole, a nie zero, żeby nie zaniżać średniej.

Dwie pozostałe metryki:
- **% etapów na czas**: ile zakończonych etapów zamknięto do terminu.
- **Liczba wdrożeń zagrożonych**: ile projektów ma dziś kolor czerwony.

## Kiedy przypomnienie, kiedy flaga ryzyka

System codziennie patrzy na **bieżący etap** każdego wdrożenia (pierwszy niezakończony).

- **Po terminie** → przypomnienie do właściciela etapu. Jeśli etap jest klienta, przypomnienie idzie do jego kontaktu technicznego, jeśli nasz, do nas.
- **To samo przypomnienie najwyżej co 3 dni robocze.** Jeśli etap był przypomniany mniej niż 3 dni robocze temu, system je wstrzymuje, żeby nikt nie dostawał tego samego maila każdego ranka. Flaga i kolor zostają bez zmian.
- **Etap stoi dłużej niż 5 dni roboczych** bez żadnej zmiany statusu → flaga ryzyka „stoi”. Dokładnie 5 dni to jeszcze nie flaga, 6 już tak (to jest sprawdzone w testach).
- **Klient oznaczył blokadę** (status „zablokowany” z notatką, co blokuje) → flaga „blokuje klient”.
- **Prognoza uruchomienia wypada po docelowej dacie** → flaga „uruchomienie zagrożone”.

Kolor zdrowia:
- **czerwony**: etap stoi, uruchomienie zagrożone albo spóźnienie większe niż 5 dni roboczych;
- **żółty**: etap po terminie albo zablokowany;
- **zielony**: nic z powyższych.

Jedna ważna zasada: jeśli poprzedni etap skończył się późno, następny dostaje swoje normalne dni od momentu, kiedy faktycznie mógł się zacząć. Bez tego jedno opóźnienie robiłoby fałszywe „po terminie” na wszystkich kolejnych etapach i ludzie przestaliby czytać alerty.

Do testów jest zmienna **„dzisiejsza data”** (węzeł *Simulated today*): pusta oznacza prawdziwą datę, a wpisana data pozwala „przewinąć czas” i sprawdzić, co system powie danego dnia.

## Dlaczego plan robią reguły, a AI tylko szkice

- **Plan i flagi to decyzje.** Muszą być takie same za każdym razem dla tych samych danych i muszą się dać wytłumaczyć („czerwony, bo etap stoi 7 dni roboczych”). Reguły w kodzie to dają, AI nie.
- **Maila łatwo poprawić, złej decyzji trudniej.** AI pisze tylko szkice: mail na kickoff i cotygodniowy status po angielsku. Kod podaje AI wyłącznie fakty z tabeli, a potem sprawdza tekst: czy każda data jest w planie, czy jest następny etap z terminem, czy żaden otwarty etap nie jest opisany jako zrobiony, czy zadania klienta są opisane jako jego zadania, czy nie ma „za darmo” i „gwarantujemy”.
- **Szkic idzie do mnie, nie do klienta.** Status zawsze „pending review”. W formularzu `OT Draft Review` widzę szkice z wynikiem kontroli i wybieram: zatwierdź, do poprawy albo odrzuć. Szkic, który nie przeszedł kontroli, mogę zatwierdzić tylko z notatką. „Zatwierdzony” znaczy „gotowy do skopiowania”, nic nie jest wysyłane.
- Nazwa klienta nie idzie do AI. Kod wstawia ją dopiero na końcu.

Czego się nauczyłem: pierwsze 10 szkiców przeszło wszystkie kontrole, a i tak 3 z nich wprowadzały w błąd (pisały „wszystko zgodnie z planem” przy zagrożonym uruchomieniu albo prosiły klienta o plik „do 3 listopada”, czyli do daty z przeszłości). Daty były „z tabeli”, ale nieaktualne. Dlatego teraz AI dostaje stan z codziennej kontroli, a kod sprawdza dodatkowo: terminy z przeszłości, „on track” przy żółtym lub czerwonym i brak realnej daty uruchomienia. Wniosek: kontrola kodem nie zastępuje przeczytania szkicu przez człowieka.

## 5 pytań rekrutacyjnych o ten projekt (po angielsku)

**1. Why do rules build the plan instead of AI?**
The plan and the risk flags are decisions. They must give the same answer for the same data and be easy to explain to a client. Rules in code do that. AI only drafts emails, and every draft is checked by code and reviewed by a person.

**2. What is time-to-first-value and why track it?**
It is the number of days from contract signature to the first result the client sees on their own data. The longer it takes, the higher the risk that the client loses interest. In my test data it was 16 days for the fastest project and 36 for the late one.

**3. How do you avoid false alerts?**
A stage gets a stall flag only after more than 5 business days with no status change, and I tested exactly 5 and 6 days. When an earlier stage finishes late, the next stage still gets its normal time, so one delay does not turn every later stage red. The test set had 0 false flags and 0 missed flags. Reminders are also limited: the same stage is reminded again only after 3 business days, so the client does not get the same email every morning.

**4. What went wrong and what did you change?**
Two things. The first test run scored 35 of 39 because of a bug in the test itself: the scoring node only read the first of five results. Then all 10 AI drafts passed the code checks, but when I read them, 3 used outdated dates or said "on schedule" while the launch was at risk. I fed the drafts the live status and added three more checks.

**5. What would you need before using this with real clients?**
A sync from the CRM and project tool instead of manual forms, real reminder sending with an approval step, a holiday calendar for business days, and thresholds tuned on real project history. It is built and tested on fictional data, not deployed.
