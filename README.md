# Analiza Sentymentu Recenzji Gier Steam

Praca magisterska poświęcona analizie sentymentu recenzji gier komputerowych z platformy Steam z podziałem na gatunki.

---

## 3. Materiały i metody

### 3.1. Gromadzenie danych

Dane zostały pobrane za pomocą biblioteki `steamreviews` z oficjalnego API platformy Steam. Zbiór danych obejmuje **23 gry** z **4 gatunków**:

| Tytuł | Autor | Kategoria | % Pozytywnych opinii |
|-------|-------|-----------|---------------------|
| Hotline Miami | Dennaton Games (2012) | Akcja (A) | 97,31% |
| Left 4 Dead 2 | Valve (2009) | Akcja (A) | 97,54% |
| Sea of Thieves | Rare Ltd. (2020) | Akcja (A) | 90,05% |
| Katana Zero | Askiisoft (2019) | Akcja (A) | 98,12% |
| Darkest Dungeon | Red Hook Studios (2016) | RPG (R) | 91,53% |
| The Wolf Among Us | Telltale (2013) | RPG (R) | 97,78% |
| Disco Elysium | ZA/UM (2019) | RPG (R) | 93,45% |
| Slay the Spire | Mega Crit Games (2019) | RPG (R) | 97,87% |
| Omori | OMOCAT, LLC (2020) | RPG (R) | 97,70% |
| The Witness | Thekla, Inc. (2016) | Puzzle (P) | 84,99% |
| Portal 2 | Valve (2011) | Puzzle (P) | 98,74% |
| Baba Is You | Oy (2019) | Puzzle (P) | 97,96% |
| Antichamber | Bruce (2013) | Puzzle (P) | 95,09% |
| The Talos Principle | Croteam (2014) | Puzzle (P) | 95,62% |
| INSIDE | Playdead (2016) | Puzzle (P) | 96,67% |
| Portal | Valve (2007) | Puzzle (P) | 98,50% |
| We Were Here Forever | Total Mayhem Games (2022) | Puzzle (P) | 91,55% |
| The Past Within | Rusty Lake (2022) | Puzzle (P) | 94,84% |
| Little Nightmares II | Tarsier Studios (2021) | Puzzle (P) | 94,35% |
| RimWorld | Ludeon Studios (2018) | Strategia (S) | 98,10% |
| Hearts of Iron IV | Paradox Development Studio (2016) | Strategia (S) | 91,83% |
| Sid Meier's Civilization VI | Firaxis Games (2016) | Strategia (S) | 85,46% |
| Factorio | Wube Software LTD. (2020) | Strategia (S) | 96,49% |

*Dane pobrane ze strony steamdb, dnia 06.10.2023*

**Parametry pobierania:**
- Język: angielski
- Początkowa liczba recenzji: **29 492**
- Liczba recenzji po filtracji: **26 663**
- Łączna liczba słów: **1 593 992**

### 3.2. Przetwarzanie danych

Wstępne przetwarzanie danych składało się z następujących kroków:

1. **Sprawdzenie braków danych** - weryfikacja zbioru w celu upewnienia się, że żadna z kolumn nie zawiera braków. Zbiór danych nie zawierał braków w żadnej z kolumn.

2. **Usunięcie duplikatów** - identyfikacja i usunięcie duplikatów w zbiorze danych. Znaczna część duplikatów stanowiła recenzje składające się z jednego słowa lub krótkiego zdania. Podczas tego procesu usunięto **2 580 recenzji**.

3. **Usunięcie recenzji krótszych niż 3 słowa** - usunięcie recenzji zawierających mało informacji, aby skupić się na bardziej obszernych opiniach. Podczas tego procesu usunięto **2 829 recenzji**.

4. **Oczyszczanie tekstu:**
   - Usunięcie znaków specjalnych i niealfanumerycznych spoza zbioru ASCII
   - Obniżenie wielkości liter (normalizacja)
   - Usunięcie znaków interpunkcyjnych i liczb

5. **Tokenizacja** - podział tekstu na elementy znaczące (zdania, frazy, pojedyncze słowa)

**Wykorzystane narzędzia NLP:**
- spaCy (`en_core_web_sm`)
- Stanza (model angielski)
- NLTK (stopwords, vader_lexicon)

### 3.3. Struktura danych

Spośród **21 663** wszystkich recenzji, **11 385** dotyczyło aspektów gier.

| Gatunek | Recenzje z aspektami | Pozytywne | Negatywne |
|---------|---------------------|-----------|-----------|
| **RPG (R)** | 2 249 | 2 065 | 184 |
| **Akcja (A)** | 3 121 | 2 676 | 445 |
| **Puzzle (P)** | 2 264 | 2 167 | 87 |
| **Strategia (S)** | 3 751 | 3 092 | 659 |

