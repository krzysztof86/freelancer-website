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
│       │   ├── api/                        # Route Handlery (webhooks, revalidate, checkout, magic-link)
│       │   │   ├── checkout/               # POST /api/checkout - tworzenie sesji Stripe Checkout
│       │   │   ├── revalidate/             # POST /api/revalidate - webhook rewalidacji ISR z WP
│       │   │   └── webhooks/               # POST /api/webhooks/stripe - webhook zdarzeń Stripe
│       │   ├── (public)/                   # Strony publiczne (Layout główny, Header, Footer)
│       │   │   ├── o-mnie/                 # Widok o mnie
│       │   │   ├── uslugi/                 # Zbiorcza strona usług i podstrony [slug]
│       │   │   ├── realizacje/             # Lista portfolio i detal [slug]
│       │   │   ├── blog/                   # Lista wpisów blogowych i detal [slug]
│       │   │   ├── cennik/                 # Strona cennika z kalkulatorem
│       │   │   ├── kontakt/                # Strona kontaktowa z formularzem wyceny
│       │   │   ├── kursy/                  # Publiczna strona sprzedażowa kursu [slug]
│       │   │   ├── regulamin/              # Strona statyczna regulaminu
│       │   │   └── polityka-prywatnosci/   # Strona statyczna polityki prywatności
│       │   └── (protected)/                # Strony chronione dla kursantów
│       │       └── panel-kursanta/         # Panel kursanta i odtwarzacz lekcji kursu [slug]
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
│       │   │   └── valuation-cta.tsx       # Sekcja 13: Formularz wyceny CTA
│       │   ├── calculator/                 # Interaktywny kalkulator ceny i logika prefillu
│       │   ├── forms/                      # Formularz wyceny i kontaktowy (React Hook Form)
│       │   ├── lms/                        # Odtwarzacz wideo z signed URL, nawigacja lekcji
│       │   └── analytics/                  # Google Tag Manager, Cookiebot CMP script loader
│       ├── lib/                            # Konfiguracja bibliotek frontendowych i pomocniki
│       │   ├── auth.ts                     # Konfiguracja NextAuth.js (magic link z Resend)
│       │   ├── stripe.ts                   # Inicjalizacja SDK Stripe po stronie serwera
│       │   └── video-signer.ts             # Generowanie podpisywanych URL dla Cloudflare Stream/Vimeo
│       ├── public/                         # Statyczne zasoby (ikony, favicony, obrazki fallback)
│       └── tests/                          # Testy jednostkowe i integracyjne (Jest/Vitest, React Testing Library)
├── wp-plugin/                              # Customowy plugin PHP dla Headless WordPress
│   └── freelancer-cms-core/               # Główna wtyczka zdefiniowana w PHP
│       ├── freelancer-cms-core.php         # Plik główny wtyczki rejestrujący CPT i hooki
│       ├── inc/                            # Logika rejestracji CPT, ACF JSON, GraphQL Mutations
│       │   ├── cpt-registration.php        # Rejestracja 7 Custom Post Types
│       │   ├── acf-fields/                 # Eksportowane definicje ACF w formacie JSON
│       │   ├── graphql-mutations.php       # Mutation dodająca wpis do CPT `dostepy`
│       │   └── isr-revalidate-hook.php     # Hook `save_post` wysyłający request do Next.js revalidate
│       └── docker-compose.yml              # Środowisko deweloperskie WordPress + MySQL + WPGraphQL
├── packages/                               # Pakiety współdzielone w monorepo
│   └── shared/                             # Pakiet ze schematami, typami i klientem API
│       ├── src/
│       │   ├── schemas/                    # Schematy walidacji Zod (formularz wyceny, kalkulator)
│       │   ├── types/                      # Definicje typów TypeScript dla GraphQL i API
│       │   └── graphql/                    # Zapytania i mutacje .graphql oraz GraphQL Codegen
│       └── package.json
├── package.json                            # Korzeń monorepo (skrypty pnpm build, dev, test, codegen)
├── pnpm-workspace.yaml                     # Definijacja przestrzeni roboczych pnpm
└── ARCHITEKTURA.md                         # Niniejszy dokument architektoniczny
```

---

## 2. MODEL DANYCH (WORDPRESS CPT + ACF)

Każda kolekcja realizowana jest jako Custom Post Type (CPT) w WordPressie z dedykowaną grupą pól ACF (Advanced Custom Fields) wystawioną w WPGraphQL.

### 1. Realizacje (`realizacje`)
- **Pola systemowe**: `ID`, `post_title` (tytuł projektu), `post_status` (`publish`/`draft`), `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `klient_nazwa` | Text | T | Nazwa firmy lub marki klienta |
  | `adres_url` | Url | N | Adres URL wdrożonej strony |
  | `kategoria_uslugi` | Select | T | Opcje: `strona-www`, `sklep-e-commerce`, `aplikacja-dedykowana`, `seo-marketing` |
  | `skrócony_opis` | Textarea | T | Zajawka na kartę portfolio (max 200 znaków) |
  | `wyzwanie_biznesowe` | Wysiwyg | T | Opis problemu klienta przed wdrożeniem |
  | `zakres_prac` | Repeater | T | Lista podpunktów realizowanych prac |
  | `zastosowane_technologie` | Select (multi) | T | Opcje: `Next.js`, `WordPress`, `WooCommerce`, `TailwindCSS`, `Payload`, `Stripe`, `SEO` |
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
  | `autor_bio` | Group | T | Podgrupa: `imię` (Text), `zdjęcie` (Image), `rola` (Text) |
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
  | `cena_od_netto` | Number | T | Minimalna kwota netto w PLN |
  | `jednostka_rozliczenia` | Select | T | Opcje: `za_projekt`, `miesięcznie`, `za_godzinę` |
  | `opis_pakietu` | Textarea | T | Krótki opis dla kogo jest ten pakiet |
  | `lista_funkcji` | Repeater | T | Lista cech/funkcji (tekst + flaga czy zawarte) |
  | `czy_wyróżniony` | True/False | T | Flaga akcentująca pakiet w UI |
  | `etykieta_promocyjna` | Text | N | Etykieta typu "Najpopularniejszy", "Rekomendowany" |

