# Étape 1 : Utiliser l'image de base Apache
FROM httpd:2.4

# Étape 2 : Copier la page HTML dans le dossier de contenu web par défaut d'Apache
COPY index.html /usr/local/apache2/htdocs/

COPY httpd.conf /usr/local/apache2/conf/httpd.conf

# Étape 3 : Exposer le port 80 
EXPOSE 80
