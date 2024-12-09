# ETL proces datasetu CSV Northwind Database

Tento repozitár obsahuje implementáciu ETL procesu v Snowflake pre analýzu dát z datasetu CSV Northwind Database. Projekt sa zameriava na analýzu predajných trendov, preferencií zákazníkov a optimalizáciu logistických procesov na základe údajov o kategóriách produktov, zákazníkoch, zamestnancoch, objednávkach a dodávkach. Výsledný dátový model umožňuje multidimenzionálnu analýzu a vizualizáciu kľúčových metrík.  

## 1. Úvod a popis zdrojových dát   

Cieľom semestrálneho projektu je analyzovať dáta týkajúce sa kategórií produktov, zákazníkov, zamestnancov, objednávok a dodávok. Táto analýza umožňuje identifikovať trendy v predaji, preferenciách zákazníkov a optimalizovať procesy v rámci spoločnosti.

Zdrojové dáta pochádzajú z datasetu CSV Northwind Database dostupného [tu](https://www.kaggle.com/datasets/cleveranjosqlik/csv-northwind-database/data). Dataset obsahuje nasledujúce hlavné tabuľky:  

- categories: Informácie o kategóriách produktov vrátane ich popisu a obrázkov.
- customers: Údaje o zákazníkoch, vrátane kontaktných informácií a geografických údajov.
- employee_territories: Prepojenie medzi zamestnancami a pridelenými teritóriami.
- employees: Informácie o zamestnancoch vrátane osobných údajov a pracovných pozícií.
- orders: Detaily o objednávkach, vrátane údajov o zákazníkoch, zamestnancoch a doprave.
- orders_details: Informácie o objednaných produktoch, cenách, množstvách a zľavách.
- products: Údaje o produktoch vrátane cien, skladových zásob a kategórií.
- regions: Informácie o regiónoch, v ktorých spoločnosť pôsobí.
- shippers: Dáta o prepravcoch, vrátane kontaktných informácií.
- suppliers: Informácie o dodávateľoch a ich kontaktné údaje.
- territories: Údaje o teritóriách vrátane ich geografických charakteristík.

Účelom ETL procesu bolo tieto dáta pripraviť, transformovať a sprístupniť pre viacdimenzionálnu analýzu.

### 1.1 Dátová architektúra
ERD diagram
Surové dáta sú usporiadané v relačnom modeli, ktorý je znázornený na entitno-relačnom diagrame (ERD):
