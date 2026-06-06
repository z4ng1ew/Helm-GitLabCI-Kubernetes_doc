## Part 1. Настройка CI и CD

## 1. 1. Найти open-source проект на GitHub или GitLab, состоящий минимум из 3 микросервисов, сервиса с базой данных и фронтенда. 



![alt text](img/image.png)

https://github.com/GoogleCloudPlatform/bank-of-anthos   - ссылка на проект


Bank of Anthos — лучший репозиторий под задание: настоящие базы, нет готового Helm и GitLab-пайплайна (значит всё делаешь сам). Демо-банковское приложение с независимыми сервисами на Python и Java, двумя базами PostgreSQL и веб-интерфейсом.
















## 2. Если необходимо, написать докерфайлы для выбранного проекта.

![alt text](<img/image copy 4.png>)

**1. Все 8 Dockerfile**






![alt text](<img/image copy.png>)



![alt text](<img/image copy 2.png>)


Скрин: успешная сборка образа (финальная строка вида naming to docker.io/library/balancereader:test ... done).






















## 3. Если необходимо, написать модульные тесты для микросервисов с бизнес-логикой приложения.




![alt text](<img/image copy 5.png>)

**Python и Java тесты**


![alt text](<img/image copy 3.png>)

**пайплайн с тестами проходит**

![alt text](<img/image copy 6.png>)

**Количество тестовых файлов**

![alt text](<img/image copy 7.png>)

**запуск тестов локально**






## 4. Разработать Helm-чарт для развертывания выбранного приложения.


![alt text](<img/image copy 8.png>)




![alt text](<img/image copy 9.png>)

структура чарта

![alt text](<img/image copy 10.png>)

метаданные чарта (Chart.yaml)




![alt text](<img/image copy 11.png>)

чарт синтаксически валиден







![alt text](<img/image copy 12.png>)

чарт рендерится (генерирует валидные манифесты)










![alt text](<img/image copy 13.png>)

Три релиза:

- boa в prod — наш чарт, revision 1 (один деплой, через manual-кнопку пайплайна) 
- boa в staging — наш чарт, revision 9 (девять деплоев — все эксперименты и финальный) 
Чарт bank-of-anthos-0.1.0 — это наш собственный чарт из src/helm/bank-of-anthos/.



![alt text](<img/image copy 14.png>)

Подробности о релизах - статус и  время деплоя










 **История изменений (revisions)**


![alt text](<img/image copy 15.png>)




![alt text](<img/image copy 16.png>)

все ревизии — для staging  много (много раз запускали upgrade), для prod — одна.












значения, с которыми задеплоен релиз

![alt text](<img/image copy 17.png>)

![alt text](<img/image copy 18.png>)




## 5. Разработать пайплайн для развертывания приложения, состоящий из четырех стадий:


![alt text](<img/image copy 23.png>)


   - build,

   ![alt text](<img/image copy 19.png>)




   - test,

   ![alt text](<img/image copy 20.png>)






   - staging,

![alt text](<img/image copy 21.png>)


   - prod.

![alt text](<img/image copy 22.png>)






![alt text](<img/image copy 24.png>)



## 6. Настроить CI/CD для развертывания приложения с помощью GitLab Runner в Kubernetes.


![alt text](<img/image copy 25.png>)

Файл пайплайна в репозитории




![alt text](<img/image copy 26.png>)

под gitlab-runner-5587cf9c89-mwjkq со статусом Running 1/1, на какой ноде запущен (IP-адрес).



![alt text](<img/image copy 27.png>)

Результат деплоя в staging

все 8 подов 1/1 Running, со свежим возрастом (например, 5m — то есть только что задеплоены пайплайном, а не руками).

![alt text](<img/image copy 28.png>)

8 подов Running. Доказывает, что manual-кнопка в пайплайне реально привела к развёртыванию в prod-namespace.











## Бонус





## 1  Нормальные сертификаты а не самоподписанные




