# howto_docker

## restart policy

```
docker inspect airflow-airflow-worker-1 | grep -C 2 RestartPolicy
            "NetworkMode": "airflow_default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
```
