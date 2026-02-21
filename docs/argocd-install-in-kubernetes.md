<img src="../img/argo-icon-color.png" align="left" width="100" height="120" />

	Argo CD это инструмент для непрырывной доставки кода в кластеры kubernetes.
	Организуем непрерывную доставку в Kubernetes.
	У вас должен быть запущен кластер K8s,	
	потому что мы собираемся делать это внутри k8s кластера.
	Обратите внимание, что в развертывании по умолчанию сервис 
	работает как тип Cluster IP, и ingress по 	умолчанию нет. 
	Поэтому, если вы хотите получить доступ к сервису, вам нужно будет
	либо выполнить 	переадресацию портов, либо изменить
	тип сервиса на балансировщик нагрузки, либо создать ingress.	
										
У Вас должна быть установлен ***kubectl**

Выполним команды установки:														

создайте новое пространство имен
```Bash
kubectl create namespace argocd
```
Установка манифестом argocd

```Bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml 
```




Создаём сервис load-balancer и прокидываем порт наружу

```Bash
nano argocd-load-balancer.yaml
```

argocd-load-balancer.yaml

```yaml

apiVersion: v1
kind: Service
metadata:
  name: argocd-load-balancer
  namespace: argocd
spec:
  type: LoadBalancer
  ports:
  - port: 443
    name: load-balancer-port-ssl
    targetPort: 8080
  selector:
   app.kubernetes.io/name: argocd-server
```

```Bash
kubectl create -f argocd-load-balancer.yaml
``` 

Какой пароль у пользователя admin

```Bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"   | openssl base64 -A -d; echo
```


 Смотрим на каком порту сервис работает 

watch -n 2 kubectl get service  -A