### 5. FAQ (`faq`)
- **Pola systemowe**: `ID`, `post_title` (pytanie), `post_status`, `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `odpowiedz` | Wysiwyg | T | Treść odpowiedzi na pytanie |
  | `kategoria_faq` | Select | T | Opcje: `ogólne`, `strony-www`, `sklepy`, `aplikacje`, `kursy`, `płatności` |
  | `kolejnosc_sortowania` | Number | T | Priorytet wyświetlania na akordeonie |

### 6. Kursy (`kursy`)
- **Pola systemowe**: `ID`, `post_title` (tytuł kursu), `post_name` (slug), `post_status`, `post_date`, `post_modified`.
- **Pola ACF**:
  | Nazwa pola (key/slug) | Typ ACF | Wymagane (T/N) | Opis / Relacja |
  | :--- | :--- | :---: | :--- |
  | `cena_brutto` | Number | T | Cena kursu w PLN brutto |
  | `stripe_price_id` | Text | T | Identyfikator ceny z dashboardu Stripe (`price_...`) |
  | `opis_krótki` | Textarea | T | Podsumowanie kursu na kartę sprzedażową |
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
| `/kontakt/` | `app/(public)/kontakt/page.tsx` | Static | Formularz Client Component + Resend API | WPYoast (Page: Kontakt) |
| `/kursy/[slug]/` | `app/(public)/kursy/[slug]/page.tsx` | ISR (`generateStaticParams`) | WPGraphQL (CPT `kursy` - strona sprzedażowa) | WPYoast (CPT kursy metadata) |
| `/panel-kursanta/` | `app/(protected)/panel-kursanta/page.tsx` | Dynamic (SSR / NextAuth session) | WPGraphQL Query po `user_email` -> CPT `dostepy` | Static ("Panel Kursanta") |
| `/panel-kursanta/kurs/[slug]/` | `app/(protected)/panel-kursanta/kurs/[slug]/page.tsx` | Dynamic (SSR) | WPGraphQL (CPT `kursy`) + Signed Video URLs API | Static ("Odtwarzacz Kursu") |
| `/regulamin/` | `app/(public)/regulamin/page.tsx` | Static | Static HTML / Markdown | Static Metadata |
| `/polityka-prywatnosci/` | `app/(public)/polityka-prywatnosci/page.tsx` | Static | Static HTML / Markdown | Static Metadata |

---

## 4. SCHEMA.ORG (JSON-LD)

| Route | Typy Schema | Wymagane pola Schema | Źródło danych w WP / App |
| :--- | :--- | :--- | :--- |
| `/` | `WebSite`, `Person`, `Organization`, `LocalBusiness` | `@id`, `name`, `url`, `logo`, `sameAs`, `address`, `telephone`, `priceRange` | Statyczne dane konfiguracji marki + WP site options |
| `/o-mnie/` | `Person`, `BreadcrumbList` | `name`, `jobTitle`, `worksFor`, `knowsAbout`, `sameAs`, `itemListElement` | Statyczna konfiguracja bio + URL |
| `/uslugi/` | `BreadcrumbList` | `itemListElement` | Statyczna struktura nawigacji |
| `/uslugi/[slug]/` | `Service`, `LocalBusiness`, `BreadcrumbList` | `name`, `serviceType`, `provider` (`@id` Organization), `areaServed`, `description` | WPGraphQL CPT Usługi / Statyczny szablon usługi |
| `/realizacje/[slug]/` | `BreadcrumbList` | `itemListElement` | WPGraphQL CPT `realizacje` |
| `/blog/[slug]/` | `Article`, `BreadcrumbList` | `headline`, `image`, `datePublished`, `dateModified`, `author` (`@id` Person), `publisher` (`@id` Organization) | WPGraphQL CPT `post` |
| `/cennik/` | `FAQPage`, `BreadcrumbList` | `mainEntity` (tablica pytania/odpowiedzi `Question`/`Answer`) | WPGraphQL CPT `faq` |
| `/kontakt/` | `LocalBusiness`, `BreadcrumbList` | `name`, `telephone`, `email`, `address`, `openingHours` | Statyczna konfiguracja firmy |

### Powiązanie encji za pomocą `@id`:
- **Organization ID**: `https://domena-freelancera.pl/#organization`
- **Person ID**: `https://domena-freelancera.pl/#person`
- **WebSite ID**: `https://domena-freelancera.pl/#website`

