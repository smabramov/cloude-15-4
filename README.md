# «Кластеры. Ресурсы под управлением облачных провайдеров» - Абрамов Сергей

### Цели задания 

1. Организация кластера Kubernetes и кластера баз данных MySQL в отказоустойчивой архитектуре.
2. Размещение в private подсетях кластера БД, а в public — кластера Kubernetes.

---
## Задание 1. Yandex Cloud

1. Настроить с помощью Terraform кластер баз данных MySQL.

 - Используя настройки VPC из предыдущих домашних заданий, добавить дополнительно подсеть private в разных зонах, чтобы обеспечить отказоустойчивость. 
 - Разместить ноды кластера MySQL в разных подсетях.
 - Необходимо предусмотреть репликацию с произвольным временем технического обслуживания.
 - Использовать окружение Prestable, платформу Intel Broadwell с производительностью 50% CPU и размером диска 20 Гб.
 - Задать время начала резервного копирования — 23:59.
 - Включить защиту кластера от непреднамеренного удаления.

 ```
 resource "yandex_mdb_mysql_cluster" "test_cluster" {
name        = "test"
environment = "PRESTABLE"
network_id  = yandex_vpc_network.VPC.id
version     = "8.0"

resources {
  resource_preset_id = "s2.medium"
  disk_type_id       = "network-ssd"
  disk_size          = 20
}

maintenance_window {
  type = "ANYTIME"
 }
 backup_window_start {
  hours = 23
  minutes = 59

}

host {
  zone      = "ru-central1-a"
  subnet_id = yandex_vpc_subnet.private_a.id
}

host {
  zone      = "ru-central1-b"
  subnet_id = yandex_vpc_subnet.private_b.id
}
deletion_protection = true
}

resource "yandex_vpc_network" "VPC" {
name = var.vpc_name
}


resource "yandex_vpc_subnet" "private_a" {
zone           = "ru-central1-a"
network_id     = yandex_vpc_network.VPC.id
v4_cidr_blocks = ["10.1.0.0/24"]
}

resource "yandex_vpc_subnet" "private_b" {
zone           = "ru-central1-b"
network_id     = yandex_vpc_network.VPC.id
v4_cidr_blocks = ["10.2.0.0/24"]
}
resource "yandex_mdb_mysql_database" "netology_db" {
  cluster_id = yandex_mdb_mysql_cluster.test_cluster.id
  name       = "netology_db"
}
```

 - Создать БД с именем `netology_db`, логином и паролем.

```
resource "yandex_mdb_mysql_database" "netology_db" {
  cluster_id = yandex_mdb_mysql_cluster.test_cluster.id
  name       = "netology_db"
}
```
Создал пользователя и назначил ему права

```
resource "yandex_mdb_mysql_user" "serg" {
  cluster_id = yandex_mdb_mysql_cluster.test_cluster.id
  name       = "serg"
  password   = "password"

  permission {
    database_name = yandex_mdb_mysql_database.netology_db.name
    roles         = ["ALL"]
  }
}
```
Применил пайплайн:

