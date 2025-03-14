# AWS Profile Switcher

**AWS Profile Switcher** – это консольная утилита, написанная на Python и предназначенная для быстрой смены профилей учетных данных AWS, которые хранятся в файле `credentials` по пути `~/.aws/credentials`. На данный момент не поддерживает какие-либо сложные системы конфигурирования профилей от AWS, по типу SSO и т.д. Больше всего подходит, когда у вас есть N-ое количество классических профилей, между которыми необходимо быстро переключаться.

## **Мотивация:**

Основной мотивацией, помимо очевидного изучения Python, послужила ситуация, когда у нас есть несколько независимых аккаунтов в AWS и с каждым из них приходится активно взаимодействовать. В частности, вносить какие-то изменения в инфраструктуру с помощью Terraform. Это значит, что когда файл credentials от AWS выглядит следующим образом:

```
# Company 1
[default]
aws_access_key_id = your_access_key
aws_secret_access_key = your_secret_key

# Company 2
#[default]
#aws_access_key_id = your_access_key
#aws_secret_access_key = your_secret_key
```

И нам нужно переключиться на работу с Company 2, приходится редактировать содержимое самого файла (а это комментирование и раскомментирование). Конечно у консольной утилиты от самого AWS есть поддержка профилей. И она отлично работает. Вот только всё это плохо вяжется с Terraform и ситуацией, когда нам необходимо выполнить `terraform apply` для одной инфраструктуры, а потом резко переключиться и сделать тоже самое для другой (в Outsource такое бывает, ага). Сильно в Интернете я не искал, но «из коробки» какого-то внятного решения не нашел, а профили менять руками уже подустал. Поэтому решил написать свою утилиту, которая в два счета подкинет тот профиль в файл `credentials`, который нужен в данный момент времени.

## **Установка:**

Для успешного взаимодействия с данной утилитой необходимо обладать минимальным количеством программных коммпонентов и вспомогательных инструментов, поскольку сама утилита не несет каких-либо зависимостей. В частности, это:

- `Python` версии 3 для запуска непосредственно самой утилиты;

- `Git` с помощью которого вами будет загружен репозиторий из Github;

- `Wget`, `curl` и `jq`, которые понадобятся для загрузки бинарной версии из релиза;

- Файл в формате JSON, в котором будут храниться все профили (об этом более подробно чуть ниже);

### **Из исходного кода:**

Чтобы начать работу с исходной версией приложения достаточно выполнить следующую последовательность команд:

```
git clone https://github.com/exitfound/aws-profile-switcher.git
cd aws-profile-switcher
python3 aws.py -h
```

### **Бинарная версия:**

Данный метод основан на упаковке Python-скрипта в бинарный файл с помощью инструмента `Pyinstaller`. Является наиболее удобным и предпочтительным вариантом. Сам бинарник хранится в релизе. Чтобы начать работу с бинарной версией необходимо выполнить следующую последовательность команд:

```
wget $(wget -q -O - https://api.github.com/repos/exitfound/aws-profile-switcher/releases/latest | jq -r '.assets[] | select(.name | contains ("aps")) | .browser_download_url')
unzip aps_linux_amd64.zip
sudo mv aps /usr/local/bin/
aps -h
```

Примечание: Альтернативный путь, по которому также можно забрать архив с бинарным файлом с помощью `wget`:

```
wget https://github.com/exitfound/aws-profile-switcher/releases/latest/download/aps_linux_amd64.zip
```

### **С помощью Docker:**

Данный метод не сильно удобен в работе с этой утилитой, поскольку необходимо монтировать директорию .aws/, что в свою очередь может повлечь за собой потерю данных. Метод является опциональным, но в случае применения необходимо выполнить следующую последовательность команд:

```
docker build -t "awstool:local" --build-arg UID=$UID .
docker run -dit --name awstool -v ~/.aws/:/home/awstool/.aws/ awstool:local
docker exec -it awstool python3 aws.py -h
```

## **Быстрый старт:**

Используйте следующую команду (в зависимости от применяемого типа утилиты) для заполнения файла с профилями своими учетными данными от AWS:

```
python3 aws.py -g
aps -g
```

Используйте следующую команду (в зависимости от применяемого типа утилиты) для вывода списка текущих профилей (опционально):

```
python3 aws.py -l
aps -l
```