W schematach podstron `provider` lub `author` odwołuje się bezpośrednio do `{"@id": "https://domena-freelancera.pl/#person"}`.

---

## 5. INTEGRACJE ZEWNĘTRZNE

| Integracja | Miejsce inicjalizacji w kodzie | Wymagane zmienne środowiskowe | Fallback przy braku klucza (Dev Mode) |
| :--- | :--- | :--- | :--- |
| **Resend** | `apps/web/lib/resend.ts` & `app/api/contact/route.ts` | `RESEND_API_KEY`, `RESEND_FROM_EMAIL`, `RESEND_TO_EMAIL` | Logowanie treści wiadomości e-mail w konsoli serwera (`console.log`), zwrot odpowiedzi statusu `200 OK` do formularza. |
| **Stripe** | `apps/web/lib/stripe.ts` & `app/api/checkout/route.ts` & `app/api/webhooks/stripe/route.ts` | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Powiadomienie błędu w interfejsie checkoutu: "Płatności są wyłączone w środowisku deweloperskim". |
| **Cookiebot** | `apps/web/components/analytics/cookiebot.tsx` | `NEXT_PUBLIC_COOKIEBOT_DOMAIN_GROUP_ID` | Komponent CMP nie renderuje skryptu zewnętrznego; analityka traktowana jako domyślnie odrzucona. |
| **GTM / GA4** | `apps/web/components/analytics/gtm.tsx` | `NEXT_PUBLIC_GTM_ID`, `NEXT_PUBLIC_GA4_ID` | Brak ładowania skryptów gtm.js / gtag.js; zdarzenia `dataLayer.push` przechwytywane przez pustą funkcję mocka. |
| **Google Maps Embed** | `apps/web/components/sections/contact-map.tsx` | `NEXT_PUBLIC_GOOGLE_MAPS_EMBED_KEY` | Wyświetlenie statycznej podglądowej grafiki karty mapy z adresem i przyciskiem nawigacji do Google Maps. |
| **ReCAPTCHA v3** | `apps/web/lib/recaptcha.ts` | `NEXT_PUBLIC_RECAPTCHA_SITE_KEY`, `RECAPTCHA_SECRET_KEY` | Pominięcie weryfikacji walidacji tokena po stronie serwera (`score = 1.0` w dev mode). |

