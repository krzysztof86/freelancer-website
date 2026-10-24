# ARCHITEKTURA SYSTEMU I PLAN IMPLEMENTACJI

Niniejszy dokument stanowi architekturę techniczną oraz szczegółowy plan wykonawczy dla serwisu osobistego freelancera-programisty (marka osobista, usługi: strony WWW, sklepy e-commerce, aplikacje webowe/dedykowane, marketing/SEO, kursy wideo).

---

## 1. DRZEWO KATALOGÓW

Monorepo zarządzone przez `pnpm workspaces`.

```
.
├── apps/
│   └── web/                                # Aplikacja frontendowa Next.js 14+ (App Router)
│       ├── app/                            # Ścieżki i routing App Router (RSC + Client Components)
│       │   ├── api/                        # Route Handlery (webhooks, revalidate, checkout, lead, contact)
│       │   │   ├── checkout/               # POST /api/checkout - tworzenie sesji Stripe Checkout
│       │   │   ├── contact/                # POST /api/contact/route.ts - zapytanie kontaktowe przez Resend
│       │   │   ├── lead/                   # POST /api/lead/route.ts - zapis leada do WPGraphQL createLead
│       │   │   ├── revalidate/             # POST /api/revalidate - webhook rewalidacji ISR z WP
│       │   │   └── webhooks/               # POST /api/webhooks/stripe - webhook zdarzeń Stripe
│       │   ├── (public)/                   # Strony publiczne (Layout główny, Header, Footer)
│       │   │   ├── page.tsx                # apps/web/app/(public)/page.tsx - Strona główna (13 sekcji)
│       │   │   ├── o-mnie/                 # Widok o mnie
│       │   │   ├── uslugi/                 # Zbiorcza strona usług i podstrony [slug]
│       │   │   ├── realizacje/             # Lista portfolio i detal [slug]
│       │   │   ├── blog/                   # Lista wpisów blogowych i detal [slug]
│       │   │   ├── cennik/                 # Strona cennika z kalkulatorem
│       │   │   ├── kontakt/                # Strona kontaktowa z formularzem wyceny
│       │   │   ├── moje-kursy/             # Publiczny katalog i opis dostępnych kursów
│       │   │   ├── dziekujemy/             # Strona podziękowania po sfinalizowaniu zakupu w Stripe
│       │   │   ├── kursy/                  # Publiczna strona sprzedażowa kursu [slug]
│       │   │   ├── regulamin/              # Strona statyczna regulaminu
│       │   │   └── polityka-prywatnosci/   # Strona statyczna polityki prywatności
│       │   ├── (protected)/                # Strony chronione dla kursantów
│       │   │   └── panel-kursanta/         # Panel kursanta i odtwarzacz lekcji kursu [slug]
│       │   ├── not-found.tsx               # Dedykowana strona 404 App Router
│       │   ├── error.tsx                   # Obsługa błędów App Router
│       │   ├── loading.tsx                 # Widok ładowania App Router
│       │   ├── sitemap.ts                  # Dynamiczna mapa strony XML z listą ścieżek ISR
│       │   └── robots.ts                   # Plik konfiguracyjny robotów indeksujących
│       ├── components/                     # Komponenty interfejsu użytkownika
│       │   ├── ui/                         # Atomowe komponenty UI (Shadcn UI: Button, Input, Select itd.)
│       │   ├── sections/                   # Komponenty sekcji strony głównej (13 sekcji)
│       │   │   ├── hero-section.tsx        # Sekcja 1: Hero
│       │   │   ├── services-overview.tsx   # Sekcja 2: Przegląd usług
│       │   │   ├── value-prop.tsx          # Sekcja 3: Wartość i korzyści
│       │   │   ├── portfolio-preview.tsx   # Sekcja 4: Wybrane realizacje
│       │   │   ├── process-steps.tsx       # Sekcja 5: Proces współpracy
│       │   │   ├── tech-stack.tsx          # Sekcja 6: Stos technologiczny
│       │   │   ├── pricing-teaser.tsx      # Sekcja 7: Zajawka cennika
│       │   │   ├── testimonials.tsx        # Sekcja 8: Opinie klientów
│       │   │   ├── courses-teaser.tsx      # Sekcja 9: Zajawka kursów wideo
│       │   │   ├── about-teaser.tsx        # Sekcja 10: O mnie / bio
│       │   │   ├── faq-section.tsx         # Sekcja 11: Najczęściej zadawane pytania
│       │   │   ├── blog-preview.tsx        # Sekcja 12: Najnowsze wpisy
│       │   │   ├── valuation-cta.tsx       # Sekcja 13: Formularz wyceny CTA
│       │   │   └── contact-map.tsx         # apps/web/components/sections/contact-map.tsx - Mapa kontaktowa Google Embed
│       │   ├── calculator/                 # Interaktywny kalkulator ceny i logika prefillu
│       │   ├── forms/                      # Formularz wyceny i kontaktowy (React Hook Form)
│       │   ├── lms/                        # Odtwarzacz wideo z signed URL, nawigacja lekcji
│       │   └── analytics/                  # Google Tag Manager, Cookiebot CMP script loader
│       ├── lib/                            # Konfiguracja bibliotek frontendowych i pomocniki
│       │   ├── auth.ts                     # Konfiguracja NextAuth.js (magic link z Resend)
│       │   ├── stripe.ts                   # Inicjalizacja SDK Stripe po stronie serwera
│       │   ├── resend.ts                   # apps/web/lib/resend.ts - Klient i konfiguracja SDK Resend
│       │   ├── recaptcha.ts                # apps/web/lib/recaptcha.ts - Weryfikacja tokena ReCAPTCHA v3
│       │   └── video-signer.ts             # Generowanie podpisywanych URL dla Cloudflare Stream/Vimeo
│       ├── public/                         # Statyczne zasoby (ikony, favicony, obrazki fallback)
│       └── tests/                          # Testy jednostkowe, integracyjne (Vitest) i e2e (Playwright)
├── wp-plugin/                              # Dedykowany moduł i środowisko Headless WordPress
│   ├── docker-compose.yml                  # Środowisko deweloperskie WordPress + MySQL + WPGraphQL
│   └── freelancer-cms-core/               # Główna wtyczka zdefiniowana w PHP
│       ├── freelancer-cms-core.php         # Plik główny wtyczki rejestrujący CPT i hooki
│       └── inc/                            # Logika rejestracji CPT, ACF JSON, GraphQL Mutations
│           ├── cpt-registration.php        # Rejestracja 8 Custom Post Types (w tym leady i dostepy)
│           ├── acf-fields/                 # Eksportowane definicje ACF w formacie JSON
│           ├── graphql-mutations.php       # Mutation `createLead` oraz dodawanie wpisu do CPT `dostepy`
│           └── isr-revalidate-hook.php     # Action Scheduler hook wysyłający request do Next.js revalidate
├── packages/                               # Pakiety współdzielone w monorepo
│   └── shared/                             # Pakiet ze schematami, typami i klientem API
│       ├── src/
│       │   ├── schemas/                    # Schematy walidacji Zod (formularz wyceny, kalkulator)
│       │   ├── types/                      # Definicje typów TypeScript dla GraphQL i API
│       │   └── graphql/                    # Zapytania i mutacje .graphql oraz GraphQL Codegen
│       └── package.json
├── .env.example                            # Szablon zmiennych środowiskowych dla aplikacji i WordPressa
├── README.md                               # Instrukcja uruchomienia projektu (pnpm install, docker, codegen)
├── package.json                            # Korzeń monorepo (skrypty pnpm build, dev, test, codegen)
├── pnpm-workspace.yaml                     # Definicja przestrzeni roboczych pnpm
└── ARCHITEKTURA.md                         # Niniejszy dokument architektoniczny
```

