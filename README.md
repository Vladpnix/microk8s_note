sudo snap install microk8s --classic --channel=1.32.3

sudo usermod -a -G microk8s $USER
sudo mkdir -p ~/.kube
sudo chown -f -R $USER ~/.kube

microk8s status --wait-ready
microk8s kubectl get nodes
microk8s kubectl get services

~/.bash_aliases
alias kubectl='microk8s kubectl'

kubectl create deployment nginx --image=nginx
kubectl get pods

Использование дополнений
Для начала рекомендуется добавить управление DNS, чтобы облегчить взаимодействие между сервисами. Для приложений, которым требуется хранилище, дополнение hostpath-storage предоставляет пространство для каталогов на хосте. Их легко настроить:

microk8s enable dns
microk8s enable hostpath-storage

Запуск и остановка MicroK8s
microk8s stop
microk8s start

Добавление узла
Чтобы создать кластер из двух или более уже работающих экземпляров MicroK8s, используйте команду microk8s add-node. Экземпляр MicroK8s, на котором будет запущена эта команда, станет ведущим в кластере и будет размещать плоскость управления Kubernetes:

microk8s add-node

Это вернет некоторые инструкции по присоединению, которые должны быть выполнены на экземпляре MicroK8s, который вы хотите присоединить к кластеру (НЕ на том узле, с которого вы выполнили команду add-node).

From the node you wish to join to this cluster, run the following:
microk8s join 185.204.117.3:25000/b83e00c714287a6fe61d9e74ac9c2d88/c59006c46eac

Use the '--worker' flag to join a node as a worker not running the control plane, eg:
microk8s join 185.204.117.3:25000/b83e00c714287a6fe61d9e74ac9c2d88/c59006c46eac --worker

If the node you are adding is not reachable through the default interface you can use one of the following:
microk8s join 185.204.117.3:25000/b83e00c714287a6fe61d9e74ac9c2d88/c59006c46eac

kubectl get no

Удаление узла
Сначала на узле, который вы хотите удалить, выполните команду microk8s leave. MicroK8s на удаляемом узле перезапустит свою собственную плоскость управления и возобновит работу как полноценный одноузловой кластер:

microk8s leave

Чтобы завершить удаление узла, вызовите microk8s remove-node с оставшихся узлов, чтобы указать, что ушедший (недоступный теперь) узел должен быть удален навсегда:

microk8s remove-node 185.204.117.7
microk8s remove-node 185.204.117.9

Высокая доступность
Начиная с версии 1.19 в MicroK8s HA включена по умолчанию. Если ваш кластер состоит из трех или более узлов, то хранилище данных будет реплицировано между узлами и будет устойчиво к единичному сбою (если на одном узле возникнут проблемы, рабочие нагрузки продолжат выполняться без перебоев).

В microk8s status теперь включена информация о состоянии HA. Например:

{ clear && \
  echo -e "\n=== Kubernetes Status ===\n" && \
  kubectl get --raw '/healthz?verbose' && \
  kubectl version --short && \
  kubectl get nodes && \
  kubectl cluster-info; 
} | grep -z 'Ready\| ok\|passed\|running'

Проверим healtz эндпоинты controlplane

kubectl get --raw /
kubectl get --raw /api/v1/namespaces/default | jq

kubectl get pods --namespace kube-system

На данном этапе перечислен только Calico, который является интерфейсом контейнера по умолчанию (CNI) в MicroK8s. Другие ключевые компоненты cp, такие как etcd, CoreDNS, kube-apiserver, kube-proxy, kube-scheduler и kube-controller-manager, отсутствуют, однако кластер сообщает, что он здоров. Что происходит? Где они?

Архитекторы MicroK8s добавили эти ключевые контролирующие компоненты Kubernetes в виде служб-демонов, управляемых systemd:

systemctl list-units 'snap.microk8s.*' --no-pager

microk8s enable dashboard dns registry metrics-server

kubectl get pods --namespace kube-system

#### Получить информацию о Pod'е
kubectl describe pod my-pod -n my-namespace

#### Определить, что он принадлежит Deployment
#### ... (вывод команды покажет, что он принадлежит Deployment) ...

#### Удалить Deployment
kubectl delete deployment my-deployment -n my-namespace

kubectl get services