---

## 6. FORMULARZ WYCENY — MACIERZ PÓL

Formularz wyceny obsługuje dynamicznie zmienny zestaw pól w zależności od wybranego typu usługi, walidowany przy użyciu biblioteki Zod (`packages/shared/src/schemas/valuation.ts`).

| Typ usługi (`serviceType`) | Widoczne pola | Typ pola | Walidacja Zod | Wymagane (T/N) |
| :--- | :--- | :--- | :--- | :---: |
| **Wszystkie typy (Pola bazowe)** | `companyName` | Text | `z.string().min(2)` | T |
| | `fullName` | Text | `z.string().min(3)` | T |
| | `email` | Email | `z.string().email()` | T |
| | `phone` | Tel | `z.string().regex(/^[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s\./0-9]*$/).optional().or(z.literal(''))` | N |
| | `budget` | Select | `z.enum(['5k-10k', '10k-20k', '20k-40k', '40k+'])` | T |
| | `timeline` | Select | `z.enum(['asap', '1-month', '2-3-months', 'flexible'])` | T |
| | `description` | Textarea | `z.string().min(20, 'Opis musi mieć min. 20 znaków')` | T |
| **`strona-wp`** | `hasExistingSite` | Radio/Select | `z.boolean()` | T |
| | `existingSiteUrl` | Url | `z.string().url().optional().or(z.literal(''))` | N |
| | `wpFeatures` | Checkbox Group | `z.array(z.enum(['blog', 'multilingual', 'cms-custom', 'seo-advanced', 'speed-optimization']))` | N |
| **`sklep-woocommerce`** | `productCountRange` | Select | `z.enum(['1-50', '51-500', '501-5000', '5000+'])` | T |
| | `integrationsNeeded` | Checkbox Group | `z.array(z.enum(['płatności-blik-card', 'kurierzy-apaczka-inpost', 'erp-faktury', 'allegro-baselinker']))` | N |
| **`aplikacja-webowa`** | `appType` | Select | `z.enum(['saas', 'portal-klienta', 'mvp', 'dysk-internal-system'])` | T |
| | `authRequirements` | Checkbox Group | `z.array(z.enum(['email-pass', 'social-auth', 'sso-saml', 'role-permissions']))` | N |
| **`inne`** | `customScope` | Textarea | `z.string().min(10)` | T |

---

## 7. KALKULATOR CENY — ALGORYTM

Kalkulator ceny jest osobnym interaktywnym komponentem na stronie `/cennik/`. Klient zaznacza preferencje, a algorytm wylicza szacowaną kwotę bazową netto.

### Pseudokod algorytmu (TypeScript):