---

## 2. MODEL DANYCH (WORDPRESS CPT + ACF)

Każda kolekcja realizowana jest jako Custom Post Type (CPT) w WordPressie z dedykowaną grupą pól ACF (Advanced Custom Fields) wystawioną w WPGraphQL. Wszystkie klucze pól ACF używają wyłącznie znaków ASCII.

### 1. Realizacje (`realizacje`)
- **Pola systemowe**: `ID`, `post_title` (tytuł projektu), `post_status` (`publish`/`draft`), `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `klient_nazwa` | Text | T | Nazwa firmy lub marki klienta |
  | `adres_url` | Url | N | Adres URL wdrożonej strony |
  | `kategoria_uslugi` | Select | T | Opcje: `strona-wp`, `sklep-woocommerce`, `aplikacja-webowa`, `seo-marketing`, `inne` |
  | `skrocony_opis` | Textarea | T | Zajawka na kartę portfolio (max 200 znaków) |
  | `wyzwanie_biznesowe` | Wysiwyg | T | Opis problemu klienta przed wdrożeniem |
  | `zakres_prac` | Repeater | T | Lista podpunktów realizowanych prac |
  | `zastosowane_technologie` | Select (multi) | T | Opcje: `Next.js`, `WordPress`, `WooCommerce`, `TailwindCSS`, `WPGraphQL`, `Cloudflare Stream`, `Stripe`, `SEO` |
  | `wyniki_metryki` | Repeater | N | Klucz-wartość (np. `Konwersja: +45%`, `LCP: 0.8s`) |
  | `glowne_zdjecie` | Image | T | Główne zdjęcie podglądowe mockupa |
  | `galeria_screenow` | Gallery | N | Lista dodatkowych zrzutów ekranu |

### 2. Wpisy blogowe (`post` / `wpisy`)
- **Pola systemowe**: `ID`, `post_title` (tytuł wpisu), `post_name` (slug), `post_status`, `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `zajawka` | Textarea | T | Opis meta/zajawka artykułu |
  | `tresc_artykulu` | Wysiwyg | T | Główna treść artykułu w formacie HTML/Gutenberg |
  | `glowne_zdjecie` | Image | T | Obrazek wyróżniający wpisu |
  | `czas_czytania_minut` | Number | T | Szacowany czas czytania w minutach |
  | `autor_bio` | Group | T | Podgrupa: `imie` (Text), `zdjecie` (Image), `rola` (Text) |
  | `powiazane_uslugi` | Relationship | N | Relacja do CPT `uslugi` lub CPT `realizacje` |

### 3. Opinie (`opinie`)
- **Pola systemowe**: `ID`, `post_title` (nagłówek opinii), `post_status`, `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `autor_imie_nazwisko` | Text | T | Imię i nazwisko zamawiającego |
  | `autor_stanowisko` | Text | T | Stanowisko (np. CEO, Marketing Director) |
  | `autor_firma` | Text | T | Nazwa firmy klienta |
  | `autor_zdjecie` | Image | N | Zdjęcie/awatar autora opinii |
  | `tresc_opinii` | Textarea | T | Pełna treść rekomendacji |
  | `ocena_gwiazdki` | Number | T | Liczba od 1 do 5 |
  | `powiazana_realizacja` | Post Object | N | Relacja 1:1 do CPT `realizacje` |

### 4. Pozycje cennika (`cennik`)
- **Pola systemowe**: `ID`, `post_title` (nazwa pakietu/usługi), `post_status`, `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `service_type_slug` | Select | T | Opcje: `strona-wp`, `sklep-woocommerce`, `aplikacja-webowa`, `seo-marketing`, `inne` |
  | `cena_od_netto` | Number | T | Minimalna kwota netto w PLN dla danego pakietu |
  | `jednostka_rozliczenia` | Select | T | Opcje: `za_projekt`, `miesiecznie`, `za_godzine` |
  | `opis_pakietu` | Textarea | T | Krótki opis dla kogo jest ten pakiet |
  | `lista_funkcji` | Repeater | T | Lista cech/funkcji (tekst + flaga czy zawarte) |
  | `czy_wyrozniony` | True/False | T | Flaga akcentująca pakiet w UI |
  | `etykieta_promocyjna` | Text | N | Etykieta typu "Najpopularniejszy", "Rekomendowany" |
  | `multiplier_srednia` | Number | N | Mnożnik średniej złożoności (np. 1.3) |
  | `multiplier_zaawansowana` | Number | N | Mnożnik zaawansowanej złożoności (np. 1.7) |
  | `cost_extra_subpage` | Number | N | Koszt dodatkowej podstrony powyżej limitu |

