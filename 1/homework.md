
# Подготовка и установка mesos на хосты с deploy приложений

## Генерируем приватный ключ на хосте с ansible

```bash
ssh-keygen
```
> Пользователь тот, под которым будем работать с ansible. Нажимаем enter несколько раз для генерации

## Копируем наш приватный ключ себе

```bash
cat /ct_admin/.ssh/id_rsa.pub
```
> далее руками будем его по всем хостам разносить

## На каждом хосте включая и сам хост ansible вставляем ранее сохраненный приватный ключ

```bash
sudo -i
echo "ранее_созданный_приватный_ключ" >> /root/.ssh/authorized_keys
```

## Устанавливаем ansible

```bash
sudo yum update -y
sudo yum install -y python3-pip
sudo pip3 install --upgrade pip
pip3 install "ansible==2.7.0"
```
## Клонируем наш репозиторий с git
- на хосте с ansible соответственно

```bash
cd /
git clone git@github.com:crowdtesting-ru/ufrpayroll-devops.git
```

## Создание и добавление jenkins как службы в автозагрузку
- создаем файл службы /etc/systemd/system/jenkins.service с следующим содержимым

```
