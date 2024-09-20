# Отчет по интеграции логгера в Fastify

## Описание экосистемы

- **Fastify** — это быстрый фреймворк для Node.js, который используется для создания серверов. Он удобен и позволяет быстро обрабатывать запросы.
- **Winston** — это библиотека для логирования, которая позволяет записывать логи в разные места: консоль, файл и другие места.
- **UDP** — это протокол для отправки сообщений по сети. Мы используем его, чтобы передавать логи на локальный сервер.

## Выбор логгера

Я выбрал **Winston**, потому что:
- Можно записывать логи в файл и выводить в консоль.
- Поддерживает разные уровни логирования (debug, info, error).
- Его часто используют в сообществе, много примеров в интернете.

**UDP** был выбран для отправки логов, потому что он быстрый и не нагружает систему.

## Как это работает

1. Настроен сервер с помощью **Fastify**.
2. Логи отправляются в консоль и файл с помощью **Winston**.
3. Дополнительно настроен клиент для отправки логов через **UDP** на локальный адрес.

## Скриншоты
![image](https://github.com/user-attachments/assets/e3592dc1-0cb5-400e-89e8-081fd81f1ebe)


## Код

```javascript
const fastify = require('fastify')({ logger: true });
const winston = require('winston');
const dgram = require('dgram');

const udpClient = dgram.createSocket('udp4');

const logger = winston.createLogger({
  level: 'debug',
  format: winston.format.json(),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' })
  ]
});

function sendLogToUDP(message) {
  udpClient.send(message, 0, message.length, 5000, '127.0.0.1', (err) => {
    if (err) logger.error('UDP Error:', err);
  });
}


fastify.get('/', (request, reply) => {
    const clientIP = request.ip;
    const requestTime = new Date().toISOString();
    const logMessage = `Request "/" from IP: ${clientIP} at ${requestTime}`;
    logger.info(logMessage);
    sendLogToUDP(Buffer.from(logMessage));

    reply.send('There nothing here');
});


fastify.listen({ port: 3000 }, (err, address) => {
  logger.info(`Server ${address}`);
});

