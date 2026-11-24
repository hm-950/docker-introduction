# Partir d'une image Node.js légère
FROM node:18-alpine

# Dossier de travail dans l'image
WORKDIR /app

# Installer les dépendances (ici, l'outil Vite qui construit le site)
COPY package.json package-lock.json ./
RUN npm install

# Copier le reste du code (HTML, JavaScript, CSS, images, questions)
COPY . .

# Construire le site dans le dossier dist, puis installer le petit serveur serve
RUN npm run build && npm install -g serve

# L'appli écoute sur le port 3000
EXPOSE 3000

# Au démarrage : servir le dossier dist
CMD ["serve", "-s", "dist"]