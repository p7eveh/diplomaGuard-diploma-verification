# Контейнерная архитектура DiplomaGuard (C4 - уровень 2)

## Диаграмма контейнеров

```mermaid
graph TB
    subgraph "Браузер пользователя"
        IssuePage[Страница выпуска<br>/issue]
        VerifyPage[Страница проверки<br>/verify]
        Web3Provider[Web3 Provider<br>ethers.js]
    end
    
    subgraph "Внешние системы"
        MetaMask[MetaMask кошелёк]
        Ethereum[Ethereum Sepolia]
        Storage[Хранилище метаданных<br>JSON / GitHub Gist]
    end
    
    subgraph "Блокчейн слой"
        Contract[Смарт-контракт DiplomaGuard<br>Solidity 0.8.x]
    end
    
    %% Связи
    IssuePage -->|подключается| Web3Provider
    Web3Provider -->|JSON-RPC| MetaMask
    MetaMask -->|подписанная транзакция| Contract
    Contract -->|деплой и вызовы| Ethereum
    
    VerifyPage -->|вызов view-функции| Web3Provider
    Web3Provider -->|JSON-RPC чтение| Contract
    Contract -->|возвращает данные| Web3Provider
    
    VerifyPage -->|HTTP GET| Storage
    
    %% Стилизация
    style IssuePage fill:#e1f5fe,stroke:#01579b
    style VerifyPage fill:#e1f5fe,stroke:#01579b
    style MetaMask fill:#fff3e0,stroke:#e65100
    style Contract fill:#e8f5e9,stroke:#1b5e20
    style Ethereum fill:#f3e5f5,stroke:#4a148c
```

## Компоненты и их обязанности

| Компонент | Технология | Обязанность |
|-----------|------------|-------------|
| **Frontend (страница выпуска)** | HTML/JS + ethers.js | Форма для админа, подключение к MetaMask, отправка транзакции |
| **Frontend (страница проверки)** | HTML/JS + ethers.js | Поле ввода ID, вызов view-функции контракта, отображение результата |
| **MetaMask** | Внешнее расширение | Подпись транзакций, управление приватными ключами |
| **Смарт-контракт** | Solidity 0.8.x | Хранение данных о дипломах, проверка прав эмитента, логика верификации |
| **Хранилище метаданных** | JSON (HTTP) | Хранение подробной информации о дипломе (ФИО, оценки и т.д.) |

## Связи между контейнерами

| От | К | Протокол | Данные |
|----|---|----------|--------|
| Страница выпуска | MetaMask | JSON-RPC | Транзакция выпуска |
| MetaMask | Смарт-контракт | JSON-RPC | Подписанная транзакция |
| Страница проверки | Смарт-контракт | JSON-RPC (view) | Запрос верификации |
| Страница проверки | Хранилище метаданных | HTTP GET | JSON с данными диплома |

## Почему такая архитектура?

| Решение | Причина |
|---------|---------|
| Только фронтенд + смарт-контракт | Нет необходимости в бэкенде (блокчейн сам сервер) |
| MetaMask для выпуска | Безопасное управление ключами, готовое решение |
| Отдельное хранилище для метаданных | Экономия газа (данные не в блокчейне) |
| Две отдельные страницы | Разделение ролей: админ (с кошельком) и HR (без кошелька) |