```
s@Vivo:/tmp/cert-manager-webhook-duckdns$ export KUBECONFIG=~/.kube/do14-config

# 1) найти под webhook (без фильтра по метке)
kubectl get pods -n cert-manager | grep duckdns

# 2) подождать и перепроверить издателей
sleep 40
kubectl get clusterissuer
cert-manager-webhook-duckdns-5d574d5f68-mkrmk   1/1     Running   0             2m55s
NAME                                      READY   AGE
cert-manager-webhook-duckdns-production   False   3m35s
cert-manager-webhook-duckdns-staging      False   3m35s
my-ca-issuer                              True    10d
selfsigned-bootstrap                      True    10d
s@Vivo:/tmp/cert-manager-webhook-duckdns$ kubectl describe clusterissuer cert-manager-webhook-duckdns-staging | tail -25
  Acme:
    Email:            ВПИШИ_СВОЮ_ПОЧТУ@example.com
    Preferred Chain:  ISRG Root X1
    Private Key Secret Ref:
      Name:  cert-manager-webhook-duckdns-staging
    Server:  https://acme-staging-v02.api.letsencrypt.org/directory
    Solvers:
      dns01:
        Webhook:
          Config:
            API Token Secret Ref:
              Key:      token
              Name:     cert-manager-webhook-duckdns
          Group Name:   acme.duckdns.org
          Solver Name:  duckdns
Status:
  Acme:
  Conditions:
    Last Transition Time:  2026-06-01T20:06:34Z
    Message:               Failed to register ACME account: 400 urn:ietf:params:acme:error:invalidContact: Error validating contact(s) :: contact email contains non-ASCII characters
    Observed Generation:   1
    Reason:                ErrRegisterACMEAccount
    Status:                False
    Type:                  Ready
Events:                    <none>
s@Vivo:/tmp/cert-manager-webhook-duckdns$   export KUBECONFIG=~/.kube/do14-config
EMAIL="z4ng1ew@gmail.com"      # ← реальная почта, только латиница

for iss in cert-manager-webhook-duckdns-staging cert-manager-webhook-duckdns-production; do
  kubectl patch clusterissuer "$iss" --type=merge \
    -p "{\"spec\":{\"acme\":{\"email\":\"$EMAIL\"}}}"
done
clusterissuer.cert-manager.io/cert-manager-webhook-duckdns-staging patched
clusterissuer.cert-manager.io/cert-manager-webhook-duckdns-production patched
s@Vivo:/tmp/cert-manager-webhook-duckdns$ sleep 30
kubectl get clusterissuer
NAME                                      READY   AGE
cert-manager-webhook-duckdns-production   True    7m26s
cert-manager-webhook-duckdns-staging      True    7m26s
my-ca-issuer                              True    10d
selfsigned-bootstrap                      True    10d
s@Vivo:/tmp/cert-manager-webhook-duckdns$ kubectl describe clusterissuer cert-manager-webhook-duckdns-staging | grep -A2 "Message"
    Message:               The ACME account was registered with the ACME server
    Observed Generation:   2
    Reason:                ACMEAccountRegistered
s@Vivo:/tmp/cert-manager-webhook-duckdns$ 
```






![alt text](<img/image copy 29.png>)










![alt text](<img/image copy 30.png>)


```
s@Vivo:/tmp/cert-manager-webhook-duckdns$ export KUBECONFIG=~/.kube/do14-config

# каким релизом helm он поставлен (нужно для правки)
helm list -n cert-manager | grep cert-manager

# текущие аргументы запуска controller
kubectl get deploy cert-manager -n cert-manager \
  -o jsonpath='{.spec.template.spec.containers[0].args}'; echo
cert-manager-webhook-duckdns    cert-manager    1               2026-06-01 23:06:31.997826101 +0300 MSK deployed        cert-manager-webhook-duckdns-v1.2.3       v1.2.3     
["--v=2","--cluster-resource-namespace=$(POD_NAMESPACE)","--leader-election-namespace=kube-system","--acme-http01-solver-image=quay.io/jetstack/cert-manager-acmesolver:v1.16.2","--max-concurrent-challenges=60"]
s@Vivo:/tmp/cert-manager-webhook-duckdns$ export KUBECONFIG=~/.kube/do14-config

kubectl patch deployment cert-manager -n cert-manager --type=json -p='[
  {"op":"add","path":"/spec/template/spec/containers/0/args/-",
   "value":"--dns01-recursive-nameservers=8.8.8.8:53,1.1.1.1:53"},
  {"op":"add","path":"/spec/template/spec/containers/0/args/-",
   "value":"--dns01-recursive-nameservers-only"}
]'

kubectl rollout status deployment/cert-manager -n cert-manager
deployment.apps/cert-manager patched
Waiting for deployment "cert-manager" rollout to finish: 0 out of 1 new replicas have been updated...
Waiting for deployment "cert-manager" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "cert-manager" rollout to finish: 1 old replicas are pending termination...
deployment "cert-manager" successfully rolled out
s@Vivo:/tmp/cert-manager-webhook-duckdns$ kubectl get deploy cert-manager -n cert-manager \
  -o jsonpath='{.spec.template.spec.containers[0].args}'; echo
["--v=2","--cluster-resource-namespace=$(POD_NAMESPACE)","--leader-election-namespace=kube-system","--acme-http01-solver-image=quay.io/jetstack/cert-manager-acmesolver:v1.16.2","--max-concurrent-challenges=60","--dns01-recursive-nameservers=8.8.8.8:53,1.1.1.1:53","--dns01-recursive-nameservers-only"]
s@Vivo:/tmp/cert-manager-webhook-duckdns$ 
```




Маленький риск-нюанс: раз cert-manager ставился манифестом, при будущем его обновлении (если будешь накатывать новую версию) этот патч сбросится — нужно будет добавить аргументы снова. Для отчёта/сдачи это неважно, просто держиМ в голове. (В «нормальной» инфраструктуре, это зашьётся в values cert-manager раз и навсегда.)


















![alt text](<img/image copy 31.png>)

```
s@Vivo:/tmp/cert-manager-webhook-duckdns$ kubectl get certificate -n staging bank-tls
NAME       READY   SECRET     AGE
bank-tls   False   bank-tls   61s
s@Vivo:/tmp/cert-manager-webhook-duckdns$ kubectl get challenge -n staging
NAME                               STATE     DOMAIN                      AGE
bank-tls-1-1118570511-3213901970   pending   spectrew-bank.duckdns.org   72s
s@Vivo:/tmp/cert-manager-webhook-duckdns$ 
```