Każda recenzja w zbiorze danych zawiera następujące atrybuty:

| Kolumna | Opis |
|---------|------|
| `title` | Tytuł gry |
| `genre` | Gatunek (A/R/P/S) |
| `steamid` | ID użytkownika Steam |
| `votes_up` | Liczba pozytywnych głosów na recenzję |
| `review` | Treść recenzji |
| `weighted_vote_score` | Ważona ocena recenzji |
| `voted_up` | Rekomendacja (True/False) |
| `num_games_owned` | Liczba gier posiadanych przez recenzenta |
| `playtime_at_review` | Czas gry w momencie pisania recenzji |
| `playtime_forever` | Całkowity czas gry |

**Statystyki opisowe:**
- Średnia długość recenzji: **54.19 słów**
- Odchylenie standardowe: **135.35 słów**
- Mediana: **13 słów**

### 3.4. Selekcja cech

#### 3.4.1. Parsowanie zależności

Analiza zależności składniowych to proces umożliwiający określenie struktury gramatycznej w zdaniu i wykrycie powiązanych ze sobą słów oraz typu relacji między nimi. Wykorzystano bibliotekę **Stanza NLP**.

**Proces ekstrakcji par cecha-opinia:**

1. **Identyfikacja przysłówków** - przysłówek (adverb) to nieodmienna część mowy służąca do modyfikacji znaczenia czasowników i przymiotników.

   Przykłady:
   - *"The soundtrack is **extremely** nice."* - przysłówek "extremely" modyfikuje przymiotnik "nice", dodając silniejszego zabarwienia emocjonalnego
   - *"This game is **no** fun."* - przysłówek "no" zmienia znaczenie przymiotnika "fun" na przeciwne

2. **Wyszukiwanie cech i opinii** - jeśli:
   - Pierwsza część zależności to rzeczownik (NN - liczba pojedyncza, NNS - liczba mnoga)
   - Druga część to przymiotnik (JJ, JJR - stopień wyższy, JJS - stopień najwyższy)
   - Relacja między nimi to **amod** (atrybutywny modyfikator)

   → następuje zapis cechy i odpowiadającej jej opinii

**Przykłady ekstrahowanych par:**
- `{'feature': 'graphic', 'opinion': 'fun'}`
- `{'feature': 'player', 'opinion': 'unbalanced'}`
- `{'feature': 'reward', 'opinion': 'risky'}`

#### 3.4.2. Klastrowanie hierarchiczne

**Proces klasteryzacji:**

1. Wybór rzeczowników występujących **co najmniej 15 razy** w korpusie
2. Przekształcenie rzeczowników na tokeny za pomocą modelu językowego **SpaCy**
3. Wykorzystanie wektorów osadzeń do hierarchicznego grupowania metodą **Ward** (minimalizacja wariancji wewnątrz klastrów)
4. Generowanie **dendrogramu** wizualizującego relacje między klastrami
5. Przypisanie etykiet klastrów na podstawie odległości odcięcia (**t=75**)

**Wyodrębnione kategorie (16):**

| Kategoria | Przykładowe słowa |
|-----------|-------------------|
| **Rozgrywka** | game, gameplay, gamer, gaming, rng, bonus, replay |
| **Walka** | enemy, fight, battle, combat, campaign, victory, weapon, gun |
| **Wyzwania** | challenge, problem, effect, reason, balance, potential |
| **Grafika** | graphic, visual, video, pixel, color, paint, light |
| **Narracja i fabuła** | character, hero, plot, episode, lore, chapter, protagonist, narrator |
| **Cena i wartość** | price, cost, worth, waste, purchase, payment, buying, credit |
| **Świat gry** | world, city, country, nation, colony, empire, journey |
| **Społeczność** | experience, community, work, environment, resource, workshop |
| **Aktualizacje** | update, feature, version, release, entry, upgrade, patch |
| **Błędy** | bug, glitch |
| ... | *oraz pozostałe kategorie* |

Słowa semantycznie bliżej powiązane zostają przypisane do tego samego klastra. Następnie do istniejących klastrów dodano nowe słowa na podstawie ich podobieństwa.

### 3.5. Analiza wydźwięku

Do analizy sentymentu wykorzystano narzędzie **VADER** (Valence Aware Dictionary and sEntiment Reasoner) z biblioteki NLTK.

**Klasyfikacja sentymentu:**
- **Pozytywny:** compound > 0
- **Negatywny:** compound < 0
- **Neutralny:** compound = 0

Każda para cecha-opinia otrzymała wynik sentymentu (compound score) w skali od -1 do +1.

---

## 4. Wyniki

### 4.1. Rozkład recenzji według gatunku

