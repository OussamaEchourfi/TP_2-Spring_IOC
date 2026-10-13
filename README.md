# TP2 / LAB2 : Inversion de Contrôle (IoC) et Injection de Dépendances (DI) avec Spring

Ce projet a pour objectif de concevoir et développer une application Java basée sur le framework **Spring**, illustrant les concepts clés de l'**Inversion de Contrôle (IoC)** et de l'**Injection de Dépendances (DI)**. Il met en pratique la configuration par annotations, la gestion de plusieurs implémentations avec `@Qualifier`, la configuration alternative via fichier XML, les profils Spring (`@Profile`), ainsi que les tests unitaires avec JUnit.

---

## 📐 Architecture & Diagramme de classes

L'application repose sur un couplage faible entre la couche DAO (accès aux données) et la couche Métier grâce à l'utilisation d'interfaces.

<p align="center">
  <img src="src/b5e16b50-e40d-4f61-9a53-3ef70cf6716a.png" alt="Diagramme de Classes UML" width="650"/>
</p>

### Composants principaux :
- **Couche DAO** :
  - Interface `IDao` : définit la méthode `double getValue()`.
  - Implémentation `DaoImpl` : retourne `100.0` (associée au profil `"prod"`).
  - Implémentation `DaoImpl2` : retourne `150.0` (associée au profil `"dev"`).
- **Couche Métier** :
  - Interface `IMetier` : définit la méthode `double calcul()`.
  - Implémentation `MetierImpl` : dépend de l'interface `IDao` et calcule `dao.getValue() * 2`.
- **Couche Présentation** :
  - `Presentation2` : configuration et instanciation du contexte Spring basé sur les annotations.
  - `PresentationXML` : chargement du contexte Spring à partir du fichier XML `applicationContext.xml`.

---

## 📁 Structure du projet

```text
TP2_LAB2/
├── pom.xml
├── README.md
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/
    │   │       └── example/
    │   │           ├── dao/
    │   │           │   ├── IDao.java
    │   │           │   ├── DaoImpl.java
    │   │           │   └── DaoImpl2.java
    │   │           ├── metier/
    │   │           │   ├── IMetier.java
    │   │           │   └── MetierImpl.java
    │   │           └── presentation/
    │   │               ├── Presentation2.java
    │   │               └── PresentationXML.java
    │   └── resources/
    │       └── applicationContext.xml
    └── test/
        └── java/
            └── com/
                └── example/
                    └── metier/
                        └── MetierImplTest.java
```

---

## 🛠️ Extraits et Résultats des Exécutions

### 1. Compilation du projet avec Maven

Compilation propre du projet avec `mvn clean compile` sous JDK 21 :

<p align="center">
  <img src="src/Screenshot 2026-10-13 163816.png" alt="Compilation Maven réussie" width="750"/>
</p>

---

### 2. Injection de dépendances via Annotations (Cas de base)

Exécution de la classe `Presentation2` avec injection par défaut du bean `DaoImpl` (`getValue() = 100.0`) :
- Calcul effectué : $100.0 \times 2 = 200.0$.

```bash
mvn exec:java "-Dexec.mainClass=com.example.presentation.Presentation2"
```

<p align="center">
  <img src="src/Screenshot 2026-10-13 163853.png" alt="Exécution Presentation2 - Résultat 200.0" width="750"/>
</p>

---

### 3. Spécification de l'implémentation avec `@Qualifier`

Lorsque plusieurs implémentations de `IDao` sont présentes (`DaoImpl` et `DaoImpl2`), l'annotation `@Qualifier("dao2")` permet d'indiquer explicitement à Spring d'injecter `DaoImpl2` (`getValue() = 150.0`) :
- Calcul effectué : $150.0 \times 2 = 300.0$.

<p align="center">
  <img src="src/Screenshot 2026-10-13 164157.png" alt="Exécution avec @Qualifier - Résultat 300.0" width="750"/>
</p>

---

### 4. Tests Unitaires avec JUnit

Validation de la couche métier indépendamment du conteneur Spring en injectant manuellement une implémentation fictive (*mock*) de `IDao` :

```bash
mvn test
```

<p align="center">
  <img src="src/Screenshot 2026-10-13 164658.png" alt="Tests unitaires réussis avec JUnit" width="750"/>
</p>

---

### 5. Configuration XML alternative

Utilisation du fichier `applicationContext.xml` pour déclarer les beans et injecter la dépendance par setter (`<property name="dao" ref="dao2"/>`), puis exécution via `PresentationXML` :

```bash
mvn exec:java "-Dexec.mainClass=com.example.presentation.PresentationXML"
```

<p align="center">
  <img src="src/Screenshot 2026-10-13 164706.png" alt="Exécution PresentationXML" width="750"/>
</p>

---

### 6. Gestion des profils Spring (`@Profile`)

Configuration dynamique des environnements :
- `DaoImpl` annoté avec `@Profile("prod")` (retourne `100.0`).
- `DaoImpl2` annoté avec `@Profile("dev")` (retourne `150.0`).

Dans `Presentation2`, le profil actif est configuré dynamiquement :
```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
context.getEnvironment().setActiveProfiles("dev");
context.register(Presentation2.class);
context.refresh();
```
Le profil `"dev"` active automatiquement `DaoImpl2`, produisant en sortie `Version de développement` et le résultat `300.0`.
