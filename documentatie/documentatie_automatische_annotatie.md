## Stap 1 - Inlezen en Voorbereiden

- Data ingelezen uit `annotatie_formulier_inductief_random42_air.xls`
- Woordenlijst ingelezen uit `woordenlijst_max_2_themas.xlsx`
- Aantal rijen in data: 994
- Aantal woorden in woordenlijst: 6472
- Kolommen in data: unique_id, document_name, date, text_title, first_paragraph, match_score, theme_1, clarification_theme_1, theme_2, clarification_theme_2, main_theme, clarification_main_theme, specific_subject, clarification_subject, scope, context, model_prediction_theme_1, model_prediction_theme_2, model_prediction_main_theme, agreement_manual_auto, doubt, opmerking 
- Kolommen in woordenlijst: thema, woord

## Stap 2.1 - Setup spaCy

- spaCy geïnstalleerd via pip: `pip install spacy`
- Nederlands taalmodel `nl_core_news_lg` succesvol gedownload via `python -m spacy download nl_core_news_lg`
- spaCy-pipeline succesvol geladen met `spacy.load('nl_core_news_lg')`
- Klaar om tekstsegmenten te verwerken via tokenisatie en lemmatisering

## Stap 2.2 - Selectie testvoorbeelden

- Functie `selecteer_meerdere_voorbeelden()` geïmplementeerd om meerdere unieke ID’s tegelijk te selecteren.
- Handmatig 8 `unique_id`-voorbeelden geselecteerd op basis van duidelijke contextbeschrijvingen in de annotatie.
- Deze selectie wordt gebruikt om de werking van de pipeline te testen op relatief eenduidige gevallen.
- Binnen deze selectie zijn 7 verschillende thema's door de handmatige annotatie geïdentificeerd.

## Stap 2.2 - Selectie testvoorbeelden

- Functie `selecteer_meerdere_voorbeelden()` geïmplementeerd om meerdere unieke ID’s tegelijk te selecteren.
- Handmatig 8 `unique_id`-voorbeelden geselecteerd op basis van duidelijke contextbeschrijvingen in de annotatie.
- Deze selectie wordt gebruikt om de werking van de pipeline te testen op relatief eenduidige gevallen.
- Binnen deze selectie zijn 7 verschillende thema's door de handmatige annotatie geïdentificeerd.

## Stap 2.3 - Woordenlijst omzetten naar .json-format

- De oorspronkelijke `df_woordenlijst` is omgezet naar een nested dict in JSON-vorm.
- Spellingfout 'gemeenschapleven' is opgeschoond en samengevoegd met 'gemeenschapsleven'.
- Het resultaat is opgeslagen als `thema_woordenlijst_gecorrigeerd.json`.

## Stap 2.4 - Inlezen van themawoordenlijst uit JSON

- Het JSON-bestand `thema_woordenlijst_gecorrigeerd.json` is ingelezen en omgezet naar een dict met sets.
- Deze structuur wordt nu gebruikt als input voor de thema-analyse.

## Stap 2.5 - Opstellen functie theme_matcher(...)

- De functie `theme_matcher(...)` verwerkt tekstsegmenten met behulp van spaCy.
- Per lemma wordt gekeken of het voorkomt in de themawoordenlijst.
- Scores per thema worden berekend op basis van frequentie en aantal unieke lemma's.
- Alle matches worden opgeslagen voor interpretatie en logging.

## Stap 2.5.1 - Test theme_matcher(...) op 8 zelf geselecteerde voorbeelden (text_title)

- De test is uitgevoerd op 8 handmatig geselecteerde items uit het corpus (met voldoende context).
- Analyse is toegepast op `text_title`.
- Resultaten:
- - 5/8 = 1 van de automatisch herkende thema’s was correct
- - 1/8 = een foutief thema werd toegekend
- - 2/8 = geen thema geïdentificeerd