### 5. FAQ (`faq`)
- **Pola systemowe**: `ID`, `post_title` (pytanie), `post_status`, `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `odpowiedz` | Wysiwyg | T | Treść odpowiedzi na pytanie |
  | `kategoria_faq` | Select | T | Opcje: `ogolne`, `strony-www`, `sklepy`, `aplikacje`, `kursy`, `platnosci` |
  | `kolejnosc_sortowania` | Number | T | Priorytet wyświetlania na akordeonie |

### 6. Kursy (`kursy`)
- **Pola systemowe**: `ID`, `post_title` (tytuł kursu), `post_name` (slug), `post_status`, `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `cena_brutto` | Number | T | Cena kursu w PLN brutto |
  | `stripe_price_id` | Text | T | Identyfikator ceny z dashboardu Stripe (`price_...`) |
  | `opis_krotki` | Textarea | T | Podsumowanie kursu na kartę sprzedażową |
  | `okladka_kursu` | Image | T | Grafika główna kursu |
  | `dlugosc_ogolna_godzin` | Number | T | Łączny czas trwania materiałów |
  | `lista_lekcji` | Repeater | T | Podgrupa lekcji: `tytul_lekcji` (Text), `video_asset_id` (Text - Cloudflare Stream/Vimeo ID), `czas_trwania_sekund` (Number), `czy_preview` (True/False) |

### 7. Dostępy (`dostepy`)
- **Pola systemowe**: `ID`, `post_title` (automatycznie: `Email - Kurs ID`), `post_status` (`publish`), `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `user_email` | Email | T | Adres e-mail kupującego pobrany ze Stripe Checkout |
  | `zakupiony_kurs` | Post Object | T | Relacja do CPT `kursy` |
  | `stripe_checkout_id` | Text | T | Identyfikator sesji / płatności Stripe (`cs_test_...` lub `pi_...`) |
  | `data_zakupu` | Date Time | T | Data i czas przyznania dostępu |

### 8. Leady z formularzy (`leady`)
- **Pola systemowe**: `ID`, `post_title` (automatycznie: `Lead: Imię - Firma - ServiceType`), `post_status` (`publish`), `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `companyName` | Text | T | Nazwa firmy klienta |
  | `fullName` | Text | T | Imię i nazwisko klienta |
  | `email` | Email | T | Adres email kontaktowy |
  | `phone` | Text | N | Numer telefonu |
  | `serviceType` | Select | T | Opcje: `strona-wp`, `sklep-woocommerce`, `aplikacja-webowa`, `seo-marketing`, `inne` |
  | `budget` | Select | T | Opcje: `do-5k`, `5k-10k`, `10k-20k`, `20k-40k`, `40k+` |
  | `timeline` | Select | T | Opcje: `asap`, `1-month`, `2-3-months`, `flexible` |
  | `description` | Textarea | T | Opis wymagań lub wiadomości |
  | `calculatorPrefill` | Textarea | N | JSON z wyliczonymi szczegółami z kalkulatora ceny |
  | `source` | Select | T | Opcje: `hero`, `kontakt`, `kalkulator` |
  | `status` | Select | T | Opcje: `nowy`, `w-kontakcie`, `zamkniety` (Domyślnie: `nowy`) |

---

## 3. ROUTING (NEXT.JS APP ROUTER)

| Ścieżka URL | Plik `page.tsx` | Renderowanie | Źródło danych | Metadata SEO Source |
| :--- | :--- | :--- | :--- | :--- |
| `/` | `app/(public)/page.tsx` | ISR (`revalidate: 86400` + webhook) | WPGraphQL (13 sekcji: CPT realizacje, cennik, opinie, faq, posty) | WPYoast / Static (Home SEO) |
| `/o-mnie/` | `app/(public)/o-mnie/page.tsx` | Static / ISR | WPGraphQL (Bio, doświadczenie, stack) | WPYoast (Page: O mnie) |
| `/uslugi/` | `app/(public)/uslugi/page.tsx` | Static / ISR | Static + WPGraphQL (Lista usług) | WPYoast (Page: Usługi) |
| `/uslugi/[slug]/` | `app/(public)/uslugi/[slug]/page.tsx` | ISR (`generateStaticParams`) | WPGraphQL (Podstrony usług: strony-www, sklepy-woocommerce, aplikacje-dedykowane, marketing-seo) | WPYoast (Dynamic per service) |
| `/realizacje/` | `app/(public)/realizacje/page.tsx` | ISR | WPGraphQL (Kolekcja CPT `realizacje`) | WPYoast (Page: Portfolio) |
| `/realizacje/[slug]/` | `app/(public)/realizacje/[slug]/page.tsx` | ISR (`generateStaticParams`) | WPGraphQL (Pojedynczy wpis CPT `realizacje`) | WPYoast (CPT realizacje metadata) |
| `/blog/` | `app/(public)/blog/page.tsx` | ISR | WPGraphQL (Kolekcja CPT `post`) | WPYoast (Page: Blog) |
| `/blog/[slug]/` | `app/(public)/blog/[slug]/page.tsx` | ISR (`generateStaticParams`) | WPGraphQL (Pojedynczy wpis CPT `post`) | WPYoast (Post metadata + OpenGraph) |
| `/cennik/` | `app/(public)/cennik/page.tsx` | ISR | WPGraphQL (CPT `cennik` + CPT `faq`) | WPYoast (Page: Cennik) |
| `/kontakt/` | `app/(public)/kontakt/page.tsx` | Static | Formularz Client Component + Resend API / Lead API | WPYoast (Page: Kontakt) |
| `/moje-kursy/` | `app/(public)/moje-kursy/page.tsx` | ISR | WPGraphQL (Katalog i lista dostępnych CPT `kursy`) | WPYoast (Page: Moje Kursy) |
| `/dziekujemy/` | `app/(public)/dziekujemy/page.tsx` | Dynamic (SSR) | Query params ze Stripe Checkout Session ID | Static ("Podziękowanie za zakup") |
| `/kursy/[slug]/` | `app/(public)/kursy/[slug]/page.tsx` | ISR (`generateStaticParams`) | WPGraphQL (CPT `kursy` - strona sprzedażowa) | WPYoast (CPT kursy metadata) |
| `/panel-kursanta/` | `app/(protected)/panel-kursanta/page.tsx` | Dynamic (SSR / NextAuth session) | WPGraphQL Query po `user_email` -> CPT `dostepy` | Static ("Panel Kursanta") |
| `/panel-kursanta/kurs/[slug]/` | `app/(protected)/panel-kursanta/kurs/[slug]/page.tsx` | Dynamic (SSR) | WPGraphQL (CPT `kursy`) + Signed Video URLs API | Static ("Odtwarzacz Kursu") |
| `/regulamin/` | `app/(public)/regulamin/page.tsx` | Static | Static HTML / Markdown | Static Metadata |
| `/polityka-prywatnosci/` | `app/(public)/polityka-prywatnosci/page.tsx` | Static | Static HTML / Markdown | Static Metadata |

