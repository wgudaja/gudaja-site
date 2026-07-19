# GUDAJA — poczta i dostarczalność (home.pl + gudaja.pl)

Cel: skrzynka `wg@gudaja.pl` działa, a maile wychodzące (w tym cold outreach) nie wpadają do spamu. Osiągasz to trzema rekordami: **SPF**, **DKIM**, **DMARC**. Wszystko ustawia się w panelu home.pl.

Gdzie: Panel Klienta home.pl → **Hosting DNS** → przy domenie `gudaja.pl` → **Opcje → Zarządzaj rekordami DNS (WWW/DNS)**.

Uwaga: rekordy pocztowe (SPF/DKIM/DMARC) są niezależne od rekordów strony (A/CNAME na Vercel). Jedno drugiemu nie przeszkadza — poczta idzie do home.pl (rekordy MX), a `www`/apex do Vercel.

---

## 1. Skrzynka

Najpierw upewnij się, że skrzynka `wg@gudaja.pl` w ogóle istnieje: panel home.pl → **Poczta** → utwórz konto `wg@gudaja.pl` (jeśli jeszcze go nie ma). Bez tego formularz i wizytówka wskazują na adres, który nie odbiera.

## 2. SPF (kto może wysyłać w imieniu domeny)

Najprościej: w edytorze DNS home.pl jest gotowy **kreator SPF** — użyj go, sam wstawi poprawny adres serwera home.pl. Jeśli wpisujesz ręcznie, dodaj jeden rekord TXT:

- Typ: `TXT`
- Host / nazwa: `@` (czyli `gudaja.pl`)
- Wartość: `v=spf1 a mx include:_spf.home.pl ~all`

Zasady: **tylko jeden** rekord SPF na domenę. Jeśli w przyszłości dodasz zewnętrznego nadawcę (np. narzędzie do mailingu), dokładasz kolejny `include:` do TEGO samego rekordu, nie tworzysz drugiego.

## 3. DKIM (podpis kryptograficzny wiadomości)

DKIM włącza się po stronie home.pl, bo to home.pl generuje klucz. Panel home.pl → **Poczta** → ustawienia domeny pocztowej → **włącz DKIM** (czasem opisane jako „podpisywanie DKIM"). Home.pl doda wtedy odpowiedni rekord (selektor + klucz publiczny) automatycznie. Jeśli trzeba dodać ręcznie, home.pl pokaże dokładny host (np. `selektor._domainkey`) i wartość — wklej 1:1.

Nie wymyślaj klucza DKIM samodzielnie — musi pochodzić od home.pl.

## 4. DMARC (polityka + raporty)

Dodaj po tym, jak SPF i DKIM już działają. Jeden rekord TXT:

- Typ: `TXT`
- Host / nazwa: `_dmarc`
- Wartość (start, tryb obserwacji): `v=DMARC1; p=none; rua=mailto:wg@gudaja.pl; adkim=s; aspf=s; fo=1`

Po 2–4 tygodniach, gdy raporty potwierdzą, że Twoja poczta przechodzi SPF i DKIM, zaostrz politykę:

- `v=DMARC1; p=quarantine; rua=mailto:wg@gudaja.pl` — podejrzane maile do spamu,
- docelowo `p=reject` — sfałszowane maile odrzucane.

`p=none` na start jest celowe: nic nie blokuje, tylko zbierasz raporty, żeby nie odciąć sobie własnej poczty.

---

## Kolejność i weryfikacja

1. Utwórz skrzynkę `wg@gudaja.pl`.
2. Dodaj SPF (kreator home.pl).
3. Włącz DKIM w panelu poczty home.pl.
4. Dodaj DMARC `p=none`.
5. Po kilku godzinach sprawdź na `https://www.mail-tester.com` — wyślij testowy mail ze skrzynki gudaja.pl na podany tam adres, wynik powinien być blisko 10/10.
6. Po 2–4 tygodniach podnieś DMARC do `quarantine`, potem `reject`.

Narzędzia kontrolne: mail-tester.com, mxtoolbox.com (SPF/DKIM/DMARC lookup).