![alt text](<img/image copy 33.png>)


```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ export KUBECONFIG=~/.kube/do14-config
cd ~/до_14/DO14/src/helm/bank-of-anthos

helm upgrade boa . -n staging --reuse-values \
  --set ingress.tls.enabled=true \
  --set ingress.tls.issuer=my-ca-issuer \
  --set ingress.tls.secretName=bank-tls \
  --timeout 3m

helm upgrade boa . -n prod --reuse-values \
  --set ingress.tls.enabled=true \
  --set ingress.tls.issuer=my-ca-issuer \
  --set ingress.tls.secretName=bank-tls \
  --timeout 3m
Release "boa" has been upgraded. Happy Helming!
NAME: boa
LAST DEPLOYED: Tue Jun  2 08:52:32 2026
NAMESPACE: staging
STATUS: deployed
REVISION: 11
TEST SUITE: None
NOTES:
Bank of Anthos развёрнут (релиз: boa, namespace: staging).

Проверь поды:
  kubectl get pods -n staging
Открой банк в браузере:
  http://bank-staging.10.174.8.95.nip.io

Демо-вход:  testuser / bankofanthos
Release "boa" has been upgraded. Happy Helming!
NAME: boa
LAST DEPLOYED: Tue Jun  2 08:52:32 2026
NAMESPACE: prod
STATUS: deployed
REVISION: 3
TEST SUITE: None
NOTES:
Bank of Anthos развёрнут (релиз: boa, namespace: prod).

Проверь поды:
  kubectl get pods -n prod
Открой банк в браузере:
  http://bank.10.174.8.95.nip.io

Демо-вход:  testuser / bankofanthos
```





![alt text](<img/image copy 32.png>)



```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ sleep 15
# аннотация теперь с именем издателя?
kubectl get ingress frontend -n staging -o jsonpath='{.metadata.annotations.cert-manager\.io/cluster-issuer}'; echo
# сертификаты появились?
kubectl get certificate -n staging
kubectl get certificate -n prod
my-ca-issuer
NAME       READY   SECRET     AGE
bank-tls   True    bank-tls   28s
NAME       READY   SECRET     AGE
bank-tls   True    bank-tls   27s
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ export KUBECONFIG=~/.kube/do14-config

kubectl get secret my-root-ca-secret -n cert-manager \
  -o jsonpath='{.data.tls\.crt}' | base64 -d > ~/my-root-ca.crt

# проверим, что это валидный сертификат CA
openssl x509 -in ~/my-root-ca.crt -noout -subject -issuer -dates
subject=CN = my-root-ca
issuer=CN = my-root-ca
notBefore=May 22 08:00:17 2026 GMT
notAfter=May 19 08:00:17 2036 GMT
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ 
```

























![alt text](<img/image copy 34.png>) 


```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ # системные CA хранятся в /usr/local/share/ca-certificates (только с расширением .crt)
sudo cp ~/my-root-ca.crt /usr/local/share/ca-certificates/my-root-ca.crt
sudo update-ca-certificates
[sudo] пароль для s: 
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
Обрабатываются триггеры для ca-certificates-java (20240118) …
Adding debian:my-root-ca.pem
done.
done.
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ # утилита для базы NSS (если нет — поставим)
sudo apt-get install -y libnss3-tools

# импорт в пользовательскую базу Chrome
certutil -d sql:$HOME/.pki/nssdb -A -t "C,," -n "my-root-ca" -i ~/my-root-ca.crt

# проверка, что добавился
certutil -d sql:$HOME/.pki/nssdb -L | grep my-root-ca
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово         
Следующие НОВЫЕ пакеты будут установлены:
  libnss3-tools
Обновлено 0 пакетов, установлено 1 новых пакетов, для удаления отмечено 0 пакетов, и 27 пакетов не обновлено.
Необходимо скачать 615 kB архивов.
После данной операции объём занятого дискового пространства возрастёт на 2 193 kB.
Пол:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 libnss3-tools amd64 2:3.98-1ubuntu0.1 [615 kB]
Получено 615 kB за 2с (389 kB/s)        
Выбор ранее не выбранного пакета libnss3-tools.
(Чтение базы данных … на данный момент установлено 288763 файла и каталога.)
Подготовка к распаковке …/libnss3-tools_2%3a3.98-1ubuntu0.1_amd64.deb …
Распаковывается libnss3-tools (2:3.98-1ubuntu0.1) …
Настраивается пакет libnss3-tools (2:3.98-1ubuntu0.1) …
Обрабатываются триггеры для man-db (2.12.0-4build2) …
my-root-ca                                                   C,,  
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ 
```









![alt text](<img/image copy 36.png>)

![alt text](<img/image copy 35.png>)

импорт в систему Ubuntu

Chrome на Linux доверяет системному хранилищу. Кладём туда CA:

update-ca-certificates пересоберёт системный список доверенных центров — в выводе будет строка вроде 1 added. После этого полностью закрыв и снова открыв Chrome (важно — он читает хранилище при старте). Увидим что браузер доверяет сертефикату.