```
serg@debian:~/git/cloude-15-4$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated
with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_mdb_mysql_cluster.test_cluster will be created
  + resource "yandex_mdb_mysql_cluster" "test_cluster" {
      + allow_regeneration_host   = false
      + backup_retain_period_days = (known after apply)
      + created_at                = (known after apply)
      + deletion_protection       = true
      + environment               = "PRESTABLE"
      + folder_id                 = (known after apply)
      + health                    = (known after apply)
      + host_group_ids            = (known after apply)
      + id                        = (known after apply)
      + mysql_config              = (known after apply)
      + name                      = "test"
      + network_id                = (known after apply)
      + status                    = (known after apply)
      + version                   = "8.0"

      + backup_window_start {
          + hours   = 23
          + minutes = 59
        }

      + host {
          + assign_public_ip   = false
          + fqdn               = (known after apply)
          + replication_source = (known after apply)
          + subnet_id          = (known after apply)
          + zone               = "ru-central1-a"
        }
      + host {
          + assign_public_ip   = false
          + fqdn               = (known after apply)
          + replication_source = (known after apply)
          + subnet_id          = (known after apply)
          + zone               = "ru-central1-b"
        }

      + maintenance_window {
          + type = "ANYTIME"
        }

      + resources {
          + disk_size          = 20
          + disk_type_id       = "network-ssd"
          + resource_preset_id = "s2.medium"
        }
    }

  # yandex_mdb_mysql_database.netology_db will be created
  + resource "yandex_mdb_mysql_database" "netology_db" {
      + cluster_id = (known after apply)
      + id         = (known after apply)
      + name       = "netology_db"
    }

  # yandex_mdb_mysql_user.serg will be created
  + resource "yandex_mdb_mysql_user" "serg" {
      + authentication_plugin = (known after apply)
      + cluster_id            = (known after apply)
      + global_permissions    = (known after apply)
      + id                    = (known after apply)
      + name                  = "serg"
      + password              = (sensitive value)

      + permission {
          + database_name = "netology_db"
          + roles         = [
              + "ALL",
            ]
        }
    }

  # yandex_vpc_network.VPC will be created
  + resource "yandex_vpc_network" "VPC" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "VPC"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.private_a will be created
  + resource "yandex_vpc_subnet" "private_a" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = (known after apply)
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.1.0.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

  # yandex_vpc_subnet.private_b will be created
  + resource "yandex_vpc_subnet" "private_b" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = (known after apply)
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.2.0.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-b"
    }

Plan: 6 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

yandex_vpc_network.VPC: Creating...
yandex_vpc_network.VPC: Creation complete after 2s [id=enp2pdb0659ggu5nr4dg]
yandex_vpc_subnet.private_b: Creating...
yandex_vpc_subnet.private_a: Creating...
yandex_vpc_subnet.private_b: Creation complete after 1s [id=e2ldmr855lodrfpg1c2t]
yandex_vpc_subnet.private_a: Creation complete after 1s [id=e9bn0iua5d2dfilnjd5k]
yandex_mdb_mysql_cluster.test_cluster: Creating...
yandex_mdb_mysql_cluster.test_cluster: Still creating... [10s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [20s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [30s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [40s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [51s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [1m1s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [1m11s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [1m21s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [1m31s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [1m41s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [1m51s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [2m1s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [2m11s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [2m21s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [2m31s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [2m41s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [2m51s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [3m1s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [3m11s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [3m21s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [3m31s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [3m41s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [3m51s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [4m1s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [4m11s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [4m21s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [4m31s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [4m41s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [4m51s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [5m1s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [5m11s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [5m21s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [5m31s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [5m41s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [5m51s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [6m1s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [6m11s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [6m21s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [6m31s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [6m41s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [6m51s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [7m1s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [7m11s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [7m22s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [7m32s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [7m42s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Still creating... [7m52s elapsed]
yandex_mdb_mysql_cluster.test_cluster: Creation complete after 7m55s [id=c9q0fpkbfsv8pbdpoi93]
yandex_mdb_mysql_database.netology_db: Creating...
yandex_mdb_mysql_database.netology_db: Still creating... [10s elapsed]
yandex_mdb_mysql_database.netology_db: Still creating... [20s elapsed]
yandex_mdb_mysql_database.netology_db: Still creating... [30s elapsed]
yandex_mdb_mysql_database.netology_db: Still creating... [40s elapsed]
yandex_mdb_mysql_database.netology_db: Still creating... [50s elapsed]
yandex_mdb_mysql_database.netology_db: Still creating... [1m0s elapsed]
yandex_mdb_mysql_database.netology_db: Creation complete after 1m2s [id=c9q0fpkbfsv8pbdpoi93:netology_db]
yandex_mdb_mysql_user.serg: Creating...
yandex_mdb_mysql_user.serg: Still creating... [10s elapsed]
yandex_mdb_mysql_user.serg: Still creating... [20s elapsed]
yandex_mdb_mysql_user.serg: Creation complete after 21s [id=c9q0fpkbfsv8pbdpoi93:serg]

Apply complete! Resources: 6 added, 0 changed, 0 destroyed.
```
![1]()

