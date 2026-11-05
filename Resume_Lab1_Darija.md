# 🛡️ Guide de Révision & Synthèse: M115 Lab 1 (Phishing Detection)
**Master SSIC (Intelligent Systems & Cyber Defense) — ENSIASD Taroudant**  
**Module M115: AI & Intelligent Systems**

Ce document résume de façon simple, claire et intuitive tout ce qu'on a compris de chaque exercice, ainsi que les concepts clés de Machine Learning et de Cybersécurité indispensables pour l'examen et la soutenance.

---

## 📌 PARTIE 1: Security-Oriented EDA (Exploratory Data Analysis)

### 🔹 Exercice 1: SOC Target Alignment (Inversion du Label)
* **L-Fekra:** F l-fichier CSV l-asli dial PhiUSIIL, kano dayrin `label = 1` pour Legitimate (site 3adi) w `label = 0` pour Phishing (Attaque).
* **3lach ghalat f SOC?** F l-Cybersecurity, la classe positive `1` khassha dima tkoun hia **l-Attaque** bash les métriques (Precision, Recall, True Positives) y9isso ch7al men attaque kchefna, machi ch7al men site 3adi l9ina!
* **Chnu drna:** 
  $$\text{IsPhishing} = 1 - \text{label}$$
  L9ina bli l-dataset fih **42.81% Phishing** w **57.19% Legitimate** (données quasi-équilibrées).

---

### 🔹 Exercice 2: Distributions & Indicateurs de Sécurité Binaires
* **L-Fekra:** Qarn-na bin Phishing (1) w Legitimate (0) f 4 dial les indicateurs binaires w 3 dial les variables numériques.
* **Ahem l-Mola7adat f l-Cybersecurity:**
  1. **`IsHTTPS`:** 100% dial sites légitimes fihom HTTPS, walakin taqriban **50% dial sites de phishing 7tta humma fihom HTTPS!** ➡️ *Règle statique "If HTTPS then Safe" raaha khatira w kat-dowwez n-noss dial les attaques.*
  2. **`HasSocialNet` & `HasCopyrightInfo`:** Tal3in bzaf f les sites officiels (~80%) w habtin bzaf f Phishing (< 6%). Hacker ma 3ndoch l-weqt y-dir comptes réseaux sociaux w mentions légales l page fake ghadi t3ich 24h.
  3. **`NoOfSelfRef`:** Les sites 3adiyin fihom dizaines dial les liens dakheliyin (navigation, footer...), alors que 90%+ dial Phishing fihom **0 liens dakheliyin** (page wa7da khawya).
  4. **`URLLength`:** Les URLs de phishing twal bzaf fihom des sous-domaines w des tokens mkhbyin.

---

### 🔹 Exercice 3: Identifier Cleanup & Corrélations de Pearson
* **L-Fekra:** Khassna n-7iydo les colonnes de texte brut: `FILENAME`, `URL`, `Domain`, `TLD`, `Title`.
* **3lach? (Overfitting par Mémorisation):** Ila khllina smiyet l-site (bhal `paypal-fake.com`), l-modèle ghadi y-7fed les domaines l-qdam (Blacklist). Ghedda f production mlli yji hacker b domaine jdid ma 3mer l-modèle chafou (Zero-day attack), l-modèle ghadi y-dokh w y-9emmer (gamble). Khasso y-t3ellem les caractéristiques universelles, machi smiyet l-site.
* **Pearson Correlation ($r$):**
  * $r < 0$ (Vert): Indicateurs dial site légitime (`HasSocialNet`, `HasCopyrightInfo`).
  * $r > 0$ (Rouge): Indicateurs dial Phishing (`SpacialCharRatioInURL`, `DegitRatioInURL`).

---

## 📌 PARTIE 2: Zero-Data Leakage Preprocessing & Standardization

