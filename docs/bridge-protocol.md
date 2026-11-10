# Контракт JSON для Native CryptoPro Bridge

Каждый вызов Bridge — отдельный процесс. Dart отправляет один JSON-объект в `stdin`, закрывает поток и ожидает один JSON-объект в `stdout`.

Успех:

```json
{
  "ok": true,
  "action": "status",
  "data": {}
}
```

Ошибка:

```json
{
  "ok": false,
  "action": "status",
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": "Optional diagnostic details"
  }
}
```

`stdout` предназначен только для JSON. Диагностика должна идти в `stderr`.

## Команды

### status

```json
{"action":"status"}
```

Должен возвращать доступность CSP/CAdES SDK и версию Bridge. Не следует заявлять, что CAdES доступен, если проверена только загрузка CSP.

### list-certificates

```json
{"action":"list-certificates"}
```

Ожидаемая структура:

```json
{
  "ok": true,
  "action": "list-certificates",
  "data": {
    "certificates": [
      {
        "thumbprint": "HEX",
        "subjectName": "CN=Test",
        "issuerName": "CN=Test CA",
        "serialNumber": "SERIAL",
        "validFrom": "ISO-8601",
        "validTo": "ISO-8601",
        "hasPrivateKey": true,
        "dateRangeValid": true,
        "isReadyForSigning": true
      }
    ]
  }
}
```

Поля `isReadyForSigning` и `dateRangeValid` являются предварительной диагностикой, а не полноценной проверкой доверия/отзыва.

### sign-detached

```json
{
  "action": "sign-detached",
  "inputPath": "C:\\CryptoProTest\\input.txt",
  "outputPath": "C:\\CryptoProTest\\input.p7s",
  "thumbprint": "CERTIFICATE_THUMBPRINT"
}
```

### sign-attached

```json
{
  "action": "sign-attached",
  "inputPath": "C:\\CryptoProTest\\input.txt",
  "outputPath": "C:\\CryptoProTest\\input-attached.p7s",
  "thumbprint": "CERTIFICATE_THUMBPRINT"
}
```

### verify-detached

```json
{
  "action": "verify-detached",
  "inputPath": "C:\\CryptoProTest\\input.txt",
  "signaturePath": "C:\\CryptoProTest\\input.p7s"
}
```

Ожидаемый результат проверки:

```json
{
  "ok": true,
  "action": "verify-detached",
  "data": {
    "valid": true
  }
}
```

Важно: `valid=true` должно означать результат криптографической проверки подписи и содержимого. Проверка доверенной цепочки, OCSP/CRL, метки времени и соответствия внутренней политике организации должна быть отдельно обозначена в API.

## Безопасность

- Не передавать PIN-коды в JSON, аргументах командной строки или логах.
- Не экспортировать закрытые ключи.
- Проверять пути и ограничивать доступ к файловой системе.
- Не принимать произвольные команды оболочки.
- Не считать найденный сертификат доверенным только на основании наличия в хранилище.
