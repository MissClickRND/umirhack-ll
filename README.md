# UMIRHACK ll

## Git и сабмодули

Этот репозиторий является superproject с сабмодулями:

- `frontend`
- `backend`
- `mobile`

Каждый сабмодуль настроен на отслеживание ветки `main` в файле `.gitmodules`.

### Клонирование с сабмодулями

```bash
git clone --recurse-submodules <repo-url>
```

Если репозиторий уже клонирован без сабмодулей:

```bash
git submodule update --init --recursive
```

### Обновление сабмодулей до последних коммитов `main`

Используйте этот вариант, когда хотите подтянуть в каждом сабмодуле самые свежие коммиты ветки `main` (а не только зафиксированные указатели):

```bash
git submodule sync --recursive
git submodule update --remote --recursive --merge
```

Опционально: проверить настроенные отслеживаемые ветки:

```bash
git config -f .gitmodules --get-regexp "^submodule\..*\.branch$"
```
