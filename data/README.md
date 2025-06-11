MLOps Meteo
=

# Structure du projet

Organisation des dossiers et fichiers du projet :

## Dossiers de données

- `data/raw` : Données brutes *(non suivies par Git - `.gitignore`)*
- `data/preprocessed` : Données traitées *(non suivies par Git - `.gitignore`)*
- `data/processed` : Données traitées *(non suivies par Git - `.gitignore`)*

## Résultats et artefacts

- `metrics` : Sauvegarde des métriques d'entraînement ou d'évaluation
- `models` : Sauvegarde des modèles entraînés
- `predictions` : Prédictions du modèle

## Documentation et analyses

- `notebooks` : Notebooks Jupyter pour l'exploration et l'analyse
- `references` : Documents ou fichiers de référence
- `reports` : Rapports générés (PDF, HTML, etc.)

## Code source

- `src/data` : Scripts de traitement des données
- `src/features` : Fonctions réutilisables (feature engineering, etc.)
- `src/models` : Scripts liés aux modèles (entraînement, évaluation)
- `src/visualization` : Scripts d'exploration et de visualisation des données

## Prérequis

- Kaggle API Key

    Pour télécharger les données brutes, vous devez disposer d’une clé API Kaggle.
    1. Créez un compte sur [Kaggle](https://www.kaggle.com/).
    2. Générez votre clé API dans votre profil (Settings > Account > API > Create New Token).
    3. Placez le fichier `kaggle.json` dans `~/.kaggle/` (Linux/Mac) ou `C:\Users\<VotreNom>\.kaggle\` (Windows).
    4. Installez la bibliothèque Kaggle : `pip install kaggle`.

- Configuration initiale

    Pour accéder au projet, vous devez cloner le répertoire correspondant.
    ```bash
    git clone https://github.com/DataScientest-Studio/AVR25-CMLOPS-METEO.git
    cd AVR25-CMLOPS-METEO
    ```

## Exécution automatisée du pipeline (avec Airflow)

1. Créer le fichier d'environnement pour Airflow
    ```bash
    echo AIRFLOW_UID=50000 > .env
    echo AIRFLOW_GID=0 >> .env
    echo _AIRFLOW_WWW_USER_USERNAME=airflow >> .env
    echo _AIRFLOW_WWW_USER_PASSWORD=airflow >> .env
    ```
2. Créer les dossiers nécessaires
    ```bash
    mkdir -p logs plugins data/raw data/preprocessed data/processed models metrics predictions
    ```
3. Définir les permissions
    ```bash
    sudo chown -R 50000:50000 dags logs plugins data/ models/ metrics/ predictions/
    sudo chmod -R 755 data/ models/ metrics/ predictions/
    ```
4. Lancement de Airflow
    - Initialiser Airflow : 
      ```bash
      docker-compose up airflow-init
      ```
   - Lancer tous les services : 
      ```bash
      docker-compose up -d
      ```
    - Vérifier l'état des services ("healthy") : 
      ```bash
      docker-compose ps
      ```
    - Aller sur http://localhost:8080 et se connecter (Username: airflow / Password: airflow)
    - Trouver le dag `meteo_daily_update`
    - Cliquer sur Trigger DAG (bouton play)


## Exécution manuelle du pipeline (sans Airflow)

- Instruction à lancer dans l'ordre suivant :

    1. python src/data/import_raw_data.py
    2. python src/data/update_data.py
    3. python src/features/preprocessing.py
    4. python src/data/split_data.py
    5. python src/models/train_model.py
    6. python src/models/evaluate_model.py

## Instruction FASTAPI - API UPDATE 

    uvicorn api.api_model:app --reload

    curl -X POST -u admin:admin http://localhost:8000/update_data
    
    curl -X POST -u admin:admin http://localhost:8000/preprocessing

    curl -X POST -u admin:admin http://localhost:8000/train

    curl -X POST -u admin:admin http://localhost:8000/evaluate

    curl -X 'POST' -u admin:admin \
    'http://127.0.0.1:8000/predict' \
        -H 'accept: application/json' \
        -H 'Content-Type: application/json' \
        -d '{
        "Location": "string",
        "MinTemp": 0,
        "MaxTemp": 0,
        "Rainfall": 0,
        "Evaporation": 0,
        "Sunshine": 0,
        "WindGustDir": "string",
        "WindGustSpeed": 0,
        "WindDir9am": "string",
        "WindDir3pm": "string",
        "WindSpeed9am": 0,
        "WindSpeed3pm": 0,
        "Humidity9am": 0,
        "Humidity3pm": 0,
        "Pressure9am": 0,
        "Pressure3pm": 0,
        "Cloud9am": 0,
        "Cloud3pm": 0,
        "Temp9am": 0,
        "Temp3pm": 0,
        "RainToday": true,
        "Month": 0,
        "Day": 0
        }'

## Streamlit
streamlit run src/streamlit/app.py

## Tests unitaires
pytest src/tests_unitaires/test_api.py