## Stap 2.6 - Test theme_matcher(...) op 8 voorbeelden (first_paragraph)

- Dezelfde 8 testvoorbeelden zijn geanalyseerd op `first_paragraph`.
- Ten opzichte van `text_title` werd in de meeste gevallen meer of rijkere thematische context herkend.
- Voorbeeld: NU_961 bevatte 6 thema's in de paragraaf t.o.v. 2 in de titel.
- NU_71 en NU_706 gaven geen match op titel, maar wel duidelijke matches op paragraaf.
- Meerdere thema’s zoals 'oorlog', 'gemeenschapsleven' en 'cultuur & entertainment' kwamen uitsluitend in de paragraaf naar voren.
- De resultaten bevestigen dat `first_paragraph` essentieel is voor een volledige thematische analyse.

## Stap 3.1 – Beslisregel segmentkeuze voor main_theme

- De segmentkeuze voor het bepalen van het dominante thema volgt vaste regels:
- - Als er overlap is tussen thema's in `text_title` en `first_paragraph`: analyseer beide segmenten gecombineerd.
- - Als beide segmenten wel thema's bevatten maar geen overlap: gebruik alleen `text_title`.
- - Als er geen thema's zijn in `text_title`, maar wel in `first_paragraph`: analyseer alleen de paragraaf.
- - Als geen van beide segmenten een thema bevat: markeer als 'geen thema gevonden'.
- Deze selectie bepaalt de tekstinput waarop de dominante thema-analyse wordt toegepast.
- De reden voor deze aanpak is dat de dataset enkele onvolkomenheden bevat:
- - Soms is de tekst in `first_paragraph` gekoppeld aan een onjuiste titel.
- - Heel sporadisch bevat `text_title` een typfout of irrelevante combinatie.
- Met deze regels zorgen we dat alleen tekst met consistente thematische inhoud wordt meegenomen in de analyse.


## Stap 3.2 – Functie opstellen voor bepaling analyse segmenten

- Functie `bepaal_analyse_input(...)` opgesteld om per voorbeeld te bepalen welk(e) tekstsegment(en) moeten worden geanalyseerd.
- De functie gebruikt de themamatches uit `text_title` en `first_paragraph` om de bron van de analyse te bepalen.
- De logica is als volgt:
- - Overlap in thema’s tussen titel en paragraaf → analyseer beide gecombineerd
- - Thema’s zonder overlap in beide → gebruik alleen de titel
- - Alleen thema’s in paragraaf → gebruik alleen de paragraaf
- - Geen thema’s in beide segmenten → geen analyse
- Deze aanpak biedt controle over segmentselectie en minimaliseert ruis door incorrecte koppeling van tekst en titel in de dataset.

## Stap 3.3 – Functie toegepast op geselecteerde testvoorbeelden

- De functie `bepaal_analyse_input(...)` is toegepast op de eerder gekozen set van 8 testvoorbeelden.
- Per voorbeeld is vastgelegd welke segmenten zijn geanalyseerd: `text_title`, `first_paragraph` of beide.
- De themamatches per segment zijn weergegeven in het volgende format:
- - Thema's in titel: [...]
- - Thema's in paragraaf: [...]
- - Analysebron: titel / paragraaf / beide / geen
- Deze stap vormt de basis voor de daaropvolgende main_theme-bepaling.

## Stap 4 – Selectie van top-2 thema’s bij >2 geclassificeerde thema’s

- Functie `selecteer_top2_themas(...)` opgesteld om uit meerdere herkende thema’s de twee meest relevante te selecteren.
- Eerste selectie gebeurt op basis van `score_freq`, met tie-breaker via `score_uniek`, en als laatste fallback alfabetische volgorde.
- Deze aanpak is voldoende voor de meeste gevallen, maar kan willekeurig worden als meerdere thema’s exact gelijke scores hebben.

## Stap 4.1 – Test `selecteer_top2_themas(...)` op voorbeeld NU_4222