закрытый замочек без предупреждения «Не защищено». Нажав на замочек → «Connection is secure» → «Certificate is valid» — видно, что сертификат выписан на bank-staging.10.174.8.95.nip.io и подписан my-root-ca. Это и есть скрин для отчёта.



Это полноценный, грамотный TLS — просто называть его надо «внутренний CA», а не «настоящий/публичный». Публично доверенный Let's Encrypt честно отметь как «недостижим на этой площадке, вынесен на нормальную инфраструктуру с доменом» — это не провал, а корректная инженерная оценка ограничений.





















![alt text](<img/image copy 37.png>)

```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ echo | openssl s_client -connect bank-staging.10.174.8.95.nip.io:443 \
  -servername bank-staging.10.174.8.95.nip.io 2>/dev/null \
  | openssl x509 -noout -ext subjectAltName
X509v3 Subject Alternative Name: critical
    DNS:bank-staging.10.174.8.95.nip.io
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ 
```



 TLS  технически безупречен — настоящий листовой сертификат, подписанный CA, с правильным SAN, выпущенный автоматически через ingress-shim. Единственное, чего нет — публичного доверия (импортировал CA только себе).



























 ![alt text](<img/image copy 38.png>)






 ```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ export KUBECONFIG=~/.kube/do14-config

# сносим застрявший заход целиком — заявку, order, challenge, временные секреты
kubectl delete certificate -n default wildcard-spectrew-staging
kubectl delete order -n default --all
kubectl delete challenge -n default --all
kubectl delete secret -n default wildcard-spectrew-staging-tls --ignore-not-found

# создаём заново
kubectl apply -f - <<'EOF'
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: wildcard-spectrew-staging
  namespace: default
spec:
  secretName: wildcard-spectrew-staging-tls
  issuerRef:
    name: letsencrypt-staging-cf
    kind: ClusterIssuer
  dnsNames:
    - "spectrew.online"
EOF - "*.spectrew.online"
certificate.cert-manager.io "wildcard-spectrew-staging" deleted from default namespace
No resources found
challenge.acme.cert-manager.io "wildcard-spectrew-staging-1-2753821521-1200705549" deleted from default namespace
certificate.cert-manager.io/wildcard-spectrew-staging created
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ sleep 90
echo "=== challenge ==="; kubectl get challenge -n default
echo "=== order ==="; kubectl get order -n default
echo "=== сертификат ==="; kubectl get certificate -n default wildcard-spectrew-staging
echo "=== ошибки уборки? ==="
kubectl logs -n cert-manager deploy/cert-manager --since=90s | grep -iE "cleaning up|valid|issued|error" | tail -6
=== challenge ===
No resources found in default namespace.
=== order ===
NAME                                     STATE   AGE
wildcard-spectrew-staging-1-2753821521   valid   96s
=== сертификат ===
NAME                        READY   SECRET                          AGE
wildcard-spectrew-staging   True    wildcard-spectrew-staging-tls   96s
=== ошибки уборки? ===
E0602 08:37:58.659436       1 sync.go:132] "error cleaning up challenge" err=<
                 Error: 7003: Could not route to /client/v4/zones/dns_records/f18cf02fad377511cdcdc4bc31efaac9, perhaps your object identifier is invalid?
E0602 08:37:58.659783       1 controller.go:157] "re-queuing item due to error processing" err=<
                 Error: 7003: Could not route to /client/v4/zones/dns_records/f18cf02fad377511cdcdc4bc31efaac9, perhaps your object identifier is invalid?
E0602 08:38:00.736416       1 sync.go:298] "error cleaning up challenge" err=<
                 Error: 7003: Could not route to /client/v4/zones/dns_records/f18cf02fad377511cdcdc4bc31efaac9, perhaps your object identifier is invalid?
 ```



 Тренировочный wildcard-сертификат *.spectrew.online от Let's Encrypt выпущен. Весь путь пройден: API Cloudflare через HTTP/1.1 (обошли DPI), самопроверка через публичные резолверы, challenge valid, order valid, сертификат готов. Чистый перезаезд разорвал тот цикл застревания.




  только что получил настоящий wildcard-сертификат Let's Encrypt через DNS-01 + Cloudflare, обойдя государственную блокировку Cloudflare переключением на HTTP/1.1. Было сложно, потому что у  был дополнительный барьер (DPI), но смог выпустить чистый сертефикат, через Cloudflare DNS-01, с обходом блокировки.








  ![alt text](<img/image copy 39.png>)


```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ export KUBECONFIG=~/.kube/do14-config

kubectl apply -f - <<'EOF'
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod-cf
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: z4ng1ew@gmail.com
    privateKeySecretRef:
      name: letsencrypt-prod-cf-account
    solvers:
      - dns01:
          cloudflare:
            apiTokenSecretRef:
              name: cloudflare-api-token
              key: api-token
EOF

sleep 15
kubectl get clusterissuer letsencrypt-prod-cf
clusterissuer.cert-manager.io/letsencrypt-prod-cf created
NAME                  READY   AGE
letsencrypt-prod-cf   True    15s
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ 
```


