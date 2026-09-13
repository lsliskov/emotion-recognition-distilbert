# Prepoznavanje emocija u tekstu pomoću DistilBERT-a

Projekt je izrađen u sklopu kolegija **Obrada prirodnog jezika metodama dubinskog učenja**. Cilj projekta je klasifikacija emocija u kraćim tekstovima pomoću modela DistilBERT te analiza robusnosti modela na promjene ulaznog teksta.

Poseban naglasak stavljen je na ponašanje modela kada tekst sadrži tipografske pogreške, ponavljanje znakova ili dodanu interpunkciju.

## Istraživačko pitanje

**Je li full fine-tuning robusniji od treniranja samo klasifikacijskog sloja i može li augmentacija trening podataka povećati otpornost modela na tipografske pogreške i neformalno pisanje?**

## Skup podataka

Korišten je skup podataka `dair-ai/emotion`, koji sadrži kraće tekstove označene jednom od šest emocija:

- sadness
- joy
- love
- anger
- fear
- surprise

Skup je podijeljen na:

- 16 000 trening primjera
- 2 000 validacijskih primjera
- 2 000 testnih primjera

Analizom skupa utvrđena je neravnoteža među klasama, zbog čega je **makro F1** korišten kao glavna metrika za usporedbu modela.

Tekstovi u skupu također su izrazito normalizirani. Gotovo ne sadrže interpunkciju, velika slova ni emojije, dok su ponavljanja znakova vrlo rijetka. Zbog toga je dodatno ispitana otpornost modela na takve promjene teksta.

## Model

Korišten je predtrenirani model `distilbert-base-uncased`.

Tekstovi su tokenizirani DistilBERT tokenizerom uz maksimalnu duljinu od 96 tokena. Svi modeli koriste istu osnovnu arhitekturu i tokenizaciju kako bi njihovi rezultati bili usporedivi.

Uspoređena su tri pristupa:

### Model A – Full fine-tuning

Treniraju se svi parametri DistilBERT enkodera i klasifikacijskih slojeva.

### Model B – Zamrznuti enkoder

Parametri DistilBERT enkodera ostaju zamrznuti, dok se treniraju samo `pre_classifier` i `classifier` slojevi.

### Model C – Full fine-tuning uz augmentaciju

Model koristi potpuno fino ugađanje kao Model A, ali je trening skup proširen augmentiranim primjerima.

Izvornih 16 000 trening primjera prošireno je s:

- 25 % primjera s tipografskim pogreškama
- 25 % primjera s ponavljanjem znakova

Ukupni trening skup Modela C tako sadrži 24 000 primjera.

## Testiranje robusnosti

Uz izvorni testni skup napravljene su tri njegove izmijenjene verzije:

- **Typo** – iz jedne riječi uklanja se jedan znak
- **Repetition** – jedan znak unutar riječi višestruko se ponavlja
- **Punctuation** – na kraj teksta dodaje se interpunkcija

Punctuation nije korišten tijekom augmentacije trening skupa Modela C, nego predstavlja dodatnu prethodno neviđenu promjenu.

Za evaluaciju su korištene točnost, preciznost, odziv, F1-mjera i makro F1. Dodatno je promatran pad makro F1 u odnosu na izvorni testni skup.

## Glavni rezultati

Na izvornom testnom skupu dobiveni su sljedeći rezultati:

| Model | Točnost | Makro F1 |
|---|---:|---:|
| Model A | 0,9310 | 0,8845 |
| Model B | 0,4960 | 0,2011 |
| Model C | 0,9220 | 0,8708 |

Model A ostvario je najbolji rezultat na izvornom testnom skupu, dok se Model B sa zamrznutim enkoderom pokazao znatno slabijim.

Model C nije poboljšao rezultat na izvornim tekstovima, ali je pokazao manju degradaciju pri ciljanim promjenama:

| Promjena | Pad makro F1 – Model A | Pad makro F1 – Model C |
|---|---:|---:|
| Typo | 0,0627 | 0,0462 |
| Repetition | 0,0445 | 0,0304 |
| Punctuation | 0,0077 | -0,0026 |

Rezultati pokazuju da je augmentacija imala umjeren pozitivan učinak na robusnost kod tipografskih pogrešaka i ponavljanja znakova.

## Demonstracija

Projekt je razvijen i treniran u **Google Colab** okruženju uz korištenje **NVIDIA T4 GPU-a**.

Za praktičnu demonstraciju konačnog modela izrađeno je jednostavno **Gradio sučelje**. Korisnik može unijeti vlastitu rečenicu, nakon čega Model C provodi klasifikaciju i prikazuje predviđenu emociju.

Sučelje omogućuje isprobavanje modela i na vlastitim primjerima koji sadrže tipografske pogreške ili neformalno pisanje.

## Ograničenja

Projekt ima nekoliko ograničenja:

- korišten je samo jedan osnovni model, DistilBERT
- eksperimenti su provedeni na jednom skupu podataka
- skup podataka je neravnotežan i izrazito normaliziran
- korištene augmentacije predstavljaju jednostavne, umjetno generirane promjene
- augmentacije ne obuhvaćaju sve oblike pogrešaka i neformalnog pisanja iz stvarnih korisničkih tekstova
- Model B nije zasebno optimiziran, nego je treniran u istom osnovnom eksperimentalnom postavu radi usporedbe

Zbog navedenih ograničenja rezultati se odnose prvenstveno na korišteni eksperimentalni postav i ne mogu se automatski generalizirati na druge modele i skupove podataka.

## Mogućnosti budućeg rada

Projekt se može proširiti na nekoliko načina. Moguće je koristiti realističnije oblike tekstualnog šuma, kao što su višestruke tipografske pogreške, skraćenice, sleng, velika i mala slova te emoji.

Također bi se mogli ispitati drugi Transformer modeli i skupovi podataka s manje normaliziranim korisničkim tekstovima. Dodatno bi se moglo istražiti kako različite količine i vrste augmentacije utječu na odnos između uspješnosti na čistom tekstu i robusnosti na izmijenjenom tekstu.

## Zaključak

Rezultati pokazuju da je **potpuno fino ugađanje DistilBERT-a znatno uspješnije od treniranja samo klasifikacijskih slojeva** u korištenom eksperimentalnom postavu.

Augmentacija nije poboljšala ukupnu uspješnost na izvornom testnom skupu, ali je smanjila pad makro F1 kod tipografskih pogrešaka i ponavljanja znakova. Rezultati stoga upućuju na to da i jednostavna augmentacija podataka može doprinijeti robusnosti modela na ciljane promjene ulaznog teksta.