### 🔹 Exercice 4: Stratified Split & StandardScaler (Règle d'or SecMLOps)
* **1. Train (75%) vs Test (25%):**
  * Train (176,846 URLs) hwa l-khedma f d-dar bash l-modèle y-trena.
  * Test (58,949 URLs) hwa l-imti7an l-finaal mkhbbi f coffre-fort ma 3mro chafou.
  * `stratify=y`: Kat-dmen bli nfs l-pourcentage dial phishing (42.8%) kayn f Train w f Test b joj.
* **2. 3lach StandardScaler? (Mouchkil dial l-Mizan):**
  * `LineOfCode` fiha arqam kbar (10,000+), w `IsHTTPS` fiha ghir (0 w 1).
  * Bla scaler, l-modèle kay-7sab lih `LineOfCode` mouhima 10,000 marra kter ghir hit arqamha kbar!
  * `StandardScaler` kay-red kolchi f nfs l-mizan: moyenne = 0, ecart-type = 1.
* **3. Zero Data Leakage:**
  * `.fit_transform(X_train)`: Kan7sbo la moyenne w l-ecart-type **ghir men Train**.
  * `.transform(X_test)`: Kan-tabb9o nfs l-mizan dial train 3la test bla ma n-choufo f test.
  * *Ila derti fit 3la data kamla qbel l-split, katssema gchiti (Data Leakage) hit l-modèle شاف l-imti7an qbel ma y-douzo.*

---

## 📌 PARTIE 3: Parameterized Logistic Regression & Interpretability

### 🔹 Exercice 5: Comparaison de 5 Modèles
On a comparé 5 configurations dial Logistic Regression:
1. **M1 (Unregularized / None):** Bla 7tta chi frein (pénalité = 0). Yqder y-overfitti w les poids dialo y-tferg3o (instabilité f production).
2. **M2 (Strong L2 / C=0.001):** Pénalité qas7a bzaf, kay-habba6 ga3 les poids 9rab l 0, zkel 105 phishing (Missed Phish).
3. **M3 (Standard L2 / C=1.0):** Le modèle standard équilibré, très performant w stable contre les attaques adversariales.
4. **M4 (Sparse L1 Lasso / C=0.01) ⭐ Le Champion du Firewall:**
   * **L1 Lasso** kat-msse7 les variables inutiles w kat-red l-poids dialhom **0.0000** (Sparsity).
   * **Natija:** Ms7at **24 variable** men asl 49 (khllat ghir 25 feature), w F1-score bqat taqriban hia hia (99.8%)!
   * **Fa2ida f l-Firewall:** Modèle 2x plus rapide f le filtrage temps réel!
5. **M5 (SOC Balanced):** Heavier penalty 3la les faux négatifs, kay-7di rasso kter m3a l-phishing.

* **Definitions Clés:**
  * **False Alarm (False Positive - FP):** Site 3adi walakin l-modèle gal phishing w bloukah (kay-derr l-khedma w kay-dir alert fatigue l l-analyste).
  * **Missed Phish (False Negative - FN):** Phishing 7a9i9i w l-modèle ma 3aqch bih w dowzou (akhtar 7aja f cybersecurity = breach).

---

### 🔹 Exercice 6: L1 Regularization Path (Feature Selection)
* **L-Fekra:** Kheddemna ghir b L1 Lasso bo7dha w bqina kan-le3bo b **$C$** (men 0.002 tal 5.0).
* **Chnu hia $C$?** Hia l-bouton dial le frein à main (inverse de la pénalité $C = 1/\lambda$):
  * **$C = 0.002$ (sghira bzaf = pénalité qas7a):** L-modèle msse7 **36 feature** w khlla ghir **13 feature** d darora l-qsswa!
  * **$C = 5.0$ (kbira bzaf = pénalité d3ifa):** L-modèle khlla ga3 les **49 features** 7eyyin (0 mms7in).
* **L-Courbe:** Kat-werri bli kol ma kber $C$, kol ma tzad 3adad les features li 7eyyin.
* **L-Calcul:** L-modèle (Scikit-Learn) hwa li kay7seb kolchi automatique f `fit()`.

---
*(Ce fichier sera complété au fur et à mesure qu'on avance dans Exercice 7, 8, 9, 10 et la synthèse!)*