- Bij NU_4222 bleken meerdere thema’s gelijke `score_freq` te hebben: politiek en economie.
- De selectie viel hierdoor willekeurig uit; dit onderstreepte de noodzaak van semantische disambiguatie.

## Stap 4.1.1 – Modelkeuze voor semantische disambiguatie

- Twee modellen zijn overwogen: SBERT (multilingual) en GroNLP/bert-base-dutch-cased (specifiek Nederlands).
- SBERT is gekozen vanwege eenvoudige integratie, snelheid, en voldoende semantische kwaliteit voor disambiguatie tussen thema’s.
- In een latere fase kan GroNLP worden overwogen voor bijvoorbeeld main_theme-identificatie, mits de performance van SBERT tekortschiet.

## Stap 5 – Semantische disambiguatie via embeddings

- Disambiguatie wordt toegepast als meerdere thema’s exact gelijke score hebben.
- SBERT vergelijkt de tekst met voorbeeldzinnen per thema, en selecteert het thema met de hoogste cosine similarity.
- Het gekozen thema vervangt de zwakkere kandidaat uit top 2 (bijv. economie ↔ politiek).

## Stap 6 – Functie `sbert_disambiguatie(...)` opgesteld

- Functie vergelijkt `analyse_input` met voorbeeldzinnen per thema via SBERT.
- Geeft cosine similarity per thema terug, inclusief de best scorende zin per thema.
- Teruggegeven thema met hoogste semantische overeenkomst wordt gekozen als tweede thema.

## Stap 6.1 – Test `sbert_disambiguatie(...)` op voorbeeld NU_4222

- Disambiguatie werd toegepast tussen `economie` en `politiek`, beide met gelijke score.
- SBERT gaf voorkeur aan `politiek` met cosine similarity 0.476 vs 0.452.
- Beslissing werd transparant gelogd in compact format: `politiek: 0.476 > economie: 0.452`.

## Stap 6.2 – SBERT als fallback bij top-2 themaselectie

- SBERT wordt nu alleen uitgevoerd wanneer disambiguatie noodzakelijk is.
- Concreet: als het tweede en derde thema gelijke `score_freq` hebben.
- Reden: bij analyse van volledige dataset zou SBERT op elk item de logboeken vervuilen en onnodig vertragen.
- Er is daarom een aparte fallback-functie opgesteld die SBERT alleen oproept als het nodig is.

## Stap 6.3 – Test nieuwe fallbackfunctie `selecteer_top2_themas_met_fallback(...)`

- Functie getest op voorbeeld `NU_4222`, waar `politiek` en `economie` exact gelijke scores hadden.
- SBERT werd correct geactiveerd, met transparante log van cosine similarity tussen beide thema’s.
- De fallbackfunctie werkt zoals bedoeld en vervangt bij ambiguïteit het tweede thema op basis van semantische nabijheid.

## Stap 7 – Bepaling van het main_theme (frequentie → uniek → SBERT → NER)

- Deze stap beschrijft de hiërarchische structuur voor themabepaling.
- In eerste instantie wordt gekeken naar `score_freq`, dan naar `score_uniek`, gevolgd door SBERT-disambiguatie.
- Indien nodig wordt in een latere stap nog een NER-booster toegepast.

## Stap 7.1 – Eerste implementatie: main_theme bepalen op basis van `score_freq`

- Bij deze aanpak werd het thema met de hoogste frequentiescore als dominant geselecteerd.
- Bij tests bleek deze aanpak onvoldoende robuust, met verkeerde keuzes bij thematische ambiguïteit.
- Er is besloten deze logica niet te gebruiken als eindselectie.

## Stap 7.2 – Directe inzet van SBERT-disambiguatie

- Om ambiguïteit structureel beter op te lossen, wordt SBERT nu standaard ingezet bij selectie van het main_theme.
- Zelfs als `score_freq` verschillen toont, wordt alsnog SBERT gebruikt om het meest semantisch passende thema te kiezen.

