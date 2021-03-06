# SklepMC Java Client
Implementacja publicznego API SklepMC w Javie.

## Przykładowe użycie
```java
// dane uwierzytelniające
String shopId = "TWOJE-ID-SKLEPU";
String secret = "TWOJ-KLUCZ-PRYWATNY";

// tworzymy context (w większości przypadków należy go gdzieś zapisać)
ShopContext shopContext = new ShopContext(shopId, secret);
```

## Pobieranie podstawowych informacji
```java
// informacje o sklepie
ShopInfo shop = ShopInfo.get(shopContext);
System.out.println(shop);

// informacje o serwerze
int serverId = 1234;
ServerInfo server = ServerInfo.get(shopContext, serverId);
System.out.println(server);

// informacje o usłudze
int serviceId = 4321;
ServiceInfo service = ServiceInfo.get(shopContext, serviceId);
System.out.println(service);

// informacje o transakcji
String transactionId = "SMC-ABCDFGHI";
TransactionInfo transaction = TransactionInfo.get(shopContext, transactionId);
System.out.println(transaction);
```

## Pobieranie i wykonywanie transakcji
```java
ExecutionInfo executionInfo = ExecutionInfo.get(shopContext, serverId);
List<ExecutionTaskInfo> executionTasks = executionInfo.getExecutionTasks();

task_execution:
for (ExecutionTaskInfo executionTask : executionTasks) {

    List<ExecutionCommandInfo> commands = executionTask.getCommands();
    String transactionId = executionTask.getTransactionId();
    boolean requireOnline = executionTask.isRequireOnline();

    // wykonujemy komendy transakcji
    for (ExecutionCommandInfo command : commands) {

        // komenda wymaga aby cel (gracz którego dotyczy) był online, 
        // sprawdzamy czy gracz jest na serwerze
        if (requireOnline) {
            // TODO: sprawdzanie online, zalezne od platformy
            // Player playerExact = Bukkit.getPlayerExact(command.getTarget());
            // if (playerExact == null) {
            //    continue task_execution;
            // }
        }

        String commandText = command.getText();
        // TODO: wykonywanie komendy, zalezne od platformy
        // Bukkit.dispatchCommand(Bukkit.getConsoleSender(), commandText);
    }

    // zmieniamy status transakcji na zakończony (COMPLETED)
    boolean updated;
    try {
        updated = TransactionInfo.updateStatus(shopContext, transactionId, TransactionInfo.TransactionStatus.COMPLETED.name());
    } catch (ApiException exception) {
        ApiError apiError = exception.getApiError();
        System.out.println("Błąd API: " + apiError.getType() + ", " + apiError.getMessage());
        continue;
    }

    // sprawdzamy czy wykonano pomyślnie zmianę
    if (!updated) {
        System.out.println("Nie udało się zmienić statusu transakcji: " + transactionId);
    }
}
```