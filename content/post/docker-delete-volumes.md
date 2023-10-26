+++
author = "Rogério Lima"
title = "Excluindo todos os volumes do Docker"
date = "2023-10-26"
description = "Como excluir todos os volumes do Docker"
tags = [
    "docker",
    "volumes",
]
thumbnail = "images/301_docker.jpg"

+++
Após muitos testes em ambiente de desenvolvimento pode restar muitos volumes sem mais utilidade.

Uma forma de apagar todos os volumes do Docker:

---

Stop the container(s) using the following command:

docker-compose down

Delete all containers using the following command:

docker rm -f $(docker ps -a -q)

Delete all volumes using the following command:

docker volume rm $(docker volume ls -q)

Restart the containers using the following command:

docker-compose up -d

