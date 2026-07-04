# Aspektna analiza sentimenta (ABSA) primenom velikih jezičkih modela

Projekat iz predmeta **Inteligentni sistemi**. Ceo rad je u jednom notebooku: [`ABSA_Amazon_LLM.ipynb`](ABSA_Amazon_LLM.ipynb).

## O čemu je projekat

Klasična analiza sentimenta ocenjuje **celu** recenziju kao pozitivnu ili negativnu. Problem je što jedna recenzija često sadrži više osobina sa različitim stavom, na primer: *"zvuk je odličan, ali baterija slaba"*. Prosečna ocena (zvezdice) to sakrije.

Zato ovaj rad radi **aspektnu** analizu (ABSA): iz teksta se izdvaja svaki **aspekt** proizvoda (baterija, zvuk, cena, ekran...) i za svaki se posebno određuje sentiment (pozitivan / negativan / neutralan).

Cilj je da se pokaže da **veliki jezički modeli (LLM) rešavaju ovaj zadatak bolje od klasičnih metoda**, jer razumeju kontekst i na koju osobinu se stav odnosi.

## Podaci

Koriste se dva skupa, sa razlogom:

- **SemEval-2014 Task 4 (Laptops)** - anotiran skup sa tačnim labelama. Služi za **merenje** tačnosti (F1), jer imamo sa čim da uporedimo predikcije.
- **Amazon Reviews 2023 (Electronics)** - realne, neanotirane recenzije. Služe za **demonstraciju** na "živim" podacima i za proširenje (rangiranje proizvoda). Skup je ogroman (~6.5 GB), pa se ne skida ceo nego se stream-uje samo potreban deo.

## Šta se radi i zašto

**1. Poređenje 4 metode** (na istom test skupu, da poređenje bude pošteno):

| Metoda | Zašto je tu |
|---|---|
| Većinska klasa | Trivijalni baseline, "donja granica" |
| TF-IDF + logistička regresija | Klasičan ML pristup po rečima |
| LLM (Claude) zero-shot | LLM samo sa instrukcijom, bez primera |
| LLM (Claude) few-shot | LLM sa par rešenih primera u promptu |

Glavna metrika je **makro-F1** (prosek F1 po klasama), jer su klase neuravnotežene. Tačnost bi varala: model koji uvek pogodi samo najčešću klasu ima visoku tačnost, a bezvredan je. Podela na train/test radi se **po rečenici**, da ista rečenica ne bude i u treningu i u testu (sprečavanje curenja podataka).

**2. Proširenje: rangiranje proizvoda** (simulacija "agenta"):

Na realnim Amazon recenzijama aspekti nisu unapred dati, pa ih LLM prvo sam **otkriva**, a zatim im određuje sentiment. Zatim:

- biraju se proizvodi **iste kategorije** (slušalice), da bi poređenje bilo smisleno,
- sentiment po aspektu se agregira **Bayesovom ocenom** (shrinkage) umesto prostog proseka. Razlog: proizvod sa jednim pohvalnim pomenom ne sme da dobije istu ocenu kao proizvod sa dvadeset pomena. Malobrojni pomeni se "vuku" ka globalnom proseku, pa jedan srećan pomen ne dominira,
- proizvodi se rangiraju prema **prioritetima korisnika** (na primer: najvažniji zvuk, pa baterija, pa cena).

Pokazuje se da rang **ne prati zvezdice**: proizvod sa manje zvezdica može biti bolji izbor ako je najbolji baš po osobinama koje su korisniku važne.

## Rezultati (ukratko)

Klasifikacija na SemEval skupu (makro-F1):

| Metoda | Makro-F1 |
|---|---|
| Većinska klasa | 0.21 |
| TF-IDF + LogReg | 0.50 |
| LLM few-shot | 0.77 |
| LLM zero-shot | **0.83** |

LLM je bolji za oko **+0.33 makro-F1** od najboljeg klasičnog pristupa, sa najvećim dobitkom na teškoj `neutral` klasi.

## Napomene

- Model: **Claude Opus 4.8** (`effort="medium"`).
- LLM je evaluiran na podskupu test skupa (zbog cene poziva); broj se menja preko `N_EVAL_LLM`.
- Amazon deo nema tačne labele, pa je tamo analiza kvalitativna.
- Svi nasumični procesi koriste `RANDOM_STATE = 42` radi reproduktivnosti.