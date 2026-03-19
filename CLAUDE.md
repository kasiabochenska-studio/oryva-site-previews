# Oryva Site Previews

Repo z preview stronami dla klientów Oryva. Każda strona to subdomena `*.oryva.site` hostowana na Vercelu (projekt `multi-tanent-website`, wildcard `*.oryva.site` już skonfigurowany).

GitHub: `kasiabochenska-studio/oryva-site-previews`
Vercel project: `multi-tanent-website`

## Struktura repo

```
oryva-site-previews/
├── CLAUDE.md
├── vercel.json          ← routing subdomen (rewrites po host)
├── .gitignore
└── <slug>/index.html    ← każda subdomena = folder z jednym plikiem HTML
```

## Tworzenie nowej subdomeny — pełny flow

### 1. Dane wejściowe

Dane pochodzą z `Oryva_Pipeline_Master.xlsx` (Google Drive → ORYVA), sheet `5_Outreach`.
Ścieżka: `/Users/kasiabochenska/Library/CloudStorage/GoogleDrive-hello@oryva.site/My Drive/ORYVA/Oryva_Pipeline_Master.xlsx`

Kluczowe kolumny (uwaga: rząd 2 ma inne nagłówki niż rząd 1, dane zaczynają się od rzędu 3):
- **B** Nazwa firmy → generuje slug subdomeny
- **C** Email
- **D** Segment (HOT/WARM)
- **F** Persona
- **G** Score
- **I** Temat email
- **J** Preview HTML ← tu wpisujemy `<slug>.oryva.site`

Dodatkowe dane z innych sheetów:
- `1_RAW_Scraping`: Branża, Miasto, Adres, Telefon, Strona WWW, Ocena Google, Ile recenzji, Opis GBP, Social media
- `3_Segmentacja`: Kluczowe problemy, Priorytet outreach

### 2. Generowanie slugu

Slug tworzymy automatycznie z nazwy firmy:
- Usuń polskie znaki (ł→l, ś→s, ą→a, ć→c, ę→e, ń→n, ó→o, ż→z, ź→z)
- Usuń słowa: salon, studio, gabinet, spa, beauty, dr, med, lek
- Lowercase, usuń spacje i znaki specjalne (|, -, ., etc.)
- Przykłady:
  - "Anna Majkowska Salon Beauty" → `annamajkowska`
  - "Nefretete | Babor Beauty Spa" → `nefretete`
  - "Dr. Kowalski Stomatologia" → `kowalski`
  - "M Studio" → `mstudio`
  - "Aspire Family Dentistry" → `aspirefamilydentistry`

### 3. Tworzenie strony

1. Utwórz folder `<slug>/`
2. Utwórz `<slug>/index.html` — samodzielny plik HTML

Wymagania techniczne:
- **Jeden plik HTML** — cały CSS inline w `<style>`, zero zewnętrznych zależności (poza Google Fonts)
- **W pełni responsywna** (mobile-first)
- **Język strony** dopasowany do klienta (pl dla polskich, en dla US)
- **Google Analytics tag** — zawsze ten sam ID, dodać w `<head>`:
  ```html
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-7YLE4F3MMN"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());
    gtag('config', 'G-7YLE4F3MMN');
  </script>
  ```

Struktura strony:
- Hero section z CTA
- Sekcja usług (dopasowana do branży)
- Sekcja o firmie/właścicielu
- Opinie/reviews (na podstawie danych z Google)
- Dane kontaktowe (telefon, adres, godziny)
- Footer z social media i linkiem "Strona stworzona przez Oryva"

Styl:
- Unikalna paleta kolorów per strona (CSS variables w `:root`)
- Google Fonts (para: serif heading + sans-serif body)
- Ikony: inline SVG (nie Font Awesome)
- Brak zewnętrznych zdjęć — gradienty, kształty CSS, emoji jako wizualne akcenty
- Alternujące tła sekcji
- Scroll animations (IntersectionObserver)

### 4. Dodanie rewrite do vercel.json

Dodaj nowy wpis na końcu tablicy `rewrites`:
```json
{
  "source": "/",
  "has": [
    {
      "type": "host",
      "value": "<slug>.oryva.site"
    }
  ],
  "destination": "/<slug>/index.html"
}
```

### 5. Commit i push

- Format commita: `add <slug> preview for <Nazwa firmy>`
- Push to main → Vercel auto-deploy (wildcard *.oryva.site jest skonfigurowany)
- **Nie trzeba ręcznie dodawać domeny w Vercelu** — wildcard obsługuje wszystkie subdomeny

### 6. Aktualizacja Pipeline Master

Po pushu zaktualizuj `Oryva_Pipeline_Master.xlsx`:
- Sheet `5_Outreach` → kolumna **J** (Preview HTML) → wpisz `<slug>.oryva.site`

## Zasady

- **Nie modyfikuj** istniejących preview bez wyraźnej prośby
- **Nie zmieniaj** Google Analytics ID (G-7YLE4F3MMN)
- **Nie dodawaj** build stepów, bundlerów ani frameworków — to statyczne pliki HTML
- Każdy `index.html` musi działać samodzielnie po otwarciu w przeglądarce
- Przy tworzeniu wielu preview na raz, przetwarzaj je sekwencyjnie (jeden folder + rewrite per lead)