| Gatunek | Recenzje z aspektami | Pozytywne | Negatywne | % Pozytywnych |
|---------|---------------------|-----------|-----------|---------------|
| RPG (R) | 2 249 | 2 065 | 184 | 91.8% |
| Akcja (A) | 3 121 | 2 676 | 445 | 85.7% |
| Puzzle (P) | 2 264 | 2 167 | 87 | 96.2% |
| Strategia (S) | 3 751 | 3 092 | 659 | 82.4% |

### 4.2. Analiza sentymentu według gatunku i kategorii

#### RPG (R)
| Kategoria | Pozytywne | Neutralne | Negatywne |
|-----------|-----------|-----------|-----------|
| Rozgrywka | 56.1% | 39.2% | 4.7% |
| Grafika | 54.6% | - | - |
| Narracja i fabuła | 39.5% | - | - |
| Walka | 24.4% | 71.9% | 3.8% |
| Wyzwania | 20.9% | - | 3.1% |

#### Akcja (A)
| Kategoria | Pozytywne | Neutralne | Negatywne |
|-----------|-----------|-----------|-----------|
| Rozgrywka | 57.0% | - | - |
| Grafika | 39.5% | - | - |
| Narracja i fabuła | 39.1% | - | - |
| Walka | 33.7% | - | - |
| Wyzwania | 25.8% | - | - |

#### Puzzle (P)
| Kategoria | Pozytywne | Neutralne | Negatywne |
|-----------|-----------|-----------|-----------|
| Rozgrywka | 64.3% | - | - |
| Narracja i fabuła | 49.9% | - | - |
| Grafika | 44.6% | - | - |
| Wyzwania | 21.9% | - | - |
| Walka | 18.4% | - | - |

#### Strategia (S)
| Kategoria | Pozytywne | Neutralne | Negatywne |
|-----------|-----------|-----------|-----------|
| Rozgrywka | 51.8% | - | - |
| Grafika | 35.4% | - | - |
| Narracja i fabuła | 31.1% | - | - |
| Wyzwania | 25.6% | - | - |
| Walka | 17.4% | - | - |

### 4.3. Kluczowe obserwacje

1. **Rozgrywka** - najwyższy odsetek pozytywnych opinii we wszystkich gatunkach (51-64%)
2. **Puzzle** - najwyższy pozytywny sentyment dla kategorii "Rozgrywka" (64.3%) oraz najwyższy % pozytywnych recenzji (96.2%)
3. **RPG** - wysoki odsetek neutralnych opinii w kategorii "Walka" (71.9%)
4. **Strategia** - najniższy pozytywny sentyment dla "Narracji i fabuły" (31.1%) oraz najniższy % pozytywnych recenzji (82.4%)
5. **Puzzle** - najniższy odsetek negatywnych recenzji (tylko 87 z 2264)

### 4.4. Testy statystyczne

#### Test Kruskal-Wallis
Nieparametryczny test do porównania rozkładów sentymentu między gatunkami.

#### Test post-hoc Dunna
Porównania parami między gatunkami z korektą na wielokrotne testowanie.

### 4.5. Korelacje
Analiza korelacji Pearsona między:
- Średnią sentymentu a wariancją
- Czasem gry a sentymentem recenzji

---

## Struktura projektu

```
Magisterka/
├── Mgr1.ipynb          # Główny notebook z analizą
├── data/               # Dane - recenzje gier ze Steam (JSON)
│   └── review_*.json   # Pliki z recenzjami poszczególnych gier (23 pliki)
└── README.md
```

---

## Wymagania

```
pandas
numpy
matplotlib
seaborn
plotly
spacy
stanza
nltk
steamreviews
wordcloud
scikit-learn
scipy
scikit-posthocs
joblib
tqdm
```

## Instalacja

```bash
# Klonowanie repozytorium
git clone https://github.com/[username]/Magisterka.git
cd Magisterka

# Instalacja zależności
pip install pandas numpy matplotlib seaborn plotly spacy stanza nltk steamreviews wordcloud scikit-learn scipy scikit-posthocs joblib tqdm

# Pobranie modeli językowych
python -m spacy download en_core_web_sm
python -c "import stanza; stanza.download('en')"
python -c "import nltk; nltk.download('vader_lexicon'); nltk.download('stopwords')"
```

## Uruchomienie

```bash
jupyter notebook Mgr1.ipynb
```

---

## Wizualizacje

Projekt generuje następujące wizualizacje:
- Chmury słów (WordCloud) dla każdego gatunku
- Wykresy słupkowe rozkładu sentymentu według kategorii
- Boxploty porównujące gatunki
- Wykresy kołowe rozkładu recenzji
- Dendrogram klasteryzacji hierarchicznej
- Heatmapa testu Dunna
- Macierz korelacji

---

## Autor

Projekt realizowany w ramach pracy magisterskiej.

## Licencja

Projekt edukacyjny - praca magisterska.
