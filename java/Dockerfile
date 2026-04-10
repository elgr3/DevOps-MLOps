# Utilisation d'une image JRE légère
#FROM eclipse-temurin:21-jre-alpine

# On définit le répertoire de travail dans le conteneur
#WORKDIR /usr/src/app

#COPY Main.class .

# Commande pour exécuter l'application au démarrage du conteneur
#CMD ["java", "Main"]

FROM eclipse-temurin:21-jdk-alpine
# Build Main.java with JDK
# TODO : in next steps (not now)

FROM eclipse-temurin:21-jre-alpine
# Copy resource from previous stage
COPY --from=0 /usr/src/Main.class .
# Run java code with the JRE
# TODO : in next steps (not now)