---

## 4. SCHEMA.ORG (JSON-LD)

| Route | Typy Schema | Wymagane pola Schema | Źródło danych w WP / App |
| :--- | :--- | :--- | :--- |
| `/` | `WebSite`, `Person`, `Organization` | `@id`, `name`, `url`, `logo`, `sameAs`, `address`, `telephone`, `priceRange` | Statyczne dane konfiguracji marki + WP site options |
| `/o-mnie/` | `Person`, `BreadcrumbList` | `name`, `jobTitle`, `worksFor`, `knowsAbout`, `sameAs`, `itemListElement` | Statyczna konfiguracja bio + URL |
| `/uslugi/` | `BreadcrumbList` | `itemListElement` | Statyczna struktura nawigacji |
| `/uslugi/[slug]/` | `Service`, `BreadcrumbList` | `name`, `serviceType`, `provider` (`@id` Organization), `areaServed`, `description` | WPGraphQL CPT Usługi / Statyczny szablon usługi |
| `/realizacje/[slug]/` | `BreadcrumbList` | `itemListElement` | WPGraphQL CPT `realizacje` |
| `/blog/[slug]/` | `Article`, `BreadcrumbList` | `headline`, `image`, `datePublished`, `dateModified`, `author` (`@id` Person), `publisher` (`@id` Organization) | WPGraphQL CPT `post` |
| `/cennik/` | `FAQPage`, `BreadcrumbList` | `mainEntity` (tablica pytania/odpowiedzi `Question`/`Answer`) | WPGraphQL CPT `faq` |
| `/kontakt/` | `BreadcrumbList` | `name`, `telephone`, `email`, `address` | Statyczna konfiguracja firmy |

### Powiązanie encji za pomocą `@id`:
- **Organization ID**: `https://domena-freelancera.pl/#organization`
- **Person ID**: `https://domena-freelancera.pl/#person`
- **WebSite ID**: `https://domena-freelancera.pl/#website`

W schematach podstron `provider` lub `author` odwołuje się bezpośrednio do `{"@id": "https://domena-freelancera.pl/#person"}`.

---

## 5. INTEGRACJE ZEWNĘTRZNE

### Tabela integracji usług zewnętrznych:

| Integracja | Miejsce inicjalizacji w kodzie | Wymagane zmienne środowiskowe | Fallback przy braku klucza (Dev Mode) |
| :--- | :--- | :--- | :--- |
| **Resend** | `apps/web/lib/resend.ts` & `app/api/contact/route.ts` & `app/api/lead/route.ts` | `RESEND_API_KEY`, `RESEND_FROM_EMAIL`, `RESEND_TO_EMAIL` | Logowanie treści wiadomości e-mail w konsoli serwera (`console.log`), zwrot odpowiedzi statusu `200 OK` do formularza. |
| **WPGraphQL Mutation createLead** | `app/api/lead/route.ts` & `wp-plugin/freelancer-cms-core/inc/graphql-mutations.php` | `WORDPRESS_API_URL`, `WP_APPLICATION_PASSWORD_USER`, `WP_APPLICATION_PASSWORD` | Zapis leada do lokalnego pliku JSON / mock response `200 OK` w środowisku deweloperskim bez połaczenia z WP. |
| **WPGraphQL Auth** | `apps/web/lib/wp-client.ts` | `WP_APPLICATION_PASSWORD_USER`, `WP_APPLICATION_PASSWORD` | Basic Auth używany dla mutacji leadów, tworzenia dostępów oraz buildów SSG; publiczne zapytania read-only chronione przez CORS. |
| **Stripe** | `apps/web/lib/stripe.ts` & `app/api/checkout/route.ts` & `app/api/webhooks/stripe/route.ts` | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Powiadomienie błędu w interfejsie checkoutu: "Płatności są wyłączone w środowisku deweloperskim". |
| **Cookiebot** | `apps/web/components/analytics/cookiebot.tsx` | `NEXT_PUBLIC_COOKIEBOT_DOMAIN_GROUP_ID` | Komponent CMP nie renderuje skryptu zewnętrznego; analityka traktowana jako domyślnie odrzucona. |
| **GTM / GA4** | `apps/web/components/analytics/gtm.tsx` | `NEXT_PUBLIC_GTM_ID`, `NEXT_PUBLIC_GA4_ID` | Brak ładowania skryptów gtm.js / gtag.js; zdarzenia `dataLayer.push` przechwytywane przez pustą funkcję mocka. |
| **Google Maps Embed** | `apps/web/components/sections/contact-map.tsx` | `NEXT_PUBLIC_GOOGLE_MAPS_EMBED_KEY` | Wyświetlenie statycznej podglądowej grafiki karty mapy z adresem i przyciskiem nawigacji do Google Maps. |
| **ReCAPTCHA v3** | `apps/web/lib/recaptcha.ts` | `NEXT_PUBLIC_RECAPTCHA_SITE_KEY`, `RECAPTCHA_SECRET_KEY` | Pominięcie weryfikacji walidacji tokena po stronie serwera (`score = 1.0` w dev mode). |

### ISR Revalidation Webhook:

| Element | Specyfikacja techniczna |
| :--- | :--- |
| **Hook WordPress** | `save_post` (dla CPT `realizacje`, `post`, `kursy`, `cennik`, `faq`) oraz `edited_term` (dla taksonomii `kategoria-realizacji`) |
| **Payload Webhooka (JSON)** | `{ "secret": "REVALIDATE_SECRET_TOKEN", "paths": ["/", "/cennik", "/realizacje/nazwa-projektu"] }` |
| **Endpoint Next.js** | `POST /api/revalidate` |
| **Autoryzacja** | Współdzielony sekret w zmiennej środowiskowej `REVALIDATE_SECRET` w nagłówku lub ciele żądania |
| **Fallback (Safety Net)** | Domyślne `revalidate: 86400` w App Routerze (odświeżanie raz na 24h w przypadku awarii webhooka) |
| **Obsługa po stronie WP** | Asynchroniczna wysyłka z wykorzystaniem **Action Scheduler** (uniknięcie blokowania panelu WP) |

---

## 6. FORMULARZ WYCENY — MACIERZ PÓL

