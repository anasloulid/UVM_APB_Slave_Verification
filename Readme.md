##### **1️⃣ Objectif du projet**



Ce projet a pour but de créer un environnement de vérification complet en UVM (Universal Verification Methodology) pour un APB Slave.



Les objectifs principaux sont :



*-Vérifier le respect du protocole APB par le DUT (Device Under Test).*



*-Valider toutes les opérations de lecture et d’écriture.*



*-Comparer automatiquement les transactions DUT avec un modèle de référence (GOLDEN).*



*-Générer des logs détaillés et des fichiers de formes d’onde pour l’analyse.*



 



##### **2️⃣Architecture de l’environnement**



######                                                                          Testbench Top

######                                                                             ├── Générateur d’horloge et de reset

######                                                                             ├── Interface APB (intf)

######                                                                             ├── DUT (apb\_slave)

######                                                                             └── UVM Environment (APB\_ENV)

######                                                                                      ├── Agent (APB\_AGENT)

######                                                                                      │     ├── Driver (APB\_DRIVER)

######                                                                                      │     ├── Monitor (APB\_MONITOR)

######                                                                                      │     └── Sequencer (APB\_SEQUENCER)

######                                                                                      ├── Scoreboard (APB\_SCOREBOARD)

######                                                                                      └── Modèle de référence

###### **Composants clés**



Interface APB : regroupe tous les signaux nécessaires (Psel, Penable, Pwrite, Paddr, Pwdata, Prdata, Pready, Pslverr) et contient deux clocking blocks :



DRV\_CB pour piloter le DUT depuis le driver.



MON\_CB pour observer les signaux côté monitor.



Testbench : génère l’horloge, le reset et configure l’interface virtuelle pour tous les composants UVM.





##### **3️⃣ Composants UVM**





###### a) Transaction – APB\_TRANS



*Contient tous les champs d’une transaction APB.*



*Champs aléatoires : PADDR, PWRITE, Pwdata.*



*Contraintes : PADDR entre 0 et 1, PWRITE distribué 70% write / 30% read.*



*Utilisation des macros UVM pour impression et gestion automatique des champs.*





###### b) Séquence – APB\_SEQUENCE



*Génère des transactions aléatoires.*



*Envoie les transactions au séquenceur via start\_item et finish\_item.*





###### c) Sequencer – APB\_SEQUENCER



*Reçoit les transactions de la séquence.*



*Distribue les transactions au driver via le port seq\_item\_port.*





###### d) Driver – APB\_DRIVER



*Reçoit la transaction et pilote le DUT selon le protocole APB :*



*Setup Phase : assignation de Paddr, Pwdata, Pwrite et activation de Psel.*



*Enable Phase : activation de Penable et attente de Pready.*



*Signale la fin de la transaction avec item\_done.*



*Log UVM pour chaque transaction.*





###### e) Monitor – APB\_MONITOR



*Observe les signaux du DUT via MON\_CB.*



*Capture la transaction complète et l’envoie au scoreboard via ap\_port.write().*



*Log UVM pour chaque transaction observée.*





###### f) Scoreboard – APB\_SCOREBOARD



*Compare les transactions DUT (actual) avec celles attendues du modèle de référence (expected).*



*Affiche un UVM\_INFO en cas de correspondance et un UVM\_ERROR en cas de mismatch.*





###### g) Modèle de référence – APB\_REFERENCE



*Simule le comportement attendu du DUT.*



*Stocke les données écrites dans une mémoire interne.*



*Fournit les valeurs attendues lors des lectures.*



*Envoie les transactions GOLDEN au scoreboard via put\_port.*





##### **4️⃣ Flux complet d’une transaction**





*La séquence génère une transaction APB\_TRANS.*



*La transaction est transmise au sequencer, puis au driver.*



*Le driver exécute la transaction sur le DUT : Setup Phase → Enable Phase → Fin.*



*Le monitor observe les signaux du DUT et crée une transaction observée.*



*Le modèle de référence génère la transaction attendue (GOLDEN).*



*Le scoreboard compare les transactions DUT et GOLDEN et affiche pass/mismatch.*



*Tous les signaux sont enregistrés dans dump.vcd pour visualisation.*





Schéma simplifié :

###### 

######           APB\_SEQUENCE

######                │

######                ▼

######          APB\_SEQUENCER

######                │

######                ▼

######           APB\_DRIVER

######                │

######                ▼

######               DUT

######                │

######                ▼

######          APB\_MONITOR ────────────────

######                │                    │

######                ▼                    ▼

######              SCOREBOARD <─ APB\_REFERENCE

######                │

######                ▼

######        Log Pass / Error

###### 