Боевой ClusterIssuer создан и работает (READY True). Это значит, что cert-manager успешно зарегистрировался на боевом сервере Let's Encrypt по адресу https://acme-v02.api.letsencrypt.org/directory и готов выпускать настоящие сертификаты.




Все пункты выполненны:
1) «нормальные сертификаты а не самоподписанные» — Let's Encrypt = нормальный публичный CA. Получилось.
2) «через ингресс контроллер nginx» —  использую nginx ingress. Подходит.
3)  «можно ли перенести записи из nip в Cloudflare? тогда не придётся вебхуки дёргать» — чтобы DNS-зона лежала в Cloudflare и cert-manager сам делал в ней записи через API. Сделал ровно это — завёл свой домен spectrew.online в Cloudflare, и cert-manager через Cloudflare API создаёт TXT-записи для DNS-01 challenge. Это и есть «не дёргать вебхуки».
4) «просто запускаешь нджинкс с сертменеджером и указываешь свой токен с Cloudflare» — ровно это: ingress-nginx + cert-manager + Cloudflare API токен в Secret. Точная схема.
5)  «но лучше API Gateway» — это альтернатива nginx ingress (Kubernetes Gateway API — новый стандарт). Сделал через классический ingress, не через Gateway. Это не блокер —  «можно через ингресс, но лучше Gateway». Пошёл по первому пути, и это валидно.





Выполнил главное требование — настоящий сертификат от настоящего CA через Cloudflare DNS-01. 
Способ через nginx-ingress — это легитимный, продакшен-уровневый способ, его используют тысячи компаний.



Технически: Let's Encrypt проверил, что домен spectrew.online действительно наш (доказал это через TXT-запись в DNS, которую может создать только владелец домена), и выписал сертификат, подписанный их корневым ключом. Корневой ключ Let's Encrypt встроен во все браузеры и операционные системы мира. Зелёный замочек везде.


Технически:

1) cert-manager запросил у Let's Encrypt: «дайте мне сертификат на *.spectrew.online».
2) Let's Encrypt ответил: «создайте TXT-запись _acme-challenge.spectrew.online со значением XYZ123».
3) cert-manager через API Cloudflare (мой токен) создал эту запись в DNS-зоне spectrew.online.
4) Let's Encrypt подождал 30 секунд и проверил DNS публично — запись на месте.
«Окей, домен твой, держи сертификат».
5) Только владелец домена может создавать TXT-записи в его зоне. Поэтому это доказательство владения.




Cloudflare заблокирован Роскомнадзором (DPI режет HTTPS-соединения по характеристикам TLS). cert-manager пытался достучаться до API Cloudflare обычным способом — обрывалось. Переключились на HTTP/1.1 (более старая версия протокола, которую DPI не режет так агрессивно) — соединение проходит, API отвечает. Это инженерное решение, не баг.


















![alt text](<img/image copy 40.png>)

```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ export KUBECONFIG=~/.kube/do14-config
echo "=== order ==="; kubectl get order -n default | grep -v staging
echo "=== сертификат ==="; kubectl get certificate -n default wildcard-spectrew
echo "=== секрет (готовый сертификат) ==="
kubectl get secret -n default wildcard-spectrew-tls 2>/dev/null && echo "✓ секрет есть"
=== order ===
NAME                                     STATE   AGE
wildcard-spectrew-1-1557615201           valid   7m55s
=== сертификат ===
NAME                READY   SECRET                  AGE
wildcard-spectrew   True    wildcard-spectrew-tls   7m55s
=== секрет (готовый сертификат) ===
NAME                    TYPE                DATA   AGE
wildcard-spectrew-tls   kubernetes.io/tls   2      6m14s
✓ секрет есть
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ kubectl describe order -n default $(kubectl get order -n default -o name | grep -v staging | cut -d/ -f2) 2>/dev/null | grep -iE "reason|state|message" | tail -6
    Initial State:  valid
    Initial State:  pending
  State:            valid
  Type    Reason    Age    From                 Message
```


настоящий сертификат от Let's Encrypt на *.spectrew.online, которому браузеры доверяют из коробки, без всякого импорта CA. Просто order финализировался с задержкой (из-за той ошибки уборки), но дошёл до valid сам. Терпение окупилось — и лимит LE не сожгли.




![alt text](<img/image copy 41.png>)


- Issuer здесь Let's Encrypt (issuer=C = US, O = Let's Encrypt, CN = YR2), а не (STAGING). Это печать настоящего доверенного центра.



- Issuer = O = Let's Encrypt, CN = YR2 — это боевой, публично доверенный Let's Encrypt (не staging!). Годен до 31 августа 2026.








![alt text](<img/image copy 42.png>)

```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ export KUBECONFIG=~/.kube/do14-config
for ns in staging prod; do
  kubectl get secret wildcard-spectrew-tls -n default -o yaml \
    | sed "s/namespace: default/namespace: $ns/" \
    | kubectl apply -n $ns -f -
done
secret/wildcard-spectrew-tls created
secret/wildcard-spectrew-tls created
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ 
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ 
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ nslookup bank.spectrew.online 1.1.1.1
Server:         1.1.1.1
Address:        1.1.1.1#53

Non-authoritative answer:
Name:   bank.spectrew.online
Address: 10.174.8.95

s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ dig +short bank.spectrew.online @1.1.1.1
10.174.8.95
```