```typescript
export interface CalculatorInput {
  serviceType: 'strona-wp' | 'sklep-woocommerce' | 'aplikacja-webowa' | 'seo-marketing';
  complexity: 'podstawowa' | 'średnia' | 'zaawansowana';
  subpagesCount: number; // min 1, max 50
  additionalFeatures: Array<'multilingual' | 'animations' | 'cms-custom' | 'speed-opt' | 'payment-integration' | 'erp-sync'>;
  expressDelivery: boolean;
}

export interface CalculatorResult {
  estimatedPriceNettoMin: number;
  estimatedPriceNettoMax: number;
  summaryText: string;
}

export function calculateProjectEstimate(input: CalculatorInput): CalculatorResult {
  // 1. Cena bazowa w zależności od typu usługi
  const basePrices: Record<CalculatorInput['serviceType'], number> = {
    'strona-wp': 4500,
    'sklep-woocommerce': 8000,
    'aplikacja-webowa': 15000,
    'seo-marketing': 3000,
  };

  // 2. Mnożnikłożoności
  const complexityMultipliers: Record<CalculatorInput['complexity'], number> = {
    'podstawowa': 1.0,
    'średnia': 1.3,
    'zaawansowana': 1.7,
  };

  // 3. Koszt dodatkowych podstron (powyżej 5 podstron zawartych w bazie)
  const extraSubpages = Math.max(0, input.subpagesCount - 5);
  const costPerExtraSubpage = 350;

  // 4. Stałe kwoty funkcji dodatkowych
  const featureCosts: Record<string, number> = {
    multilingual: 1500,
    animations: 1200,
    'cms-custom': 2000,
    'speed-opt': 800,
    'payment-integration': 1500,
    'erp-sync': 3500,
  };

  let totalMin = basePrices[input.serviceType] * complexityMultipliers[input.complexity];
  totalMin += extraSubpages * costPerExtraSubpage;

  for (const feature of input.additionalFeatures) {
    if (featureCosts[feature]) {
      totalMin += featureCosts[feature];
    }
  }

  // 5. Dopłata za tryb ekspresowy (+30%)
  if (input.expressDelivery) {
    totalMin *= 1.30;
  }

  // Zaokrąglenie do pełnych setek PLN
  const roundedMin = Math.round(totalMin / 100) * 100;
  const roundedMax = Math.round((roundedMin * 1.25) / 100) * 100;

  return {
    estimatedPriceNettoMin: roundedMin,
    estimatedPriceNettoMax: roundedMax,
    summaryText: `Szacunkowa wycena: od ${roundedMin.toLocaleString('pl-PL')} PLN netto (zakres szacunkowy: ${roundedMin.toLocaleString('pl-PL')} - ${roundedMax.toLocaleString('pl-PL')} PLN netto)`,
  };
}
```

### Tabela składników cenowych:

| Opcja | Wpływ na cenę |
| :--- | :--- |
| **Typ: Strona WP** | Bazowo 4 500 PLN netto |
| **Typ: Sklep WooCommerce** | Bazowo 8 000 PLN netto |
| **Typ: Aplikacja Webowa** | Bazowo 15 000 PLN netto |
| **Mnożnik Średnia Złożoność** | +30% do kwoty bazowej (mnożnik 1.3) |
| **Mnożnik Zaawansowana Złożoność** | +70% do kwoty bazowej (mnożnik 1.7) |
| **Dodatkowa podstrona (>5)** | +350 PLN netto za każdą podstronę |
| **Wersja wielojęzyczna** | +1 500 PLN netto |
| **Zaawansowane animacje (Framer Motion)** | +1 200 PLN netto |
| **Integracja płatności / ERP** | +1 500 PLN - +3 500 PLN netto |
| **Tryb ekspresowy** | +30% do całkowitej wartości wyceny |

### Przekazanie prefillu kalkulatora do formularza wyceny:
1. Przycisk w kalkulatorze: **„Wyślij zapytanie na podstawie tej wyceny”**.
2. Kliknięcie przekierowuje na `/kontakt/?service=...&budget=...&calc_min=...&calc_max=...&features=...`.
3. Komponent `ValuationForm` na stronie kontaktowej odczytuje parametry z `useSearchParams()`, wstępnie wypełnia selekty usług i budżetu oraz dołącza wygenerowane podsumowanie z kalkulatora do pola `description`.
4. **Zastrzeżenie w UI**: Pod wynikiem kalkulatora wyświetlany jest jawny komunikat: *„Prezentowana kwota jest szacunkiem wstępnym i nie stanowi oferty handlowej w rozumieniu Art. 66 Par. 1 Kodeksu Cywilnego. Wiążąca wycena przedstawiana jest po analizie szczegółowych wymagań.”*

---

## 8. PLAN PRACY — ROZBICIE NA ZADANIA

Podział projektu na 35 granularnych zadań implementacyjnych (szacowany czas realizacji pojedynczego zadania: 10–30 minut).

