[terraform.txt](https://github.com/user-attachments/files/24956383/terraform.txt)Задание 1
Возьмите за основу решение к заданию 1 из занятия «Подъём инфраструктуры в Яндекс Облаке».

Теперь вместо одной виртуальной машины сделайте terraform playbook, который:
создаст 2 идентичные виртуальные машины. Используйте аргумент count для создания таких ресурсов;
создаст таргет-группу. Поместите в неё созданные на шаге 1 виртуальные машины;
создаст сетевой балансировщик нагрузки, который слушает на порту 80, отправляет трафик на порт 80 виртуальных машин и http healthcheck на порт 80 виртуальных машин.
Рекомендуем изучить документацию сетевого балансировщика нагрузки для того, чтобы было понятно, что вы сделали.

Установите на созданные виртуальные машины пакет Nginx любым удобным способом и запустите Nginx веб-сервер на порту 80.

Перейдите в веб-консоль Yandex Cloud и убедитесь, что:

созданный балансировщик находится в статусе Active,
обе виртуальные машины в целевой группе находятся в состоянии healthy.
Сделайте запрос на 80 порт на внешний IP-адрес балансировщика и убедитесь, что вы получаете ответ в виде дефолтной страницы Nginx.
В качестве результата пришлите:

1. Terraform Playbook.

2. Скриншот статуса балансировщика и целевой группы.

3. Скриншот страницы, которая открылась при запросе IP-адреса балансировщика.

        [Uploaditerraform {
       required_providers {
       yandex = {
        source = "yandex-cloud/yandex"
        }
         }
       }

       provider "yandex" {
       token = "${var.yc_token}"
       cloud_id  = "b1ggel59310trksk1fu4"
       folder_id = "b1g9oing6niujio3j61t"
       zone      = "ru-central1-a"
       }
       data "template_file" "metadata" {
       template = file("./metadata.yaml")
        }

       resource "yandex_compute_instance" "ubuntu" {
       count = 2
       name = "ubuntu${count.index}"
       platform_id = "standard-v3"
       allow_stopping_for_update = true
       resources {
       core_fraction = 50
       cores  = 2
       memory = 4
       }
       boot_disk {
       initialize_params {
       image_id = "fd8ps4vdhf5hhuj8obp2"
       size = 10
       type = "network-ssd"
       }
       }

       network_interface {
       subnet_id = yandex_vpc_subnet.subnet-1.id
       nat       = true
       }

       metadata = {
       user-data = data.template_file.metadata.rendered
       }

       scheduling_policy {
       preemptible = true
       }

       }

       resource "yandex_vpc_network" "network-2" {
       name = "network2"
       }

       resource "yandex_vpc_subnet" "subnet-1" {
       name           = "subnet1"
       zone           = "ru-central1-a"
       network_id     = yandex_vpc_network.network-2.id
       v4_cidr_blocks = ["192.168.1.0/24"]
       }

       resource "yandex_lb_target_group" "target_group1" {
        name      = "my-target-group"
       dynamic "target" {
       for_each = yandex_compute_instance.ubuntu
       content {
        subnet_id = yandex_vpc_subnet.subnet-1.id
        address   = target.value.network_interface[0].ip_address
        }
        }
        }

       resource "yandex_lb_network_load_balancer" "lb-1" {
       name = "network-load-balancer-1"

       listener {
       name = "network-load-balancer-1-listener"
       port = 80
        external_address_spec {
       ip_version = "ipv4"
       }
       }
attached_target_group 
target_group_id = yandex_lb_target_group.target_group1.id

    healthcheck {
      name = "http"
      http_options {
        port = 80
        path = "/"
      }
    }
    }
    }
    output "external_ip_address_ubuntu" {
    value = "${yandex_compute_instance.ubuntu.*.network_interface.0.nat_ip_address}"
     }

     output "external_ip_address_lb" {
     value = [
    for listener in yandex_lb_network_load_balancer.lb-1.listener :
    listener.external_address_spec
    ]
    }ng terraform.txt…]()
     }

    Применяем аргумент count:
    resource "yandex_compute_instance" "ubuntu" {
    count = 2
    name = "ubuntu${count.index}"
    ...
     }

     Далее создаем таргет:
    resource "yandex_lb_target_group" "target_group1" {
     name      = "my-target-group"
     dynamic "target" {
    for_each = yandex_compute_instance.ubuntu
    content {
      subnet_id = yandex_vpc_subnet.subnet-1.id
      address   = target.value.network_interface[0].ip_address
    }
     }
     }
    далее
    resource "yandex_lb_network_load_balancer" "lb-1" {
     name = "network-load-balancer-1"

     listener {
    name = "network-load-balancer-1-listener"
    port = 80
    external_address_spec {
      ip_version = "ipv4"
    }
    }

     attached_target_group {
    target_group_id = yandex_lb_target_group.target_group1.id

    healthcheck {
      name = "http"
      http_options {
        port = 80
        path = "/"
      }
    }
  }
}      

<img width="631" height="297" alt="task1" src="https://github.com/user-attachments/assets/a298547f-a1ca-40f0-bdb8-f241c5319a24" />
<img width="763" height="576" alt="image" src="https://github.com/user-attachments/assets/6d9bb867-f58d-4dd1-9489-30196ce988d8" />
![task1-2](https://github.com/user-attachments/assets/49545d56-6982-48c1-8a21-b494234af6e7)



