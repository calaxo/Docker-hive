# docker-hive

une stack docker basé sur Vivekpawar07/Docker-hive, elle meme basé sur https://github.com/big-data-europe/docker-hadoop pour avoir un environnement de développement rapide avec hdfs hive et hue, les images docker sont normalement encore dispo sur dockerhub, mais je ne sais pas d'ou elles viennent ni si elle sont sécurisé


utilisé durant un bloc projet Big Data pour l'année 4 d'ingenieur informatique au CESI pour remplacer la vm cloudera quickstart qui est trop compliqué a metre en place


peut avoir de nombreux bug attention pour l'utilisation en production

un ficher .jar est présent pour connecter Talend( en l'occurence Talaxie 8, https://deilink.fr/#/download ) a Hive et HDFS avec un connection custom, il a été contruit dans le container namenode

un fichier host fichier conf est permet de ne pas utiliser localhost(dans talend)


## Contributors
* Ivan Ermilov [@earthquakesan](https://github.com/earthquakesan) (maintainer)
* Yiannis Mouchakis [@gmouchakis](https://github.com/gmouchakis)
* Ke Zhu [@shawnzhu](https://github.com/shawnzhu)