| ID | Nazwa zadania | Zależności | Kryteria akceptacji |
| :---: | :--- | :---: | :--- |
| **TASK-01** | Inicjalizacja Monorepo pnpm & TypeScript | - | 1. Tworzy pliki `pnpm-workspace.yaml`, `package.json`, `.gitignore`.<br>2. Konfiguruje bazowy `tsconfig.json` z trybem `strict: true`.<br>3. Pomyślne wykonanie `pnpm install`. |
| **TASK-02** | Konfiguracja środowiska Docker dla WordPress | TASK-01 | 1. Plik `wp-plugin/docker-compose.yml` uruchamia kontenery WordPress, MySQL, Mailpit.<br>2. Automatyczny montaż katalogu `wp-plugin/freelancer-cms-core`.<br>3. Dostępność panelu WP pod `localhost:8000`. |
| **TASK-03** | Inicjalizacja skryptu pluginu PHP `freelancer-cms-core` | TASK-02 | 1. Plik `freelancer-cms-core.php` prawidłowo aktywuje się w panelu WP.<br>2. Rejestruje stałe i strukturę katalogów `inc/`. |
| **TASK-04** | Rejestracja 7 CPT w pluginie PHP | TASK-03 | 1. Kod PHP rejestruje CPT: `realizacje`, `post`, `opinie`, `cennik`, `faq`, `kursy`, `dostepy`.<br>2. Pola CPT widoczne w menu bocznym WordPress Admin. |
| **TASK-05** | Definicje pól ACF w JSON w pluginie PHP | TASK-04 | 1. Pliki JSON w `inc/acf-fields/` definiują komplet pól z Sekcji 2 dokumentu.<br>2. Pola widoczne w edytorze każdego CPT w panelu WP. |
| **TASK-06** | Instalacja i aktywacja WPGraphQL & ACF for WPGraphQL | TASK-04 | 1. Wtyczki WPGraphQL, WPGraphQL for ACF, WPGraphQL Yoast są aktywne.<br>2. Endpoint `/graphql` odpowiada na zapytanie GraphQL Introspection. |
| **TASK-07** | Tworzenie struktury Next.js 14 App Router w `apps/web` | TASK-01 | 1. Aplikacja ustrukturyzowana w `apps/web/app`.<br>2. Tailwind CSS, Shadcn UI oraz Framer Motion zainstalowane i skonfigurowane w `tailwind.config.ts`. |
| **TASK-08** | Setup pakietu `@freelancer/shared` & GraphQL Codegen | TASK-01 | 1. Pliki zapytań `.graphql` umieszczone w `packages/shared/src/graphql`.<br>2. Skrypt `pnpm codegen` generuje typowane React Hooks i typy TS z schematu WPGraphQL. |
| **TASK-09** | Schematy walidacji Zod w `@freelancer/shared` | TASK-08 | 1. Exporter schematów Zod dla formularza wyceny, kalkulatora i zapytania kontaktowego w `packages/shared/src/schemas`.<br>2. Testy jednostkowe walidatorów przechodzą pomyślnie. |
| **TASK-10** | Implementacja Layoutu głównego (Header, Footer, Nav) | TASK-07 | 1. Komponenty `Header` z menu mobilnym i `Footer` w `apps/web/components/ui/`.<br>2. Prawidłowa nawigacja i responsywność RWD. |
| **TASK-11** | Strona Główna — Sekcja 1: Hero Section | TASK-10 | 1. Komponent `hero-section.tsx` z animacją wejścia Framer Motion.<br>2. Przyciski CTA prowadzące do wyceny i portfolio. |
| **TASK-12** | Strona Główna — Sekcja 2: Przegląd Usług | TASK-10 | 1. Komponent `services-overview.tsx` prezentujący 4 główne usługi w siatce kart. |
| **TASK-13** | Strona Główna — Sekcja 3: Wartość i Korzyści | TASK-10 | 1. Komponent `value-prop.tsx` z wyróżnikami oferty freelancera. |
| **TASK-14** | Strona Główna — Sekcja 4: Wybrane Realizacje | TASK-08, TASK-10 | 1. Komponent `portfolio-preview.tsx` pobierający 3 realizacje z WPGraphQL.<br>2. Obsługa kart z tagami technologii. |
| **TASK-15** | Strona Główna — Sekcja 5: Proces Współpracy | TASK-10 | 1. Komponent `process-steps.tsx` z osi czasu kroku po kroku. |
| **TASK-16** | Strona Główna — Sekcja 6: Stos Technologiczny | TASK-10 | 1. Komponent `tech-stack.tsx` z logotypami i opisem klasycznych oraz modern web stacków. |
| **TASK-17** | Strona Główna — Sekcja 7: Zajawka Cennika | TASK-10 | 1. Komponent `pricing-teaser.tsx` prezentujący pakiety cenowe z przyciskiem do kalkulatora. |
| **TASK-18** | Strona Główna — Sekcja 8: Opinie Klientów | TASK-08, TASK-10 | 1. Komponent `testimonials.tsx` wyciągający dane z CPT `opinie` z ocenami gwiazdkowymi. |
| **TASK-19** | Strona Główna — Sekcja 9: Zajawka Kursów Wideo | TASK-08, TASK-10 | 1. Komponent `courses-teaser.tsx` pobierający dostępne kursy cyfrowe. |
| **TASK-20** | Strona Główna — Sekcja 10: O mnie / Bio | TASK-10 | 1. Komponent `about-teaser.tsx` z bio i zdjęciem programisty. |
| **TASK-21** | Strona Główna — Sekcja 11: FAQ Section | TASK-08, TASK-10 | 1. Komponent `faq-section.tsx` wykorzystujący akordeon Shadcn UI ze sterowaniem animacji. |
| **TASK-22** | Strona Główna — Sekcja 12: Najnowsze Wpisy Blogowe | TASK-08, TASK-10 | 1. Komponent `blog-preview.tsx` wyświetlający 3 najnowsze artykuły z bloga. |
| **TASK-23** | Strona Główna — Sekcja 13: Formularz Wyceny CTA | TASK-09, TASK-10 | 1. Komponent `valuation-cta.tsx` zamontowany w stopce sekcji głównej. |
| **TASK-24** | Podstrona O mnie (`/o-mnie/`) | TASK-10 | 1. Strona statyczna z pełną historią, certyfikatami i stosem technologicznym.<br>2. Dodany JSON-LD `Person`. |
| **TASK-25** | Podstrony Usług (`/uslugi/` & `/uslugi/[slug]/`) | TASK-08, TASK-10 | 1. Szablon podstrony dla 4 typów usług (`generateStaticParams`).<br>2. Prawidłowy JSON-LD `Service`. |
| **TASK-26** | Podstrony Portfolio (`/realizacje/` & `/realizacje/[slug]/`) | TASK-08, TASK-10 | 1. Lista portfolio z filtrowaniem wg kategorii.<br>2. Detal realizacyjny z wyzwaniem biznesowym i galeriami. |
| **TASK-27** | Podstrony Bloga (`/blog/` & `/blog/[slug]/`) | TASK-08, TASK-10 | 1. Wierszowana i siatkowa lista artykułów blogowych.<br>2. Strona artykułu z renderowaniem HTML i JSON-LD `Article`. |
| **TASK-28** | Podstrona Cennika & Komponent Kalkulatora (`/cennik/`) | TASK-09, TASK-10 | 1. Strona z pakietami CPT `cennik`.<br>2. Komponent interaktywnego kalkulatora wyliczający estymację w czasie rzeczywistym z wyjściem prefill. |
| **TASK-29** | Podstrona Kontaktowa & Formularz Wyceny (`/kontakt/`) | TASK-09, TASK-10 | 1. Komponent formularza z obsługą prefilla z query params (`useSearchParams`).<br>2. Integracja ze skryptem ReCAPTCHA v3. |
| **TASK-30** | Integracja Resend API & Route Handler | TASK-29 | 1. Handler `app/api/contact/route.ts` waliduje dane Zod i wysyła powiadomienie e-mail.<br>2. Obsługa trybu dev (fallback console.log). |
| **TASK-31** | Moduł LMS — Checkout Stripe & Webhook (`/api/checkout`) | TASK-08 | 1. Handler `app/api/checkout` tworzy sesję Stripe Checkout dla CPT `kursy`.<br>2. Webhook `/api/webhooks/stripe` zapisuje zakup do CPT `dostepy` w WP. |
| **TASK-32** | Moduł LMS — Autoryzacja NextAuth Magic Link & Panel | TASK-31 | 1. Konfiguracja NextAuth.js z dostawcą Email (Resend).<br>2. Pomyślna weryfikacja zakupu na podstawie e-maila w CPT `dostepy`. |
| **TASK-33** | Moduł LMS — Odtwarzacz Wideo z Signed URL | TASK-32 | 1. Route handler generujący signed URL dla lekcji z Cloudflare Stream/Vimeo.<br>2. Odtwarzacz lekcji na chronionej ścieżce `/panel-kursanta/kurs/[slug]`. |
| **TASK-34** | Integracja Analityki, GTM & Cookiebot CMP | TASK-10 | 1. Skrypty Cookiebot i GTM z integracją Consent Mode v2.<br>2. Przechwytywanie zdarzeń `form_submit` i `calculator_use`. |
| **TASK-35** | Integracja Rewalidacji ISR Webhook | TASK-03, TASK-08 | 1. Hook PHP w WP wysyła zdarzenie po zapisie wpisu.<br>2. Endpoint `/api/revalidate` odświeża pamięć podręczną Next.js per tag/path. |

