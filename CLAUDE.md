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

## Jak triggerować

Użytkownik mówi np.:
- "Zrób preview dla HOT leadów z Pipeline Master"
- "Zrób preview dla ORYVA-RZE-013"
- "Zrób preview dla wszystkich nowych z outreach"

Claude wykonuje pełny flow od A do Z (kroki 1–7 poniżej) dla każdego wskazanego rekordu.

---

## Pełny flow — krok po kroku

### 1. Odczytaj dane z Pipeline Master

Ścieżka: `/Users/kasiabochenska/Library/CloudStorage/GoogleDrive-hello@oryva.site/My Drive/ORYVA/Oryva_Pipeline_Master.xlsx`

Odczytaj dane leada z wielu sheetów (użyj openpyxl, matchuj po kolumnie H = ID w `5_Outreach`):

**Sheet `5_Outreach`** (dane od rzędu 2):

Kolumny A–G są kompatybilne z Google Sheets cold email senderem (można kopiować 1:1):
| Kolumna | Nagłówek | Opis |
|---------|----------|------|
| **A** | Email | adres email kontaktu |
| **B** | Imię | imię osoby kontaktowej |
| **C** | Firma | nazwa firmy |
| **D** | Temat | temat maila |
| **E** | Treść maila (HTML) | pełna treść z linkiem do preview |
| **F** | Follow-up 1 (HTML) | treść pierwszego follow-upu |
| **G** | Follow-up 2 (HTML) | treść drugiego follow-upu |

Kolumny H–O to dane pipeline'owe:
| Kolumna | Nagłówek | Opis |
|---------|----------|------|
| **H** | ID | np. ORYVA-RZE-013 |
| **I** | Segment | HOT / WARM |
| **J** | Ścieżka | A (email z analizą) / B (Shock & Awe) |
| **K** | Persona | Invisible Craftsman / Frustrated Modernizer |
| **L** | Score | 0–100 |
| **M** | Preview URL | `<slug>.oryva.site` ← tu wpisz po deployu |
| **N** | Status | Przygotowano / Wysłano / Odpowiedź |
| **O** | Data wysyłki | YYYY-MM-DD |

**Sheet `1_RAW_Scraping`** (dane od rzędu 2, matchuj po ID w kolumnie A):
- Branża, Miasto, Adres, Telefon, Strona WWW, Ocena Google, Ile recenzji, Opis GBP, Godziny otwarcia, Social media

**Sheet `3_Segmentacja`**:
- Kluczowe problemy, Priorytet outreach

### 2. Wygeneruj slug

Z nazwy firmy:
- Usuń polskie znaki (ł→l, ś→s, ą→a, ć→c, ę→e, ń→n, ó→o, ż→z, ź→z)
- Usuń słowa: salon, studio, gabinet, spa, beauty, dr, med, lek
- Lowercase, usuń spacje i znaki specjalne (|, -, ., etc.)
- Przykłady:
  - "Anna Majkowska Salon Beauty" → `annamajkowska`
  - "Nefretete | Babor Beauty Spa" → `nefretete`
  - "M Studio" → `mstudio`

### 3. Stwórz stronę preview

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

Struktura strony (dopasuj do branży i persony):
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

### 4. Dodaj rewrite do vercel.json

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
- Przy wielu naraz: `add previews for <firma1>, <firma2>, ...`
- Push to main → Vercel auto-deploy
- **Nie trzeba ręcznie dodawać domeny** — wildcard obsługuje wszystkie subdomeny

### 6. Wygeneruj treść maila i wpisz do Pipeline Master

Wszystko trafia bezpośrednio do sheetu `5_Outreach` w Pipeline Master (openpyxl).

Dla każdego leada wypełnij:
- **E** (Treść maila HTML) — pełna treść maila z linkiem `https://<slug>.oryva.site`
- **F** (Follow-up 1 HTML) — pierwszy follow-up (po 3 dniach)
- **G** (Follow-up 2 HTML) — drugi follow-up (po 7 dniach)
- **M** (Preview URL) — `<slug>.oryva.site`

Treść maila musi zawierać live link:
```
👉 https://<slug>.oryva.site
```

Dopasuj ton maila do:
- **Persona** (Invisible Craftsman, Frustrated Modernizer, etc.)
- **Segment** (HOT = bardziej bezpośredni / Shock & Awe, WARM = edukacyjny)
- **Branża i dane** (Ocena Google, ile recenzji, opis GBP)

Szablon bazowy maila: `/Users/kasiabochenska/Library/CloudStorage/GoogleDrive-hello@oryva.site/My Drive/ORYVA/cold-email-template.md`

Kolumny A–G są gotowe do skopiowania 1:1 do Google Sheets cold email sendera.

---

## Weryfikacja po deployu

Po pushu sprawdź czy subdomena działa:
```bash
curl -s -o /dev/null -w "%{http_code}" https://<slug>.oryva.site
```
Oczekiwany kod: `200`

---

## Zasady

- **Nie modyfikuj** istniejących preview bez wyraźnej prośby
- **Nie zmieniaj** Google Analytics ID (G-7YLE4F3MMN)
- **Nie dodawaj** build stepów, bundlerów ani frameworków — to statyczne pliki HTML
- Każdy `index.html` musi działać samodzielnie po otwarciu w przeglądarce
- Przy wielu preview: przetwarzaj sekwencyjnie, jeden commit na końcu z wszystkimi naraz