## Stap 7.4 – Optimalisatie: standaardisering van themanamen in SBERT-disambiguatie

- Er is een helperfunctie toegevoegd binnen `sbert_disambiguatie(...)` om hoofdletters, spaties en tekens te normaliseren.
- Dit voorkomt fouten bij het koppelen van thema’s aan contextzinnen in `thema_seedzinnen`.
- Sinds deze fix zijn alle 8 testvoorbeelden correct en zonder foutmeldingen geclassificeerd.

## Stap 7.5 – Check op ontbrekende thema’s in `thema_seedzinnen`

- Een controleloop vergelijkt de gedetecteerde thema’s in de testvoorbeelden met de keys in `thema_seedzinnen`.
- Resultaat: er zijn geen ontbrekende thema’s meer.
- Dit garandeert dat SBERT-disambiguatie altijd uitgevoerd kan worden indien nodig.

## Stap 8.0 – Inbouwen NER-booster

- Een extra themabepalingslaag is toegevoegd op basis van named entity recognition (NER).
- Doel: als SBERT onvoldoende onderscheid maakt, kan een thematische entiteit in de tekst helpen om het juiste main_theme te bepalen.
- NER-entiteiten zijn verzameld in het bestand `ner_thema_boosters.py`, waarin per thema relevante entiteiten zijn gekoppeld.
- Het scriptiepad is ingesteld om dit bestand correct te importeren in JupyterLab.

## Stap 8.1 – Thema-booster toepassen bij twijfelachtige SBERT-scores

- De functie `pas_ner_booster_toe(...)` is ingevoerd.
- Deze functie wordt alleen geactiveerd wanneer het verschil tussen de SBERT-scores van de top-2 thema’s kleiner is dan 0.02.
- Daarnaast geldt dat de frequentiescores van beide thema’s dicht bij elkaar moeten liggen (verschil ≤ 1.0).
- Bij een overlap tussen NER-entiteiten in de tekst en een van de top-2 thema’s wordt dit thema als main_theme gekozen.

## Stap 8.2 – Test van de NER-booster

- De functie is getest op de 8 eerder geselecteerde voorbeelden.
- De implementatie werkt zoals bedoeld, maar leverde in deze testfase nog geen booster-hits op.
- Dit komt waarschijnlijk doordat de lijst met boosters in `ner_boosters` nog beperkt is.
- De lijst met entiteiten per thema zal later worden uitgebreid op basis van de inhoud van het corpus en geobserveerde named entities.

## Stap 9.0 – Opstellen van datasets voor scope-herkenning

- Er zijn drie afzonderlijke lijsten opgesteld om geografische scope te kunnen detecteren in tekst:
- - Een landenlijst (UN data)
- - Een lijst met wereldsteden (GeoNames)
- - Een lijst met Nederlandse steden, gemeenten, provincies en landsdelen (CBS)
- De drie lijsten worden opgeslagen in afzonderlijke JSON-bestanden en later samengevoegd in de scope-analyse.

## Stap 9.1 – Voorbereiden vertaling van landennamen

- De landenlijst van de Verenigde Naties bevat Engelse namen.
- Om deze bruikbaar te maken binnen een Nederlandstalige dataset is vertaling noodzakelijk.
- Er is gekozen voor het gebruik van het open source pakket `argos-translate`.
- Model voor EN → NL succesvol gedownload en geïnstalleerd.
- Vertaalfunctie opgesteld en getest op losse voorbeelden.

## Stap 9.1.2 – Landenlijst als JSON-bestand opslaan

- De originele CSV van de UN bevat de kolom ‘Country or Area’.
- Deze kolom is automatisch vertaald naar het Nederlands.
- De uiteindelijke JSON bevat een lijst van Nederlandstalige landennamen.
- Dit bestand zal worden gebruikt als scope-lijst bij landenherkenning.

