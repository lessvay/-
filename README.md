# Лабораторная работа №1:

## Установка и настройка

### 1. Создание папок для виртуальных дисков

Создание папки для хранения образа диска:

```bash
mkdir ~/wsl_disks
```

Создание образов дисков по 1 гб каждый:

```bash
dd if=/dev/zero of=~/wsl_disks/log_partition.img bs=1M count=1024
dd if=/dev/zero of=~/wsl_disks/backup_partition.img bs=1M count=1024
```

Форматирование образов

```bash
mkfs.ext4 ~/wsl_disks/log_partition.img
mkfs.ext4 ~/wsl_disks/backup_partition.img
```

Создание точек монтирования

```bash
sudo mkdir /LOG
sudo mkdir /BACKUP
```

Монтирование образов

```bash
sudo mount ~/wsl_disks/log_partition.img /LOG
sudo mount ~/wsl_disks/backup_partition.img /BACKUP
```

Проверка монтирования

```bash
df -h | grep /LOG
df -h | grep /BACKUP
```

### 2. Скрипт для проверки заполненности LOG

Cоздание файла check_log_usage.sh

```bash
nano check.sh
```

Скрипт

```bash
#!/bin/bash

LOG_DIR="/LOG"
BACKUP_DIR="/BACKUP"
THRESHOLD=70
N=5  # Количество последних файлов для архивирования

# Информацию о заполнении папки в процентах
USAGE=$(df -h "$LOG_DIR" | awk 'NR==2 {print $5}' | sed 's/%//')

# Проверяем, сколько процентов заполнено
if [ "$USAGE" -ge "$THRESHOLD" ]; then
    echo "Заполнение папки $LOG_DIR составляет $USAGE%. Начинаем архивирование..."

    # Получаем список последних N файлов по дате модификации
    FILES_TO_ARCHIVE=$(ls -t "$LOG_DIR" | head -n "$N")

    if [ -z "$FILES_TO_ARCHIVE" ]; then
        echo "Нет файлов для архивирования."
        exit 1
    fi

    # Создаём архив с выбранными файлами из /LOG
    TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
    ARCHIVE_NAME="$BACKUP_DIR/log_backup_$TIMESTAMP.tar.gz"

    # Архивируем последние N файлов и удаляем их из /LOG
    tar -czf "$ARCHIVE_NAME" -C "$LOG_DIR" $FILES_TO_ARCHIVE && rm -rf "$LOG_DIR"/$FILES_TO_ARCHIVE

    echo "Файлы из $LOG_DIR заархивированы в $ARCHIVE_NAME и удалены из $LOG_DIR."
else
    echo "Заполнение папки $LOG_DIR составляет $USAGE%. Архивирование не требуется."
fi
```
Делаем скрипт исполняемым 
```bash
chmod +x check.sh
```
Запуск скрипта
```bash
./check.sh
```
### 3. Тестовый скрипт

Создание файла test.sh

```bash
nano test.sh
```

Скрипт

```bash
#!/bin/bash

# Скрипт для проверки основного скрипта

LOG_DIR="/LOG"
BACKUP_DIR="/BACKUP"
THRESHOLD=70
N=5  # Количество последних файлов для архивирования


# Очистка старых тестовых файлов
echo "Очистка папок /LOG и /BACKUP..."
sudo rm -rf "$LOG_DIR"/*
sudo rm -rf "$BACKUP_DIR"/*

# Шаг 2: Создание тестовых файлов в /LOG
echo "Создание тестовых файлов в /LOG..."
for i in {1..10}; do
    dd if=/dev/zero of="$LOG_DIR/test_file_$i.txt" bs=1M count=10
    sleep 1  # Разные даты модификации для файлов
done

# Шаг 3: Заполняем файловую систему до порога
echo "Заполнение файловой системы до порога $THRESHOLD%..."
FILLER_FILE="$LOG_DIR/filler_file.bin"
dd if=/dev/zero of="$FILLER_FILE" bs=1M count=700  # Добавляем файл для достижения 70% заполненности

# Шаг 4: Проверка заполненности папки перед запуском основного скрипта
USAGE=$(df -h "$LOG_DIR" | awk 'NR==2 {print $5}' | sed 's/%//')
echo "Текущее заполнение папки /LOG: $USAGE%"

# Шаг 5: Запуск основного скрипта
echo "Запуск основного скрипта..."
./check_log_usage.sh

# Шаг 6: Проверка результата архивирования
echo "Проверка папки /BACKUP после архивирования..."
if [ "$(ls -A $BACKUP_DIR)" ]; then
    echo "Файлы были заархивированы:"
    ls -lh "$BACKUP_DIR"
else
    echo "Файлы не были заархивированы."
fi

# Шаг 7: Проверка очистки папки /LOG
echo "Проверка папки /LOG после архивирования..."
if [ "$(ls -A $LOG_DIR)" ]; then
    echo "Файлы остались в /LOG:"
    ls -lh "$LOG_DIR"
else
    echo "/LOG пуста после архивирования."
fi
```
Делаем скрипт исполняемым 
```bash
chmod +x test.sh
```
Запуск скрипта
```bash
./test.sh
```
Установка прав доступа

```bash
sudo chmod 777 /BACKUP
sudo chmod 777 /LOG
```

Запуск теста

```bash
./test_full_functionality.sh
```
