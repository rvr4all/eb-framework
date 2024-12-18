# eb-framework
+----------------------+
|   ControlPlane       |
+----------------------+
| - rulesEngine: RulesEngine       |
+----------------------+
| + createEventBus(name: String, tenantId: String): String            |
| + listEventBuses(tenantId: String): Collection<EventBus>            |
| + addRule(busName: String, name: String, tenantId: String, rule: Rule): String |
| + listRules(busName: String, tenantId: String): Collection<Rule>    |
+----------------------+
                  |
                  v
+-----------------------------+
|         RulesEngine         |
+-----------------------------+
| - eventBuses: Map<String, EventBus>                                |
+-----------------------------+
| + getEventBus(name: String): EventBus                              |
| + addEventBus(name: String, tenantId: String): void                |
| + addRule(busName: String, ruleId: String, rule: Rule): void       |
| + getRules(busName: String): Collection<Rule>                      |
| + getAllEventBusesForTenant(tenantId: String): Collection<EventBus>|
+-----------------------------+
                  |
                  v
+-----------------------------+
|        EventBusRepository   |
+-----------------------------+
| + findByTenantId(tenantId: String): List<EventBus> |
| + findByName(name: String): Optional<EventBus>     |
+-----------------------------+

+-----------------------------+
|           EventBus          |
+-----------------------------+
| - id: Long                  |
| - name: String              |
| - tenantId: String          |
| - rules: List<Rule>         |
+-----------------------------+
| + belongsToTenant(tenantId: String): boolean |
+-----------------------------+

+-----------------------------+
|           Rule              |
+-----------------------------+
| - id: Long                  |
| - ruleId: String            |
| - eventBus: EventBus        |
| - conditionsJson: String    |
| - actions: List<Action>     |
| - schedule: String          |
+-----------------------------+
| + getConditions(): Map<String, Object> |
| + setConditions(Map<String, Object>): void |
+-----------------------------+

+-----------------------------+
|        RuleRepository       |
+-----------------------------+
| + findByEventBus(bus: EventBus): List<Rule> |
| + findByRuleId(ruleId: String): Optional<Rule> |
+-----------------------------+

+------------------------------------------------------------------------------+

+---------------------------+
|         DataPlane         |
+---------------------------+
| - camelContext: CamelContext             |
| - routeBuilderService: RouteBuilderService       |
| - eventListenerService: EventListenerService     |
| - scheduledEventDeliveryService: ScheduledEventDeliveryService |
+---------------------------+
| + initializeDataPlane(): void             |
| + executeScheduledRules(): void           |
| + sendEventToBus(event: Event): void      |
| + shutdownDataPlane(): void               |
+---------------------------+
        |                 |                  |
        v                 v                  v
+-------------------+   +----------------------+   +----------------------------+
| RouteBuilderService|   | EventListenerService |   | ScheduledEventDeliveryService |
+-------------------+   +----------------------+   +----------------------------+
| + configureRoutes(context: CamelContext): void |
| + createBusRouteListener(busName: String): void|
+-------------------+
| + startEventBusListener(): void |
| + stopEventBusListener(): void |
| + sendEventToEventBus(event: Event): void |
+----------------------+   
| + executeScheduledRules(): void |
+----------------------------+
        |
        v
+-------------------+
| EventDeliveryService |
+-------------------+
| - targetHandlers: TargetHandlers          |
| - eventStorageService: EventStorageService|
| - deadLetterService: DeadLetterService    |
+-------------------+
| + processAndDeliverEvent(event: Event, exchange: Exchange): boolean |
| + publishEventToRoute(event: Event): void                           |
+-------------------+
        |                             |
        v                             v
+-------------------+        +-----------------------+
|   Event           |        |  ProcessedEvent      |
+-------------------+        +-----------------------+
| - id: Long        |        | - eventId: String    |
| - busName: String |        | - delivered: Boolean |
| - eventType: String|       +-----------------------+
| - source: String  |        | + isDelivered(): Boolean |
| - processed: Boolean       +-----------------------+
+-------------------+
| + toEventMap(): Map<String, Object> |
+-------------------+
        |
        v
+----------------------------+
|   EventRepository          |
+----------------------------+
| + findByProcessed(processed: Boolean): List<Event>    |
| + findById(id: Long): Optional<Event>                 |
| + save(event: Event): Event                           |
+----------------------------+

+----------------------------+
|   DeadLetterEvent          |
+----------------------------+
| - id: Long                 |
| - payload: String          |
| - errorReason: String      |
| - timestamp: LocalDateTime |
+----------------------------+
| + setPayload(String): void |
| + setErrorReason(String): void |
+----------------------------+
        |
        v
+----------------------------+
|   DeadLetterRepository     |
+----------------------------+
| + save(deadLetterEvent: DeadLetterEvent): void |
+----------------------------+