- Секрет скопирован в staging и prod.
- A-запись bank.spectrew.online → 10.174.8.95 уже резолвится (и nslookup, и dig показывают). Серое облако сработало — отдаёт мой приватный IP напрямую.





```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ export KUBECONFIG=~/.kube/do14-config
cd ~/до_14/DO14/src/helm/bank-of-anthos

# staging → bank-staging.spectrew.online, prod → bank.spectrew.online
helm upgrade boa . -n prod --reuse-values \
  --set ingress.host=bank.spectrew.online \
  --set ingress.tls.enabled=true \
  --set ingress.tls.issuer=my-ca-issuer \
  --set ingress.tls.secretName=wildcard-spectrew-tls \
  --timeout 3m
Release "boa" has been upgraded. Happy Helming!
NAME: boa
LAST DEPLOYED: Tue Jun  2 12:57:26 2026
NAMESPACE: prod
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
Bank of Anthos развёрнут (релиз: boa, namespace: prod).

Проверь поды:
  kubectl get pods -n prod
Открой банк в браузере:
  http://bank.spectrew.online

Демо-вход:  testuser / bankofanthos
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ 
```



![alt text](<img/image copy 43.png>)

 ingress на bank.spectrew.online + Let's Encrypt-сертификат

 Деплоим через Helm с новым хостом и готовым секретом. Поскольку сертификат уже выпущен и лежит секретом, нам не нужен ingress-shim/issuer — просто укажем ingress использовать готовый секрет wildcard-spectrew-tls. Передаём параметры явно (не через --reuse-values, чтобы не словить пустые значения, как было раньше).

 Тут хитрость: issuer оставляем my-ca-issuer (он безвреден — ingress-shim увидит, что секрет wildcard-spectrew-tls уже существует и не будет ничего перевыпускать, просто использует готовый). Главное — secretName=wildcard-spectrew-tls (наш боевой Let's Encrypt) и host=bank.spectrew.online.

Почему меняем хост на bank.spectrew.online: сертификат wildcard *.spectrew.online покрывает bank.spectrew.online, но не покрывает старый bank.10.174.8.95.nip.io. Поэтому хост ingress должен стать поддоменом spectrew.online, чтобы имя совпало с сертификатом.



















![alt text](<img/image copy 44.png>)

выпустил настоящий wildcard через Let's Encrypt + Cloudflare DNS-01, причём пришлось ещё и блокировку Cloudflare обойти

bank.spectrew.online теперь отдаёт настоящий, публично доверенный сертификат Let's Encrypt. Не внутренний CA, не самоподписанный — тот самый «нормальный» сертификат, о котором говорили. Годен до 31 августа 2026.

перечислю честно:

1) Купил домен spectrew.online, делегировал DNS в Cloudflare.
2) cert-manager + Cloudflare DNS-01 — настроил автоматический выпуск через API-токен с минимальными правами (только DNS одной зоны — грамотно по безопасности).
3) Выпустил wildcard *.spectrew.online от Let's Encrypt — именно то, что требовалось (wildcard возможен только через DNS-01, ты это и сделал).
4) Обошёл государственную блокировку Cloudflare — диагностировал, что РКН режет HTTP/2 к api.cloudflare.com (curl по HTTP/1.1 проходил, cert-manager по HTTP/2 — нет), и обошёл флагом GODEBUG=http2client=0. обход DPI.
5) Привязал к банку через ingress-nginx, публично доверенный TLS.






![alt text](<img/image copy 45.png>)



![alt text](<img/image copy 47.png>)

![alt text](<img/image copy 48.png>)


![alt text](<img/image copy 46.png>)

железобетонное доказательство, в одну фразу: «Подтверждено: Let's Encrypt» в браузере, без импорта какого-либо CA. Это и есть определение «настоящего» (публично доверенного) сертификата — браузер доверяет ему сам, из коробки.








![alt text](<img/image copy 49.png>)

