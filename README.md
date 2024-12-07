# ETL proces datasetu Walmart Sales Forecast

Tento repozitár obsahuje implementáciu ETL procesu v Snowflake pre analýzu dát z datasetu Walmart Sales Forecast. Projekt sa zameriava na preskúmanie predajov, sezónnych výkyvov a faktorov ovplyvňujúcich spotrebiteľský dopyt, čo umožňuje presnejšie predpovedať budúce predaje. Výsledný dátový model umožňuje multidimenzionálnu analýzu a vizualizáciu kľúčových metrík.  

## 1. Úvod a popis zdrojových dát  

Cieľom semestrálneho projektu je analyzovať dáta súvisiace s predajmi obchodov Walmart, aby sa identifikovali sezónne trendy, predpovedali tržby a optimalizovali stratégie riadenia zásob. Táto analýza pomáha spoločnosti prijímať informovanejšie rozhodnutia týkajúce sa nákupov, investícií a marketingových kampaní.  

Zdrojové dáta pochádzajú z datasetu Walmart Sales Forecast dostupného [tu](https://www.kaggle.com/datasets/aslanahmedov/walmart-sales-forecast/data). Dataset obsahuje tri hlavné tabuľky:  

- features.csv
- stores.csv
- train.csv 

Účelom ETL procesu bolo tieto dáta pripraviť, transformovať a sprístupniť pre viacdimenzionálnu analýzu.

### 1.1 Dátová architektúra
ERD diagram
Surové dáta sú usporiadané v relačnom modeli, ktorý je znázornený na entitno-relačnom diagrame (ERD):
