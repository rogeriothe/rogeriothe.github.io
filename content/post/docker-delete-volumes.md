+++
author = "Hugo Authors"
title = "Rich Content"
date = "2023-10-26"
description = "A brief description of Hugo Shortcodes"
tags = [
    "docker",
    "volumes",
]
thumbnail = "images/dollar.png"

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

