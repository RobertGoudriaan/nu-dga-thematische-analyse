## Stap 1 - Inlezen Excel-bestand

- Excel-bestand `annotatie_formulier_automatisch_random42_air_DEF.xlsx` succesvol ingeladen.
- Bevat zowel handmatige als automatische annotatie in één dataset.
- Gebruik van `pd.read_excel()` en controle op correcte kolomnamen uitgevoerd.

## Stap 2 - Overlap tussen thema 1 en 2 (top-2 overlap)

- Functie `thema_overlap` toegepast.
- Meet overlap tussen handmatige `theme_1`, `theme_2` en automatische `model_prediction_theme_1`, `model_prediction_theme_2`.
- Resultaten gelogd als `beide_themas_juist`, `een_thema_juist`, `geen_match`, inclusief percentages.

## Stap 3 - Prestatie per thema

- Overzicht gegenereerd op basis van lange themalijst (via `theme_1` en `theme_2`).
- Politiek, economie en oorlog worden goed herkend.
- Sport, reizen en identiteit & ongelijkheid hebben hoge herkenning, maar lage representatie (n<10).

## Stap 4 - Main_theme-identificatie

- Match tussen `theme_1` en `model_prediction_main_theme` berekend.
- Drie categorieën onderscheiden: `volledige match`, `in top-2, niet gekozen`, `geen match`.
- Slechts 56% exacte overeenstemming; 15% totale mismatch.

## Stap 5.0 - SBERT-scoreanalyse

- Aantal observaties met lage SBERT-scoreverschillen (< 0.05): 37 van 200.
- Waarschijnlijk indicatie van semantische ambiguïteit of thematische overlap.

## Stap 5.1 - Interpretatie SBERT

- SBERT-scoreverschillen zijn laag in 18,5% van de gevallen.
- Aanbevolen: uitbreiding van contextzinnen in `sbert_contextzinnen.py` naar minimaal 20 per thema.

## Stap 6.0 - NER-booster analyse

- NER-booster geactiveerd bij slechts 6 observaties (3%).
- In sommige gevallen correct, maar beperkt bereik door incomplete lijst.

## Stap 6.1 - Interpretatie NER-booster

- NER-lijsten aanvullen voor hogere dekking:
- - `nederlandse_entiteiten.py`: >50 per onderdeel
- - `ner_thema_boosters.py`: >50 lemma’s per thema

## Stap 7.0 - Frequentieverschil-analyse

- 21 observaties met klein frequentieverschil tussen top 2 thema’s.
- SBERT-fallback correct ingeschakeld in deze gevallen.

## Stap 7.1 - Interpretatie frequentieanalyse

- Klein verschil in frequentie (<1.0) komt voor bij 10,5% van de gevallen.
- SBERT-fallback functioneert, maar kan verbeterd worden in combinatie met NER-booster en contextzinnen.

## Stap 8.0 - Samenvatting en vervolgstappen

- Acties uitgevoerd:
- - `sbert_contextzinnen.py` uitgebreid tot ≥20 per thema [✓]
- - `nederlandse_entiteiten.py` aangevuld tot ≥50 per type [✓]
- - `ner_thema_boosters.py` uitgebreid tot ≥50 per thema [✓]
- Volgende stap: annotatie opnieuw uitvoeren en evaluatie herhalen.

## Stap 9 - Opnieuw runnen van annotatielus en scopelus

- Annotatielus en scopelus opnieuw uitgevoerd op volledige dataset.
- Scopelus functioneert nu correct en slaat informatie weg in de kolommen `scope_resultaat`, `scope_hits`, `scope_log`.
- Annotatielus opnieuw doorlopen zonder fouten.

## Stap 10 - Themaherkenning per thema (tweede run)

- Overzicht gegenereerd op basis van herkenning van `theme_1` en `theme_2`.
- Verbetering in herkenning van `economie` (100% juist) en afname van `geen_match` bij andere thema's.
- Algemeen beeld: themabereik verbeterd.

## Stap 10.1 - Interpretatie van stap 10

- Thematische herkenning licht verbeterd ten opzichte van eerste run.
- Vooral minder `geen_match`, wat wijst op breder themadekkingsbereik.
- Algoritme heeft meer titels correct gelabeld met minstens één correct thema.

## Stap 11 - Vergelijking main_theme: handmatig vs automatisch

- Vergelijking uitgevoerd tussen `main_theme` en `model_prediction_main_theme`.
- Indeling: 'volledige match', 'in top-2, niet gekozen', 'geen match'.
- Resultaat: 55.5% exacte match, 28% correct thema niet gekozen als main, 16.5% mismatch.

## Stap 11.1 - Interpretatie van stap 11

- Vergelijkbaar resultaat als in eerste run: minimale verbetering/verslechtering.
- Geen significante vooruitgang op main_theme-detectie.
- Volledige match blijft stabiel rond ~55%.

## Stap 12 - SBERT-score analyse

- SBERT-scoreverschil geanalyseerd: 45 van 200 observaties met verschil < 0.05.
- Deze lage verschillen duiden op onzekerheid in thematische disambiguatie.
- Aantal observaties met lage SBERT-scores gestegen t.o.v. eerste run.

## Stap 12.1 - Interpretatie van stap 12

- Toename van lage SBERT-scoreverschillen (van 37 naar 45).
- Model twijfelt vaker tussen thema’s, mogelijk door bredere dekking of overlap in contextzinnen.
- Aanvulling van contextzinnen blijft essentieel.

## Stap 13 - NER-booster analyse

- Aantal keer dat de NER-booster het main_theme heeft aangepast: 6 van 200.
- Log toont beperkte bijdrage van de booster in huidige versie.
- Geen foutmeldingen bij gebruik van `ner_thema_mapping` of entiteitendetectie.

## Stap 13.1 - Interpretatie van stap 13

- NER-booster is slechts in 3% van de gevallen geactiveerd.
- Geen verbetering t.o.v. eerste run.
- Aanbevolen: uitbreiding van `nederlandse_entiteiten.py` en `ner_thema_boosters.py` voor grotere impact.

## Stap 14 - Frequentieverschil analyse

- Frequentieverschil tussen top 2 thema's < 0.05 in 19 observaties.
- Iets lager dan in eerste run (21 observaties).
- Analyse uitgevoerd op basis van `frequentie_scoreverschil_main`.

## Stap 14.1 - Interpretatie van stap 14

- Lichte afname in frequentieverschillen toont dat het model minder afhankelijk is van fallbackmechanismen.
- SBERT-fallback wordt dus minder vaak geactiveerd.
- Toont meer overtuiging van het model in eerste score-keuze.

## Stap 15 - Vergelijking van scope handmatig vs automatisch

- Scope uit `scope_resultaat` vergeleken met handmatige `scope`.
- Resultaten geclassificeerd als: 'volledige match', 'geen match', 'ontbreekt handmatig'.
- Grootste categorie is 'geen match' met 67.5%.

## Stap 16.1 - Interpretatie van stap 15

- Automatische scope-detectie werkt ondermaats.
- Veel handmatige scopes worden niet herkend of correct geclassificeerd.
- Wordt niet meegenomen in eindanalyse.

## Stap 17.1 - Afsluitende beslissing m.b.t. scope

- Scope wordt niet verder geoptimaliseerd wegens tijdsdruk.
- Beperkte betrouwbaarheid wordt transparant besproken in de discussie.
- Alle overige componenten zijn stabiel genoeg om door te zetten naar volledige annotatie.

