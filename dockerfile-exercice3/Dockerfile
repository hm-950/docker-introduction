# Partir d'une image Python légère
FROM python:3.12-slim

# Dossier de travail dans l'image
WORKDIR /app

# Installer les bibliothèques listées dans requirements.txt (Flask, gunicorn...)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code de l'appli (app.py et le dossier templates)
COPY . .

# L'appli écoute sur le port 5000
EXPOSE 5000

# Au démarrage : lancer l'appli avec le serveur gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]