Команда (issuer от Let's Encrypt, не самоподписанный):

echo | openssl s_client -connect bank.spectrew.online:443 -servername bank.spectrew.online 2>/dev/null | openssl x509 -noout -issuer
   → issuer = O = Let's Encrypt, CN = YR2







![alt text](<img/image copy 50.png>)



редиректит на https











## 2  Билд-кит ( BuildKit)  - вместо канико.

![alt text](<img/image copy 51.png>)








## 3. Реплики по разными нодам(антиафинити)

![alt text](<img/image copy 52.png>)




![alt text](<img/image copy 53.png>)




![alt text](<img/image copy 54.png>)


```
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ # поднимаем 2 реплики frontend в prod
kubectl scale deploy/frontend -n prod --replicas=2
sleep 20

# КЛЮЧЕВОЙ СКРИН: смотрим, на каких нодах оказались реплики
kubectl get pods -n prod -l app=frontend -o wide
deployment.apps/frontend scaled
NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
frontend-86d6dc594d-rlf5q   1/1     Running   0          20s   10.42.2.133   k3s-worker2   <none>           <none>
frontend-86d6dc594d-sxkqm   1/1     Running   0          14m   10.42.1.159   k3s-worker1   <none>           <none>
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ kubectl scale deploy/frontend -n prod --replicas=1
deployment.apps/frontend scaled
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ 
```



В выводе будет колонка NODE — две реплики frontend  стоят на разных воркерах (k3s-worker1 и k3s-worker2). Это и есть доказательство антиаффинити/





- Лимиты: «Каждому сервису заданы resources.requests (гарантированная память) и resources.limits (потолок, защита от OOM и монополизации памяти ноды)».
- Антиаффинити: «Для stateless-сервисов настроено podAntiAffinity (preferred, weight 100, topologyKey hostname): при наличии нескольких реплik они разносятся по разным нодам для отказоустойчивости. Продемонстрировано на frontend (2 реплики → worker1+worker2)».
- Разделение окружений: «staging — по 1 реплике (экономия ресурсов); prod — антиаффинити готово к масштабированию. Реплики на проде не подняты постоянно из-за ограничений тестового стенда (воркеры по 2 ГБ); на полноценной инфраструктуре включаются изменением replicas в values».


добавил мягкое (preferred) антиаффинити, чтобы реплики разносились по воркерам, но не висели Pending при нехватке.











![alt text](<img/image copy 55.png>)





















## Развёртывание через CI/CD на self-hosted GitLab (VPS)

http://gitlab.185.50.202.99.nip.io:30080/students/bank-of-anthos/-/pipelines    - ссылка на self-hosted GitLab (VPS)

https://bank.spectrew.online:30443/    - ссылка на сайт


![alt text](img_2/image.png)

master@vm-c965dd:~$ sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/cf.ini --dns-cloudflare-propagation-seconds 30 -d "spectrew.online" -d "*.spectrew.online" --email z4ng1ew@gmail.com --agree-tos --non-interactive
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Account registered.
Requesting a certificate for spectrew.online and *.spectrew.online
Waiting 30 seconds for DNS changes to propagate

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/spectrew.online/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/spectrew.online/privkey.pem
This certificate expires on 2026-09-03.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
master@vm-c965dd:~$ 





Successfully received certificate — настоящий wildcard *.spectrew.online от Let's Encrypt, выпущен напрямую через certbot за один заход. Истекает 2026-09-03. Весь клубок с cert-manager обойдён.
Что  сделалано: то же доказательство владения доменом через DNS-01 Cloudflare, но проверенным плагином certbot, который сам корректно создал TXT-запись, дождался распространения и удалил её. Без петель, без webhook-конфликтов.



Файлы лежат в /etc/letsencrypt/live/spectrew.online/:

fullchain.pem — сертификат + цепочка.
privkey.pem — приватный ключ.








![alt text](<img_2/image copy.png>)


```
master@vm-c965dd:~$ 
master@vm-c965dd:~$ sudo KUBECONFIG=/etc/rancher/k3s/k3s.yaml sh -c 'for ns in default staging prod; do echo "=== $ns ==="; kubectl get secret wildcard-spectrew-tls -n $ns; done'
=== default ===
NAME                    TYPE                DATA   AGE
wildcard-spectrew-tls   kubernetes.io/tls   2      6m8s
=== staging ===
NAME                    TYPE                DATA   AGE
wildcard-spectrew-tls   kubernetes.io/tls   2      6m7s
=== prod ===
NAME                    TYPE                DATA   AGE
wildcard-spectrew-tls   kubernetes.io/tls   2      6m7s
master@vm-c965dd:~$ helm upgrade boa . -n prod --reuse-values \
  --set ingress.host=bank.spectrew.online \
  --set ingress.tls.enabled=true \
  --set ingress.tls.secretName=wildcard-spectrew-tls
Error: Chart.yaml file is missing
master@vm-c965dd:~$ cd ~/до_14/DO14/src/helm/bank-of-anthos && ls Chart.yaml
-bash: cd: /home/master/до_14/DO14/src/helm/bank-of-anthos: No such file or directory
master@vm-c965dd:~$ 

```


























































- Настоящий wildcard-сертификат Let's Encrypt (*.spectrew.online) выпущен через certbot + Cloudflare DNS-01.
- Лежит в кластере как Secret wildcard-spectrew-tls (в default, staging, prod), привязан к ingress банка в prod.
- Банк отдаётся по HTTPS с этим сертификатом — curl https://bank.spectrew.online с ноды возвращает страницу SIGN IN, openssl подтверждает issuer = Let's Encrypt (не самоподписанный, не staging), годен до 3 сентября 2026.






![alt text](<img_2/image copy 4.png>)

 «Защита соединения с spectrew.online:30443: Подтверждено: Let's Encrypt». Это финальное железобетонное доказательство: настоящий публично доверенный сертификат, банк открывается снаружи по HTTPS.





изначально ingress был доступен снаружи только на нестандартном NodePort 30443, после открытия порта 443 на стороне провайдера повесили на стандартный — теперь без указания порта.






 ![alt text](<img_2/image copy 6.png>)


и prod на 443, и staging отдают настоящий Let's Encrypt снаружи


```
s@Vivo:~/до_14/DO14$ 
s@Vivo:~/до_14/DO14$ 
s@Vivo:~/до_14/DO14$ echo "=== PROD :443 ==="; echo | openssl s_client -connect bank.spectrew.online:443 -servername bank.spectrew.online 2>/dev/null | openssl x509 -noout -issuer -dates; echo "=== STAGING :443 ==="; echo | openssl s_client -connect bank-staging.spectrew.online:443 -servername bank-staging.spectrew.online 2>/dev/null | openssl x509 -noout -issuer -dates
=== PROD :443 ===
issuer=C = US, O = Let's Encrypt, CN = YR2
notBefore=Jun  5 19:33:13 2026 GMT
notAfter=Sep  3 19:33:12 2026 GMT
=== STAGING :443 ===
issuer=C = US, O = Let's Encrypt, CN = YR2
notBefore=Jun  5 19:33:13 2026 GMT
notAfter=Sep  3 19:33:12 2026 GMT
s@Vivo:~/до_14/DO14$ 
```






























![alt text](<img_2/image copy 7.png>)





```
s@Vivo:~/до_14/DO14$ ssh -i ~/"Рабочий стол/do_14 (2)/master" master@185.50.202.99 "sudo KUBECONFIG=/etc/rancher/k3s/k3s.yaml sh -c 'echo \"===== STAGING (1 реплика) =====\"; kubectl get deploy -n staging; echo; echo \"===== PROD (2 реплики на stateless) =====\"; kubectl get deploy -n prod'"
===== STAGING (1 реплика) =====
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
accounts-db          1/1     1            1           9h
balancereader        1/1     1            1           9h
contacts             1/1     1            1           9h
frontend             1/1     1            1           9h
ledger-db            1/1     1            1           9h
ledgerwriter         1/1     1            1           9h
transactionhistory   1/1     1            1           9h
userservice          1/1     1            1           9h

===== PROD (2 реплики на stateless) =====
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
accounts-db          1/1     1            1           8h
balancereader        2/2     2            2           8h
contacts             2/2     2            2           8h
frontend             2/2     2            2           8h
ledger-db            1/1     1            1           8h
ledgerwriter         2/2     2            2           8h
transactionhistory   2/2     2            2           8h
userservice          2/2     2            2           8h
s@Vivo:~/до_14/DO14$ 
```






 таблицу деплойментов обеих сред с колонкой READY (например frontend 1/1 в staging и frontend 2/2 в prod) — наглядно видно разницу.




![alt text](<img_2/image copy 10.png>)

джоб прода


Две среды (канон Helm)  — один чарт + values-staging.yaml / values-prod.yaml.





















 ![alt text](<img_2/image copy 8.png>)


 ```
s@Vivo:~/до_14/DO14$ cd ~/до_14/DO14/src/helm/bank-of-anthos && grep -rn "resources:\|limits:\|requests:\|memory:\|cpu:" templates/ | head -40
templates/balancereader.yaml:66:          resources:
templates/balancereader.yaml:67:            requests:
templates/balancereader.yaml:68:              cpu: 100m
templates/balancereader.yaml:69:              memory: 256Mi
templates/balancereader.yaml:70:            limits:
templates/balancereader.yaml:71:              memory: 512Mi
templates/loadgenerator.yaml:27:          resources:
templates/loadgenerator.yaml:28:            requests:
templates/loadgenerator.yaml:29:              cpu: 100m
templates/loadgenerator.yaml:30:              memory: 128Mi
templates/loadgenerator.yaml:31:            limits:
templates/loadgenerator.yaml:32:              memory: 256Mi
templates/frontend.yaml:71:          resources:
templates/frontend.yaml:72:            requests:
templates/frontend.yaml:73:              cpu: 100m
templates/frontend.yaml:74:              memory: 128Mi
templates/frontend.yaml:75:            limits:
templates/frontend.yaml:76:              memory: 256Mi
templates/accounts-db.yaml:31:          resources:
templates/accounts-db.yaml:32:            requests:
templates/accounts-db.yaml:33:              cpu: 100m
templates/accounts-db.yaml:34:              memory: 256Mi
templates/accounts-db.yaml:35:            limits:
templates/accounts-db.yaml:36:              memory: 512Mi
templates/contacts.yaml:58:          resources:
templates/contacts.yaml:59:            requests:
templates/contacts.yaml:60:              cpu: 100m
templates/contacts.yaml:61:              memory: 128Mi
templates/contacts.yaml:62:            limits:
templates/contacts.yaml:63:              memory: 256Mi
templates/userservice.yaml:62:          resources:
templates/userservice.yaml:63:            requests:
templates/userservice.yaml:64:              cpu: 100m
templates/userservice.yaml:65:              memory: 128Mi
templates/userservice.yaml:66:            limits:
templates/userservice.yaml:67:              memory: 256Mi
templates/ledgerwriter.yaml:64:          resources:
templates/ledgerwriter.yaml:65:            requests:
templates/ledgerwriter.yaml:66:              cpu: 100m
templates/ledgerwriter.yaml:67:              memory: 256Mi
s@Vivo:~/до_14/DO14/src/helm/bank-of-anthos$ 
 ```









 ![alt text](<img_2/image copy 9.png>)


 пайплайн прошёл успешно!