Formularz wyceny obsługuje dynamicznie zmienny zestaw pól w zależności od wybranego typu usługi (`serviceType`), walidowany przy użyciu biblioteki Zod (`packages/shared/src/schemas/valuation.ts`).

| Typ usługi (`serviceType`) | Widoczne pola | Typ pola | Walidacja Zod | Wymagane (T/N) |
| :--- | :--- | :--- | :--- | :---: |
| **Wszystkie typy (Pola bazowe)** | `companyName` | Text | `z.string().min(2)` | T |
| | `fullName` | Text | `z.string().min(3)` | T |
| | `email` | Email | `z.string().email()` | T |
| | `phone` | Tel | `z.string().regex(/^[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s\./0-9]*$/).optional().or(z.literal(''))` | N |
| | `serviceType` | Select | `z.enum(['strona-wp', 'sklep-woocommerce', 'aplikacja-webowa', 'seo-marketing', 'inne'])` | T |
| | `budget` | Select | `z.enum(['do-5k', '5k-10k', '10k-20k', '20k-40k', '40k+'])` | T |
| | `timeline` | Select | `z.enum(['asap', '1-month', '2-3-months', 'flexible'])` | T |
| | `description` | Textarea | `z.string().min(20, 'Opis musi mieć min. 20 znaków')` | T |
| | `website` (Honeypot) | Text (hidden) | `z.string().max(0, 'Bot detected')` (ukryte w CSS) | N (musi być puste) |
| **`strona-wp`** | `hasExistingSite` | Radio/Select | `z.boolean()` | T |
| | `existingSiteUrl` | Url | `z.string().url().optional().or(z.literal(''))` | N |
| | `wpFeatures` | Checkbox Group | `z.array(z.enum(['blog', 'multilingual', 'cms-custom', 'seo-advanced', 'speed-optimization']))` | N |
| **`sklep-woocommerce`** | `productCountRange` | Select | `z.enum(['1-50', '51-500', '501-5000', '5000+'])` | T |
| | `integrationsNeeded` | Checkbox Group | `z.array(z.enum(['płatności-blik-card', 'kurierzy-apaczka-inpost', 'erp-faktury', 'allegro-baselinker']))` | N |
| **`aplikacja-webowa`** | `appType` | Select | `z.enum(['saas', 'portal-klienta', 'mvp', 'dysk-internal-system'])` | T |
| | `authRequirements` | Checkbox Group | `z.array(z.enum(['email-pass', 'social-auth', 'sso-saml', 'role-permissions']))` | N |
| **`seo-marketing`** | `seoScope` | Select | `z.enum(['local', 'national'])` | T |
| | `seoKeywordsCount` | Select | `z.enum(['10-50', '50-200', '200+'])` | T |
| **`inne`** | `customScope` | Textarea | `z.string().min(10)` | T |

---

## 7. KALKULATOR CENY — ALGORYTM

Kalkulator ceny jest osobnym interaktywnym komponentem na stronie `/cennik/`. Ceny i mnożniki pobierane są dynamicznie z CPT `cennik` przez WPGraphQL.

### Struktura `PriceConfig` i zapytanie GraphQL:

```typescript
export interface PriceConfigItem {
  serviceTypeSlug: 'strona-wp' | 'sklep-woocommerce' | 'aplikacja-webowa' | 'seo-marketing' | 'inne';
  basePriceNetto: number;
  multiplierSrednia: number;
  multiplierZaawansowana: number;
  costExtraSubpage: number;
}

export interface PriceConfig {
  items: Record<string, PriceConfigItem>;
  featureCosts: Record<string, number>;
  expressDeliveryMultiplier: number;
}
```

#### GraphQL Query (`GetPriceConfig.graphql`):
```graphql
query GetPriceConfig {
  cennikItems(first: 50) {
    nodes {
      cennikFields {
        serviceTypeSlug
        cenaOdNetto
        multiplierSrednia
        multiplierZaawansowana
        costExtraSubpage
      }
    }
  }
}
```

### Algorytm kalkulatora (TypeScript):

```typescript
export interface CalculatorInput {
  serviceType: 'strona-wp' | 'sklep-woocommerce' | 'aplikacja-webowa' | 'seo-marketing' | 'inne';
  complexity: 'podstawowa' | 'średnia' | 'zaawansowana';
  subpagesCount: number;
  additionalFeatures: Array<'multilingual' | 'animations' | 'cms-custom' | 'speed-opt' | 'payment-integration' | 'erp-sync'>;
  expressDelivery: boolean;
}

export interface CalculatorResult {
  estimatedPriceNettoMin: number;
  estimatedPriceNettoMax: number;
  summaryText: string;
}

export function calculateProjectEstimate(
  input: CalculatorInput,
  priceConfig: PriceConfig
): CalculatorResult {
  const config = priceConfig.items[input.serviceType] || {
    basePriceNetto: 5000,
    multiplierSrednia: 1.3,
    multiplierZaawansowana: 1.7,
    costExtraSubpage: 350,
  };

  const complexityMultipliers: Record<CalculatorInput['complexity'], number> = {
    'podstawowa': 1.0,
    'średnia': config.multiplierSrednia || 1.3,
    'zaawansowana': config.multiplierZaawansowana || 1.7,
  };

  const extraSubpages = Math.max(0, input.subpagesCount - 5);
  let totalMin = (config.basePriceNetto * complexityMultipliers[input.complexity]) + (extraSubpages * config.costExtraSubpage);

  for (const feature of input.additionalFeatures) {
    if (priceConfig.featureCosts[feature]) {
      totalMin += priceConfig.featureCosts[feature];
    }
  }

  if (input.expressDelivery) {
    totalMin *= (priceConfig.expressDeliveryMultiplier || 1.30);
  }

  const roundedMin = Math.round(totalMin / 100) * 100;
  const roundedMax = Math.round((roundedMin * 1.25) / 100) * 100;

  return {
    estimatedPriceNettoMin: roundedMin,
    estimatedPriceNettoMax: roundedMax,
    summaryText: `Szacunkowa wycena: od ${roundedMin.toLocaleString('pl-PL')} PLN netto (zakres szacunkowy: ${roundedMin.toLocaleString('pl-PL')} - ${roundedMax.toLocaleString('pl-PL')} PLN netto)`,
  };
}
```

