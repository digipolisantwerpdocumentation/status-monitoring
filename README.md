# Status monitoring

Dit document is een richtlijn naar inrichting van health monitoring voor verschillende componenten binnen een oplossing.

1) Kubernetes probes versus health checks
2) Metrics


## Health
Dit endpoint beschrijft de werking van de service.
Alle endpoints komen onder de resource `/status` terecht.

#### Status niveau's
Er zijn 3 verschillende niveaus van health status gedefinieerd:

status|betekenis
-------|-------
Ok|de service werkt
Degraded|de service functioneert niet meer op 100%, niet kritische systemen zijn niet bereikbaar of de applicatie verwerkt requests trager
Outage|de service werkt niet meer omwille van ernstige problemen

#### Basis response
Default zouden we volgende status aanbieden omdat consumers alleen willen weten of de service werkt. Dit endpoint wordt publiek mee ontsloten op de API Gateway.
```
/status/ping
```
Response:
```
{
  "status": "ok"
}
```
De afnemer van de service kan dan zelf de status pagina opvragen en naargelang de response acties ondernemen. (bv. een error pagina weergeven, bepaalde transacties bufferen, ...).

#### Status types

##### Status ok
Betekenis: de applicatie heeft status 'ok' indien de apllicatie en afhankelijkheden correct werken.

Probe /status/live geeft een HTTP200, de pod wordt niet herstart niet door Kubernetes.
Probe /status/ready geeft een HTTP200, de Kubernetes ingress controller laat requests op de toepassing toe.

##### Status degraded
Betekenis: de applicatie heeft status 'degraded' indien bepaalde niet-kritische systemen onbeschikbaar zijn (bijvoorbeeld logging, limiet van DB connecties is bereikt).

HTTP500


#####  Status outage
Betekenis: de applicatie heeft status 'outage' indien de de applicatie op dit moment zijn primaire functie niet uitvoeren.

HTTP500

#### Components
Betekenis: het components endpoint geeft een overzicht van alle afhankelijkheden en hun status.

```
/status/health/components
```
Voorbeeld response:
```
{
  "status": "outage",
  "components": [
    {
      "errormessage": "",
      "name": "database",
      "details": "database is working",
      "type": "db",
      "code":"",
      "status": "ok"
    },
    {
      "errormessage": "timeout",
      "name": "eventhandler",
      "details": "eventhandler timed out",
      "type": "event",
      "code":"TIMEOUT504",
      "status": "outage"
    }
  ]
}
```

## Metrics
Endpoint waar het continuous monitoring systeem metrics kan opvragen van de status.
```
/status/metrics
```
Default supported layout van continuous monitoring is in prometheus plain/text formaat. Beschrijving van de layout is [hier](#prometheus-metrics-format) terug te vinden.

Flask (Python web framework) metrics example:
```
process_virtual_memory_bytes 897339392.0
process_resident_memory_bytes 123064320.0
process_start_time_seconds 1530601590.11
process_cpu_seconds_total 2.8
process_open_fds 46.0
process_max_fds 1024.0
python_info{implementation="CPython",major="3",minor="6",patchlevel="5",version="3.6.5"} 1.0
flask_http_request_duration_seconds_bucket{le="0.005",method="GET",path="/favicon.ico",status="404"} 0.0
flask_http_request_duration_seconds_bucket{le="0.01",method="GET",path="/favicon.ico",status="404"} 0.0
flask_http_request_duration_seconds_bucket{le="0.025",method="GET",path="/favicon.ico",status="404"} 0.0
flask_http_request_duration_seconds_bucket{le="0.05",method="GET",path="/favicon.ico",status="404"} 0.0
flask_http_request_duration_seconds_bucket{le="0.075",method="GET",path="/favicon.ico",status="404"} 0.0
flask_http_request_duration_seconds_bucket{le="0.1",method="GET",path="/favicon.ico",status="404"} 0.0
flask_http_request_duration_seconds_bucket{le="0.25",method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_bucket{le="0.5",method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_bucket{le="0.75",method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_bucket{le="1.0",method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_bucket{le="2.5",method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_bucket{le="5.0",method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_bucket{le="7.5",method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_bucket{le="10.0",method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_bucket{le="+Inf",method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_count{method="GET",path="/favicon.ico",status="404"} 1.0
flask_http_request_duration_seconds_sum{method="GET",path="/favicon.ico",status="404"} 0.11943134200009808
flask_http_request_duration_seconds_bucket{le="0.005",method="POST",path="/uploads",status="202"} 0.0
flask_http_request_duration_seconds_bucket{le="0.01",method="POST",path="/uploads",status="202"} 0.0
flask_http_request_duration_seconds_bucket{le="0.025",method="POST",path="/uploads",status="202"} 0.0
flask_http_request_duration_seconds_bucket{le="0.05",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="0.075",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="0.1",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="0.25",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="0.5",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="0.75",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="1.0",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="2.5",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="5.0",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="7.5",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="10.0",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_bucket{le="+Inf",method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_count{method="POST",path="/uploads",status="202"} 1.0
flask_http_request_duration_seconds_sum{method="POST",path="/uploads",status="202"} 0.03629215999990265
```

## References
- <a name="prometheus-metrics-format"/>[Prometheus metrics format](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md)
- [Prometheus client libraries](https://prometheus.io/docs/instrumenting/clientlibs/)
- [Spring default endpoints](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html)
- [Microsoft health endpoint monitoring pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/health-endpoint-monitoring)

