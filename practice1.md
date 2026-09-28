# Практическая работа №1 — «Командная строка»

**Дисциплина:** Конфигурационное управление  
**Тема:** Командная строка

## Задание 1

Вывести отсортированный в алфавитном порядке список имён пользователей из `/etc/passwd`.

```bash
grep -o '^[^:]*' /etc/passwd | sort
```

## Задание 2

Вывести данные `/etc/protocols` в отформатированном и отсортированном виде для 5 наибольших номеров протоколов.

```bash
grep -v '^#' /etc/protocols | grep -v '^$' | awk '{print $2, $1}' | sort -nr | head -5
```

Пример результата:

```text
262 mptcp
143 ethernet
142 rohc
141 wesp
140 shim6
```

## Задание 3

Написать программу `banner` средствами Bash для вывода текста в рамке. Размер рамки должен меняться в зависимости от длины текста.

```bash
cat > banner <<'EOF'
#!/bin/bash

text="$*"
border=$(printf '%*s' "$(( ${#text} + 2 ))" '' | tr ' ' '-')

printf '+%s+\n' "$border"
printf '| %s |\n' "$text"
printf '+%s+\n' "$border"
EOF
```

```bash
chmod +x banner
./banner "Hello from RTU MIREA!"
shellcheck banner
```

Пример результата:

```text
+-----------------------+
| Hello from RTU MIREA! |
+-----------------------+
```

## Задание 4

Написать программу для вывода всех идентификаторов по правилам C/C++/Java без повторений.

```bash
cat > identifiers <<'EOF'
#!/bin/bash
grep -oE '[A-Za-z_][A-Za-z0-9_]*' "$1" | sort -u
EOF

chmod +x identifiers
```

Тестовый файл:

```bash
cat > hello.c <<'EOF'
#include <stdio.h>

int main(void) {
    printf("hello world\n");
    return 0;
}
EOF
```

Запуск:

```bash
./identifiers hello.c
```

## Задание 5

Написать программу для регистрации пользовательской команды: установить права доступа и скопировать программу в `/usr/local/bin`.

```bash
cat > reg <<'EOF'
#!/bin/bash

chmod 755 "$1"
cp "$1" /usr/local/bin/
EOF

chmod +x reg
```

Проверка на программе `banner`:

```bash
./reg banner
ls -l /usr/local/bin/banner
banner "Hello from RTU MIREA!"
```

## Задание 6

Проверить наличие комментария в первой строке файлов `.c`, `.js` и `.py`.

```bash
cat > comments <<'EOF'
#!/bin/bash

dir="${1:-.}"

find "$dir" -type f \( -name '*.c' -o -name '*.js' -o -name '*.py' \) -print0 |
while IFS= read -r -d '' file
do
    first=$(head -n 1 "$file")

    case "$file" in
        *.py)
            echo "$first" | grep -Eq '^[[:space:]]*#' && echo "$file"
            ;;
        *.c|*.js)
            echo "$first" | grep -Eq '^[[:space:]]*(//|/\*)' && echo "$file"
            ;;
    esac
done
EOF

chmod +x comments
```

Тест:

```bash
printf '// comment\nint main(){return 0;}\n' > test.c
printf '# comment\nprint("hello")\n' > test.py
printf 'console.log("hello");\n' > test.js

./comments .
```

## Задание 7

Найти файлы-дубликаты по содержимому в указанной директории и её подкаталогах.

```bash
cat > duplicates <<'EOF'
#!/bin/bash

find "$1" -type f -exec sha256sum {} + | sort |
awk '
$1 == prev {
    if (!shown) print prev_line
    print
    shown=1
}
$1 != prev {
    shown=0
}
{
    prev=$1
    prev_line=$0
}'
EOF

chmod +x duplicates
```

Тест:

```bash
mkdir -p testdup/sub
echo "hello duplicate" > testdup/a.txt
cp testdup/a.txt testdup/b.txt
cp testdup/a.txt testdup/sub/c.txt
echo "different" > testdup/d.txt

./duplicates testdup
```

## Задание 8

Найти все файлы указанного расширения и архивировать их в `tar`.

```bash
cat > archive_ext <<'EOF'
#!/bin/bash

dir="$1"
ext="$2"

find "$dir" -type f -name "*.$ext" -print0 |
tar --null -T - -cf archive.tar
EOF

chmod +x archive_ext
```

Тест:

```bash
mkdir -p testarchive/sub
touch testarchive/a.txt
touch testarchive/b.txt
touch testarchive/sub/c.txt
touch testarchive/test.py

./archive_ext testarchive txt
tar -tf archive.tar
```

## Задание 9

Заменить последовательности из 4 пробелов на символ табуляции.

```bash
cat > spaces_to_tabs <<'EOF'
#!/bin/bash

sed $'s/    /\t/g' "$1" > "$2"
EOF

chmod +x spaces_to_tabs
```

Тест:

```bash
printf 'hello    world\none    two\n' > input.txt
./spaces_to_tabs input.txt output.txt
cat -T output.txt
```

Пример результата:

```text
hello^Iworld
one^Itwo
```

## Задание 10

Вывести названия всех пустых файлов в указанной директории.

```bash
cat > empty_files <<'EOF'
#!/bin/bash

find "$1" -maxdepth 1 -type f -empty -print
EOF

chmod +x empty_files
```

Тест:

```bash
mkdir -p testempty
touch testempty/empty1.txt
touch testempty/empty2.txt
echo "hello" > testempty/notempty.txt

./empty_files testempty
```

Пример результата:

```text
testempty/empty1.txt
testempty/empty2.txt
```

## Docker

Практическая выполнялась в Ubuntu-контейнере Docker с именем `management`.

Повторный запуск:

```bash
docker start -ai management
```

Рабочая директория внутри контейнера:

```text
/work
```