### Przekazanie prefillu kalkulatora do formularza wyceny:
1. Przycisk w kalkulatorze: **„Wyślij zapytanie na podstawie tej wyceny”**.
2. Przekierowanie na `/kontakt/?service=...&budget=...&calc_min=...&calc_max=...&features=...`.
3. Komponent `ValuationForm` odczytuje parametry z `useSearchParams()`, wstępnie wypełnia polach i przekazuje podsumowanie JSON w polu `calculatorPrefill`.
4. **Zastrzeżenie w UI**: *„Prezentowana kwota jest szacunkiem wstępnym wyliczonym dynamicznie i nie stanowi oferty handlowej w rozumieniu Art. 66 Par. 1 Kodeksu Cywilnego.”*

---

## 8. PLAN PRACY — ROZBICIE NA ZADANIA

Podział projektu na 42 granularne zadania implementacyjne (szacowany czas realizacji pojedynczego zadania: 10–30 minut).

| ID | Nazwa zadania | Zależności | Kryteria akceptacji |
| :---: | :--- | :---: | :--- |
| **TASK-01** | Inicjalizacja Monorepo pnpm & TypeScript | - | 1. Tworzy pliki `pnpm-workspace.yaml`, `package.json`, `.gitignore`.<br>2. Konfiguruje bazowy `tsconfig.json` z trybem `strict: true`.<br>3. Pomyślne wykonanie `pnpm install`. |
| **TASK-02** | Pliki konfiguracyjne i dokumentacja setupu | TASK-01 | 1. Tworzy plik `.env.example` ze wszystkimi nazwami zmiennych środowiskowych.<br>2. Tworzy `README.md` z krokami setupu (`pnpm install`, `docker compose up`, `pnpm codegen`, `pnpm dev`). |
| **TASK-03** | Konfiguracja środowiska Docker dla WordPress | TASK-01 | 1. Plik `wp-plugin/docker-compose.yml` uruchamia kontenery WordPress, MySQL, Mailpit.<br>2. Dostępność panelu WP pod `localhost:8000`. |
| **TASK-04** | Inicjalizacja wtyczki PHP `freelancer-cms-core` | TASK-03 | 1. Plik `freelancer-cms-core.php` aktywuje się w panelu WP.<br>2. Rejestruje stałe i strukturę katalogów w `wp-plugin/freelancer-cms-core/inc/`. |
| **TASK-05** | Rejestracja 8 CPT w pluginie PHP | TASK-04 | 1. Kod PHP rejestruje 8 CPT: `realizacje`, `post`, `opinie`, `cennik`, `faq`, `kursy`, `dostepy`, `leady`.<br>2. Wszystkie CPT widoczne w menu WordPress Admin. |
| **TASK-06** | Definicje pól ACF w JSON (pola ASCII) | TASK-05 | 1. Pliki JSON w `inc/acf-fields/` definiują komplet pól z Sekcji 2 bez polskich znaków w kluczach.<br>2. Pola widoczne w edytorze każdego CPT w panelu WP. |
| **TASK-07** | Instalacja i aktywacja WPGraphQL & ACF GraphQL | TASK-05 | 1. Wtyczki WPGraphQL, WPGraphQL for ACF, WPGraphQL Yoast są aktywne.<br>2. Endpoint `/graphql` odpowiada na zapytanie Introspection. |
| **TASK-08** | Mutacja WPGraphQL `createLead` i obsługa zapisów | TASK-07 | 1. Plik `inc/graphql-mutations.php` rejestruje mutację `createLead`.<br>2. Mutacja przyjmuje pola z Sekcji 2 CPT `leady` i tworzy wpis o statusie `nowy`. |
| **TASK-09** | Action Scheduler Hook dla ISR Revalidation | TASK-07 | 1. Plik `inc/isr-revalidate-hook.php` podczepia pod `save_post` i `edited_term` asynchroniczny task w Action Scheduler.<br>2. Wysyła request POST do Next.js `/api/revalidate`. |
| **TASK-10** | Tworzenie struktury Next.js 14 App Router | TASK-01 | 1. Aplikacja ustrukturyzowana w `apps/web/app`.<br>2. Tailwind CSS, Shadcn UI oraz Framer Motion zainstalowane i skonfigurowane w `tailwind.config.ts`. |
| **TASK-11** | Setup pakietu `@freelancer/shared` & GraphQL Codegen | TASK-01 | 1. Pliki zapytań `.graphql` umieszczone w `packages/shared/src/graphql`.<br>2. Skrypt `pnpm codegen` generuje typowane React Hooks i typy TS z schematu WPGraphQL. |
| **TASK-12** | Schematy walidacji Zod w `@freelancer/shared` | TASK-11 | 1. Exporter schematów Zod dla formularza wyceny (z enum `serviceType`, budget, honeypot), kalkulatora i kontaktowego.<br>2. Testy jednostkowe walidatorów Zod przechodzą pomyślnie. |
| **TASK-13** | Dedykowane strony błędu i ładowania w App Router | TASK-10 | 1. Tworzy pliki `not-found.tsx`, `error.tsx`, `loading.tsx` w `apps/web/app/`.<br>2. Wyświetlają dopracowane komponenty UI z możliwością ponowienia akcji. |
| **TASK-14** | Dynamiczna mapa strony i plik robotów (`sitemap.ts`, `robots.ts`) | TASK-10, TASK-11 | 1. Plik `sitemap.ts` generuje XML z listą wszystkich statycznych i dynamicznych ścieżek ISR.<br>2. Plik `robots.ts` definiuje reguły dla botów i wskazuje sitemap. |
| **TASK-15** | Implementacja Layoutu głównego (Header, Footer, Nav) | TASK-10 | 1. Komponenty `Header` z menu mobilnym i `Footer` w `apps/web/components/ui/`.<br>2. Prawidłowa nawigacja i responsywność RWD. |
| **TASK-16** | Strona Główna — Sekcja 1: Hero Section | TASK-15 | 1. Komponent `hero-section.tsx` z animacją wejścia Framer Motion.<br>2. Przyciski CTA prowadzące do wyceny i portfolio. |
| **TASK-17** | Strona Główna — Sekcja 2: Przegląd Usług | TASK-15 | 1. Komponent `services-overview.tsx` prezentujący 5 głównych usług w siatce kart. |
| **TASK-18** | Strona Główna — Sekcja 3: Wartość i Korzyści | TASK-15 | 1. Komponent `value-prop.tsx` z wyróżnikami oferty freelancera. |
| **TASK-19** | Strona Główna — Sekcja 4: Wybrane Realizacje | TASK-11, TASK-15 | 1. Komponent `portfolio-preview.tsx` pobierający 3 realizacje z WPGraphQL.<br>2. Obsługa kart z tagami technologii. |
| **TASK-20** | Strona Główna — Sekcja 5: Proces Współpracy | TASK-15 | 1. Komponent `process-steps.tsx` z osi czasu kroku po kroku. |
| **TASK-21** | Strona Główna — Sekcja 6: Stos Technologiczny | TASK-15 | 1. Komponent `tech-stack.tsx` z logotypami i opisem klasycznych oraz modern web stacków. |
| **TASK-22** | Strona Główna — Sekcja 7: Zajawka Cennika | TASK-15 | 1. Komponent `pricing-teaser.tsx` prezentujący pakiety cenowe z przyciskiem do kalkulatora. |
| **TASK-23** | Strona Główna — Sekcja 8: Opinie Klientów | TASK-11, TASK-15 | 1. Komponent `testimonials.tsx` wyciągający dane z CPT `opinie` z ocenami gwiazdkowymi. |
| **TASK-24** | Strona Główna — Sekcja 9: Zajawka Kursów Wideo | TASK-11, TASK-15 | 1. Komponent `courses-teaser.tsx` pobierający dostępne kursy cyfrowe. |
| **TASK-25** | Strona Główna — Sekcja 10: O mnie / Bio | TASK-15 | 1. Komponent `about-teaser.tsx` z bio i zdjęciem programisty. |
| **TASK-26** | Strona Główna — Sekcja 11: FAQ Section | TASK-11, TASK-15 | 1. Komponent `faq-section.tsx` wykorzystujący akordeon Shadcn UI ze sterowaniem animacji. |
| **TASK-27** | Strona Główna — Sekcja 12: Najnowsze Wpisy Blogowe | TASK-11, TASK-15 | 1. Komponent `blog-preview.tsx` wyświetlający 3 najnowsze artykuły z bloga. |
| **TASK-28** | Strona Główna — Sekcja 13: Formularz Wyceny CTA | TASK-12, TASK-15 | 1. Komponent `valuation-cta.tsx` zamontowany w stopce sekcji głównej. |
| **TASK-29** | Podstrona O mnie (`/o-mnie/`) | TASK-15 | 1. Strona statyczna z pełną historią, certyfikatami i stosem technologicznym.<br>2. Dodany JSON-LD `Person`. |
| **TASK-30** | Podstrony Usług (`/uslugi/` & `/uslugi/[slug]/`) | TASK-11, TASK-15 | 1. Szablon podstrony dla 5 typów usług (`generateStaticParams`).<br>2. Prawidłowy JSON-LD `Service`. |
| **TASK-31a** | Lista Portfolio z filtracją (`/realizacje/`) | TASK-11, TASK-15 | 1. Strona listy z filtracją wg taksonomii/kategorii usług. |
| **TASK-31b** | Detal Portfolio (`/realizacje/[slug]/`) | TASK-31a | 1. Podstrona pojedynczego projektu z wyzwaniem biznesowym, metrykami i galerią screenów. |
| **TASK-32a** | Lista Bloga (`/blog/`) | TASK-11, TASK-15 | 1. Strona listy wpisów z czasem czytania i podziałem na kategorie. |
| **TASK-32b** | Detal Artykułu Blogowego (`/blog/[slug]/`) | TASK-32a | 1. Strona artykułu z renderowaniem HTML, sekcją bio autora i JSON-LD `Article`. |
| **TASK-33** | Podstrona Cennika & Kalkulator z dynamicznym PriceConfig (`/cennik/`) | TASK-11, TASK-12, TASK-15 | 1. Server Component pobiera `PriceConfig` z CPT `cennik` przez GraphQL i przekazuje do Client Component kalkulatora.<br>2. Kalkulator przelicza dynamicznie kwoty i umożliwia przekierowanie z prefill.<br>3. Test jednostkowy logiki (Vitest) pokrywa happy path + 2 przypadki brzegowe. |
| **TASK-34** | Podstrona Kontaktowa & Formularz Wyceny (`/kontakt/`) | TASK-12, TASK-15 | 1. Komponent formularza z obsługą prefilla z query params, polem honeypot i ReCAPTCHA v3.<br>2. Test jednostkowy logiki (Vitest) pokrywa happy path + 2 przypadki brzegowe. |
| **TASK-35** | Integracja Route Handler Contact & Lead API | TASK-34 | 1. Handler `app/api/lead/route.ts` waliduje dane Zod, wysyła mutację `createLead` do WP i powiadomienie przez Resend.<br>2. Test jednostkowy logiki (Vitest) pokrywa happy path + 2 przypadki brzegowe. |
| **TASK-36** | Moduł LMS — Checkout Stripe & Webhook (`/api/checkout`) | TASK-11 | 1. Handler `app/api/checkout` tworzy sesję Stripe Checkout dla CPT `kursy`.<br>2. Webhook `/api/webhooks/stripe` zapisuje zakup do CPT `dostepy` w WP.<br>3. Test jednostkowy logiki (Vitest) pokrywa happy path + 2 przypadki brzegowe. |
| **TASK-37a** | LMS — NextAuth Magic Link & Konfiguracja Resend | TASK-36 | 1. Konfiguracja NextAuth.js z dostawcą Email przez Resend SDK. |
| **TASK-37b** | LMS — Middleware Ochrony Ścieżek Kursanta | TASK-37a | 1. Middleware Next.js weryfikujący sesję na ścieżkach `/panel-kursanta/*`. |
| **TASK-37c** | LMS — Weryfikacja Dostępów Kursanta po Emailu | TASK-37b | 1. Zapytanie do WPGraphQL pobierające wpisy CPT `dostepy` dla zalogowanego e-maila i odblokowujące kursy. |
| **TASK-38** | Moduł LMS — Odtwarzacz Wideo z Signed URL | TASK-37c | 1. Route handler generujący signed URL dla lekcji z Cloudflare Stream/Vimeo.<br>2. Odtwarzacz lekcji na chronionej ścieżce `/panel-kursanta/kurs/[slug]`. |
| **TASK-39** | Podstrona Katalogu Kursów (`/moje-kursy/`) & Podziękowania (`/dziekujemy/`) | TASK-36 | 1. Strona `/moje-kursy/` wyświetlająca publiczne karty kursów.<br>2. Strona `/dziekujemy/` weryfikująca sesję Stripe Checkout. |
| **TASK-40** | Integracja Analityki, GTM & Cookiebot CMP | TASK-15 | 1. Skrypty Cookiebot i GTM z integracją Consent Mode v2.<br>2. Przechwytywanie zdarzeń `form_submit` i `calculator_use`. |
| **TASK-41** | Testy End-to-End (Playwright) | TASK-35 | 1. Test e2e ścieżki: wypełnienie formularza wyceny -> zapis leada w WP przez `createLead` -> wysyłka e-maila w Resend (mock). |
| **TASK-42** | Konfiguracja Wdrożenia i CI/CD | TASK-02 | 1. Konfiguracja wdrożenia Next.js na Vercel z wprowadzonymi zmiennymi produkcyjnymi.<br>2. Konfiguracja środowiska Staging WordPress na serwerze docelowym. |

