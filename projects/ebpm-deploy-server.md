---
description: Прослойка между клиентом и ebpm.
devs:
  - "[[intkgc]]"
status: "#✒"
---

Этот проект запускается на сервере и при отправке файла через POST запрос "/upload" анализирует [[jwt-and-what-to-eat-it-with|токен]] пользователя, если аутентификация прошла успешно, то он принимает файл и дальше передаёт его [[ebpm]] для развёртывания.

Так же ebpm-deploy-server генерирует токены пользователя.