Используйте следующую команду (в зависимости от применяемого типа утилиты) для сохранения оригинального файла `credentials` (опционально):

```
python3 aws.py -o
aps -o
```

Используйте следующую команду (в зависимости от применяемого типа утилиты) для загрузки желаемого профиля в файл `credentials`:

```
python3 aws.py -p [имя_профиля]
aps -g [имя_профиля]
```

После этого вы можете перейти к работе с указанными учетными данными AWS. Это может быть как Terraform, так и консольная утилита от AWS. Ниже представлен весь процесс работы, а также более подробно рассказано о том, для чего нужен каждый аргумент.

## **Работа с утилитой:**

Для успешного взаимодействия с утилитой необходимо создать файл в формате JSON с именем `profiles.json` и разместить его там же, где находится файл `credentials`, то бишь в директории `.aws/`. В самом репозитории находится пример данного файла, имя которого `profiles.json.example`. Это поведение по умолчанию, но при желании и использовании специального ключа, данный файл можно хранить где угодно. Сам файл будет содержать в себе структуру следующего вида:

```
{
    "profiles": [
        {
            "name": "medoed",
            "aws_access_key": "access_key",
            "aws_secret_key": "secret_key"
        },
        {
            "name": "dvp",
            "aws_access_key": "access_key",
            "aws_secret_key": "secret_key"
        },
        {
            "name": "limbo",
            "aws_access_key": "access_key",
            "aws_secret_key": "secret_key"
        },
        {
            "name": "tesla",
            "aws_access_key": "access_key",
            "aws_secret_key": "secret_key"
        }
    ]
}
```

Это и есть место хранения всех наших учетных данных от AWS. Вы можете удалять или добавлять желаемые профили в зависимости от ситуации. При этом данный файл также можно сгенерировать с помощью используемой утилиты. По умолчанию, без указания пути, он (файл) будет размещён по пути `~/.aws/profiles.json`. Сделать это можно с помощью следующей команды:

```
python3 aws.py -g [путь_расположения (опционально)]
```

После выполнения команды выше вам будет предложено ввести произвольное имя профиля, а также access и secret ключи. Последние два значения представляют собой аналог `aws_access_key_id` и `aws_secret_access_key` в оригинальном файле `credentials`. Также стоит понимать, что файл с исходными значениями будет перезатерт после вызова того или иного профиля, поэтому если вы хотите сохранить файл, используйте следующую команду:

```
python3 aws.py -o [путь_расположения (опционально)]
```

По умолчанию, без указания пути, аргумент, представленный выше, сохранит оригинальный файл `credentials` с именем `credentials.original` в той же директории. Если вы хотите вывести список всех профилей, которые на текущий момент записаны в файл `profiles.json`, используйте следующую команду:

```
python3 aws.py -l
```

Чтобы загрузить один из существующих профилей, которые находятся в файле `profiles.json`, используйте следующую команду:

```
python3 aws.py -p [имя_профиля]
```

После выполнения команды из примера выше в файл `credentials` будет загружен указанный ранее профиль. Чтобы загрузить новый профиль просто выполните команду ещё раз, только теперь укажите имя нового профиля. Если по каким-то причинам вы хотите разместить содержимое файла `credentials` по иному пути, указав соответствующее имя, используйте следующую команду:

```
python3 aws.py -a [путь_к_файлу] -p [имя_профиля]
```

Если вы сгенерировали файл в формате JSON, который не соответствует пути по умолчанию, то в таком случае следующая команда поможет вам указать путь до этого файла (где хранятся ваши профили):

```
python3 aws.py -j [путь_к_JSON_файлу] -p [имя_профиля]
```

Как уже было отмечено ранее, если вы хотите расширить или сократить файл с профилями, вы можете просто отредактировать его с помощью любого удобного для вас редактора. Однако новый профиль можно подгрузить к уже существующему списку профилей, использовав для этого следующую команду:

```
python3 aws.py -e [путь_к_JSON_файлу (опционально)]
```

А если вы хотите удалить тот или иной профиль, используйте вот такую команду:

```
python3 aws.py -d [имя_профиля] -j [путь_к_JSON_файлу (опционально)]
```

## Бинарная сборка:

Если у вас возникла потребность в самостоятельной сборке бинарного файла, можно воспользоваться следующей командой:

```
pip3 install pyinstaller
pyinstaller --onefile --noconfirm --clean --name aps aws.py
./dist/aps -h
```
