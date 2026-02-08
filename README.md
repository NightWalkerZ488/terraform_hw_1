# Домашнее задание к занятию «Введение в Terraform» - Лоскутов В.В.

### Цели задания

1. Установить и настроить Terrafrom.
2. Научиться использовать готовый код.

------

### Чек-лист готовности к домашнему заданию

1. Скачайте и установите **Terraform** версии >=1.12.0 . Приложите скриншот вывода команды ```terraform --version```.
2. Скачайте на свой ПК этот git-репозиторий. Исходный код для выполнения задания расположен в директории **01/src**.
3. Убедитесь, что в вашей ОС установлен docker.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. Репозиторий с ссылкой на зеркало для установки и настройки Terraform: [ссылка](https://github.com/netology-code/devops-materials).
2. Установка docker: [ссылка](https://docs.docker.com/engine/install/ubuntu/). 
------
### Внимание!! Обязательно предоставляем на проверку получившийся код в виде ссылки на ваш github-репозиторий!
------

### Задание 1

1. Перейдите в каталог [**src**](https://github.com/netology-code/ter-homeworks/tree/main/01/src). Скачайте все необходимые зависимости, использованные в проекте. 
2. Изучите файл **.gitignore**. В каком terraform-файле, согласно этому .gitignore, допустимо сохранить личную, секретную информацию?(логины,пароли,ключи,токены итд)
3. Выполните код проекта. Найдите  в state-файле секретное содержимое созданного ресурса **random_password**, пришлите в качестве ответа конкретный ключ и его значение.
4. Раскомментируйте блок кода, примерно расположенный на строчках 29–42 файла **main.tf**.
Выполните команду ```terraform validate```. Объясните, в чём заключаются намеренно допущенные ошибки. Исправьте их.
5. Выполните код. В качестве ответа приложите: исправленный фрагмент кода и вывод команды ```docker ps```.
6. Замените имя docker-контейнера в блоке кода на ```hello_world```. Не перепутайте имя контейнера и имя образа. Мы всё ещё продолжаем использовать name = "nginx:latest". Выполните команду ```terraform apply -auto-approve```.
Объясните своими словами, в чём может быть опасность применения ключа  ```-auto-approve```. Догадайтесь или нагуглите зачем может пригодиться данный ключ? В качестве ответа дополнительно приложите вывод команды ```docker ps```.
7. Уничтожьте созданные ресурсы с помощью **terraform**. Убедитесь, что все ресурсы удалены. Приложите содержимое файла **terraform.tfstate**. 
8. Объясните, почему при этом не был удалён docker-образ **nginx:latest**. Ответ **ОБЯЗАТЕЛЬНО НАЙДИТЕ В ПРЕДОСТАВЛЕННОМ КОДЕ**, а затем **ОБЯЗАТЕЛЬНО ПОДКРЕПИТЕ** строчкой из документации [**terraform провайдера docker**](https://library.tf/providers/kreuzwerker/docker/latest).  (ищите в классификаторе resource docker_image )

------

### Выполнение:

## 1.

Установка терраформа:

![install](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/install_tf.png)

Клонирование репо:

![clone](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/clone_git.png)

Инициализация:

![init](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/init.png)

## 2. Согласно содержимому файла .gitignore, личную секретную информацию можно хранить в файле personal.auto.tfvars, т.к. он добавлен в игнор.

## 3. Выполняем код и находим содержимое "random_password":

![run](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/tf_apply.png)

![pass](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/tfstate.png)

## 4. Раскомментируем блок кода и выполним "terraform validate":

![validate](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/validate.png)

Итак, "resource "docker_image"" — отсутствует имя ресурса (должно быть docker_image.nginx), далее
имя контейнера "1nginx" начинается с цифры — недопустимо в Terraform. Также "random_password.random_string_FAKE.resulT" — опечатки в имени ресурса (_FAKE) и атрибуте (resulT вместо result).

## 5. Исправляем и выполняем код:

 ```
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string.result}"

  ports {
    internal = 80
    external = 9090
  }
}
```

![run_validate](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/run_validate.png)


## 6. Заменяем имя контейнера и выполняем код: 


![run_hw](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/h_world.png)

Опасность ключа "-auto-approve" заключается в том, что он ропускает интерактивное подтверждение плана изменений, что может привести к случайному удалению или изменению критичных ресурсов. В продакшене такое опасно.
Данный ключ может быть поллезен в CI/CD пайплайнах, где нет интерактивного ввода, в скрипитах и автоматизированных процессах.

## 7. Уничтожаем ресурсы и смотрим содержимое "tfstate":

```
terraform destroy -auto-approve
cat terraform.tfstate

```

![remove](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/tf_final.png)

## 8. Образ nginx:latest не удалился потому, что в ресурсе "docker_image.nginx" установлен параметр "keep_locally = true", так образ не удаляется при выполнении команды "terraform destroy".

![1](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/locally.png)

![2](https://github.com/NightWalkerZ488/terraform_hw_1/blob/main/main_tf.png)