---

## 9. RYZYKA I DECYZJE OTWARTE

Wykaz decyzji architektonicznych i technologicznych, które wymagają zatwierdzenia przez Architekta / Zamawiającego przed fazą realizacyjną.

| ID | ZAGADNIENIE / DECYZJA | OPCJE DO WYBORU | REKOMENDACJA ARCHITEKTA |
| :---: | :--- | :--- | :--- |
| **DEC-01** | **Hosting dla Headless WordPress** | **Opcja A**: Managed WordPress (Kinsta / WP Engine)<br>**Opcja B**: VPS (Hetzner / DigitalOcean) z Docker / CapRover<br>**Opcja C**: Cloudways / CyberFolks | **Rekomendacja: Opcja B (VPS Hetzner)**. Zapewnia najniższe opóźnienia terytorialne w PL, pełną kontrolę nad Dockerem z WPGraphQL i niski stały koszt miesięczny. |
| **DEC-02** | **Hosting i Podpisywanie Plików Wideo LMS** | **Opcja A**: Cloudflare Stream (signed URL / HMAC token)<br>**Opcja B**: Vimeo OTT / Pro (Domain restriction + JWT)<br>**Opcja C**: AWS S3 + CloudFront Signed URLs | **Rekomendacja: Opcja A (Cloudflare Stream)**. Najprostsze API, natywna obsługa odtwarzacza HLS, bardzo dobre zabezpieczenie przed pobieraniem i niskie koszty za minutę streamingu. |
| **DEC-03** | **Zarządzanie Sesją i Autoryzacją Kursanta** | **Opcja A**: NextAuth.js z Passwordless Magic Link (Resend)<br>**Opcja B**: Supabase Auth (tylko do obsługi logowania)<br>**Opcja C**: WordPress Native Auth JWT | **Rekomendacja: Opcja A (NextAuth.js Magic Link)**. Eliminacja konieczności pamiętania haseł przez kursantów, bezpośrednie mapowanie e-maila zakupu ze Stripe bez dodatkowego zewnętrznego providera bazodanowego. |
| **DEC-04** | **Platforma CMP dla Consent Mode v2** | **Opcja A**: Cookiebot (oficjalna wtyczka / skrypt)<br>**Opcja B**: CookieFirst<br>**Opcja C**: Customowy banner zgodny z GTM Consent Mode v2 | **Rekomendacja: Opcja A (Cookiebot)**. Pełna automatyzacja skanowania ciasteczek i natywna integracja z szablonem GTM Google Consent Mode v2. |
| **DEC-05** | **Rozszerzenie Typów Schema.org w Przyszłości** | **Opcja A**: Pozostanie ściśle przy zdefiniowanej liście (Person, Organization, Service, FAQPage, Article)<br>**Opcja B**: Rozszerzenie w kolejnej iteracji o `Course` i `Offer` dla strony kursu | **Rekomendacja: Opcja B**. Aktualnie zachowujemy 1:1 listę ze specyfikacji, lecz w sekcji kursowej w przyszłości warto dodać Schema `Course` i `Offer` w celu uzyskania Rich Snippets w wyszukiwarce. |
| **DEC-06** | **Mechanizm Obsługi Błędów w Wysyłce Webhooka ISR z WP** | **Opcja A**: Synchroniczny request HTTP z cURL przy `save_post`<br>**Opcja B**: Asynchroniczna kolejka WP Action Scheduler | **Rekomendacja: Opcja B (Action Scheduler)**. Zapobiega opóźnieniom i zawieszaniu panelu administracyjnego WordPressa w przypadku chwilowej niedostępności API Next.js. |
| **DEC-07** | **Wykorzystanie WooCommerce do Sprzedaży Kursów** | **Opcja A**: Kursy tylko przez Stripe w Next.js + zapis do CPT `dostepy` (BEZ WooCommerce)<br>**Opcja B**: Zainstalowanie i konfiguracja WooCommerce na WordPressie | **Rekomendacja: Opcja A**. Mniej ruchomych części, szybszy proces checkoutu w Next.js oraz natywna integracja ze Stripe bez konieczności utrzymywania ciężkiej wtyczki WooCommerce w MVP. |
| **DEC-08** | **Obsługa Trybu Podglądu (Preview / Draft Mode)** | **Opcja A**: Natywny Next.js Draft Mode + edycja w panelu WP z podglądem na żywo<br>**Opcja B**: Pomijamy podgląd w MVP, praca na opublikowanych wpisach | **Rekomendacja: Opcja B dla MVP** (wdrożenie natywnego Draft Mode w 2. iteracji po uruchomieniu podstawowej wersji serwisu). |
| **DEC-09** | **Strategia Kopii Zapasowych i Migracji Bazy WP** | **Opcja A**: Wtyczka WP Migrate DB Pro / All-in-One WP Migration<br>**Opcja B**: Zautomatyzowane zrzuty zwięzłe MySQL (mysqldump) przez cron na VPS<br>**Opcja C**: Automatyczne backupy po stronie dostawcy hostingowego | **Rekomendacja: Zależy od wyboru DEC-01** (jeśli VPS - Opcja B; jeśli hosting managed - Opcja C). |
| **DEC-10** | **Publikacja Fizycznego Adresu w Schema LocalBusiness** | **Opcja A**: Publikacja pełnego adresu zamieszkania/biura i użycie typu `LocalBusiness`<br>**Opcja B**: Użycie wyłącznie encji `Person` + `areaServed` (Polska / Zdalnie) bez adresu fizycznego | **Rekomendacja: Opcja B**, jeśli praca odbywa się 100% zdalnie bez przyjmowania klientów w biurze stacjonarnym. |

---