![2]()

![3]()

2. Настроить с помощью Terraform кластер Kubernetes.

 - Используя настройки VPC из предыдущих домашних заданий, добавить дополнительно две подсети public в разных зонах, чтобы обеспечить отказоустойчивость.

 ```
 resource "yandex_vpc_subnet" "public_a" {
 zone           = "ru-central1-a"
 network_id     = yandex_vpc_network.VPC.id
 v4_cidr_blocks = ["10.0.1.0/24"]
}

resource "yandex_vpc_subnet" "public_b" {
 zone           = "ru-central1-b"
 network_id     = yandex_vpc_network.VPC.id
 v4_cidr_blocks = ["10.0.2.0/24"]
}
resource "yandex_vpc_subnet" "public_d" {
 zone           = "ru-central1-d"
 network_id     = yandex_vpc_network.VPC.id
 v4_cidr_blocks = ["10.0.3.0/24"]
}
```

 - Создать отдельный сервис-аккаунт с необходимыми правами. 

 ```
 # Сервисный аккаунт 
resource "yandex_iam_service_account" "k8s" {
 name        = "k8s"
}

#Назначение роли для сервисного аккаунта
resource "yandex_resourcemanager_folder_iam_member" "editor" {
 folder_id = var.folder_id
 role      = "editor"
 member    = "serviceAccount:${yandex_iam_service_account.k8s.id}"
}
#Создание статического ключа доступа
resource "yandex_iam_service_account_static_access_key" "k8s-key" {
 service_account_id = yandex_iam_service_account.k8s.id
 description        = "static access key for object storage"
}
```

 - Создать региональный мастер Kubernetes с размещением нод в трёх разных подсетях.
 - Добавить возможность шифрования ключом из KMS, созданным в предыдущем домашнем задании.

 ```
 # Создание ключа для шифрования
resource "yandex_kms_symmetric_key" "encryptkey" {
 name              = "encryptkey"
 default_algorithm = "AES_256"
 rotation_period   = "8760h"
}
# Создание регионального кластера k8s
resource "yandex_kubernetes_cluster" "regional_k8s" {
 name        = "regional_k8s"

 network_id = yandex_vpc_network.VPC.id

 master {
   regional {
     region = "ru-central1"

     location {
       zone      = yandex_vpc_subnet.public_a.zone
       subnet_id = yandex_vpc_subnet.public_a.id
     }

     location {
       zone      = yandex_vpc_subnet.public_b.zone
       subnet_id = yandex_vpc_subnet.public_b.id
     }

     location {
       zone      = yandex_vpc_subnet.public_d.zone
       subnet_id = yandex_vpc_subnet.public_d.id
     }
   }
  
   version   = "1.14"
   public_ip = true

 }

 service_account_id      = yandex_iam_service_account.k8s.id
 node_service_account_id = yandex_iam_service_account.k8s.id
kms_provider {
 key_id = yandex_kms_symmetric_key.encryptkey.id
}
 
 release_channel = "STABLE"
}
```
 - Создать группу узлов, состояющую из трёх машин с автомасштабированием до шести.

 ```

resource "yandex_kubernetes_node_group" "k8s-node-group-a" {
 cluster_id  = yandex_kubernetes_cluster.regional-k8s.id
 name        = "k8s-node-group-a"

 version     = "1.30"

 labels = {
   "key" = "k8s-node-group-a"
 }

 instance_template {
   platform_id = "standard-v2"

   network_interface {
     nat        = true
     subnet_ids = ["${yandex_vpc_subnet.public_a.id}"]
      }

   resources {
     memory = 2
     cores  = 2
   }

   boot_disk {
     type = "network-hdd"
     size = 64
   }

   scheduling_policy {
     preemptible = true
   }

   container_runtime {
     type = "containerd"
   }
     metadata = {
       ssh-keys  = "ubuntu:${var.ssh_public_key_path}"
   }
 }

 scale_policy {
   auto_scale {
     initial = 3
     min = 3
     max = 6
   }
 }

 allocation_policy {
   location {
     zone = "ru-central1-a"

   }
   
 }
}
```
```
serg@k8snode:~/yandex-cloud/bin$ yc managed-kubernetes cluster list
+----------------------+--------------+---------------------+---------+---------+------------------------+-------------------+
|          ID          |     NAME     |     CREATED AT      | HEALTH  | STATUS  |   EXTERNAL ENDPOINT    | INTERNAL ENDPOINT |
+----------------------+--------------+---------------------+---------+---------+------------------------+-------------------+
| catj5hdcd4er47v2fl9c | regional-k8s | 2025-01-18 13:03:02 | HEALTHY | RUNNING | https://158.160.139.48 | https://10.0.1.20 |
+----------------------+--------------+---------------------+---------+---------+------------------------+-------------------+
```

 - Подключиться к кластеру с помощью `kubectl`.

 ```
 serg@k8snode:~/yandex-cloud/bin$ yc managed-kubernetes cluster get-credentials catj5hdcd4er47v2fl9c --external

Context 'yc-regional-k8s' was added as default to kubeconfig '/home/serg/.kube/config'.
Check connection to cluster using 'kubectl cluster-info --kubeconfig /home/serg/.kube/config'.

Note, that authentication depends on 'yc' and its config profile 'default'.
To access clusters using the Kubernetes API, please use Kubernetes Service Account
serg@k8snode:~/yandex-cloud/bin$ kubectl cluster-info 
Kubernetes control plane is running at https://158.160.139.48
CoreDNS is running at https://158.160.139.48/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
serg@k8snode:~/yandex-cloud/bin$ kubectl get all -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-577d65f588-nk44p               1/1     Running   0          23m
kube-system   pod/coredns-577d65f588-z4ljw               1/1     Running   0          17m
kube-system   pod/ip-masq-agent-84zgp                    1/1     Running   0          18m
kube-system   pod/ip-masq-agent-bzpwk                    1/1     Running   0          18m
kube-system   pod/ip-masq-agent-k6j2x                    1/1     Running   0          18m
kube-system   pod/kube-dns-autoscaler-697d688488-nxzfs   1/1     Running   0          23m
kube-system   pod/kube-proxy-2k9tz                       1/1     Running   0          18m
kube-system   pod/kube-proxy-6nthv                       1/1     Running   0          18m
kube-system   pod/kube-proxy-l52s8                       1/1     Running   0          18m
kube-system   pod/metrics-server-9f7b47c55-l6lgj         2/2     Running   0          18m
kube-system   pod/npd-v0.8.0-4sdj7                       1/1     Running   0          18m
kube-system   pod/npd-v0.8.0-xc75q                       1/1     Running   0          18m
kube-system   pod/npd-v0.8.0-xtn9q                       1/1     Running   0          18m
kube-system   pod/yc-disk-csi-node-v2-5sr7q              6/6     Running   0          18m
kube-system   pod/yc-disk-csi-node-v2-f7fv8              6/6     Running   0          18m
kube-system   pod/yc-disk-csi-node-v2-x7flr              6/6     Running   0          18m

NAMESPACE     NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes       ClusterIP   10.96.128.1     <none>        443/TCP                  23m
kube-system   service/kube-dns         ClusterIP   10.96.128.2     <none>        53/UDP,53/TCP,9153/TCP   23m
kube-system   service/metrics-server   ClusterIP   10.96.200.185   <none>        443/TCP                  23m

NAMESPACE     NAME                                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                                                        AGE
kube-system   daemonset.apps/ip-masq-agent                    3         3         3       3            3           beta.kubernetes.io/os=linux,node.kubernetes.io/masq-agent-ds-ready=true              23m
kube-system   daemonset.apps/kube-proxy                       3         3         3       3            3           kubernetes.io/os=linux,node.kubernetes.io/kube-proxy-ds-ready=true                   23m
kube-system   daemonset.apps/npd-v0.8.0                       3         3         3       3            3           beta.kubernetes.io/os=linux,node.kubernetes.io/node-problem-detector-ds-ready=true   23m
kube-system   daemonset.apps/nvidia-device-plugin-daemonset   0         0         0       0            0           beta.kubernetes.io/os=linux,node.kubernetes.io/nvidia-device-plugin-ds-ready=true    23m
kube-system   daemonset.apps/yc-disk-csi-node                 0         0         0       0            0           <none>                                                                               23m
kube-system   daemonset.apps/yc-disk-csi-node-v2              3         3         3       3            3           yandex.cloud/pci-topology=k8s                                                        23m

NAMESPACE     NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns               2/2     2            2           23m
kube-system   deployment.apps/kube-dns-autoscaler   1/1     1            1           23m
kube-system   deployment.apps/metrics-server        1/1     1            1           23m

NAMESPACE     NAME                                             DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-577d65f588               2         2         2       23m
kube-system   replicaset.apps/kube-dns-autoscaler-697d688488   1         1         1       23m
kube-system   replicaset.apps/metrics-server-66ddbc9fc5        0         0         0       23m
kube-system   replicaset.apps/metrics-server-9f7b47c55         1         1         1       18m
```

 - *Запустить микросервис phpmyadmin и подключиться к ранее созданной БД.
 - *Создать сервис-типы Load Balancer и подключиться к phpmyadmin. Предоставить скриншот с публичным адресом и подключением к БД.

 ```
 serg@k8snode:~/k8s-cloud$ kubectl apply -f ./phpmyadmin.yaml 
deployment.apps/phpmyadmin-deployment created
service/phpmyadmin-service created
serg@k8snode:~/k8s-cloud$ kubectl get deployments.apps
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
phpmyadmin-deployment   3/3     3            3           38s
serg@k8snode:~/k8s-cloud$ kubectl get svc
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)        AGE
kubernetes           ClusterIP      10.96.128.1    <none>            443/TCP        30m
phpmyadmin-service   LoadBalancer   10.96.223.15   158.160.140.170   80:30682/TCP   52s
```
![4]()

