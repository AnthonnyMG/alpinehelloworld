### ETAPE pour test deploiement

1- Creer un job dans Jenkins avec les parametres correspondant au jenkinsfile
    >> dont le credentials docker hub et l'username docker hub dans les parametres "ID_DOCKER_PARAMS"

** Option Jenkins public -- EAZYLABTRAINING 
2- Creer des environments de deploiement sur EAZyLab (docker.labs.eazytraining.fr)
    2.1 : ADD TWO INSTANCE : franela/dind image instance
    2.2 :  Lanch EAZYLABS API Container ==> https://github.com/eazytraining/eazylabs

        docker run -d --name eazylabs --privileged -v /var/run/docker.sock:/var/run/docker.sock -p 1993:1993 eazytraining/eazylabs:latest
    2.3 : Verifier les urls de l'API dans "OPEN PORT" sur le port 1993 : 
        URL de type : ip10-0-1-3-cc7bafssrdn0fvnms4tg-1993.direct.docker.labs.eazytraining.fr
    2.4 : Remplacer les URL du Jenkinsfile par ce URL

** Option Jenkins local -- docker host localhost
2- Creer l'environment de deploiement local 
    2.1 : Lanch EAZYLABS API Container ==> https://github.com/eazytraining/eazylabs

        docker run -d --name eazylabs-staging  --privileged -v /var/run/docker.sock:/var/run/docker.sock -p 1994:1993 eazytraining/eazylabs:latest
        docker run -d --name eazylabs-staging  --privileged -v /var/run/docker.sock:/var/run/docker.sock -p 1993:1993 eazytraining/eazylabs:latest

    2.2 : Verifier les urls de l'API sur les url
        http://ip-host-docker.local:1994
        http://ip-host-docker.local:1993

    2.4 : Remplacer les URL du Jenkinsfile local par ce URL
    
3 - Launch Build