---

## 9. RYZYKA I DECYZJE OTWARTE

Wykaz decyzji architektonicznych i technologicznych, które wymagają zatwierdzenia przez Architekta / Zamawiającego przed fazą realizacyjną.

| ID | ZAGADNIENIE / DECYZJA | OPCJE DO WYBORU | REKOMENDACJA ARCHITEKTA |
| :---: | :--- | :--- | :--- |
| **DEC-01** | **Hosting dla Headless WordPress** | **Opcja A**: Managed WordPress (Kinsta / WP Engine)<br>**Opcja B**: VPS (Hetzner / DigitalOcean) z Docker / CapRover<br>**Opcja C**: Cloudways / CyberFolks | **Rekomendacja: Opcja B (VPS Hetzner)**. Zapewnia najniższe opóźnienia terytorialne w PL, pełną kontrolę nad Dockerem z WPGraphQL i niski stały koszt miesięczny. |
| **DEC-02** | **Hosting i Podpisywanie Plików Wideo LMS** | **Opcja A**: Cloudflare Stream (signed URL / HMAC token)<br>**Opcja B**: Vimeo OTT / Pro (Domain restriction + JWT)<br>**Opcja C**: AWS S3 + CloudFront Signed URLs | **Rekomendacja: Opcja A (Cloudflare Stream)**. Najprostsze API, natywna obsługa odtwarzacza HLS, bardzo dobre zabezpieczenie przed pobieraniem i niskie koszty za minutę streamingu. |
| **DEC-03** | **Zarządzanie Sesją i Autoryzacją Kursanta** | **Opcja A**: NextAuth.js z Passwordless Magic Link (Resend)<br>**Opcja B**: Supabase Auth (tylko do obsługi logowania)<br>**Opcja C**: WordPress Native Auth JWT | **Rekomendacja: Opcja A (NextAuth.js Magic Link)**. Eliminacja konieczności pamiętania haseł przez kursantów, bezpośrednie mapowanie e-maila zakupu ze Stripe bez dodatkowego zewnętrznego providera bazodanowego. |
| **DEC-04** | **Platforma CMP dla Consent Mode v2** | **Opcja A**: Cookiebot (oficjalna wtyczka / skrypt)<br>**Opcja B**: CookieFirst<br>**Opcja C**: Customowy banner zgodny z GTM Consent Mode v2 | **Rekomendacja: Opcja A (Cookiebot)**. Pełna automatyzacja skanowania ciasteczek i natywna integracja z szablonem GTM Google Consent Mode v2. |
| **DEC-05** | **Rozszerzenie Typów Schema.org w Przyszłości** | **Opcja A**: Pozostanie ściśle przy zdefiniowanej liście (Person, Organization, Service, FAQPage, Article, LocalBusiness)<br>**Opcja B**: Rozszerzenie w kolejnej iteracji o `Course` i `Offer` dla strony kursu | **Rekomendacja: Opcja B**. Aktualnie zachowujemy 1:1 listę ze specyfikacji, lecz w sekcji kursowej w przyszłości warto dodać Schema `Course` i `Offer` w celu uzyskania Rich Snippets w wyszukiwarce. |
| **DEC-06** | **Mechanizm Obsługi Błędów w Wysyłce Webhooka ISR z WP** | **Opcja A**: Synchroniczny request HTTP z cURL przy `save_post`<br>**Opcja B**: Asynchroniczna kolejka wp-cron / Action Scheduler | **Rekomendacja: Opcja B (Action Scheduler)**. Zapobiega opóźnieniom i zawieszaniu panelu administracyjnego WordPressa w przypadku chwilowej niedostępności API Next.js. |

---