Полезные документы:

- [MySQL cluster](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/mdb_mysql_cluster).
- [Создание кластера Kubernetes](https://cloud.yandex.ru/docs/managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-create)
- [K8S Cluster](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_cluster).
- [K8S node group](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_node_group).

--- 
## Задание 2*. Вариант с AWS (задание со звёздочкой)

Это необязательное задание. Его выполнение не влияет на получение зачёта по домашней работе.

**Что нужно сделать**

1. Настроить с помощью Terraform кластер EKS в три AZ региона, а также RDS на базе MySQL с поддержкой MultiAZ для репликации и создать два readreplica для работы.
 
 - Создать кластер RDS на базе MySQL.
 - Разместить в Private subnet и обеспечить доступ из public сети c помощью security group.
 - Настроить backup в семь дней и MultiAZ для обеспечения отказоустойчивости.
 - Настроить Read prelica в количестве двух штук на два AZ.

2. Создать кластер EKS на базе EC2.

 - С помощью Terraform установить кластер EKS на трёх EC2-инстансах в VPC в public сети.
 - Обеспечить доступ до БД RDS в private сети.
 - С помощью kubectl установить и запустить контейнер с phpmyadmin (образ взять из docker hub) и проверить подключение к БД RDS.
 - Подключить ELB (на выбор) к приложению, предоставить скрин.

Полезные документы:

- [Модуль EKS](https://learn.hashicorp.com/tutorials/terraform/eks).

### Правила приёма работы

Домашняя работа оформляется в своём Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