## Stap 9.2.0 – Wereldstedenlijst opstellen via GeoNames

- De lijst is gebaseerd op `cities500.txt`, afkomstig van GeoNames.
- Deze bevat 183.630 steden wereldwijd (met >500 inwoners).
- Alle stadsnamen zijn uit het txt-bestand gehaald en als JSON opgeslagen.
- Namen zijn (nog) niet genormaliseerd naar lowercase – dit gebeurt pas bij gebruik.

## Stap 9.3.0 – Nederlandse geografische entiteiten via CBS

- De bron is CBS open data (dataset: 85877NED).
- De volledige dataset is via een OData-API opgehaald en in pandas ingeladen.
- De dataset bevat: woonplaatsnamen (`Naam_2`), gemeente (`Naam_2`), provincie (`Naam_4`) en landsdeel (`Naam_6`).

## Stap 9.3.1 – Opslaan van gescheiden JSON-bestanden

- Op basis van de CBS-data zijn drie lijsten opgebouwd:
- - `nederlandse_woonplaatsen.json` met alle unieke plaatsnamen
- - `nederlandse_provincies.json` met alle provincies
- - `nederlandse_landsdelen.json` met alle landsdelen
- Deze bestanden zijn bedoeld voor latere identificatie van geografische scopes op verschillende niveaus.

## Stap 10 – Scope-detectie op basis van geografische entiteiten

- Regels opgesteld voor herkenning van geografische scopes in vaste volgorde: NL-steden → provincies → landsdelen → landen → wereldsteden.
- Scope wordt geclassificeerd als binnenland/buitenland/geen scope o.b.v. prioriteit en overlap tussen lijsten.

## Stap 10.1 – Eerste test op 8 voorbeelden

- Eerste test op 8 unieke ID’s uitgevoerd met basisversie van `detecteer_scope(...)`.
- Resultaat: Te veel false positives uit wereldstedenlijst (bijv. 'Een', 'Over').

## Stap 10.2 – Nieuwe `detecteer_scope(...)` functie opgesteld

- Wereldstedenlijst wordt nu alleen gebruikt indien de plaatsnaam met hoofdletter wordt genoemd én geen stopwoord is.
- Functie controleert expliciet op woordgrenzen en hoofdlettergebruik.

## Stap 10.3 – Tweede test op 8 voorbeelden

- False positives sterk gereduceerd.
- Nieuwe foutcategorie ontstaan: enkele false negatives, zoals ‘Amsterdam’ wordt niet altijd herkend.

## Stap 10.4 – Optimalisatie via woordgrenzen en hoofdlettergevoeligheid

- Custom `bevat_woordgrens(...)` functie toegevoegd die SpaCy gebruikt voor nauwkeurige matchcontrole.
- Lijsten opgeschoond (trailing whitespace verwijderd).

## Stap 10.5 – Test geoptimaliseerde functie met locatiehits

- Amsterdam, Gaza, Sri Lanka worden nu correct herkend.
- False positives als 'Moth' en 'Heel' komen niet meer voor — werking verbeterd.

## Stap 10.6 – Functie `bepaal_scope_uitkomst(...)` toegevoegd

- Definieert scope op basis van combinatie van hits (binnenlandse entiteiten hebben voorrang).
- Wereldsteden worden alleen als buitenlands geteld als ze niet ook in de Nederlandse stedenlijst staan.

## Stap 10.7 – Test van `bepaal_scope_uitkomst(...)` op 8 voorbeelden

- Functioneert zoals bedoeld. Correcte toewijzing aan binnenland of buitenland.
- Artikelen zonder geografische entiteiten geven ‘geen scope’ terug.

## Stap 11.0 – Fallback gebaseerd op named entities

- Wanneer geen geografische entiteiten worden herkend, wordt een fallback geactiveerd.
- Fallback controleert of bekende Nederlandse personen of instellingen worden genoemd.
- Resultaat: classificatie ‘binnenland’ of ‘onbekend’.

## Stap 12.0 – Volledige scope-analyse pipeline getest

- Scope-analyse succesvol toegepast op 8 testvoorbeelden.
- Functioneert correct met hiërarchische structuur: geografische hits → fallback op entiteiten.
- Fallback-structuur en lijsten kunnen in een latere fase nog worden uitgebreid.

## Stap 13.0 – Opstellen nieuw .xls-bestand voor automatische annotatie

- Het bestaande handmatig geannoteerde Excelbestand is ingeladen.
- Voorbereidingen getroffen om automatisch gegenereerde metadata toe te voegen.

## Stap 13.1 – Toevoegen van kolommen voor automatische voorspellingen

- Kolommen toegevoegd voor: model_prediction_theme_1, model_prediction_theme_1_clarification, model_prediction_theme_2, model_prediction_theme_2_clarification, model_prediction_main_theme, model_prediction_main_theme_clarification.
- Daarnaast voorbereid op logging van scope-informatie: scope_detectie_input, scope_detectie_hits, scope_detectie_eindscope, scope_detectie_fallback.

## Stap 13.2 – Herschikken van kolommen

- Kolommen herschikt in logische structuur: metadata, handmatige annotatie, automatische voorspellingen, scope-analyse, overig.
- Volgorde afgestemd op leesbaarheid en vergelijkbaarheid tussen handmatige en automatische velden.

## Stap 13.3 – Opslaan van tussentijdse versie van Excelbestand

- Excelbestand opgeslagen als `annotatie_formulier_automatisch_random42_air.xlsx`.
- Structuur gecontroleerd; bestaande waarden behouden, nieuwe velden initieel op `---` gezet.

## Stap 13.4 – Controle op metadata en kolomvolgorde

- Visuele en programmatische controle uitgevoerd op aanwezigheid en volgorde van kolommen.
- Alle geplande metadata zijn aanwezig in het bestand.

## Stap 13.5 – Toevoegen van extra kolommen voor logging

- Kolommen toegevoegd voor logging van tussenstappen in de thematische analyse en scope-analyse.
- Nieuwe kolommen: analyse_input_segmenten, top2_themas_freq, top2_themas_fallback, sbert_scores, main_theme_reden, ner_booster_hits, sbert_scoreverschil_main, frequentie_scoreverschil_main, scope_detectie_nl_steden, scope_detectie_provincies, scope_detectie_landsdelen, scope_detectie_landen, scope_detectie_wereldsteden.

## Stap 13.6 – Kolommen herschikt in definitieve volgorde

- Alle kolommen opnieuw geordend zodat inhoudelijke samenhang tussen handmatige en automatische annotatie behouden blijft.
- Loggingvelden zijn gegroepeerd per fase in de analysepipeline (segmentselectie, themabepaling, scope).

## Stap 13.7 – Opslaan als definitieve versie

- Het Excelbestand met alle nieuwe kolommen en loggingstructuur is opgeslagen als `annotatie_formulier_automatisch_random42_air_def.xlsx`.
- Dit bestand dient nu als centrale dataset voor de automatische annotatierondes.

## Stap 13.8 – Laatste metadata-check op bestand

- Alle kolomnamen, waarden en volgorde gecontroleerd.
- Voorbeeldwaarden visueel geïnspecteerd om te garanderen dat er geen opschoonfouten zijn opgetreden.

## Tussenstap – Vooruitblik op verdere verwerking

- Alle afzonderlijke functies voor themabepaling en scopedetectie zijn getest en werken stabiel.
- Het bestand wordt nu gebruikt om de eerste 200 rijen automatisch te annoteren.
- Na deze batch volgt analyse en finetuning van specifieke componenten zoals disambiguatie, boosterstrategie en scope-inferentie.
- Daarna worden alle functies opgeschoond en overgezet naar een apart `.py`-bestand voor herbruikbaarheid en overzichtelijkheid.

